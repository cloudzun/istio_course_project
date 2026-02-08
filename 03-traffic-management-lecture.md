# 第三章：流量治理基础 —— 南北向与东西向流量管理

> **章节目标**：深入理解 Istio 的流量管理模型，掌握 VirtualService 和 DestinationRule 的设计原理，理解网格内流量路由和入口网关的工作机制。

---

## 1. Istio 流量管理的核心模型

### 为什么需要重新定义流量管理？

Kubernetes 的 Service 在设计时，只考虑了**基于 Pod 标签的分组**和**简单的负载均衡**：

```
┌─────────────────────────────────────────┐
│ Kubernetes Service                      │
│ • 基于 Label Selector 发现 Pod          │
│ • 提供 ClusterIP（虚拟 IP）             │
│ • Round-Robin 负载均衡（kube-proxy）    │
│ • 仅支持 L4（TCP/UDP）层操作            │
└─────────────────────────────────────────┘
```

**K8s Service 的局限性：**

| 功能需求 | K8s Service 能力 | 网格能力 |
|---------|----------------|--------|
| **基于 HTTP 路径路由** | ✗ 无法（L4 层） | ✓ 支持（L7 层） |
| **金丝雀灰度（权重路由）** | ✗ 不支持 | ✓ 精确控制百分比 |
| **请求头匹配** | ✗ 不支持 | ✓ 支持（基于 Header） |
| **故障转移** | ✓ 有（但粗糙） | ✓ 精细的熔断和重试 |
| **运行时修改路由** | ✗ 需要 kube-proxy 重新计算 | ✓ Envoy 动态感知 |

Istio 的核心创新就是：**在 K8s Service 之上，加一层高级的 L7 流量管理**。

### VirtualService 与 DestinationRule 的设计哲学

Istio 使用两个 CRD 来定义流量管理：

```
┌────────────────────────────────────────────────────────┐
│ VirtualService（路由规则）                              │
│ • "我如何将流量路由到目标服务？"                       │
│ • 定义：匹配条件、目标服务、权重、超时等               │
│ • 类比：传统的 nginx upstream + location 块            │
└────────────────────────────────────────────────────────┘
                            ↓
              ┌─────────────────────────────┐
              │ 流量到达 details:9080        │
              └─────────────────────────────┘
                            ↓
┌────────────────────────────────────────────────────────┐
│ DestinationRule（目标规则）                             │
│ • "到达目标后，如何处理连接？"                         │
│ • 定义：版本分组、连接池、异常检测、负载均衡算法      │
│ • 类比：传统的 upstream 内部的性能参数                 │
└────────────────────────────────────────────────────────┘
```

**关键区别：**

- **VirtualService**：控制**流量去向**（Where to send）
- **DestinationRule**：控制**连接行为**（How to send）

### 实际例子：拆解一个流量管理场景

假设你想：**将 80% 的流量转到 reviews:v1，20% 转到 reviews:v2，并对 v2 启用熔断**。

```yaml
# 1. VirtualService：定义路由规则
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews                    # 匹配这个 Service 的流量
  http:
  - route:
    - destination:
        host: reviews          # 目标 Service FQDN
        subset: v1             # 使用 v1 版本
      weight: 80               # 80% 流量
    - destination:
        host: reviews
        subset: v2             # 使用 v2 版本
      weight: 20               # 20% 流量
    timeout: 10s               # 整体超时 10 秒

---
# 2. DestinationRule：定义版本分组和连接策略
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 100
    outlierDetection:
      consecutive5xxErrors: 5
      interval: 30s
      baseEjectionTime: 30s
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
    # 仅对 v2 启用特殊的异常检测
    trafficPolicy:
      outlierDetection:
        consecutive5xxErrors: 3    # v2 更敏感，3 次错误就标记为异常
        interval: 10s
        baseEjectionTime: 60s
```

**工作流程：**

```
客户端请求到达 reviews Service
           ↓
VirtualService 匹配规则：
  • Host = reviews ✓
  • 没有其他匹配条件，使用默认 HTTP 路由
           ↓
决策：80% 流量 → v1，20% 流量 → v2
           ↓
DestinationRule 应用连接策略：
  • 对于 v1：使用默认策略
  • 对于 v2：连接池（100 个连接），异常检测（3 次 5xx 触发熔断）
           ↓
Envoy 执行路由决策，连接到具体 Pod
```

---

## 2. VirtualService 详解：流量路由的艺术

