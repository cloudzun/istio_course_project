# 第二章：数据平面的魔法 —— Sidecar 与应用接入

> **章节目标**：理解 Envoy Sidecar 的工作原理，掌握应用接入 Istio 的机制，学会通过日志和工具诊断数据平面的问题。

---

## 1. Envoy 代理基础知识

### 什么是 Envoy？

Envoy 是一个用 C++ 编写的**网络代理**，由 Lyft 开源。它的设计哲学是：

> 成为应用和网络之间的**透明中介**。应用无需知道代理的存在，但代理可以对所有流量进行精细控制。

**核心特点：**

| 特点 | 说明 |
|------|------|
| **性能** | C++ 编写，单进程可处理数百万连接 |
| **动态配置** | 支持 xDS API，配置无需重启即时生效 |
| **协议支持** | HTTP/1.1, HTTP/2, HTTP/3, gRPC, WebSocket, TCP 等 |
| **可观测性** | 内置访问日志、指标导出、追踪支持 |
| **热重启** | 支持平滑升级，无连接中断 |

### Envoy 的线程模型与处理流程

当一个网络包到达 Envoy 时，会经过以下处理流程：

```
入站流量
   ↓
┌─────────────────────────────────┐
│   Listener（监听器）             │
│   绑定到特定 IP:Port             │
└────────────┬────────────────────┘
             ↓
┌─────────────────────────────────┐
│   Filter Chain（过滤链）          │
│   L4（TCP） 过滤器                │
│   • TLS 终止                     │
│   • TCP 代理                     │
└────────────┬────────────────────┘
             ↓
┌─────────────────────────────────┐
│   L7（HTTP） 过滤器               │
│   • HTTP 路由                    │
│   • Header 修改                  │
│   • 故障注入                     │
└────────────┬────────────────────┘
             ↓
┌─────────────────────────────────┐
│   Cluster（集群）                 │
│   • 负载均衡算法选择目标           │
│   • 连接池管理                   │
│   • 熔断判断                     │
└────────────┬────────────────────┘
             ↓
出站流量（到达目标服务）
```

**关键概念解释：**

- **Listener**：Envoy 绑定的入站端口。在 Sidecar 模式下，有两个主要 Listener：
  - `0.0.0.0:15000`（管理员接口，用于调试）
  - `0.0.0.0:15006`（代理入站流量）

- **Filter Chain**：按顺序应用的过滤规则。L4 处理 TCP，L7 处理 HTTP。

- **Cluster**：一组具有相同功能的后端服务实例，例如所有 `productpage:v1` 的 Pod。

- **Endpoint**：Cluster 中的单个实例，由 IP:Port 标识。

### xDS 协议详解

Istiod 通过 xDS 将配置实时推送给 Envoy。xDS 包含四个主要服务：

```
Istiod 维护配置           Envoy 订阅配置
     ↓                        ↑
  ┌──────┐              ┌──────────┐
  │ LDS  │←──────────── │ Listener │
  │(监听│  gRPC        │ Discovery│
  │ 器) │              │ Service  │
  └──────┘              └──────────┘
     
  ┌──────┐              ┌──────────┐
  │ RDS  │←──────────── │  Route   │
  │(路由)│  gRPC        │ Discovery│
  └──────┘              │ Service  │
     
  ┌──────┐              ┌──────────┐
  │ CDS  │←──────────── │ Cluster  │
  │(集群)│  gRPC        │ Discovery│
  └──────┘              │ Service  │
     
  ┌──────┐              ┌──────────┐
  │ EDS  │←──────────── │ Endpoint │
  │(端点)│  gRPC        │ Discovery│
  └──────┘              │ Service  │
```

**四个 D 的含义：**

1. **LDS (Listener Discovery Service)**
   - Envoy 需要监听哪些端口？
   - 示例：监听 `0.0.0.0:15006` 接收入站流量

2. **RDS (Route Discovery Service)**
   - 对于某个 HTTP Listener，流量应该如何路由？
   - 示例：请求 `Host: productpage` 时，根据路径前缀转发到不同的服务

