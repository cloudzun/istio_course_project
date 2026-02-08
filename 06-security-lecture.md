# 第六章：安全基础 —— 零信任网络架构

> **章节目标**：理解 Istio 的安全模型，掌握 mTLS 自动化、认证和授权的设计原理，构建一个真正安全的微服务系统。

---

## 1. 传统安全模型的困境

### 边界安全的假设

传统网络架构建立在一个核心假设之上：

```
┌─────────────────────────────────────┐
│       "信任的边界"（防火墙）         │
│                                     │
│  ┌───────────────────────────────┐ │
│  │      内部网络（可信任）        │ │
│  │  ├─ Pod A                     │ │
│  │  ├─ Pod B                     │ │
│  │  └─ Pod C                     │ │
│  │  所有内部通信不加密            │ │
│  └───────────────────────────────┘ │
│         ↑ 防火墙阻止外部访问        │
│    互联网（不可信任）              │
└─────────────────────────────────────┘
```

**这个模型的问题：**

1. **内部威胁被忽视**
   ```
   情景：某个 Pod 被攻击者入侵
         ↓
   攻击者获得 Pod 内的进程权限
         ↓
   可以访问同网络的所有其他 Pod（无密钥要求）
         ↓
   从数据库到 API，全部被攻击者窃取
   ```

2. **无法识别服务身份**
   ```
   问题：Pod A 收到来自网络的请求
         ↓
   无法确认请求来自真正的 Pod B（可能是仿冒）
         ↓
   无法细粒度控制访问（只能按网络隔离）
   ```

3. **加密部署复杂**
   ```
   传统方式：应用负责 TLS
         ↓
   每个应用都要生成证书、管理密钥
         ↓
   证书过期、轮换是噩梦
         ↓
   不同团队的应用可能使用不同的加密方式（不统一）
   ```

### 零信任安全（Zero Trust）

Istio 采用**零信任**模型：

```
假设：网络不可信（即使是内部网络）

原则：
  1. 所有通信必须加密
  2. 所有服务必须相互验证身份
  3. 每个请求都需要授权检查
  4. 一切流量都可审计
```

**零信任的架构：**

```
┌──────────────────────────────────┐
│  Pod A                           │
│  ├─ 应用进程                     │
│  └─ Envoy Sidecar                │
│     ├─ 自动加密所有出站流量      │
│     └─ 验证出站的目标身份        │
└──────────────────────────────────┘
         ↓ TLS 加密通道
┌──────────────────────────────────┐
│  Pod B                           │
│  ├─ Envoy Sidecar                │
│  │  ├─ 解密入站流量              │
│  │  ├─ 验证客户端身份            │
│  │  ├─ 检查授权策略              │
│  │  └─ 审计请求                  │
│  └─ 应用进程                     │
│     （应用收到明文请求）         │
└──────────────────────────────────┘
```

---

## 2. mTLS（相互 TLS 认证）的自动化

### TLS 基础回顾

**单向 TLS（HTTPS）：**
- 客户端验证服务器身份（通过证书）
- 服务器**不**验证客户端身份
- 常见于 Web 浏览器 vs 网站

**双向 TLS（mTLS）：**
- 服务器验证客户端身份
- 客户端也验证服务器身份
- 通信双方都有证书

### Istio 的 mTLS 自动化

**关键创新：** Istiod 自动生成和轮换证书，应用完全无感知。

```
时刻 T0：新 Pod 启动
   ↓
Envoy Sidecar 启动
   ↓
Envoy 向 Istiod 请求证书
   ↓
Istiod：
  1. 验证请求来自真实的 Kubernetes Pod
  2. 生成私钥和证书
  3. 返回给 Envoy
   ↓
Envoy 在内存中保存证书和密钥
（不写入磁盘，更安全）
   ↓
Envoy 使用证书进行 mTLS 通信

时刻 T0+90天：证书即将过期
   ↓
Istiod 自动推送新证书
   ↓
Envoy 无缝切换（无连接中断）
```

### Istio mTLS 的三种模式

#### 模式 1：PERMISSIVE（宽松模式，过渡期）

```
Envoy 接受两种流量：
  ├─ mTLS 加密流量（验证客户端身份）
  └─ 明文流量（不验证）
```