### VirtualService 的核心字段

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my-service
spec:
  # ========== 基础配置 ==========
  hosts:                          # 这些流量由本规则处理
  - my-service                    # K8s Service 名称
  - my-service.default            # 完整域名
  - my-service.default.svc        # FQDN
  - "*.example.com"               # 支持通配符（仅 Gateway 流量）
  
  gateways:                        # 关联哪些 Gateway
  - my-gateway                     # 处理 Gateway 的入站流量
  - mesh                           # 默认：处理网格内的流量
  
  # ========== HTTP 路由规则（第一个匹配的规则生效）==========
  http:
  - name: "review-v1"             # 规则名称（可选，便于追踪）
    
    # 流量匹配条件
    match:
    - uri:                         # URI 匹配（前缀、精确、正则）
        prefix: "/reviews"
    - headers:                     # 请求头匹配
        user-agent:
          regex: ".*Chrome.*"
    - sourceLabels:                # 来源 Pod 标签匹配
        version: v2                # 仅来自 version=v2 的 Pod
    - withoutHeaders:              # 不含某个 Header
        cookie:
    
    # 路由目标
    route:
    - destination:
        host: reviews              # 目标 Service
        subset: v1                 # 目标版本（定义在 DestinationRule）
        port:
          number: 9080
      weight: 50                   # 权重 50%（与其他目标权重相加应为 100）
    - destination:
        host: reviews
        subset: v2
      weight: 50
    
    # 请求修改
    headers:
      request:
        add:
          x-custom-header: "value"
        set:
          x-user-id: "12345"
      response:
        add:
          x-response-time: "100ms"
    
    # 超时和重试
    timeout: 10s                   # 请求超时
    retries:
      attempts: 3                  # 最多重试 3 次
      perTryTimeout: 2s            # 每次重试的超时
      retryOn: "5xx,retriable-4xx" # 触发重试的条件
    
    # 故障注入（用于测试）
    fault:
      delay:
        percentage: 10             # 10% 请求延迟
        fixedDelay: 500ms          # 延迟 500ms
      abort:
        percentage: 5              # 5% 请求中止
        httpStatus: 500            # 返回 500 错误

  # ========== TCP 路由（非 HTTP）==========
  tcp:
  - match:
    - destinationSubnets:
      - "10.0.0.0/8"
    route:
    - destination:
        host: mysql
        port:
          number: 3306

  # ========== TLS 路由（HTTPS）==========
  tls:
  - match:
    - sniHosts:
      - "example.com"
    route:
    - destination:
        host: my-service
        port:
          number: 443
  
  # ========== 全局配置 ==========
  exportTo:
  - "."                            # 仅本命名空间可见
  - "*"                            # 全网格可见（默认）
```

### 匹配规则的优先级与顺序

**重要概念：** VirtualService 中 `http[]` 数组的**第一个匹配规则生效**。

```yaml
http:
# 规则 1：精确匹配，优先级最高
- match:
  - uri:
      exact: "/health"
  route:
  - destination:
      host: health-service

# 规则 2：路径前缀匹配
- match:
  - uri:
      prefix: "/api/v2"
  route:
  - destination:
      host: api-v2

# 规则 3：兜底规则（所有其他流量）
- route:
  - destination:
      host: api-v1
```

**处理流程：**

```
请求：GET /api/v2/users

检查规则 1：URI = /health？不匹配 → 继续
检查规则 2：URI 前缀 = /api/v2？✓ 匹配！
→ 路由到 api-v2，规则生效，不继续检查规则 3
```

**最佳实践：**
1. **从特殊到一般排列**：精确匹配 → 前缀匹配 → 兜底规则
2. **避免重叠条件**：否则后续规则永不执行
3. **使用 `match` 的多条件 AND 逻辑**：一个 match 块内的条件都要满足才生效

### 匹配条件的详细解析

#### URI 匹配

```yaml
match:
- uri:
    exact: "/reviews/1"           # 精确匹配
- uri:
    prefix: "/api/v2"             # 前缀匹配
- uri:
    regex: "^/reviews/[0-9]+$"    # 正则匹配
```

#### 请求头匹配

```yaml
match:
- headers:
    user-agent:
      regex: ".*Firefox.*"         # 来自 Firefox 浏览器的请求
    x-user-id:
      exact: "alice"               # 用户 ID 为 alice