3. **CDS (Cluster Discovery Service)**
   - 有哪些后端服务集群？
   - 示例：`productpage` Cluster 的负载均衡策略是什么？

4. **EDS (Endpoint Discovery Service)**
   - 某个 Cluster 有哪些实例（Pod）？
   - 示例：`productpage:v1` Cluster 包含 IP 为 10.0.1.5 和 10.0.2.3 的两个 Pod

---

## 2. Sidecar 注入机制

### 自动注入 vs 手动注入

Istio 提供两种方式将 Envoy 注入到 Pod：

#### 方式 1：自动注入（推荐）

通过 Webhook 在 Pod 创建时自动注入 Sidecar。

**步骤 1：标记命名空间**

```bash
kubectl label namespace default istio-injection=enabled
```

**步骤 2：创建任何 Pod，Sidecar 会自动被注入**

```bash
kubectl apply -f deployment.yaml
# Pod 创建时，Istio 的 Mutating Webhook 会自动修改 Pod 定义
```

**工作原理：**

```
用户提交 Pod Spec
       ↓
Kubernetes API 收到请求
       ↓
检查命名空间标签 istio-injection=enabled？
       ↓
是 → 调用 istio-sidecar-injector Webhook
       ↓
Webhook 修改 Pod Spec，添加 Sidecar 容器
       ↓
修改后的 Pod 被创建
```

#### 方式 2：手动注入

直接使用 `istioctl kube-inject` 生成修改后的 YAML。

```bash
# 不修改原文件，只输出修改后的 YAML
istioctl kube-inject -f deployment.yaml | kubectl apply -f -

# 或修改文件后再应用
istioctl kube-inject -f deployment.yaml -o deployment-injected.yaml
kubectl apply -f deployment-injected.yaml
```

**使用场景：**
- 开启了自动注入但需要排除特定 Pod（加 `sidecar.istio.io/inject: "false"`）
- 集群中多个 Istio 实例，需要注入特定版本

### Sidecar 容器详解

当 Pod 被注入后，会添加以下容器：

```yaml
# 原始 Pod
apiVersion: v1
kind: Pod
metadata:
  name: productpage-v1-xxx
spec:
  containers:
  - name: productpage
    image: gcr.io/istio-release/productpage:1.16
    ports:
    - containerPort: 8000

---
# 注入后的 Pod（由 Webhook 自动修改）
apiVersion: v1
kind: Pod
metadata:
  name: productpage-v1-xxx
spec:
  initContainers:                        # ← 新增：初始化容器
  - name: istio-init
    image: gcr.io/istio-release/proxyv2:1.26
    args:                                # ← iptables 规则配置
    - istio-iptables
    - -p
    - "15001"                            # Envoy 出站拦截端口
    - -z
    - "15006"                            # Envoy 入站拦截端口
    - -u
    - "1337"                             # Envoy 运行用户 UID
    - -m
    - REDIRECT                           # 拦截模式
    - -i
    - "*"                                # 拦截所有网络接口
    - -x
    - ""                                 # 排除的 IP（CIDR）
    - -b
    - "*"                                # 被拦截的入站端口
    - -d
    - "15090,15021,15020"                # 排除的出站端口（Envoy 自身）
    securityContext:
      privileged: true                   # 需要特权才能配置 iptables
  containers:
  - name: productpage                    # 原容器
    image: gcr.io/istio-release/productpage:1.16
    ports:
    - containerPort: 8000
    env:
    - name: ENABLE_PROFILER
      value: "false"
  - name: istio-proxy                    # ← 新增：Sidecar 容器
    image: gcr.io/istio-release/proxyv2:1.26
    ports:
    - name: http-envoy-prom
      containerPort: 15000               # 管理员接口 & 指标
      protocol: TCP
    - name: http-envoy-metrics
      containerPort: 15090
      protocol: TCP
    args:
    - proxy
    - sidecar
    - --domain
    - default.svc.cluster.local          # K8s DNS 域名
    - --proxyLogLevel=warning
    - --proxyComponentLogLevel=misc:error
    env:
    - name: ISTIO_META_POD_PORTS
      value: '[{"containerPort":8000,"protocol":"TCP"}]'
    - name: ISTIO_META_APP_CONTAINERS
      value: productpage                 # 应用容器名称
    - name: POD_NAME
      valueFrom:
        fieldRef:
          fieldPath: metadata.name
    - name: POD_NAMESPACE
      valueFrom:
        fieldRef:
          fieldPath: metadata.namespace
    securityContext:
      privileged: false
      runAsUser: 1337                    # Envoy 运行用户（非 root）
      allowPrivilegeEscalation: false
    resources:
      limits:
        cpu: 2000m
        memory: 1024Mi
      requests:
        cpu: 100m
        memory: 128Mi
    volumeMounts:
    - name: istio-envoy
      mountPath: /etc/istio/proxy        # Envoy 配置目录
  volumes:
  - name: istio-envoy
    emptyDir: {}                         # 临时配置目录
```