**使用场景：**
- 网格迁移期（部分应用还未启用 Sidecar）
- 临时兼容旧系统

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: bookinfo
spec:
  mtls:
    mode: PERMISSIVE    # 接受加密和明文
```

**工作流程：**

```
客户端请求到达（可能是明文或加密）
   ↓
Envoy 检查：是否加密？
   ├─ 是 → 验证 mTLS 证书
   └─ 否 → 接受明文（仅在 PERMISSIVE 模式）
   ↓
请求继续转发给应用
```

#### 模式 2：STRICT（严格模式，生产环境）

```
Envoy 只接受 mTLS 加密流量
拒绝所有明文请求（返回 PERMISSION_DENIED）
```

**使用场景：**
- 生产环境（强制加密）
- 网格完全迁移后

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: bookinfo
spec:
  mtls:
    mode: STRICT        # 仅接受 mTLS
```

**工作流程：**

```
客户端请求到达（可能是明文或加密）
   ↓
Envoy 检查：是否加密？
   ├─ 是 → 验证 mTLS 证书 ✓
   └─ 否 → 拒绝，返回 PERMISSION_DENIED ✗
```

#### 模式 3：DISABLE（禁用，仅特殊情况）

```
Envoy 不进行 TLS 验证，接受所有明文流量
风险：应该在迁移期或测试环境使用
```

### 配置 mTLS 的分阶段策略

**第 1 周：诊断现状**

```yaml
# 全网络 PERMISSIVE（宽松）
# 目的：让 mTLS 正常运行，但不拒绝明文流量
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: istio-system
spec:
  mtls:
    mode: PERMISSIVE
```

执行：
```bash
# 查看日志，确认哪些流量是明文、哪些是加密的
kubectl logs <pod> -c istio-proxy | grep tls_version
```

**第 2-3 周：逐命名空间切换为 STRICT**

```yaml
# bookinfo 命名空间切换为严格模式
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: strict-mtls
  namespace: bookinfo
spec:
  mtls:
    mode: STRICT
```

执行：
```bash
# 监控 bookinfo 命名空间的错误
kubectl logs -n bookinfo <pod> -c istio-proxy | grep PERMISSION_DENIED

# 若有错误，回退到 PERMISSIVE 并调查
```

**第 4 周：全网 STRICT**

```yaml
# 全网络切换为严格模式
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: istio-system
spec:
  mtls:
    mode: STRICT
```

### mTLS 对应用的影响

**好消息：** 应用**完全无需改动**

```
应用代码：
  curl http://localhost:8000/api

Envoy Sidecar 处理：
  1. 拦截请求
  2. 建立 mTLS 连接到目标 Pod 的 Sidecar
  3. 发送请求
  4. 接收响应
  5. 返回给应用

应用看到的：
  HTTP 明文响应（Sidecar 已解密）
```

**所有的 TLS 管理都由 Istio 在后台自动完成。**

---

## 3. 请求认证（Request Authentication）：验证身份

### 认证 vs 授权的区别

```
认证（Authentication）：验证"你是谁"
  ├─ 提交证明（如身份证、JWT Token）
  ├─ 系统验证证明的有效性
  └─ 确定请求者的身份

授权（Authorization）：验证"你能做什么"
  ├─ 知道了你是谁（认证完成）
  ├─ 检查你是否被允许进行此操作
  └─ 例如：只有 admin 用户才能删除数据库
```

**实际场景：**

```
用户 Alice 想删除某条记录

1. 认证（RequestAuthentication）：
   Alice 提交 JWT Token
   Istio 验证 Token 签名和过期时间
   确认请求来自真实的 Alice

2. 授权（AuthorizationPolicy）：
   检查：Alice 是否在 "admin" 组？
   若是 → 允许删除
   若否 → 拒绝删除（403 Forbidden）
```

### JWT Token 认证

JWT（JSON Web Token）是一个标准的身份证明格式。

**JWT 的结构：**

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJhbGljZSIsImV4cCI6MTcwMzI3NTIwMH0.xxx

三部分（用 . 分隔）：
  1. Header（算法信息）
  2. Payload（用户信息）
  3. Signature（签名，防篡改）
```

**Payload 示例：**

```json
{
  "sub": "alice",                    # 用户 ID
  "email": "alice@example.com",     # 邮箱
  "iss": "https://auth.example.com", # 签发者
  "exp": 1703275200,                # 过期时间
  "groups": ["admin", "developers"]  # 用户组
}
```

### 配置 RequestAuthentication

```yaml
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: jwt-auth
  namespace: bookinfo