```

**实际应用：** 灰度发布时，可以用 Header 标识特殊用户组。

```yaml
# 示例：将特定用户路由到新版本
match:
- headers:
    x-test-user:
      exact: "true"
route:
- destination:
    host: reviews
    subset: v3-beta              # 新版本
```

#### 来源标签匹配

```yaml
match:
- sourceLabels:
    version: v2                    # 仅来自 version=v2 的 Pod
    team: frontend                 # 且来自 team=frontend 的 Pod
route:
- destination:
    host: database
    subset: replica-preferred      # 路由到副本数据库（减少主库压力）
```

**使用场景：** 
- 某个版本的应用对另一个后端有特殊需求
- A/B 测试：不同源的流量路由到不同版本

---

## 3. DestinationRule 详解：连接与负载均衡策略

### DestinationRule 的核心结构

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews                    # 应用到哪个 Service

  # ========== 全局流量策略 ==========
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100        # 最多 100 个 TCP 连接
      http:
        http1MaxPendingRequests: 100  # HTTP/1.1：最多 100 个待处理请求
        http2MaxRequests: 100         # HTTP/2：最多 100 个并发请求
        http2MaxRequestsPerConnection: 2  # 每个连接最多 2 个请求
        maxRequestsPerConnection: 2   # HTTP/1.1：keep-alive 连接最多 2 个请求
        idleTimeout: 300s             # 空闲连接超时
        h2UpgradePolicy: UPGRADE      # 是否升级到 HTTP/2
    
    outlierDetection:              # 异常检测与熔断
      consecutive5xxErrors: 5      # 5 次 5xx 错误标记为异常
      consecutiveGatewayErrors: 5  # 5 次网关错误
      consecutive4xxErrors: 0      # 0 表示不检测 4xx（4xx 通常不是后端问题）
      interval: 30s                # 每 30 秒检查一次
      baseEjectionTime: 30s        # 第一次隔离时长 30 秒
      maxEjectionPercent: 50       # 最多隔离 50% 的实例
      minRequestVolume: 5          # 至少 5 个请求才开始检测
      splitExternalLocalOriginErrors: true  # 区分外部和本地错误
    
    loadBalancer:                  # 负载均衡算法
      simple: ROUND_ROBIN          # Round Robin（默认）
      # 其他算法：LEAST_REQUEST, RANDOM, PASSTHROUGH
      consistentHash:              # 一致性哈希
        httpCookie:
          name: "user_id"
          ttl: 3600s
        # 或基于 Header：
        # httpHeaderName: "x-session-id"
        # 或基于 TCP 连接源 IP：
        # useSourceIp: true
    
    tls:                           # TLS 配置
      mode: SIMPLE                 # DISABLE, SIMPLE, MUTUAL, ISTIO_MUTUAL
      clientCertificate: /etc/certs/cert.pem
      privateKey: /etc/certs/key.pem
      caCertificates: /etc/certs/ca.pem

  # ========== 版本分组（Subsets）==========
  subsets:
  - name: v1
    labels:
      version: v1                  # 匹配 Pods 的标签
    trafficPolicy:                 # 仅对此 subset 的特殊策略
      connectionPool:
        http:
          http1MaxPendingRequests: 50  # v1 更保守：50 个待处理请求
  
  - name: v2
    labels:
      version: v2
    trafficPolicy:
      outlierDetection:
        consecutive5xxErrors: 3    # v2 更敏感：3 次错误就隔离
        interval: 10s
  
  - name: canary
    labels:
      version: v3
      env: canary
    trafficPolicy:
      connectionPool:
        http:
          http1MaxPendingRequests: 200  # 金丝雀版本：允许更多请求

  # ========== 导出配置 ==========
  exportTo:
  - "."                            # 仅本命名空间
  - "*"                            # 全网格
```

### 连接池（Connection Pool）的深度理解

连接池是防止**级联故障**的关键机制。

**场景：** 假设后端服务突然变慢（响应时间从 10ms 增加到 1s）。

```
无连接池限制的情况：
客户端持续发送请求
   ↓
由于后端慢，请求堆积在 TCP 缓冲区
   ↓
缓冲区填满，新请求被拒
   ↓
客户端和 Envoy 之间的连接积压
   ↓
可能导致客户端也变慢（级联效应）

有连接池限制的情况（如 maxPendingRequests=100）：
客户端持续发送请求
   ↓
Envoy 的待处理队列达到 100
   ↓
新请求立即返回 503（Service Unavailable）
   ↓
客户端快速感知故障，可以重试其他服务或熔断
   ↓
防止了级联故障的扩散
```