### iptables 流量拦截原理

Init Container `istio-init` 在 Pod 启动时执行，配置 Linux iptables 规则，使所有流量都通过 Envoy。

**具体规则逻辑：**

```bash
# 伪代码，实际规则更复杂
# 1. 创建自定义链
iptables -t nat -N ISTIO_REDIRECT
iptables -t nat -N ISTIO_INBOUND

# 2. 所有出站流量重定向到 Envoy 15001 端口
iptables -t nat -A OUTPUT -j ISTIO_REDIRECT
iptables -t nat -A ISTIO_REDIRECT -p tcp -j REDIRECT --to-port 15001

# 3. 排除 Envoy 自身流量（防止无限循环）
iptables -t nat -A ISTIO_REDIRECT -m owner --uid-owner 1337 -j RETURN

# 4. 所有入站流量（来自其他 Pod）重定向到 Envoy 15006 端口
iptables -t nat -A PREROUTING -j ISTIO_INBOUND
iptables -t nat -A ISTIO_INBOUND -p tcp -j REDIRECT --to-port 15006
```

**流量流向图：**

```
应用容器发送请求到 Service IP（如 10.0.0.5）
           ↓
    iptables 规则匹配 OUTPUT
           ↓
    重定向到 Envoy 15001 端口
           ↓
  Envoy 查询 RDS 配置，判断路由
           ↓
  Envoy 连接目标服务的 Sidecar
           ↓
目标 Pod 的 iptables 规则匹配 PREROUTING
           ↓
   重定向到 Envoy 15006 端口
           ↓
目标 Pod 的 Envoy 将流量转发给应用
```

**关键细节：**

- **Envoy 运行为 UID 1337**：避免 Envoy 的流量被自身的 iptables 规则拦截
- **Init Container 需要特权**：配置 iptables 需要 `CAP_NET_ADMIN` 权限
- **流量完全对应用透明**：应用不知道流量被代理，仍然按正常方式编程

---

## 3. 示例应用：Bookinfo 微服务架构

### Bookinfo 应用概览

Bookinfo 是 Istio 官方提供的示例应用，用于演示服务网格的各种功能。它模拟了一个在线书店。

**应用架构：**

```
┌──────────────────────────────────────────────────────────┐
│                     Ingress Gateway                       │
│              (入口网关，对外暴露 HTTP 服务)                │
└──────────────────────┬───────────────────────────────────┘
                       │ 流量入口 :80, :443
                       ↓
      ┌────────────────────────────────┐
      │     productpage Pod (v1)       │
      │  ┌──────────────┐┌──────────┐  │
      │  │ productpage  ││Envoy     │  │
      │  │ (Python)     ││Sidecar   │  │
      │  │ :8000        ││:15000    │  │
      │  └──────────────┘└──────────┘  │
      └────────┬───────────────────────┘
               │ 调用 reviews, details, ratings 服务
               ↓
   ┌───────────────────────────────────────────┐
   │                                           │
   ↓                ↓                          ↓
┌──────────┐  ┌──────────┐           ┌──────────────┐
│ details  │  │ reviews  │           │   ratings    │
│ Pod(v1)  │  │ Pod(多版)│           │  Pod(v1)     │
│ Java:9080│  │ Java:9080│           │  Node:9080   │
│+Envoy    │  │+Envoy    │           │  +Envoy      │
└──────────┘  └──────────┘           └──────────────┘
              
              reviews 有 3 个版本：
              v1: 无星级显示
              v2: 黑色星级
              v3: 红色星级
```