spec:
  jwtRules:
  - issuer: "https://auth.example.com"      # 谁签发的 Token？
    jwksUri: "https://auth.example.com/certs" # 验证签名的公钥在哪里？
    audiences:
    - "bookinfo-api"                         # 这个 Token 给谁用？
```

**工作流程：**

```
客户端发送请求
  Header: Authorization: Bearer <JWT Token>
         ↓
Envoy 拦截请求
         ↓
检查 Header 中是否有 Bearer Token
  ├─ 有 → 提取 Token，验证签名和过期时间
  │       若验证失败 → 返回 401 Unauthorized
  │       若验证成功 → 从 Token 中提取用户信息（sub, groups 等）
  │
  └─ 无 → 继续转发（RequestAuthentication 本身不拒绝）

添加 Header 传递用户信息给应用
  X-Auth-Principal: alice
  X-Auth-Groups: admin,developers
         ↓
应用收到请求（现在知道了请求者的身份）
```

**关键点：** RequestAuthentication 仅**验证**Token，不**拒绝**请求。

即使没有 Token 或 Token 无效，请求仍会被转发给应用。

拒绝无效请求需要配合 **AuthorizationPolicy**。

---

## 4. 授权策略（AuthorizationPolicy）：访问控制

### 默认拒绝（Deny-by-Default）原则

Istio 授权遵循**最小权限原则**：

```
默认状态：所有请求都被拒绝
         （即使没有明确的 AuthorizationPolicy）

需要明确配置允许规则才能通过
```

**配置流程：**

```
1. 部署应用，不配置任何 AuthorizationPolicy
   ↓
   所有入站请求被拒绝（403 Forbidden）

2. 创建 AuthorizationPolicy，允许特定源
   ↓
   只有符合条件的请求才被允许

3. 细化规则，针对不同端点配置不同权限
```

### AuthorizationPolicy 的三种规则类型

#### 规则类型 1：ALLOW（允许）

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-reviews
  namespace: bookinfo
spec:
  rules:
  # 规则 1：允许来自 productpage 的请求
  - from:
    - source:
        principals:
        - "cluster.local/ns/bookinfo/sa/productpage"  # productpage 的 Service Account
    to:
    - operation:
        methods: ["GET"]                              # 仅允许 GET
  
  # 规则 2：允许来自任何 Pod 的 GET 请求
  - from:
    - source:
        namespaces:
        - "bookinfo"
    to:
    - operation:
        methods: ["GET"]
        paths:
        - "/reviews"
```

**工作流程：**

```
请求进入 reviews Pod
   ↓
检查是否有匹配的 ALLOW 规则
   ├─ 找到匹配的规则 → 请求被允许 ✓
   └─ 没找到匹配的规则 → 请求被拒绝 ✗（403）
```

#### 规则类型 2：DENY（拒绝）

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: deny-suspicious
  namespace: bookinfo
spec:
  rules:
  - from:
    - source:
        namespaces:
        - "untrusted"         # 拒绝来自不信任命名空间的请求
    to:
    - operation:
        methods: ["DELETE"]   # 拒绝 DELETE 操作
```

**与 ALLOW 的关系：**

```
DENY 规则用于"黑名单"（拒绝特定的请求）
ALLOW 规则用于"白名单"（仅允许特定的请求）

通常在生产环境使用 ALLOW（更安全），不用 DENY
```

#### 规则类型 3：CUSTOM（自定义）

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: custom-policy
spec:
  rules:
  - from:
    - source:
        principals:
        - "cluster.local/ns/default/sa/alice"
    to:
    - operation:
        methods: ["*"]
    when:
    - key: request.auth.claims[groups]
      values:
      - "admin"               # 仅当 JWT 中的 groups 包含 "admin"
```

### 实际场景：微服务访问控制