**实际配置建议：**

```yaml
# 针对不同类型的服务调整
trafficPolicy:
  connectionPool:
    tcp:
      maxConnections: 100          # 一般情况
    http:
      http1MaxPendingRequests: 100 # 普通 API
      
# 如果是流式服务或长连接：
      http2MaxRequests: 50000      # HTTP/2 更高效，可以更高
      
# 如果是批处理服务（对延迟不敏感）：
      http1MaxPendingRequests: 1000
```

### 异常检测（Outlier Detection）的工作原理

异常检测是 Istio 内置的**故障转移**机制。

```yaml
outlierDetection:
  consecutive5xxErrors: 5          # 触发条件
  interval: 30s                    # 检测周期
  baseEjectionTime: 30s            # 隔离时长
  maxEjectionPercent: 50           # 最多隔离 50%
  minRequestVolume: 5              # 流量太少时不检测
```

**执行流程（时间轴）：**

```
时刻 T0：
  Pod-A 收到请求 → 返回 500 错误（计数 1）

时刻 T0+1s：
  Pod-A 再次返回 500（计数 2）

时刻 T0+3s：
  Pod-A 连续 5 次返回 500（计数 5）✓ 达到触发条件
  → Pod-A 被标记为"异常"，进入隔离列表

时刻 T0+3s 到 T0+33s（30 秒隔离期间）：
  Envoy 停止向 Pod-A 发送流量，转向其他健康的 Pod

时刻 T0+33s：
  隔离期结束，Envoy 尝试向 Pod-A 发送**少量探测流量**
  
  如果探测成功 → Pod-A 恢复，重新加入负载均衡
  如果探测失败 → 进入更长的隔离期（指数退避）
```

**关键参数详解：**

| 参数 | 含义 | 实际影响 |
|------|------|--------|
| `consecutive5xxErrors: 5` | 错误触发阈值 | 值越小，越容易触发熔断；值越大，对瞬时错误容忍度高 |
| `interval: 30s` | 检测周期 | 越短，越快发现故障；越长，越少 CPU 消耗 |
| `baseEjectionTime: 30s` | 首次隔离时长 | 第一次隔离 30s，之后如继续失败会指数增长 |
| `maxEjectionPercent: 50` | 最多隔离比例 | 防止一次故障导致所有实例都被隔离 |
| `minRequestVolume: 5` | 最小流量阈值 | 防止低流量服务频繁误判 |

---

## 4. 实际场景：从 K8s Service 到 VirtualService 的演进

### 场景：productpage 应用的流量路由演进

#### 阶段 1：纯 K8s Service（无 Istio）

```yaml
# K8s 原生 Service
apiVersion: v1
kind: Service
metadata:
  name: productpage
spec:
  type: ClusterIP
  selector:
    app: productpage
  ports:
  - port: 8000
    targetPort: 8000

# 所有流量通过 kube-proxy 的 iptables 均匀分发到所有 Pod
```

**局限性：**
- 无法基于 HTTP 路径或 Header 路由
- 无法实现灰度发布（权重路由）
- 故障检测不细致

#### 阶段 2：添加 VirtualService（实现灰度发布）

```yaml
# Deployment：部署两个版本
apiVersion: apps/v1
kind: Deployment
metadata:
  name: productpage-v1
spec:
  selector:
    matchLabels:
      app: productpage
      version: v1
  template:
    metadata:
      labels:
        app: productpage
        version: v1
    spec:
      containers:
      - name: productpage
        image: productpage:v1

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: productpage-v2
spec:
  selector:
    matchLabels:
      app: productpage
      version: v2
  template:
    metadata:
      labels:
        app: productpage
        version: v2
    spec:
      containers:
      - name: productpage
        image: productpage:v2

---
# K8s Service（无版本区分，包含所有 Pod）
apiVersion: v1
kind: Service
metadata:
  name: productpage
spec:
  selector:
    app: productpage                # 不区分版本
  ports:
  - port: 8000
    targetPort: 8000

---
# VirtualService：基于权重路由到不同版本
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: productpage
spec:
  hosts:
  - productpage
  http:
  - route:
    - destination:
        host: productpage
        subset: v1
      weight: 90                   # 90% 流量 → v1（稳定版本）
    - destination:
        host: productpage
        subset: v2
      weight: 10                   # 10% 流量 → v2（测试版本）

---
# DestinationRule：定义版本分组
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: productpage
spec:
  host: productpage
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

**改进：**
- ✓ 90% 用户用稳定的 v1
- ✓ 10% 用户体验 v2，快速发现问题
- ✓ 问题严重可以 0 秒下线 v2（改权重为 0，无需重新部署）

#### 阶段 3：添加高级流量控制

```yaml
# VirtualService：基于 Header 和权重的混合路由
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: productpage
spec:
  hosts:
  - productpage
  http:
  # 规则 1：内部测试用户路由到 v2
  - match:
    - headers:
        x-test-user:
          exact: "true"
    route:
    - destination:
        host: productpage
        subset: v2
  
  # 规则 2：普通用户的灰度发布
  - route:
    - destination:
        host: productpage
        subset: v1
      weight: 90
    - destination:
        host: productpage
        subset: v2
      weight: 10