### 部署 Bookinfo 应用

**步骤 1：创建命名空间并启用 Sidecar 注入**

```bash
# 创建命名空间
kubectl create namespace bookinfo

# 启用自动 Sidecar 注入
kubectl label namespace bookinfo istio-injection=enabled

# 验证标签
kubectl get namespace bookinfo --show-labels
# 输出：istio-injection=enabled
```

**步骤 2：部署应用**

```bash
# 部署 Bookinfo 应用
kubectl apply -n bookinfo -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/bookinfo/platform/kube/bookinfo.yaml

# 等待所有 Pod 启动（约 30-60 秒）
kubectl get pods -n bookinfo -w

# 预期输出（所有 Pod 应为 Running，Ready 为 2/2）：
# NAME                           READY   STATUS    RESTARTS   AGE
# details-v1-5498c8f4c4-xxxxx    2/2     Running   0          10s  ← 2/2 表示应用+Sidecar
# productpage-v1-6b7f99f6d8-xxxx 2/2     Running   0          10s
# ratings-v1-5f8c45f7b-xxxxx     2/2     Running   0          10s
# reviews-v1-8676bb4789-xxxxx    2/2     Running   0          10s
# reviews-v2-86dbf4b6bd-xxxxx    2/2     Running   0          10s
# reviews-v3-5d6fb7cc6f-xxxxx    2/2     Running   0          10s
```

**步骤 3：验证 Sidecar 注入**

```bash
# 查看某个 Pod 的详细信息
kubectl describe pod -n bookinfo <pod-name>

# 输出应该包含：
# Containers:
#   productpage:                        ← 应用容器
#     Image: gcr.io/istio-release/productpage
#   istio-proxy:                        ← Sidecar 容器
#     Image: gcr.io/istio-release/proxyv2
# Init Containers:
#   istio-init:                         ← 初始化容器（配置 iptables）
#     Image: gcr.io/istio-release/proxyv2
```

**步骤 4：验证服务间通信**

使用 `kubectl exec` 在 Pod 内测试服务发现和通信：

```bash
# 进入 productpage Pod
kubectl exec -it -n bookinfo <productpage-pod-name> -c productpage -- sh

# 在 Pod 内测试连接到其他服务
curl -v http://details:9080/details/1

# 预期看到 200 OK 响应，包含图书详情信息
```

---

## 4. 流量拦截与 Sidecar 通信流程

### 详细流程图

当 productpage 应用发起请求时：

```
步骤 1: productpage 应用进程
   curl http://details:9080/details/1
   ↓
步骤 2: DNS 解析
   Kubernetes DNS 将 details 解析为 details.bookinfo.svc.cluster.local
   得到 Cluster IP: 10.0.0.8
   ↓
步骤 3: 应用建立 TCP 连接
   connect(10.0.0.8:9080)
   ↓
步骤 4: iptables 拦截（OUTPUT 链）
   匹配规则：目标是 TCP，且不是 UID 1337（Envoy 自身）
   重定向到 127.0.0.1:15001（Envoy 出站端口）
   ↓
步骤 5: Envoy 接收（出站）
   监听在 127.0.0.1:15001 的 Filter Chain
   ↓
步骤 6: Envoy 查询 RDS（Route Discovery Service）
   Istiod 推送的配置告诉 Envoy：
   "对于 details 服务，使用 RoundRobin 负载均衡，转发到 Cluster: details:9080"
   ↓
步骤 7: Envoy 查询 EDS（Endpoint Discovery Service）
   找出 details:9080 的所有 Pod：
   - 10.0.1.5:9080（details-v1 的 Pod）
   ↓
步骤 8: Envoy 选择目标端点
   使用负载均衡算法（如 RoundRobin）选择一个端点：
   假设选中 10.0.1.5:9080
   ↓
步骤 9: Envoy 建立连接到目标
   如果启用 mTLS，进行 TLS 握手；否则直接建立 TCP 连接
   ↓
步骤 10: 请求到达目标 Pod 的网络接口
   iptables 规则匹配 PREROUTING 链
   重定向到 127.0.0.1:15006（Envoy 入站端口）
   ↓
步骤 11: 目标 Pod 的 Envoy 接收（入站）
   监听在 127.0.0.1:15006 的 Filter Chain
   ↓
步骤 12: Envoy 解密（如果启用了 mTLS）并转发给应用
   127.0.0.1:9080（本地应用容器）
   ↓
步骤 13: 应用处理请求
   details 应用处理请求，返回响应
   ↓
步骤 14: 响应返回
   响应经过 Envoy 代理层层返回，最终回到 productpage 应用
```