```yaml
# ========== reviews 服务授权策略 ==========
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: reviews-policy
  namespace: bookinfo
spec:
  selector:
    matchLabels:
      app: reviews            # 应用到 reviews Pod
  
  rules:
  # 规则 1：允许来自 productpage 的 GET 请求
  - from:
    - source:
        principals:
        - "cluster.local/ns/bookinfo/sa/productpage"
    to:
    - operation:
        methods: ["GET"]
        paths:
        - "/reviews/*"
  
  # 规则 2：允许来自同命名空间的任何 Pod 的 GET 请求
  - from:
    - source:
        namespaces:
        - "bookinfo"
    to:
    - operation:
        methods: ["GET"]
  
  # 规则 3：允许使用有效 JWT 的外部请求（仅特定路径）
  - from:
    - source:
        requestPrincipals:
        - "https://auth.example.com/users/*"  # JWT issuer
    to:
    - operation:
        methods: ["GET"]
        paths:
        - "/reviews/public"

---
# ========== 同时配置认证 ==========
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: jwt-auth
  namespace: bookinfo
spec:
  jwtRules:
  - issuer: "https://auth.example.com"
    jwksUri: "https://auth.example.com/.well-known/jwks.json"
```

---

## 5. 跨命名空间的安全通信

### 场景：应用跨越命名空间

```
生产环境中常见的架构：
  ├─ 命名空间 A：前端服务（nginx）
  ├─ 命名空间 B：业务逻辑（productpage）
  └─ 命名空间 C：数据服务（database）
```

**默认情况下，跨命名空间通信受限：**

```yaml
# 这个策略仅应用到 bookinfo 命名空间的 Pod
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-from-frontend
  namespace: bookinfo                    # 仅在此命名空间生效
spec:
  rules:
  - from:
    - source:
        principals:
        - "cluster.local/ns/frontend/sa/nginx"  # 来自 frontend 命名空间
```

### 网格级别的全局策略

```yaml
# 在 istio-system 命名空间定义，应用到整个网格
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: global-policy
  namespace: istio-system
spec:
  # 未指定 selector，表示应用到所有命名空间
  rules:
  - from:
    - source:
        namespaces:
        - "istio-system"    # 允许 istio 自身的流量
  - from:
    - source:
        namespaces:
        - "*"               # 允许所有命名空间
    to:
    - operation:
        methods: ["GET"]    # 但仅允许 GET
```

---

## 6. 常见的安全配置陷阱

### 陷阱 1：PeerAuthentication 和 AuthorizationPolicy 不配套

```yaml
# ❌ 错误：启用了 STRICT mTLS，但没有 AuthorizationPolicy
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
spec:
  mtls:
    mode: STRICT
# 结果：所有网格内流量都被拒绝！（因为没有 ALLOW 规则）

# ✓ 正确：mTLS + AuthorizationPolicy 配套
# 1. 启用 mTLS
# 2. 定义明确的 ALLOW 规则
```

### 陷阱 2：授权策略过宽松

```yaml
# ❌ 错误：允许所有请求
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-all
spec:
  rules:
  - from:
    - source:
        principals: ["*"]  # 任何人都可以
    to:
    - operation:
        methods: ["*"]     # 执行任何操作

# ✓ 正确：最小权限原则
# 仅允许具体的源访问具体的资源
```

### 陷阱 3：JWT 验证不严格

```yaml
# ❌ 错误：RequestAuthentication 存在但无 AuthorizationPolicy
# 结果：Token 被验证但无效 Token 不会被拒绝
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: jwt-auth
spec:
  jwtRules:
  - issuer: "https://auth.example.com"
    jwksUri: "https://auth.example.com/.well-known/jwks.json"

# ✓ 正确：配合 AuthorizationPolicy 拒绝无 Token 的请求
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: require-token
spec:
  rules:
  - from:
    - source:
        requestPrincipals:
        - "https://auth.example.com/*"  # 拒绝没有有效 Token 的请求
```

### 陷阱 4：遗忘配置 Service Account

```yaml
# ❌ 错误：授权策略引用了不存在的 Service Account
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: policy
spec:
  rules:
  - from:
    - source:
        principals:
        - "cluster.local/ns/bookinfo/sa/unknown-sa"  # 此 SA 不存在

# ✓ 正确：验证 Service Account 存在
kubectl get sa -n bookinfo
# 必须看到对应的 SA
```

---

## 7. 诊断安全问题

### 问题 1：403 Forbidden - 请求被拒绝

**诊断步骤：**