---
# DestinationRule：细粒度的连接控制
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: productpage
spec:
  host: productpage
  
  # 全局策略
  trafficPolicy:
    connectionPool:
      http:
        http1MaxPendingRequests: 200
    outlierDetection:
      consecutive5xxErrors: 5
      interval: 30s
  
  subsets:
  - name: v1
    labels:
      version: v1
    # v1 是稳定版本，允许更多连接
    trafficPolicy:
      connectionPool:
        http:
          http1MaxPendingRequests: 300
  
  - name: v2
    labels:
      version: v2
    # v2 是新版本，更保守
    trafficPolicy:
      connectionPool:
        http:
          http1MaxPendingRequests: 50
      outlierDetection:
        consecutive5xxErrors: 3    # 更敏感
        interval: 10s
```

**优势：**
- 内部测试用户总是用 v2（通过设置 Header）
- 普通用户 90% 用 v1，10% 用 v2
- v1 被信任，允许更高的并发
- v2 被谨慎对待，快速发现问题

---

## 5. 南北向流量：Ingress Gateway 与虚拟主机

### Ingress Gateway 与 K8s Ingress 的区别

```
K8s Ingress                      Istio Ingress Gateway
├─ 基于虚拟主机和路径           ├─ 基于虚拟主机、路径、协议
├─ 依赖 Ingress Controller       ├─ 由 Istio 提供（更强大）
├─ 配置存储在 Ingress 对象      ├─ 分离为 Gateway + VirtualService
├─ 更新快但功能有限             ├─ 功能完整（支持 TLS、超时等）
└─ 较轻量级                      └─ 部署网格级 Gateway Pod
```

### Gateway 与 VirtualService 的协作

**重要概念：** Gateway 只定义**如何暴露端口**，VirtualService 定义**流量如何路由**。

```yaml
# 步骤 1：创建 Gateway（绑定端口、TLS）
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  selector:
    istio: ingressgateway          # 选择标签为 istio:ingressgateway 的 Pod（通常是 ingress-gateway）
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "bookinfo.com"               # 暴露这个域名的 HTTP 流量
    - "www.bookinfo.com"
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: bookinfo-tls # Secret 中的证书
    hosts:
    - "bookinfo.com"

---
# 步骤 2：创建 VirtualService（关联 Gateway，定义路由）
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - "bookinfo.com"                 # 必须与 Gateway 的 hosts 匹配
  gateways:
  - bookinfo-gateway               # 关联上面的 Gateway
  http:
  - match:
    - uri:
        prefix: "/productpage"
    route:
    - destination:
        host: productpage
        port:
          number: 8000
  - match:
    - uri:
        prefix: "/details"
    route:
    - destination:
        host: details
        port:
          number: 9080
```

**流量路径：**

```
外部请求
GET http://bookinfo.com/productpage
        ↓
Gateway Listener（0.0.0.0:80）拦截
        ↓
匹配 host = bookinfo.com ✓
        ↓
转给 VirtualService bookinfo 处理
        ↓
匹配 path prefix = /productpage ✓
        ↓
路由到 productpage:8000
        ↓
Sidecar 拦截，最终到达 productpage 应用
```

### 实际案例：多版本应用的南北向暴露

假设你要：
- productpage 暴露给外部用户
- 内部测试用户通过特殊 Header 到 v2
- 外部用户 90% 用 v1，10% 用 v2

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: productpage
spec:
  hosts:
  - "bookinfo.example.com"
  gateways:
  - bookinfo-gateway
  http:
  # 规则 1：内部测试用户（通过 internal-test:true Header）
  - match:
    - headers:
        internal-test:
          exact: "true"
    route:
    - destination:
        host: productpage
        subset: v2
    timeout: 10s

  # 规则 2：普通外部用户的灰度
  - route:
    - destination:
        host: productpage
        subset: v1
      weight: 90
    - destination:
        host: productpage
        subset: v2
      weight: 10
    timeout: 15s
    retries:
      attempts: 3
      perTryTimeout: 5s
```