### 验证流量拦截

进入 Pod 查看 iptables 规则：

```bash
# 进入 Pod 的应用容器
kubectl exec -it -n bookinfo <productpage-pod-name> -c productpage -- sh

# 查看 iptables 规则（需要 Pod 内有 iptables 工具）
iptables -t nat -L -n

# 预期输出（简化版）：
# Chain PREROUTING (policy ACCEPT)
# target     prot opt source               destination
# ISTIO_INBOUND  all  --  0.0.0.0/0        0.0.0.0/0
# 
# Chain OUTPUT (policy ACCEPT)
# target     prot opt source               destination
# ISTIO_REDIRECT all  --  0.0.0.0/0        0.0.0.0/0
```

---

## 5. 诊断工具与日志分析

### 工具 1：istioctl proxy-config

查看 Envoy 的配置状态。

**查看路由（Routes）**

```bash
# 查看某个 Pod 的所有路由规则
istioctl proxy-config route -n bookinfo <productpage-pod-name>

# 输出示例：
# NAME                              DOMAINS            MATCH                 VIRTUAL SERVICE
# http.8000                          *                  /                     productpage:8000
# http.9080                          *                  /                     details
# ...
```

**查看集群（Clusters）**

```bash
# 查看所有 Cluster（后端服务）
istioctl proxy-config cluster -n bookinfo <productpage-pod-name>

# 输出示例：
# SERVICE FQDN                      PORT     SUBSET     DIRECTION     TYPE
# details.bookinfo.svc.cluster.local 9080     -          outbound      EDS
# reviews.bookinfo.svc.cluster.local 9080     -          outbound      EDS
# ratings.bookinfo.svc.cluster.local 9080     -          outbound      EDS
# productpage.bookinfo.svc.cluster.local 8000 -          outbound      EDS
```

**查看端点（Endpoints）**

```bash
# 查看某个 Cluster 的所有端点
istioctl proxy-config endpoints -n bookinfo <productpage-pod-name> | grep details

# 输出示例：
# ENDPOINT                     STATUS      OUTLIER CHECK
# 10.0.1.5:9080               HEALTHY     OK
# 10.0.2.3:9080               HEALTHY     OK
```

### 工具 2：Envoy 访问日志

Envoy 可以记录每一条通过代理的请求，帮助诊断流量问题。

**启用访问日志**

```bash
# 查看 Pod 的 Sidecar 日志（Envoy 输出）
kubectl logs -n bookinfo <productpage-pod-name> -c istio-proxy

# 或实时查看日志
kubectl logs -n bookinfo <productpage-pod-name> -c istio-proxy -f

# 预期输出（Envoy 访问日志）：
# [2026-02-08T06:30:00.000Z] "GET /details/1 HTTP/1.1" 200 - 0 256 45 40 "-" "curl/7.85.0" "xxx" "details:9080" "10.0.1.5:9080"
```

**日志字段解释：**

```
[时间戳] "HTTP 方法 路径 协议版本" 响应码 响应码详情 
        请求体字节 响应体字节 处理时间(ms) 连接时间(ms)
        Referrer User-Agent Authority Upstream Host 
        Upstream Host IP
```

### 工具 3：Envoy Admin 界面

Envoy 提供了一个强大的管理接口，可以在线查看和修改配置。

**端口转发到管理接口**