```bash
# 步骤 1：检查是否有 AuthorizationPolicy
kubectl get authorizationpolicies -n bookinfo
kubectl describe ap <policy-name> -n bookinfo

# 步骤 2：检查 mTLS 模式
kubectl get peerauthentication -n bookinfo
kubectl describe pa <pa-name> -n bookinfo

# 步骤 3：查看 Envoy 日志（关键！）
kubectl logs -n bookinfo <pod-name> -c istio-proxy | grep "RBAC"

# 输出可能包含：
# "RBAC: access denied"
# 这表示授权策略拒绝了请求

# 步骤 4：检查请求的来源
# 在 AuthorizationPolicy 中查看 principals 配置
# 确认请求来自预期的 Service Account

# 步骤 5：检查 Token 有效性（若使用 JWT）
kubectl logs -n bookinfo <pod-name> -c istio-proxy | grep "JWT"

# 输出可能包含：
# "JWT validation failed"
```

**常见原因及解决方案：**

| 问题 | 原因 | 解决 |
|------|------|------|
| 403 无故出现 | AuthorizationPolicy 规则不匹配 | 检查 principals 和 namespaces |
| 跨命名空间请求失败 | 来源的 Service Account 不被信任 | 添加对应的 principal 规则 |
| JWT 验证失败 | Token 过期或签名不对 | 检查 JWT issuer 和 jwksUri |
| STRICT mTLS 后全部 403 | 缺少 AuthorizationPolicy 规则 | 添加 ALLOW 规则 |

### 问题 2：mTLS 握手失败

```bash
# 症状：连接重置、连接超时

# 检查 mTLS 配置
istioctl get peerauthentication -n bookinfo
istioctl describe peerauthentication <name> -n bookinfo

# 检查证书是否存在
kubectl exec -it -n bookinfo <pod-name> -c istio-proxy -- \
  ls -la /etc/certs/workload_cert.pem

# 查看 Envoy 日志
kubectl logs -n bookinfo <pod-name> -c istio-proxy | grep "TLS"

# 若看到 "TLS alert"，可能原因：
# 1. 客户端和服务器 mTLS 模式不匹配（一个 STRICT，一个 PERMISSIVE）
# 2. 证书验证失败（CA 证书不匹配）
```

---

## 8. 本章小结

### 安全的三层防护

```
层 1：网络加密（mTLS）
  ├─ PeerAuthentication 配置
  └─ 作用：所有流量加密，防止窃听

层 2：身份认证（Authentication）
  ├─ RequestAuthentication 配置（JWT）
  └─ 作用：确认请求来自真实用户

层 3：访问控制（Authorization）
  ├─ AuthorizationPolicy 配置
  └─ 作用：确认用户有权限做此操作
```

### 快速参考：完整的安全配置

```yaml
# ========== 1. 启用 mTLS ==========
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: bookinfo
spec:
  mtls:
    mode: STRICT  # 强制 TLS

---
# ========== 2. 配置 JWT 认证 ==========
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: jwt-auth
  namespace: bookinfo
spec:
  jwtRules:
  - issuer: "https://auth.example.com"
    jwksUri: "https://auth.example.com/.well-known/jwks.json"

---
# ========== 3. 定义授权策略 ==========
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: reviews-policy
  namespace: bookinfo
spec:
  selector:
    matchLabels:
      app: reviews
  rules:
  # 允许来自 productpage 的请求
  - from:
    - source:
        principals:
        - "cluster.local/ns/bookinfo/sa/productpage"
    to:
    - operation:
        methods: ["GET"]
  # 允许带有有效 JWT 的请求
  - from:
    - source:
        requestPrincipals:
        - "https://auth.example.com/*"
```

### 验证安全配置

```bash
# 检查 mTLS 是否启用
kubectl get peerauthentication -n bookinfo

# 检查授权策略
kubectl get authorizationpolicies -n bookinfo

# 检查 JWT 配置
kubectl get requestauthentication -n bookinfo

# 测试访问控制（应该被拒绝）
kubectl exec -it -n bookinfo <pod> -c <app> -- \
  curl -v http://reviews:9080/reviews

# 查看拒绝日志
kubectl logs -n bookinfo <pod> -c istio-proxy | grep RBAC
```

---

## 9. 下一步

第七章将深入讲解：
- **可观测性深化**：全面的监控、告警和日志分析
- **故障恢复与灾备**：多集群部署、容灾策略
- **性能优化**：Istio 的性能调优和资源优化

掌握这些，你就能运维一个大规模、高可用的微服务系统。 🚀