---

## 6. 常见路由陷阱与最佳实践

### 陷阱 1：VirtualService 与 Gateway 的 hosts 不匹配

```yaml
# ❌ 错误示例
Gateway:
  hosts:
  - "example.com"

VirtualService:
  hosts:
  - "api.example.com"  # 不匹配！流量无法路由

# ✓ 正确示例
Gateway:
  hosts:
  - "*.example.com"     # 通配符

VirtualService:
  hosts:
  - "api.example.com"   # 现在匹配 ✓
```

### 陷阱 2：Subset 标签不存在

```yaml
# ❌ 错误示例
DestinationRule:
  subsets:
  - name: v1
    labels:
      version: v1       # 但没有 Pod 带有这个标签！

# 诊断：
kubectl get pods --show-labels | grep version

# ✓ 修复：
# 确保 Pod 的标签与 DestinationRule 定义的标签一致
kubectl label pods <pod-name> version=v1
```

### 陷阱 3：规则顺序导致匹配失败

```yaml
# ❌ 错误示例
http:
- route:              # 规则 1：兜底规则（所有流量）
  - destination:
      host: default-service
      
- match:              # 规则 2：永不执行！所有流量已在规则 1 消耗
  - uri:
      exact: "/health"
  route:
  - destination:
      host: health-service

# ✓ 正确示例
http:
- match:              # 规则 1：先匹配特殊情况
  - uri:
      exact: "/health"
  route:
  - destination:
      host: health-service
      
- route:              # 规则 2：最后的兜底规则
  - destination:
      host: default-service
```

### 最佳实践总结

1. **分离关注点**
   - VirtualService：定义"路由去向"
   - DestinationRule：定义"连接策略"
   - Gateway：定义"如何暴露"
   
2. **使用命名规范**
   ```yaml
   VirtualService:
     metadata:
       name: productpage-vs
   
   DestinationRule:
     metadata:
       name: productpage-dr
   
   Gateway:
     metadata:
       name: productpage-gateway
   ```

3. **验证配置**
   ```bash
   # 检查配置是否有问题
   istioctl analyze -n bookinfo
   
   # 验证路由规则是否生效
   istioctl proxy-config routes -n bookinfo <pod-name>
   ```

4. **渐进式灰度发布的标准流程**
   ```yaml
   Day 1：
     v1: 100%, v2: 0%
   
   Day 2-3：
     v1: 90%, v2: 10%
   
   Day 4-5：
     v1: 75%, v2: 25%
   
   Day 6-7：
     v1: 50%, v2: 50%
   
   Day 8：
     v1: 0%, v2: 100%  # 完全切换
   ```

---

## 7. 本章小结

### 核心概念

| 概念 | 角色 |
|------|------|
| **VirtualService** | 流量路由表（Where） |
| **DestinationRule** | 连接策略（How） |
| **Gateway** | 入口网关配置（How to expose） |
| **Subset** | 版本分组（基于 Pod 标签） |
| **outlierDetection** | 自动故障转移 |
| **connectionPool** | 防止级联故障 |

### 快速参考

**创建基本的灰度发布：**

```bash
# 1. 创建 Gateway（允许外部访问）
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: app-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      protocol: HTTP
    hosts:
    - "*.example.com"
EOF

# 2. 创建 VirtualService（路由规则）
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: app-vs
spec:
  hosts:
  - "app.example.com"
  gateways:
  - app-gateway
  http:
  - route:
    - destination:
        host: app
        subset: v1
      weight: 90
    - destination:
        host: app
        subset: v2
      weight: 10
EOF

# 3. 创建 DestinationRule（版本分组）
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: app-dr
spec:
  host: app
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
EOF
```

---

## 8. 下一步

第四章将深入讲解：
- **弹性与韧性**：超时、重试、熔断、限流的详细机制
- **故障注入**：如何安全地测试应用的容错能力
- **调试与观测**：如何快速诊断流量问题

掌握这些，你就能构建一个真正稳定、可控的微服务系统！ 🚀