```bash
# 转发本地 15000 到 Pod 的 Envoy 管理端口
kubectl port-forward -n bookinfo <productpage-pod-name> 15000:15000 &

# 在浏览器访问
http://localhost:15000

# 或在命令行查询
curl http://localhost:15000/config_dump | jq
curl http://localhost:15000/clusters
curl http://localhost:15000/stats
```

**Admin 接口常用端点：**

| 端点 | 用途 |
|------|------|
| `/config_dump` | 查看 Envoy 的完整配置（JSON 格式） |
| `/clusters` | 列出所有 Cluster 及其健康状态 |
| `/stats` | 查看实时统计信息（连接数、请求数等） |
| `/stats/prometheus` | Prometheus 格式的指标 |
| `/logging` | 动态调整日志级别 |

**示例：查看 productpage Pod 的 Cluster 信息**

```bash
# 端口转发
kubectl port-forward -n bookinfo <productpage-pod-name> 15000:15000 &

# 查看所有 Cluster
curl http://localhost:15000/clusters | grep -A 10 "details"

# 输出示例：
# details.bookinfo.svc.cluster.local:9080::default_priority::max_connections::1000
# details.bookinfo.svc.cluster.local:9080::default_priority::max_pending_requests::1000
# 10.0.1.5:9080 10.0.1.5:9080 HEALTHY OK
# 10.0.2.3:9080 10.0.2.3:9080 HEALTHY OK
```

---

## 6. 常见问题与排查指南

### 问题 1：Pod 中缺少 Sidecar（Ready: 1/1 而不是 2/2）

**原因排查：**

```bash
# 1. 检查命名空间标签
kubectl get namespace bookinfo --show-labels
# 如果没有 istio-injection=enabled，加上标签
kubectl label namespace bookinfo istio-injection=enabled

# 2. 检查 Pod 创建时间
# Sidecar 只在 Pod 创建时注入，已有的 Pod 不会自动更新
# 解决：删除 Pod，让 Deployment 重新创建
kubectl rollout restart deployment -n bookinfo productpage-v1

# 3. 检查 Webhook 是否运行
kubectl get mutatingwebhookconfigurations | grep istio
kubectl get validatingwebhookconfigurations | grep istio

# 4. 检查 Webhook 日志
kubectl logs -n istio-system -l app=sidecar-injector
```

### 问题 2：服务间无法通信（503 Service Unavailable）

**排查流程：**

```bash
# 步骤 1：验证应用容器是否健康
kubectl exec -it -n bookinfo <productpage-pod-name> -c productpage -- curl http://localhost:8000

# 步骤 2：验证 Sidecar 是否正常
kubectl logs -n bookinfo <productpage-pod-name> -c istio-proxy | grep error

# 步骤 3：检查目标服务是否存在
kubectl get svc -n bookinfo
kubectl get endpoints -n bookinfo

# 步骤 4：检查 Envoy 配置（是否有路由规则）
istioctl proxy-config routes -n bookinfo <productpage-pod-name>

# 步骤 5：检查 Endpoint 健康状态
istioctl proxy-config endpoints -n bookinfo <productpage-pod-name>

# 步骤 6：查看 Envoy 访问日志，找出具体错误
kubectl logs -n bookinfo <productpage-pod-name> -c istio-proxy | tail -20
```

**常见错误码解释：**

| 错误码 | 含义 | 常见原因 |
|--------|------|--------|
| `NR` | No Route | VirtualService 规则不匹配，或未创建 |
| `UO` | Upstream Overflow | Cluster 连接池满，触发限流 |
| `UH` | Upstream Healthcheck Failed | 后端实例故障 |
| `DC` | Downstream Connection Termination | 客户端关闭连接 |

### 问题 3：Sidecar 启动失败（CrashLoopBackOff）

**排查步骤：**

```bash
# 查看 Pod 事件
kubectl describe pod -n bookinfo <pod-name>

# 查看 Sidecar 日志
kubectl logs -n bookinfo <pod-name> -c istio-proxy --previous

# 常见原因：
# 1. 节点 CPU/内存不足 → 增加节点或调整资源限制
# 2. 权限问题 → 检查 SecurityContext 配置
# 3. Init Container 失败 → 查看 istio-init 日志

# 查看 Init Container 日志
kubectl logs -n bookinfo <pod-name> -c istio-init
```

---

## 7. 性能考量与优化建议

### Sidecar 资源消耗

默认情况下，Envoy Sidecar 的资源占用：

```yaml
resources:
  requests:
    cpu: 100m              # 最低保证 100m CPU
    memory: 128Mi          # 最低保证 128MB 内存
  limits:
    cpu: 2000m             # 最多可用 2 核 CPU
    memory: 1024Mi         # 最多可用 1GB 内存
```

**优化建议：**

1. **对于小型集群或演示环境**
   - 降低资源限制：`cpu: 500m, memory: 256Mi`

2. **对于生产环境**
   - 根据实际负载进行压测和调整
   - 监控 Pod 的实际资源使用情况

3. **使用 Ambient Mesh**
   - 减少 Sidecar 数量，降低总体资源消耗

### 启动顺序与竞态条件

在某些情况下，应用容器可能在 Sidecar 启动前就尝试连接：

```bash
# Pod 启动日志示例：
# 10:20:00 istio-init started
# 10:20:01 istio-init completed
# 10:20:02 productpage started  ← 应用启动，但 Sidecar 还未完全初始化
# 10:20:03 Connection refused!
# 10:20:04 istio-proxy started  ← 太晚了
```

**解决方案：**

在 Deployment 中配置 `holdApplicationUntilProxyStarts`：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: productpage-v1
spec:
  template:
    metadata:
      annotations:
        proxy.istio.io/config: |
          {
            "holdApplicationUntilProxyStarts": true
          }
    spec:
      containers:
      - name: productpage
        image: gcr.io/istio-release/productpage:1.16
```

---

## 8. 本章小结

### 核心概念回顾

| 概念 | 说明 |
|------|------|
| **Envoy** | 高性能的 L4/L7 网络代理 |
| **Sidecar 注入** | 在 Pod 创建时自动/手动添加 Envoy 容器 |
| **iptables 拦截** | Init Container 配置规则，使流量无感知地通过 Envoy |
| **xDS 协议** | 实时推送 LDS/RDS/CDS/EDS，Envoy 动态感知配置变化 |
| **Bookinfo** | 官方示例应用，包含 productpage/details/reviews/ratings 四个微服务 |

### 工具清单

```bash
# 查看 Sidecar 配置
istioctl proxy-config routes -n <namespace> <pod-name>
istioctl proxy-config cluster -n <namespace> <pod-name>
istioctl proxy-config endpoints -n <namespace> <pod-name>

# 查看 Sidecar 日志
kubectl logs -n <namespace> <pod-name> -c istio-proxy
kubectl logs -n <namespace> <pod-name> -c istio-proxy -f

# 访问 Envoy Admin 接口
kubectl port-forward -n <namespace> <pod-name> 15000:15000
curl http://localhost:15000/config_dump
curl http://localhost:15000/stats

# 诊断应用和 Sidecar
kubectl describe pod -n <namespace> <pod-name>
kubectl exec -it -n <namespace> <pod-name> -c <container-name> -- <command>
```

### 思考题

1. **为什么 Init Container 需要特权（privileged），而应用和 Sidecar 容器不需要？**
   - 提示：iptables 配置需要什么权限？

2. **Envoy 为什么要以 UID 1337 运行？这样做有什么好处？**
   - 提示：防止什么问题？

3. **如果你要在一个 1000 个 Pod 的集群上部署 Istio，Sidecar 模式和 Ambient Mesh 各需要多少代理进程？**
   - 提示：按节点数和 Pod 数计算

---

## 9. 下一步

现在你已经理解了 Istio 的数据平面如何工作，以及如何部署和诊断 Sidecar。

**第三章** 会开始真正的流量管理实战：
- 创建 VirtualService 和 DestinationRule
- 实现蓝绿发布和金丝雀灰度发布
- 配置 Ingress Gateway 暴露应用给外部用户

准备好了吗？开始流量管理的魔法！ 🚀
