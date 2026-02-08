

# Lab 1 Istio 环境的安装

下载当前最新版本 Istio

```Bash
curl -L https://istio.io/downloadIstio | sh -
root@node1:~# curl -L https://istio.io/downloadIstio | sh -
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   101  100   101    0     0    109      0 --:--:-- --:--:-- --:--:--   109
100  5124  100  5124    0     0   2064      0  0:00:02  0:00:02 --:--:--  4856

Downloading istio-1.26.0 from https://github.com/istio/istio/releases/download/1.26.0/istio-1.26.0-linux-amd64.tar.gz ...

Istio 1.26.0 download complete!

The Istio release archive has been downloaded to the istio-1.26.0 directory.

To configure the istioctl client tool for your workstation,
add the /root/istio-1.26.0/bin directory to your environment path variable with:
         export PATH="$PATH:/root/istio-1.26.0/bin"

Begin the Istio pre-installation check by running:
         istioctl x precheck

Try Istio in ambient mode
        https://istio.io/latest/docs/ambient/getting-started/
Try Istio in sidecar mode
        https://istio.io/latest/docs/setup/getting-started/
Install guides for ambient mode
        https://istio.io/latest/docs/ambient/install/
Install guides for sidecar mode
        https://istio.io/latest/docs/setup/install/

Need more information? Visit https://istio.io/latest/docs/
```

根据上述步骤的输出设置环境变量

```Bash
export PATH="$PATH:/root/istio-1.26.0/bin"
```

进入目录

```Bash
cd istio-1.26.0/
```

检查安装前提条件

```Bash
istioctl x precheck 
controlplane $ istioctl x precheck 
✔ No issues found when checking the cluster. Istio is safe to install or upgrade!
  To get started, check out https://istio.io/latest/docs/setup/getting-started/
```

执行安装

```Bash
istioctl manifest apply --set profile=demo
```

> `demo` 配置文件是一个较为简单的配置，适合学习和试验 Istio 的基本功能。它包括了 Istio 的核心组件，如 Pilot、Envoy 代理、Ingress Gateway 等，但没有启用一些高级功能，如遥测数据收集、访问日志等。

```Bash
root@node1:~/istio-1.26.0# istioctl manifest apply --set profile=demo
        |\
        | \
        |  \
        |   \
      /||    \
     / ||     \
    /  ||      \
   /   ||       \
  /    ||        \
 /     ||         \
/______||__________\
____________________
  \__       _____/
     \_____/

This will install the Istio 1.26.0 profile "demo" into the cluster. Proceed? (y/N) y
✔ Istio core installed ⛵️
✔ Istiod installed 🧠
✔ Egress gateways installed 🛫
✔ Ingress gateways installed 🛬
✔ Installation complete
root@node1:~/istio-1.26.0#
```

（可选）安装仪表板：

```Bash
kubectl apply -f samples/addons 
```

> 这个命令会将 `samples/addons` 目录下的所有资源清单文件应用到您的 Kubernetes 集群中。这些资源清单文件定义了一些附加组件，如：
>
> - `kiali.yaml`： Kiali 是一个用于可视化 Istio 服务网格的 Web UI。
> - `jaeger.yaml`： Jaeger 是一个分布式跟踪系统，用于监控和调试 Istio 服务网格中的请求。
> - `prometheus.yaml`： Prometheus 是一个监控和警报系统，用于收集和查询 Istio 服务网格的指标数据。
> - `grafana.yaml`： Grafana 是一个数据可视化和监控工具，与 Prometheus 集成，用于查看 Istio 服务网格的指标数据。
>
> 通过应用这些附加组件，您可以获得更多的可观察性和监控功能，以便更好地了解和管理 Istio 服务网格。但是，请注意，这些附加组件通常需要更多的资源，因此在生产环境中使用时需要进行适当的规划和配置。

```Bash
controlplane $ kubectl apply -f samples/addons
serviceaccount/grafana created
configmap/grafana created
service/grafana created
deployment.apps/grafana created
configmap/istio-grafana-dashboards created
configmap/istio-services-grafana-dashboards created
deployment.apps/jaeger created
service/tracing created
service/zipkin created
service/jaeger-collector created
serviceaccount/kiali created
configmap/kiali created
clusterrole.rbac.authorization.k8s.io/kiali-viewer created
clusterrole.rbac.authorization.k8s.io/kiali created
clusterrolebinding.rbac.authorization.k8s.io/kiali created
role.rbac.authorization.k8s.io/kiali-controlplane created
rolebinding.rbac.authorization.k8s.io/kiali-controlplane created
service/kiali created
deployment.apps/kiali created
serviceaccount/prometheus created
configmap/prometheus created
clusterrole.rbac.authorization.k8s.io/prometheus created
clusterrolebinding.rbac.authorization.k8s.io/prometheus created
service/prometheus created
deployment.apps/prometheus created
```

检查 istio 安装版本：

```Bash
istioctl version
controlplane $ istioctl version
client version: 1.26.0
control plane version: 1.26.0
data plane version: 1.26.0 (2 proxies)
```

> 这个命令输出显示了 Istio 服务网格的版本信息。Istio 是一个开源的服务网格，用于连接、监控和保护微服务。
>
> 输出中包含以下几个部分：
>
> 1. `client version: 1.26.0` 这是 `istioctl` 客户端工具的版本，用于与 Istio 控制平面进行交互。
> 2. `control plane version: 1.26.0` 这是 Istio 控制平面的版本，包括 Pilot、Galley、Citadel 等控制平面组件。
> 3. `data plane version: 1.26.0 (2 proxies)` 这是 Istio 数据平面的版本，即 Envoy 代理的版本。括号中的 `(2 proxies)` 表示当前集群中有 2 个 Envoy 代理实例。
>
> 通过运行 `istioctl version` 命令，您可以查看当前 Istio 安装的版本信息，包括客户端、控制平面和数据平面的版本。这对于诊断问题、规划升级等场景非常有用。如果版本号不一致，可能会导致兼容性问题。

查看 crd：

```Bash
kubectl get crd | grep istio
root@node1:~/istio-1.26.0# kubectl get crd | grep istio
authorizationpolicies.security.istio.io               2025-05-09T00:44:20Z
destinationrules.networking.istio.io                  2025-05-09T00:44:19Z
envoyfilters.networking.istio.io                      2025-05-09T00:44:19Z
gateways.networking.istio.io                          2025-05-09T00:44:19Z
peerauthentications.security.istio.io                 2025-05-09T00:44:20Z
proxyconfigs.networking.istio.io                      2025-05-09T00:44:19Z
requestauthentications.security.istio.io              2025-05-09T00:44:20Z
serviceentries.networking.istio.io                    2025-05-09T00:44:19Z
sidecars.networking.istio.io                          2025-05-09T00:44:19Z
telemetries.telemetry.istio.io                        2025-05-09T00:44:20Z
virtualservices.networking.istio.io                   2025-05-09T00:44:19Z
wasmplugins.extensions.istio.io                       2025-05-09T00:44:19Z
workloadentries.networking.istio.io                   2025-05-09T00:44:20Z
workloadgroups.networking.istio.io                    2025-05-09T00:44:20Z
```

> 这个命令输出显示了当前 Kubernetes 集群中安装的 Istio 相关的自定义资源定义 （CRD， Custom Resource Definitions）。
>
> CRD 是 Kubernetes 的一个扩展机制，允许用户定义自己的自定义资源类型，并像使用内置资源一样管理它们。Istio 利用了这个机制来定义自己的资源类型，用于配置和管理服务网格。
>
> 输出中列出了 Istio 安装的所有 CRD，包括：
>
> - `authorizationpolicies.security.istio.io`： Istio 授权策略
> - `destinationrules.networking.istio.io`： Istio 目标规则
> - `envoyfilters.networking.istio.io`： Istio Envoy 过滤器
> - `gateways.networking.istio.io`： Istio 网关
> - `peerauthentications.security.istio.io`： Istio 对等认证
> - `proxyconfigs.networking.istio.io`： Istio 代理配置
> - `requestauthentications.security.istio.io`： Istio 请求认证
> - `serviceentries.networking.istio.io`： Istio 服务条目
> - `sidecars.networking.istio.io`: Istio sidecar
> - `telemetries.telemetry.istio.io`： Istio 遥测
> - `virtualservices.networking.istio.io`： Istio 虚拟服务
> - `wasmplugins.extensions.istio.io`： Istio WebAssembly 插件
> - `workloadentries.networking.istio.io`： Istio 工作负载条目
> - `workloadgroups.networking.istio.io`： Istio 工作负载组
>
> 这些 CRD 允许您使用 Kubernetes 原生的方式来配置和管理 Istio 服务网格。例如，您可以创建一个 `VirtualService` 资源来配置流量路由，或者创建一个 `DestinationRule` 资源来配置负载均衡策略。
>
> 通过列出已安装的 CRD，您可以了解 Istio 提供了哪些配置功能，并根据需要进一步学习和使用这些资源。

查看 api 资源：

```Bash
kubectl api-resources | grep istio
root@node1:~# kubectl api-resources | grep istio
wasmplugins                                    extensions.istio.io/v1alpha1           true         WasmPlugin
destinationrules                  dr           networking.istio.io/v1beta1            true         DestinationRule
envoyfilters                                   networking.istio.io/v1alpha3           true         EnvoyFilter
gateways                          gw           networking.istio.io/v1beta1            true         Gateway
proxyconfigs                                   networking.istio.io/v1beta1            true         ProxyConfig
serviceentries                    se           networking.istio.io/v1beta1            true         ServiceEntry
sidecars                                       networking.istio.io/v1beta1            true         Sidecar
virtualservices                   vs           networking.istio.io/v1beta1            true         VirtualService
workloadentries                   we           networking.istio.io/v1beta1            true         WorkloadEntry
workloadgroups                    wg           networking.istio.io/v1beta1            true         WorkloadGroup
authorizationpolicies                          security.istio.io/v1                   true         AuthorizationPolicy
peerauthentications               pa           security.istio.io/v1beta1              true         PeerAuthentication
requestauthentications            ra           security.istio.io/v1                   true         RequestAuthentication
telemetries                       telemetry    telemetry.istio.io/v1alpha1            true         Telemetry
```

> 这个命令输出列出了当前 Kubernetes 集群中安装的所有与 Istio 相关的 API 资源。它提供了更详细的信息，包括资源名称、缩写、API 组和版本等。
>
> 输出中包含以下 Istio 相关的 API 资源：
>
> - `wasmplugins` （API 组： `extensions.istio.io/v1alpha1`）
> - `destinationrules` （API 组： `networking.istio.io/v1beta1`）
> - `envoyfilters` （API 组： `networking.istio.io/v1alpha3`）
> - `gateways` （API 组： `networking.istio.io/v1beta1`）
> - `proxyconfigs` （API 组： `networking.istio.io/v1beta1`）
> - `serviceentries` （API 组： `networking.istio.io/v1beta1`）
> - `sidecars` （API 组： `networking.istio.io/v1beta1`）
> - `virtualservices` （API 组： `networking.istio.io/v1beta1`）
> - `workloadentries` （API 组： `networking.istio.io/v1beta1`）
> - `workloadgroups` （API 组： `networking.istio.io/v1beta1`）
> - `authorizationpolicies` （API 组： `security.istio.io/v1`）
> - `peerauthentications` （API 组： `security.istio.io/v1beta1`）
> - `requestauthentications` （API 组： `security.istio.io/v1`）
> - `telemetries` （API 组： `telemetry.istio.io/v1alpha1`）
>
> 这些 API 资源对应于 Istio 服务网格中的各种配置资源，例如虚拟服务、目标规则、网关、授权策略等。您可以使用 Kubernetes 原生的方式（如 `kubectl` 命令）来创建、更新和删除这些资源，从而配置和管理 Istio 服务网格。
>
> 此外，输出还提供了每个资源的缩写（如 `vs` 代表 `virtualservices`）和 API 版本信息，这对于编写 YAML 配置文件或使用 Kubernetes API 非常有用。

查看命名空间：

```Bash
kubectl get namespaces 
```

查看 istio 相关 pod：

```Bash
kubectl get pods -n istio-system
controlplane $ kubectl get pods -n istio-system
NAME                                    READY   STATUS    RESTARTS   AGE
grafana-56bdf8bf85-xm6gn                1/1     Running   0          5m2s
istio-egressgateway-5bdd756dfd-tqmz9    1/1     Running   0          6m31s
istio-ingressgateway-67f7b5f88d-j4v8z   1/1     Running   0          6m31s
istiod-58c6454c57-vq7l4                 1/1     Running   0          6m47s
jaeger-c4fdf6674-ltgkn                  1/1     Running   0          5m2s
kiali-5ff49b9f69-sf42z                  1/1     Running   0          5m2s
prometheus-85949fddb-z57vs              2/2     Running   0          5m1s
```

> 这些 Pod 都是 Istio 服务网格的核心组件，让我们逐一解释它们的作用：
>
> - `grafana`： Grafana 是一个开源的数据可视化和监控工具，用于展示 Istio 的指标和日志数据。
> - `istio-egressgateway`： Egress Gateway 用于监控和控制离开服务网格的流量。
> - `istio-ingressgateway`： Ingress Gateway 用于接收进入服务网格的流量。
> - `istiod`： Istiod 是 Istio 控制平面的核心组件，提供服务发现、配置和证书管理等功能。
> - `jaeger`： Jaeger 是一个开源的分布式跟踪系统，用于监控和排查 Istio 服务网格中的请求。
> - `kiali`： Kiali 是 Istio 的可观测性控制台，提供服务网格配置和监控功能。
> - `prometheus`： Prometheus 是一个开源的监控和警报系统，用于从 Istio 收集指标数据。
>
> 这些组件共同构成了 Istio 服务网格的控制平面和数据平面。控制平面包括 istiod、Prometheus 和其他配置组件，负责管理和配置服务网格。数据平面包括 Ingress Gateway、Egress Gateway 和 Sidecar 代理，负责处理实际的网络流量。
>
> 监控和可视化组件如 Grafana、Jaeger 和 Kiali 则提供了对服务网格的可观测性，帮助诊断和了解整个系统的运行状况。

查看 istio 服务状态：

```Bash
kubectl get svc -n istio-system 
controlplane $ kubectl get svc -n istio-system 
NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                                      AGE
grafana                ClusterIP      10.104.87.225    <none>        3000/TCP                                                                     5m43s
istio-egressgateway    ClusterIP      10.103.43.184    <none>        80/TCP,443/TCP                                                               7m11s
istio-ingressgateway   LoadBalancer   10.105.52.139    <pending>     15021:30275/TCP,80:31216/TCP,443:31018/TCP,31400:30363/TCP,15443:32013/TCP   7m11s
istiod                 ClusterIP      10.110.235.100   <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP                                        7m28s
jaeger-collector       ClusterIP      10.109.101.148   <none>        14268/TCP,14250/TCP,9411/TCP                                                 5m42s
kiali                  ClusterIP      10.101.69.18     <none>        20001/TCP,9090/TCP                                                           5m42s
prometheus             ClusterIP      10.100.129.105   <none>        9090/TCP                                                                     5m42s
tracing                ClusterIP      10.96.122.246    <none>        80/TCP,16685/TCP                                                             5m42s
zipkin                 ClusterIP      10.106.66.61     <none>        9411/TCP                                                                     5m42s
```

> 这些服务都是 Istio 服务网格的核心组件，让我们逐一解释它们的作用：
>
> - `grafana`： Grafana 服务提供了 Istio 的监控和可视化界面，用于查看指标和日志数据。
> - `istio-egressgateway`： Egress Gateway 是一个专门用于处理离开服务网格流量的网关。它允许 Istio 集群内的服务访问集群外的服务，同时提供了安全性、监控和控制功能。Egress Gateway 充当了集群内服务与外部服务之间的统一入口。
> - `istio-ingressgateway`： Ingress Gateway 是一个专门用于处理进入服务网格流量的网关。它是进入 Istio 服务网格的入口点，负责捕获所有进入集群的流量，并将其路由到合适的服务。Ingress Gateway 还提供了丰富的流量管理功能，如负载均衡、访问控制、TLS 终止等。
> - `istiod`： Istiod 是 Istio 控制平面的核心组件，提供服务发现、配置分发和证书管理等功能。它维护了服务网格的整体状态和配置信息。
> - `jaeger-collector`： Jaeger 是一个开源的分布式跟踪系统，用于监控和排查 Istio 服务网格中的请求。`jaeger-collector` 服务收集和处理跟踪数据。
> - `kiali`： Kiali 是 Istio 的可观测性控制台，提供服务网格配置和监控功能。
> - `prometheus`： Prometheus 是一个开源的监控和警报系统，用于从 Istio 收集指标数据。
> - `tracing`、`zipkin`： 这两个服务也与分布式跟踪相关，提供了跟踪数据的收集和查询功能。
>
> 重点介绍 `istio-egressgateway` 和 `istio-ingressgateway`：
>
> **Ingress Gateway**:
>
> - 作为服务网格的入口网关，接收进入服务网格的所有外部流量
> - 所有进入服务网格的流量都需要先经过 Ingress Gateway
> - 可以在 Ingress Gateway 上配置流量路由规则，实现高级流量管理
> - 支持 HTTP、HTTPS、TCP 等多种协议
> - 通过 LoadBalancer 类型暴露服务，可通过外部 IP 访问
>
> **Egress Gateway**:
>
> - 作为服务网格的出口网关，管控离开服务网格的所有流量
> - 服务网格内部服务访问外部服务时，流量需要经过 Egress Gateway
> - 可以在 Egress Gateway 上配置出口流量策略，实现安全控制
> - 支持定义出口主机列表，只允许访问列表内的外部服务
> - 通过 ClusterIP 类型暴露服务，只能在集群内部访问
>
> Ingress Gateway 和 Egress Gateway 共同构成了 Istio 服务网格的流量入口和出口，是实现流量管理和安全控制的关键组件。通过在网关层配置规则，可以实现高级的流量路由、安全策略等功能，从而加强对服务网格的控制和管理。

查看各组件状态：

```Bash
kubectl get svc,pod,hpa,pdb,Gateway,VirtualService -n istio-system
```

加载实验脚本目录

```Bash
git clone https://github.com/cloudzun/istiolabmanual
root@node1:~/istio-1.26.0# git clone https://github.com/cloudzun/istiolabmanual
Cloning into 'istiolabmanual'...
remote: Enumerating objects: 270, done.
remote: Counting objects: 100% (112/112), done.
remote: Compressing objects: 100% (96/96), done.
remote: Total 270 (delta 51), reused 67 (delta 16), pack-reused 158
Receiving objects: 100% (270/270), 3.39 MiB | 3.42 MiB/s, done.
Resolving deltas: 100% (137/137), done.
root@node1:~/istio-1.26.0# dir
bin  istiolabmanual  LICENSE  manifests  manifest.yaml  README.md  samples  tools
```

# Lab 2 Bookinfo 示例程序安装

Bookinfo 是 Istio 社区官方推荐的示例应用之一。它可以用来演示多种 Istio 的特性，并且它是一个异构的微服务应用。这个应用模仿在线书店的一个分类，显示一本书的信息。 页面上会显示一本书的描述，书籍的细节（ISBN、页数等），以及关于这本书的一些评论。

Bookinfo 应用分为四个单独的微服务：

- `productpage`：这个微服务会调用 `details` 和 `reviews` 两个微服务，用来生成页面。
- `details`：这个微服务中包含了书籍的信息。
- `reviews`：这个微服务中包含了书籍相关的评论。它还会调用 `ratings` 微服务。
- `ratings`：这个微服务中包含了由书籍评价组成的评级信息。

`reviews` 微服务有 3 个版本：

- v1 版本不会调用 `ratings` 服务。
- v2 版本会调用 `ratings` 服务，并使用 1 到 5 个黑色星形图标来显示评分信息。
- v3 版本会调用 `ratings` 服务，并使用 1 到 5 个红色星形图标来显示评分信息。

下图展示了这个应用的端到端架构。

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=Njc0Y2QyNThhMDViNGNhY2UwYjM1NWVhOGVkMmIzNWZfYTZtVnB6UlBlMkdPczJOS1Jwa1p4UjMwTFo4d3RkbTRfVG9rZW46V0tpWmJYVVNUb3hrMEp4SU1HSGNxSEJlbnFjXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

启动自动注入 sidecar

```Bash
kubectl label namespace default istio-injection=enabled
```

> 这条命令用于为 Kubernetes 的 `default` 命名空间打上一个标签 `istio-injection=enabled`。这个标签是 Istio 自动注入 Sidecar 代理的关键。
>
> **Istio 自动注入 Sidecar**
>
> Istio 通过自动注入的方式，在应用程序 Pod 中注入一个 Envoy 代理作为 Sidecar 容器运行。Envoy 代理拦截并处理 Pod 中应用容器的入口和出口流量，从而使应用程序无需做任何更改就能接入 Istio 服务网格。
>
> **istio-injection 标签**
>
> - `istio-injection=enabled`：标记该命名空间下新创建的 Pod 将自动注入 Istio Sidecar
> - `istio-injection=disabled`：标记该命名空间下新创建的 Pod 将不会注入 Sidecar
> - 未设置标签：默认情况下不会注入 Sidecar
>
> 注意，该标签只对新创建的 Pod 生效。对于已存在的 Pod，需要删除并重新创建才会生效。
>
> **为什么需要自动注入？**
>
> 自动注入 Sidecar 的目的是让应用程序无需任何代码修改，就能无缝接入 Istio 服务网格，享受 Istio 提供的流量管理、安全控制、策略执行等功能。这极大简化了应用程序的集成过程。
>
> 需要注意的是，并非所有命名空间都需要启用自动注入。对于某些系统组件或不需要接入 Istio 的应用，可以选择不启用该标签。通常建议为包含应用服务的命名空间启用自动注入。

安装 bookinfo

```Bash
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
controlplane $ kubectl label namespace default istio-injection=enabled
namespace/default labeled
controlplane $ kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
service/details created
serviceaccount/bookinfo-details created
deployment.apps/details-v1 created
service/ratings created
serviceaccount/bookinfo-ratings created
deployment.apps/ratings-v1 created
service/reviews created
serviceaccount/bookinfo-reviews created
deployment.apps/reviews-v1 created
deployment.apps/reviews-v2 created
deployment.apps/reviews-v3 created
service/productpage created
serviceaccount/bookinfo-productpage created
deployment.apps/productpage-v1 created
```

确认服务和 pod 状态

```Bash
kubectl get svc

kubectl get pod
```

此处需要等待大概 2 分钟，等到所有的 pod 都 ready 再进行下一步

```Bash
controlplane $ kubectl get svc
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
details       ClusterIP   10.103.216.219   <none>        9080/TCP   40s
kubernetes    ClusterIP   10.96.0.1        <none>        443/TCP    17d
productpage   ClusterIP   10.96.27.197     <none>        9080/TCP   39s
ratings       ClusterIP   10.96.54.189     <none>        9080/TCP   39s
reviews       ClusterIP   10.96.4.49       <none>        9080/TCP   39s
controlplane $ 
controlplane $ kubectl get pod
NAME                             READY   STATUS            RESTARTS   AGE
details-v1-5ffd6b64f7-s7src      2/2     Running           0          41s
productpage-v1-979d4d9fc-l97ck   0/2     PodInitializing   0          40s
ratings-v1-5f9699cfdf-465tm      2/2     Running           0          41s
reviews-v1-569db879f5-44qtz      0/2     PodInitializing   0          41s
reviews-v2-65c4dc6fdc-xpb2d      1/2     Running           0          41s
reviews-v3-c9c4fb987-xwdmj       0/2     PodInitializing   0          41s
```

检查 sidecar 自动注入

```Bash
kubectl describe pod productpage-v1-979d4d9fc-l97ck
```

*重点关注 Container 部分和 Events 部分

```Bash
……
Containers:
  productpage:
    Container ID:   containerd://8a0f4d1e955d4b0c5349d5bc56467d6bb135bf06031dc397d483dda64fcd9e89
    Image:          docker.io/istio/examples-bookinfo-productpage-v1:1.17.0
    Image ID:       docker.io/istio/examples-bookinfo-productpage-v1@sha256:6668bcf42ef0afb89d0ccd378905c761eab0f06919e74e178852b58b4bbb29c5
    Port:           9080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 01 Dec 2022 03:42:06 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /tmp from tmp (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-n7v78 (ro)
  istio-proxy:
    Container ID:  containerd://01ec70eef67d93f2b67b0c274edf98c6c14718a61fe8e97137f792ec3023105b
    Image:         docker.io/istio/proxyv2:1.26.0
    Image ID:      docker.io/istio/proxyv2@sha256:f6f97fa4fb77a3cbe1e3eca0fa46bd462ad6b284c129cf57bf91575c4fb50cf9
    Port:          15090/TCP
    Host Port:     0/TCP
    Args:
      proxy
      sidecar
      --domain
      $(POD_NAMESPACE).svc.cluster.local
      --proxyLogLevel=warning
      --proxyComponentLogLevel=misc:error
      --log_output_level=default:info
      --concurrency
      2
    State:          Running
      Started:      Thu, 01 Dec 2022 03:42:06 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     2
      memory:  1Gi
    Requests:
      cpu:      10m
      memory:   40Mi
    Readiness:  http-get http://:15021/healthz/ready delay=1s timeout=3s period=2s #success=1 #failure=30
    Environment:
      JWT_POLICY:                    third-party-jwt
      PILOT_CERT_PROVIDER:           istiod
      CA_ADDR:                       istiod.istio-system.svc:15012
      POD_NAME:                      productpage-v1-979d4d9fc-l97ck (v1:metadata.name)
      POD_NAMESPACE:                 default (v1:metadata.namespace)
      INSTANCE_IP:                    (v1:status.podIP)
      SERVICE_ACCOUNT:                (v1:spec.serviceAccountName)
      HOST_IP:                        (v1:status.hostIP)
      PROXY_CONFIG:                  {}
                                     
      ISTIO_META_POD_PORTS:          [
                                         {"containerPort":9080,"protocol":"TCP"}
                                     ]
      ISTIO_META_APP_CONTAINERS:     productpage
      ISTIO_META_CLUSTER_ID:         Kubernetes
      ISTIO_META_INTERCEPTION_MODE:  REDIRECT
      ISTIO_META_WORKLOAD_NAME:      productpage-v1
      ISTIO_META_OWNER:              kubernetes://apis/apps/v1/namespaces/default/deployments/productpage-v1
      ISTIO_META_MESH_ID:            cluster.local
      TRUST_DOMAIN:                  cluster.local
    Mounts:
      /etc/istio/pod from istio-podinfo (rw)
      /etc/istio/proxy from istio-envoy (rw)
      /var/lib/istio/data from istio-data (rw)
      /var/run/secrets/credential-uds from credential-socket (rw)
      /var/run/secrets/istio from istiod-ca-cert (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-n7v78 (ro)
      /var/run/secrets/tokens from istio-token (rw)
      /var/run/secrets/workload-spiffe-credentials from workload-certs (rw)
      /var/run/secrets/workload-spiffe-uds from workload-socket (rw)
……
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  96s                default-scheduler  Successfully assigned default/productpage-v1-979d4d9fc-l97ck to controlplane
  Normal   Pulling    95s                kubelet            Pulling image "docker.io/istio/proxyv2:1.26.0"
  Normal   Pulled     85s                kubelet            Successfully pulled image "docker.io/istio/proxyv2:1.26.0" in 10.334996634s
  Normal   Created    85s                kubelet            Created container istio-init
  Normal   Started    84s                kubelet            Started container istio-init
  Normal   Pulling    84s                kubelet            Pulling image "docker.io/istio/examples-bookinfo-productpage-v1:1.17.0"
  Normal   Pulled     37s                kubelet            Successfully pulled image "docker.io/istio/examples-bookinfo-productpage-v1:1.17.0" in 46.9444707s
  Normal   Created    37s                kubelet            Created container productpage
  Normal   Started    36s                kubelet            Started container productpage
  Normal   Pulled     36s                kubelet            Container image "docker.io/istio/proxyv2:1.26.0" already present on machine
  Normal   Created    36s                kubelet            Created container istio-proxy
  Normal   Started    36s                kubelet            Started container istio-proxy
  Warning  Unhealthy  33s (x4 over 35s)  kubelet            Readiness probe failed: Get "http://192.168.0.7:15021/healthz/ready": dial tcp 192.168.0.7:15021: connect: connection refused
```

> 这个 Pod 中包含三个容器：
>
> - **istio-init 容器：**这是一个 Init 容器，在 Pod 启动时首先运行。它的主要作用是设置 iptables 规则，将应用容器的入口流量重定向到 Sidecar 代理容器。这样 Sidecar 就能够拦截并处理应用容器的所有入口流量。
> - **productpage 容器：**这是应用程序的主容器，运行着 Bookinfo 示例应用的 Product Page 服务。它对外暴露 9080 端口，提供访问服务的入口。
> - **istio-proxy 容器：**这是 Istio 自动注入的 Envoy 代理容器，作为 Sidecar 与应用容器一起运行。它拦截应用容器的入口和出口流量，并根据 Istio 控制平面的配置执行流量管理策略，包括路由、负载均衡、重试、安全等功能。
>
> **三者关系：**
>
> - istio-init 容器先运行，设置 iptables 规则将入口流量重定向到 Sidecar
> - productpage 容器是应用程序本身，提供服务的入口
> - istio-proxy 容器是 Sidecar 代理，拦截并处理应用容器的所有入口和出口流量
>
> 所有进入 Pod 的流量，都会先被 iptables 规则重定向到 Sidecar 代理。Sidecar 根据 Istio 的配置，对流量执行管理策略后，再将流量转发给应用容器。应用容器的出口流量，也需要先经过 Sidecar 的处理。
>
> 通过这种无侵入的方式，Istio 实现了对应用的透明代理，使应用程序无需修改代码，就能接入服务网格，享受 Istio 提供的流量管理、安全、可观测等功能。

检查 productpage 页面访问

```Bash
kubectl exec -it $(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}') -c ratings -- curl productpage:9080/productpage | grep -o "<title>.*</title>"
controlplane $ kubectl exec -it $(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}') -c ratings -- curl productpage:9080/productpage | grep -o "<title>.*</title>"
<title>Simple Bookstore App</title>
```

启动默认网关

```Bash
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
```

使用默认目标规则

```Bash
kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
```

针对 productpage 启用 nodeport，并确认对外访问路径和端口

```Bash
export INGRESS_HOST=$(kubectl get po -l istio=ingressgateway -n istio-system -o jsonpath='{.items[0].status.hostIP}')
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
echo $GATEWAY_URL
echo http://$GATEWAY_URL/productpage
```

> 这是一组 shell 命令，用于从 Kubernetes 集群中获取 Istio Ingress Gateway 的相关信息，将这些信息分别赋值给环境变量，并输出 Istio Ingress Gateway 的 URL 以及应用程序的产品页面 URL。
>
> - `export INGRESS_HOST=$(kubectl get po -l istio=ingressgateway -n istio-system -o jsonpath='{.items[0].status.hostIP}')`： 这条命令使用`kubectl`查询标签为`istio=ingressgateway`的 pod，位于名称空间`istio-system`中，然后从结果中提取第一个 pod 的`hostIP`。这个 IP 地址被赋值给环境变量`INGRESS_HOST`。
> - `export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')`： 这条命令查询名称空间`istio-system`中名为`istio-ingressgateway`的服务，并从该服务的端口规格中提取名为`http2`的端口的`nodePort`。这个端口号被赋值给环境变量`INGRESS_PORT`。
> - `export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')`： 类似于上一条命令，此命令查询名称空间`istio-system`中名为`istio-ingressgateway`的服务，但这次是从服务的端口规格中提取名为`https`的端口的`nodePort`。这个端口号被赋值给环境变量`SECURE_INGRESS_PORT`。
> - `export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT`： 使用前面获取的`INGRESS_HOST`和`INGRESS_PORT`变量，将它们组合成一个完整的 URL，并将其赋值给环境变量`GATEWAY_URL`。
> - `echo $GATEWAY_URL`： 输出`GATEWAY_URL`的值，这是 Istio Ingress Gateway 的 URL。
> - `echo http://$GATEWAY_URL/productpage`： 在`GATEWAY_URL`基础上添加应用程序的产品页面路径`/productpage`，输出应用程序的产品页面 URL。
>
> 这组命令的目的是收集 Istio Ingress Gateway 的信息，这将帮助你通过 URL 访问应用程序，验证 Istio 配置是否正确。

```Bash
controlplane $ kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
destinationrule.networking.istio.io/productpage created
destinationrule.networking.istio.io/reviews created
destinationrule.networking.istio.io/ratings created
destinationrule.networking.istio.io/details created
controlplane $ export INGRESS_HOST=$(kubectl get po -l istio=ingressgateway -n istio-system -o jsonpath='{.items[0].status.hostIP}')
controlplane $ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
controlplane $ export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
controlplane $ export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
controlplane $ echo $GATEWAY_URL
172.30.2.2:30175
controlplane $ echo http://$GATEWAY_URL/productpage
http://172.30.2.2:30175/productpage
controlplane $ 
```

> - `echo $GATEWAY_URL`
>   - 这条命令输出了 Istio Ingress Gateway 的 IP 地址和端口号。
>   - 在这个例子中， Ingress Gateway 的 IP 是 `172.30.2.2`， 端口是 `30175`。
>   - Ingress Gateway 是一个负载均衡器，为外部流量提供入口，将请求转发到内部服务。
> - `echo http://$GATEWAY_URL/productpage`
>   - 这条命令输出了访问 Bookinfo 示例应用 ProductPage 服务的 URL。
>   - URL 是 `http://172.30.2.2:30175/productpage`
>   - 通过这个 URL，可以在浏览器或使用 curl 等工具访问 Bookinfo 应用的 Product Page 服务。
>
> 这些输出展示了如何通过 Istio Ingress Gateway 公开的端点来访问内部服务。Ingress Gateway 充当反向代理和负载均衡器的角色，将外部流量路由到内部的 Istio 服务网格中。在这个例子中，我们可以使用输出的 URL 来访问 Bookinfo 应用的 Product Page 服务。

# Lab 3 服务路由和流量管理

## 1.动态路由

（可选）启用默认目标规则

```Bash
kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
```

查看目标规则

```Bash
nano samples/bookinfo/networking/destination-rule-all.yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: productpage
spec:
  host: productpage
  subsets:
  - name: v1
    labels:
      version: v1
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
  - name: v3
    labels:
      version: v3
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: ratings
spec:
  host: ratings
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
  - name: v2-mysql
    labels:
      version: v2-mysql
  - name: v2-mysql-vm
    labels:
      version: v2-mysql-vm
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: details
spec:
  host: details
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
---
```

> 这是一组 Istio 的 YAML 配置文件，用于定义目标规则 （DestinationRules）。Istio 是一个开源的服务网格，提供了服务间通信、安全和流量管理的功能。在这组配置文件中，定义了 4 个目标规则，分别针对 `productpage`、`reviews`、`ratings` 和 `details` 服务。
>
> - 第一个 DestinationRule（目标规则）针对 `productpage` 服务：
>   - host（主机）：目标规则所应用的服务的名称，即 productpage。
>   - subsets（子集）：包含一个名为 v1 的子集，标签 version 为 v1。
> - 第二个 DestinationRule 针对 `reviews` 服务：
>   - host：目标规则所应用的服务的名称，即 reviews。
>   - subsets：包含 v1、v2 和 v3 三个子集，对应的 version 标签分别为 v1、v2 和 v3。
> - 第三个 DestinationRule 针对 `ratings` 服务：
>   - host：目标规则所应用的服务的名称，即 ratings。
>   - subsets：包含 v1、v2、v2-mysql 和 v2-mysql-vm 四个子集，对应的 version 标签分别为 v1、v2、v2-mysql 和 v2-mysql-vm。
> - 第四个 DestinationRule 针对 `details` 服务：
>   - host：目标规则所应用的服务的名称，即 details。
>   - subsets：包含 v1 和 v2 两个子集，对应的 version 标签分别为 v1 和 v2。
>
> 这些配置文件定义了这些服务之间的子集和版本信息，以便在路由规则和流量管理中使用。

创建将`review`流量都指向`v1`虚拟服务

```Bash
kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
```

查看该虚拟服务

```Plaintext
nano samples/bookinfo/networking/virtual-service-all-v1.yaml
```

这个配置文件明确定义了任何情况下只呈现 v1 版本的 reviews

```YAML
apiVersion: networking.istio.io/v1alpha3
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
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - route:
    - destination:
        host: ratings
        subset: v1
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: details
spec:
  hosts:
  - details
  http:
  - route:
    - destination:
        host: details
        subset: v1
---
```

> 这个 YAML 配置文件包含了 4 个虚拟服务（VirtualService）配置，分别为 productpage、reviews、ratings 和 details。它们在核心配置上与之前提及的基本配置相似，但涉及到了 4 个不同的服务。
>
> 这些是一系列 Istio VirtualService 资源的定义，它们用于管理服务网格中的流量路由。
>
> VirtualService 是 Istio 中的一种配置资源，用于定义如何路由服务网格内的流量。每个 VirtualService 资源都包含一组路由规则，描述了将流量从入口网关路由到相应服务的方式。
>
> 让我们逐个解释这些 VirtualService 资源：
>
> - `productpage` VirtualService:
>   - 它将目标主机为 `productpage` 的流量路由到 `productpage` 服务的 `v1` 子集。
> - `reviews` VirtualService:
>   - 它将目标主机为 `reviews` 的流量路由到 `reviews` 服务的 `v1` 子集。
> - `ratings` VirtualService:
>   - 它将目标主机为 `ratings` 的流量路由到 `ratings` 服务的 `v1` 子集。
> - `details` VirtualService:
>   - 它将目标主机为 `details` 的流量路由到 `details` 服务的 `v1` 子集。
>
> 这些 VirtualService 资源定义了一个简单的流量拓扑，将所有流量路由到各个服务的 `v1` 子集。在更复杂的场景中，VirtualService 可以配置高级路由规则，例如基于权重的流量分割、基于 HTTP 头的路由等。通过应用这些 VirtualService 资源，您可以精确控制服务网格中的流量分发，实现灰度发布、蓝绿部署等功能。

使用浏览器查看效果，即使反复 F5，也是无星星版

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=Yjc5Mzc4ZGE5YjFjZGNhYzliY2RmNmUzMmExZDcwYzlfdnQ3QWZBaXoyNWZnanpUQzBXbWR0S2Z5cUc4bzFFU3FfVG9rZW46S0VNdmJXS3JQb08ybFV4b0N6NmN5WmptbmFnXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

创建将登录用户的 review 流量都指向 v2 的虚拟服务

```Bash
kubectl apply -f  samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
```

Jason 同志应该可以看到黑星星

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=NTdmMmIyMjk2M2E2MzE1YjNlNDQ3Mzk0NTY0MzVkY2ZfZGVGWEFER2FKTjV4MHFZZG9QdUwyUWx4THBiZFB2NWdfVG9rZW46TmJybGJiR1NCb3NFcTV4aFg5QWM0WEdybnFjXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

查看该虚拟服务

```Bash
nano samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1
```

> 这个 VirtualService 定义了如何处理目标为`reviews`服务的流量。当流量包含一个名为`end-user`、内容为`jason`的 HTTP 头时，流量将被路由至 `reviews` 服务的 `v2` 子集。在其他情况下（无匹配的头信息），流量将被路由至 `reviews` 服务的 `v1` 子集。

清理环境

```Bash
kubectl delete -f samples/bookinfo/networking/virtual-service-all-v1.yaml
kubectl delete -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
```

## 2.流量转移

（可选）启用默认目标规则

```Bash
kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
```

> 使用浏览器查看页面效果，主要是关注 reviews 的版本

将所有流量指向`reviews:v1`

```Bash
kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
```

查看该虚拟服务

```Bash
nano samples/bookinfo/networking/virtual-service-all-v1.yaml
apiVersion: networking.istio.io/v1alpha3
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
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - route:
    - destination:
        host: ratings
        subset: v1
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: details
spec:
  hosts:
  - details
  http:
  - route:
    - destination:
        host: details
        subset: v1
---
```

> 这个 VirtualService 确保了所有发送到 `reviews` 服务的 HTTP 流量都被路由到该服务的 `v1` 子集。如果你有多个版本的 `reviews` 服务在运行，你可以使用其他规则将流量分割到不同的子集。

使用浏览器查看页面效果，主要是关注 reviews 的版本

将`50%` 的流量从 `reviews:v1` 转移到 `reviews:v3`

```Bash
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml
```

查看该虚拟服务

```Bash
nano samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 50
    - destination:
        host: reviews
        subset: v3
      weight: 50
```

> 与之前的 VirtualService 配置相比，这个新的配置文件有以下不同之处：
>
> - 路由规则变更：
>   - 之前的配置只有一个路由规则，将所有流量路由到 `reviews` 服务的 `v1` 子集。
>   - 新的配置包含两个路由规则，每个规则都指定了一个目标子集和一个权重。
> - 流量分割：
>   - 新的配置通过指定权重来实现流量分割。
>   - 第一个规则将 50% 的流量路由到 `reviews` 服务的 `v1` 子集。
>   - 第二个规则将另外 50% 的流量路由到 `reviews` 服务的 `v3` 子集。
> - 多版本支持：
>   - 新的配置支持将流量路由到 `reviews` 服务的不同版本（`v1` 和 `v3`）。
>   - 这种设置通常用于渐进式部署、蓝绿部署或金丝雀发布等场景，允许同时测试和运行多个版本的服务。
>
> 这个新的 VirtualService 配置利用了 Istio 的流量权重和版本路由功能，实现了对 `reviews` 服务的流量分割和多版本支持。相比之前的配置，它提供了更好的灵活性和控制力，有助于平滑地进行应用程序的升级和测试。

使用浏览器查看页面效果，主要是关注 reviews 的版本

将 `100%` 的流量路由到 `reviews:v3`

```Bash
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-v3.yaml
```

查看该虚拟服务

```Bash
nano samples/bookinfo/networking/virtual-service-reviews-v3.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v3
```

> 这个配置文件同样定义了一个 Istio VirtualService，它针对 reviews 服务设置了流量路由规则。然而，与之前的配置文件不同，这个配置文件没有为流量设置权重分配。
>
> 在这个配置文件中，所有进入 reviews 服务的流量将被直接路由至 reviews 服务的`v3`子集（即版本 3）。因为这个配置文件没有为不同版本的 reviews 服务设置权重分配，所以`v3`子集会接收到全部的流量。这意味着仅使用 reviews 服务的`v3`版本，而不再对其他版本进行任何负载均衡。

使用浏览器查看页面效果，主要是关注`reviews`的版本

清理

```Bash
kubectl delete -f samples/bookinfo/networking/virtual-service-all-v1.yaml
```

## 3.网关

查看现有网关

```Bash
kubectl get gw
root@node1:~/istio-1.26.0# kubectl get gw
NAME               AGE
bookinfo-gateway   13m
```

增加网关

```Bash
kubectl apply -f istiolabmanual/gateway.yaml 
```

查看网关定义文件

```Plaintext
nano istiolabmanual/gateway.yaml 
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: test-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
```

> 这个配置文件定义了一个 Istio Gateway，它指定如何将外部流量引入到 Istio 服务网格中。Gateway 是 Istio 中用于在服务网格和外部网络之间建立连接关系的关键组件。
>
> 以下是关于这个配置文件的主要部分：
>
> - `metadata.name` 指定了 Gateway 的名称，也就是 "test-gateway"。
> - 在 `spec.selector` 中，配置文件使用键值对 "istio: ingressgateway" 来指定 Gateway 应绑定到 Istio 网格中的 Ingress Gateway 组件。这意味着所有进入 Istio 网格的流量将通过此 Ingress Gateway 处理。
> - `spec.servers` 定义了一个服务器以接收进入服务网格的流量。
>   - 使用端口号 80，名为"http"，以及 HTTP 协议，这指定了 Ingress Gateway 接收 HTTP 流量的端口。
>   - `hosts` 列表中的通配符 "*" 表示此 Gateway 将接受发送到任何域名的流量。
>
> 总之，这个 Gateway 配置文件定义了一个名为 "`test-gateway`" 的 Ingress Gateway，在 80 端口上接收任何 HTTP 流量，并将这些流量引入到 Istio 服务网格中。

查看现有网关

```Bash
kubectl get gw
root@node1:~/istio-1.26.0# kubectl get gw
NAME               AGE
bookinfo-gateway   15m
test-gateway       62s
```

增加虚拟服务

```Bash
kubectl apply -f istiolabmanual/virtualservice.yaml
```

查看虚拟服务定义文件

```Bash
nano istiolabmanual/virtualservice.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: test-gateway
spec:
  hosts:
  - "*"
  gateways:
  - test-gateway
  http:
  - match:
    - uri:
        prefix: /details
    - uri:
        exact: /health
    route:
    - destination:
        host: details
        port:
          number: 9080
```

> 这是一个 Istio VirtualService 配置文件，用于定义基于特定条件将请求路由到目标服务。具体来说，这个配置文件指定了与网关 "test-gateway" 和 HTTP 请求路径相关的路由规则。
>
> - `apiVersion`： 这表示 Istio API 的版本号。
> - `kind`： 表明这是一个 VirtualService 配置。
>
> 该配置文件具有以下元信息（metadata）：
>
> - `name`： 配置文件的名称（在这里为 "test-gateway"）。
>
> 核心逻辑如下：
>
> - `hosts`： 列出了匹配的主机名，星号表示匹配任何主机。
> - `gateways`： 这里只有一个网关（"test-gateway"）。这是请求需要从该网关经过才能进入 Istio 服务网格的规则。
>
> 接着，定义了 HTTP 请求的路由规则：
>
> - `match`： 包含两个匹配条件：
>   - 第一个条件匹配所有以 `/details` 为前缀的 URI。
>   - 第二个条件匹配准确的 URI `/health`。
> - 当请求的 URI 与上述任一条件匹配时，路由规则将被触发。在这种情况下，请求将路由到以下目的地：
>   - `destination`： 目标服务：
>     - `host`： 目标服务的名称（这里是 "details"）。
>     - `port`： 目标服务的端口号（这里是 9080）。
>
> 总之，这个配置文件定义了一个 VirtualService，当请求从"test-gateway"网关进入 Istio 服务网格并满足指定的 URI 匹配条件（以 `/details` 为前缀或准确匹配 `/health`）时，请求将被路由到 "`details`" 服务的 9080 端口。

查看虚拟服务

```Bash
kubectl get vs
root@node1:~/istio-1.26.0# kubectl get vs
NAME           GATEWAYS               HOSTS   AGE
bookinfo       ["bookinfo-gateway"]   ["*"]   18m
test-gateway   ["test-gateway"]       ["*"]   100s
```

随后使用浏览器访问`/details/0` 和 `/health`，检查效果

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=MjQ3YzYzZGExMjEyM2M0NWRmNjUwNDE5ZTZlZTAyNDBfQUVENzBaUXQyaFNzZ0pjT1ZQM0xlYjhHZjB4ZTlTSEhfVG9rZW46VU9ZcGJkM0dpb3VpS0R4MUZCbmNBM0lUbk9HXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=ZGE4ZDU2MjdiYzk3MDRiZWI3MTdmNTU2NmU4MzFiNTRfS2R4bkFyeEJKU2ZFcmdTYnVZQndVdDVCaGlYd0preTNfVG9rZW46V3ZEYmJ3RlBFb3phbHZ4ZmpBM2N0VlVRbkxnXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

清理环境

```Bash
kubectl delete -f istiolabmanual/gateway.yaml 
kubectl delete -f istiolabmanual/virtualservice.yaml
```

## 4.服务入口

安装 sleep 应用

```Bash
kubectl apply -f samples/sleep/sleep.yaml
```

查看 pod

```Bash
kubectl  get pod 
root@node1:~/istio-1.26.0# kubectl  get pod
NAME                             READY   STATUS    RESTARTS   AGE
details-v1-698b5d8c98-6qksf      2/2     Running   0          31m
productpage-v1-bf4b489d8-b6k9m   2/2     Running   0          31m
ratings-v1-5967f59c58-wstvp      2/2     Running   0          31m
reviews-v1-9c6bb6658-zwhcl       2/2     Running   0          31m
reviews-v2-8454bb78d8-fbj4k      2/2     Running   0          31m
reviews-v3-6dc9897554-n6947      2/2     Running   0          31m
sleep-75bbc86479-rz9hv           2/2     Running   0          44s
```

设置 source_pod 变量

```Bash
export SOURCE_POD=$(kubectl get pod -l app=sleep -o jsonpath={.items..metadata.name})
```

查看出站访问效果

```Bash
kubectl exec -it $SOURCE_POD -c sleep -- curl http://httpbin.org/headers
root@node1:~/istio-1.26.0# kubectl exec -it $SOURCE_POD -c sleep -- curl http://httpbin.org/headers
{
  "headers": {
    "Accept": "*/*",
    "Host": "httpbin.org",
    "User-Agent": "curl/7.81.0-DEV",
    "X-Amzn-Trace-Id": "Root=1-6388586a-60445856410d4db06df83a5c",
    "X-B3-Sampled": "1",
    "X-B3-Spanid": "15e9d5ae982425d4",
    "X-B3-Traceid": "08e4b47d73abff7a15e9d5ae982425d4",
    "X-Envoy-Attempt-Count": "1",
    "X-Envoy-Peer-Metadata": "ChkKDkFQUF9DT05UQUlORVJTEgcaBXNsZWVwChoKCkNMVVNURVJfSUQSDBoKS3ViZXJuZXRlcwofCgxJTlNUQU5DRV9JUFMSDxoNMTAuMjQ0LjEwNC4xMgoZCg1JU1RJT19WRVJTSU9OEggaBjEuMTYuMAqhAQoGTEFCRUxTEpYBKpMBCg4KA2FwcBIHGgVzbGVlcAokChlzZWN1cml0eS5pc3Rpby5pby90bHNNb2RlEgcaBWlzdGlvCioKH3NlcnZpY2UuaXN0aW8uaW8vY2Fub25pY2FsLW5hbWUSBxoFc2xlZXAKLwojc2VydmljZS5pc3Rpby5pby9jYW5vbmljYWwtcmV2aXNpb24SCBoGbGF0ZXN0ChoKB01FU0hfSUQSDxoNY2x1c3Rlci5sb2NhbAogCgROQU1FEhgaFnNsZWVwLTc1YmJjODY0Nzktcno5aHYKFgoJTkFNRVNQQUNFEgkaB2RlZmF1bHQKSQoFT1dORVISQBo+a3ViZXJuZXRlczovL2FwaXMvYXBwcy92MS9uYW1lc3BhY2VzL2RlZmF1bHQvZGVwbG95bWVudHMvc2xlZXAKFwoRUExBVEZPUk1fTUVUQURBVEESAioAChgKDVdPUktMT0FEX05BTUUSBxoFc2xlZXA=",
    "X-Envoy-Peer-Metadata-Id": "sidecar~10.244.104.12~sleep-75bbc86479-rz9hv.default~default.svc.cluster.local"
  }
}
```

> 这个命令执行了以下操作：
>
> 1. `export SOURCE_POD=$(kubectl get pod -l app=sleep -o jsonpath={.items..metadata.name})` 获取标签为 `app=sleep` 的 Pod 名称，并将其赋值给环境变量 `SOURCE_POD`。
> 2. `kubectl exec -it $SOURCE_POD -c sleep -- curl ``http://httpbin.org/headers` 在具有标签 `app=sleep` 的 Pod 中的 `sleep` 容器中执行 `curl ``http://httpbin.org/headers` 命令。
>
> 这个命令的目的是从 Istio 服务网格内部发送一个 HTTP 请求到外部的 `httpbin.org` 服务，并获取响应头信息。
>
> 响应结果显示了由 Istio 代理 （Envoy sidecar） 添加的一些特殊头信息，例如：
>
> - `X-Envoy-*` 头信息由 Envoy 代理添加，包含了代理的一些元数据。
> - `X-B3-*` 头信息与分布式跟踪相关，由 Istio 自动注入。
>
> 这些头信息对于调试和监控 Istio 服务网格非常有用。它们提供了有关请求路由、代理元数据和分布式跟踪的信息。通过分析这些头信息，您可以了解请求是如何在 Istio 服务网格中流动的，以及 Istio 代理添加了哪些功能。

关闭默认出站访问

```Bash
istioctl install  --set meshConfig.outboundTrafficPolicy.mode=REGISTRY_ONLY -y
```

如果出现报错，尝试重新定义环境变量

```Bash
export PATH="$PATH:/root/istio-1.26.0/bin"
root@node1:~/istio-1.26.0# istioctl install  --set meshConfig.outboundTrafficPolicy.mode=REGISTRY_ONLY -y
✔ Istio core installed
✔ Istiod installed
✔ Ingress gateways installed
- Pruning removed resources                                                                                                              Removed Deployment:istio-system:istio-egressgateway.
  Removed Service:istio-system:istio-egressgateway.
  Removed ServiceAccount:istio-system:istio-egressgateway-service-account.
  Removed RoleBinding:istio-system:istio-egressgateway-sds.
  Removed Role:istio-system:istio-egressgateway-sds.
  Removed PodDisruptionBudget:istio-system:istio-egressgateway.
✔ Installation complete                                                                                                                Making this installation the default for injection and validation.

Thank you for installing Istio 1.21.  Please take a few minutes to tell us about your install/upgrade experience!  https://forms.gle/99uiMML96AmsXY5d6
```

查看出站访问效果

```Bash
kubectl exec -it $SOURCE_POD -c sleep -- curl http://httpbin.org/headers
root@node1:~/istio-1.26.0# kubectl exec -it $SOURCE_POD -c sleep -- curl http://httpbin.org/headers
root@node1:~/istio-1.26.0#
```

创建指向 `httpbin.org` 的 ServiceEntry

```Bash
kubectl apply -f istiolabmanual/serviceentry.yaml
```

查看 ServiceEntry 配置文件

```YAML
nano istiolabmanual/serviceentry.yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: httpbin-ext
spec:
  hosts:
  - httpbin.org
  ports:
  - number: 80
    name: http
    protocol: HTTP
  resolution: DNS
  location: MESH_EXTERNAL
```

> 这个配置文件是一个 Istio ServiceEntry 资源的定义，主要用于将外部服务（不属于服务网格的服务）纳入 Istio 服务网格内，使得网格内的服务能够调用这些外部服务并对流量应用 Istio 的流量管理和安全策略。
>
> 具体解释如下：
>
> - `apiVersion: networking.istio.io/v1alpha3`：声明了这个资源使用 Istio 网络 API 的 v1alpha3 版本。
> - `kind: ServiceEntry`：定义了这个资源的类型，即一个 Istio ServiceEntry。
> - `metadata:`：资源的元数据。
>   - `name: httpbin-ext`：给这个资源分配了一个名字，叫做 "httpbin-ext"。
> - `spec:`：资源的详细配置。
>   - `hosts:`：一个外部服务的域名列表。
>     - `httpbin.org`：指定了需要访问的外部服务的域名为"httpbin.org"。
>   - `ports:`：定义外部服务需要暴露的端口信息。
>     - `number: 80`：暴露的端口号为 80。
>     - `name: http`：给此端口分配一个名字，叫做 "http"。
>     - `protocol: HTTP`：声明此端口上运行的协议是 HTTP。
>   - `resolution: DNS`：表示使用 DNS 解析来找到外部服务的 IP 地址。
>   - `location: MESH_EXTERNAL`：指明这是一个外部服务（不在 Istio 服务网格内）。对此类服务，Istio 将透明地执行域名解析和负载均衡。
>
> 总之，这个配置文件创建了一个名为 "`httpbin-ext`" 的 Istio ServiceEntry，允许服务网格内的服务调用外部域名 "httpbin.org" 上的 HTTP 服务，访问端口为 80。

查看 ServiceEntry

```Bash
kubectl get se
root@node1:~/istio-1.26.0# kubectl get se
NAME          HOSTS             LOCATION        RESOLUTION   AGE
httpbin-ext   ["httpbin.org"]   MESH_EXTERNAL   DNS          75s
```

稍等数秒钟之后，再次查看出站访问效果

```Bash
kubectl exec -it $SOURCE_POD -c sleep -- curl http://httpbin.org/ip
root@node1:~/istio-1.26.0# kubectl exec -it $SOURCE_POD -c sleep -- curl http://httpbin.org/ip
{
  "origin": "20.205.45.50"
}
```

清理

```Bash
kubectl delete -f samples/sleep/sleep.yaml
kubectl delete -f istiolabmanual/serviceentry.yaml
istioctl install --set profile=demo -y
```

## 5.Ingress

创建 httpbin 服务

```Bash
kubectl apply -f samples/httpbin/httpbin.yaml
```

查看 pod

```Bash
kubectl get pods
root@node1:~/istio-1.26.0# kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
details-v1-698b5d8c98-6qksf      2/2     Running   0          42m
httpbin-85d76b4bb6-brrkx         2/2     Running   0          84s
productpage-v1-bf4b489d8-b6k9m   2/2     Running   0          42m
ratings-v1-5967f59c58-wstvp      2/2     Running   0          42m
reviews-v1-9c6bb6658-zwhcl       2/2     Running   0          42m
reviews-v2-8454bb78d8-fbj4k      2/2     Running   0          42m
reviews-v3-6dc9897554-n6947      2/2     Running   0          42m
```

查看 ingressgateway

```Bash
kubectl get svc istio-ingressgateway -n istio-system
root@node1:~/istio-1.26.0# kubectl get svc istio-ingressgateway -n istio-system
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                                      AGE
istio-ingressgateway   LoadBalancer   10.105.85.107   <pending>     15021:31215/TCP,80:30193/TCP,443:32696/TCP,31400:32528/TCP,15443:32399/TCP   48m
```

设置 ingress 主机和端口变量

```Bash
export INGRESS_HOST=$(kubectl get po -l istio=ingressgateway -n istio-system -o jsonpath='{.items[0].status.hostIP}')
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
export TCP_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="tcp")].nodePort}')
```

> 这组命令帮助我们收集了 Istio Ingress Gateway 的主机 IP、HTTP/2 端口、HTTPS 端口以及 TCP 端口，以便在以后的操作中使用。

创建 ingress gateway，定义接入点

```Bash
kubectl apply -f  istiolabmanual/ingressgateway.yaml 
```

查看 ingress gateway 配置文件

```Bash
nano istiolabmanual/ingressgateway.yaml 
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: httpbin-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "httpbin.example.com"
```

> 这是一个 Istio Gateway 配置文件，用于定义一个初始接入点，以使外部用户能够访问到在 Istio Service Mesh 内部运行的服务。以下是详细解释：
>
> - `apiVersion: networking.istio.io/v1alpha3`：配置文件的 API 版本，这里使用的是 Istio 网络 API 的 v1alpha3 版本。
> - `kind: Gateway`：表明这是一个 Istio Gateway 资源类型。
> - `metadata`：资源的元数据。
>   - `name: httpbin-gateway`：Gateway 资源的名称，这里叫做`httpbin-gateway`。
> - `spec`：配置资源的详细规格。
>   - `selector`：用于选择一个或多个与此 Gateway 关联的 Istio Ingress Gateway 负载均衡器。
>     - `istio: ingressgateway`：选择所有标签为`istio=ingressgateway`的 Ingress Gateway。
>   - `servers`：一个服务器列表，每个服务器对应一个端口和协议，以及允许连接的主机名。
>     - `port`： 定义服务器监听的端口和协议。
>       - `number: 80`：此服务器监听 80 端口。
>       - `name: http`：端口名称定义为 http。
>       - `protocol: HTTP`：指定此端口使用 HTTP 协议。
>     - `hosts`：允许访问这个 Gateway 的外部主机名列表。
>       - `"httpbin.example.com"`：外部访问访问此服务需要通过`httpbin.example.com`的域名。
>
> 总结：这个配置文件定义了一个 Gateway（名为`httpbin-gateway`），该 Gateway 允许外部请求通过主机名`httpbin.example.com`访问到 Istio Service Mesh 内部的服务，监听 80 端口并使用 HTTP 协议。

创建 virtual service 定义路由规则

```Bash
kubectl apply -f istiolabmanual/ingressvs.yaml 
```

查看 ingress vs 的配置

```Bash
nano istiolabmanual/ingressvs.yaml 
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: httpbin
spec:
  hosts:
  - "httpbin.example.com"
  gateways:
  - httpbin-gateway
  http:
  - match:
    - uri:
        prefix: /status
    - uri:
        prefix: /delay
    route:
    - destination:
        port:
          number: 8000
        host: httpbin
```

> 这是一个 Istio VirtualService 配置文件。其主要作用是来定义流量路由规则，将访问特定主机名（本例中为 httpbin.example.com）的流量导向指定的 Istio 服务。以下是配置文件各个部分的解析：
>
> - `apiVersion`：这个配置文件遵循的 API 版本，为 networking.istio.io/v1alpha3。
> - `kind`：配置类型为 VirtualService。
> - `metadata`：元数据部分。
>   - `name`：VirtualService 的名称，为 httpbin。
> - `spec`：以下是具体的 VirtualService 规格配置。
>   - `hosts`：流量路由规则适用的匹配域名列表。本例中，匹配域名为 httpbin.example.com。
>   - `gateways`：定义此 VirtualService 通过哪些 Gateway 发布。本例中，使用 httpbin-gateway。
>   - `http`：定义 HTTP 路由规则。
>     - `match`：用于匹配特定条件的流量。
>       - `uri`：根据请求 URI 进行匹配。
>         - 第一个匹配规则：匹配以/status 开头的 URL 路径。
>         - 第二个匹配规则：匹配以/delay 开头的 URL 路径。
>     - `route`：指定匹配规则后要执行的路由操作。
>       - `destination`：设定流量路由目标。
>         - `port`：目标服务的端口，本例中为 8000。
>         - `host`：目标服务的主机名，本例中为 httpbin。
>
> 综上所述，这个配置文件定义了一个 VirtualService，名为`httpbin`，用于将主机名为 httpbin.example.com 的流量，通过`httpbin-gateway`网关，根据相应的 URL 匹配将流量路由至目标服务`httpbin`的 8000 端口。

我们可以使用 Mermaid 来可视化这个 VirtualService 资源中定义的路由规则。以下是使用 Mermaid 绘制的流程图：

```Plain
graph LR
    subgraph VirtualService[httpbin.example.com]
        gateway[httpbin-gateway]
        match1{"/status<br>/delay"}
        match2{"其他路径"}
        route1[httpbin:8000]
        route2>其他路由规则]
        gateway --> match1
        gateway --> match2
        match1 --> route1
        match2 --> route2
    end
```

在这个流程图中：

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=ZWE1NTdlYmRmZTE2NTU1OThkMTI0NzI1NzNlYWM5NGJfMmxrcEJ4MEhXN0NiNDZDRUM0MURHTDNoZ05yVURkd3JfVG9rZW46RDloMmJ2QTVPb1dDM2l4WjYybWNTaFVobko2XzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

- `VirtualService` 子图表示 `httpbin.example.com` 虚拟服务。
- `gateway` 节点表示 `httpbin-gateway` 网关。
- `match1` 决策节点表示匹配 `/status` 或 `/delay` 路径前缀的请求。
- `match2` 决策节点表示匹配其他路径的请求。
- `route1` 节点表示将 `/status` 或 `/delay` 路径前缀的请求路由到 `httpbin:8000` 服务。
- `route2` 节点表示将其他路径的请求路由到其他路由规则。

这个流程图清楚地展示了 VirtualService 资源中定义的路由规则：

1. 所有进入 `httpbin-gateway` 网关的请求都将被评估路由规则。
2. 如果请求的路径前缀是 `/status` 或 `/delay`，则将被路由到 `httpbin:8000` 服务。
3. 对于其他路径的请求，将根据其他路由规则进行路由。

查看 Virtual Service 信息，重点关注服务 网关和主机的绑定关系

```Bash
kubectl get vs
root@node1:~/istio-1.26.0# kubectl get vs
NAME       GATEWAYS               HOSTS                     AGE
bookinfo   ["bookinfo-gateway"]   ["*"]                     43m
httpbin    ["httpbin-gateway"]    ["httpbin.example.com"]   80s
```

访问已发布的 httpin 接口

```Bash
curl -I -HHost:httpbin.example.com http://$INGRESS_HOST:$INGRESS_PORT/status/200

curl -I -HHost:httpbin.example.com http://$INGRESS_HOST:$INGRESS_PORT/delay/2
root@node1:~/istio-1.26.0# curl -I -HHost:httpbin.example.com http://$INGRESS_HOST:$INGRESS_PORT/status/200
HTTP/1.1 200 OK
server: istio-envoy
date: Thu, 01 Dec 2022 07:45:37 GMT
content-type: text/html; charset=utf-8
access-control-allow-origin: *
access-control-allow-credentials: true
content-length: 0
x-envoy-upstream-service-time: 28

root@node1:~/istio-1.26.0# curl -I -HHost:httpbin.example.com http://$INGRESS_HOST:$INGRESS_PORT/delay/2
HTTP/1.1 200 OK
server: istio-envoy
date: Thu, 01 Dec 2022 07:45:50 GMT
content-type: application/json
content-length: 738
access-control-allow-origin: *
access-control-allow-credentials: true
x-envoy-upstream-service-time: 2006
```

访问未经定义的目标

```Bash
curl -I -HHost:httpbin.example.com http://$INGRESS_HOST:$INGRESS_PORT/headers
root@node1:~/istio-1.26.0# curl -I -HHost:httpbin.example.com http://$INGRESS_HOST:$INGRESS_PORT/headers
HTTP/1.1 404 Not Found
date: Thu, 01 Dec 2022 07:46:32 GMT
server: istio-envoy
transfer-encoding: chunked
```

设置规则将 headers 服务发布到外网

```Bash
kubectl apply -f istiolabmanual/ingressgateway2.yaml 
```

查看 ingress gate2 的配置文件

```Bash
nano istiolabmanual/ingressgateway2.yaml 
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: httpbin-gateway
spec:
  selector:
    istio: ingressgateway # use Istio default gateway implementation
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: httpbin
spec:
  hosts:
  - "*"
  gateways:
  - httpbin-gateway
  http:
  - match:
    - uri:
        prefix: /headers
    route:
    - destination:
        port:
          number: 8000
        host: httpbin
```

> 这是一个新的 Istio Gateway 和 VirtualService 配置文件。
>
> Gateway 配置部分：
>
> - `kind: Gateway` 定义了这是一个 Gateway 资源。
> - `metadata.name` 定义了 Gateway 的名称为：`httpbin-gateway`。
> - `spec.selector.istio` 使用默认的 Istio ingressgateway。
> - `spec.servers` 部分定义了 1 个服务器，其使用了 80 端口，协议为 HTTP，名称为 http。此服务器接受来自任何 host 的请求（hosts 设定为`*`）。
>
> VirtualService 配置部分：
>
> - `kind: VirtualService` 定义了这是一个 VirtualService 资源。
> - `metadata.name` 定义了 VirtualService 的名称为：`httpbin`。
> - `spec.hosts` 设置了所有的主机（`*`）。
> - `spec.gateways` 将 VirtualService 绑定到之前创建的`httpbin-gateway`。
> - `spec.http.match` 包含一个匹配规则，要求 URI 路径以`/headers`作为前缀。
> - `spec.http.route.destination` 部分定义了匹配规则后的流量将被发送到`httpbin`服务的 8000 端口。
>
> 此配置文件实现的功能是：将所有访问`httpbin.example.com`域名以及 URL 路径前缀为`/headers`的请求，通过`httpbin-gateway`网关转发至`httpbin`服务的 8000 端口。

使用浏览器加 `/headers` 在群集外进行访问

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=YjI4YWZlYTllYTM5NGVkMmI3ZTI1YTVkMDA3YzI2ZGNfTno1RjRkSnR2Ykd0U3laS0dEMTFWQlRoN2hWUHVPbExfVG9rZW46RXk1ZGJTTWNXbzhKOFp4d3VxeWNzWUh2bkRiXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

查看 Virtual Service 信息，重点关注服务 网关和主机的绑定关系

```Bash
kubectl get vs
root@node1:~/istio-1.26.0# kubectl get vs
NAME       GATEWAYS               HOSTS   AGE
bookinfo   ["bookinfo-gateway"]   ["*"]   47m
httpbin    ["httpbin-gateway"]    ["*"]   5m52s
```

清理资源

```Bash
kubectl delete gateway httpbin-gateway
kubectl delete virtualservice httpbin
kubectl delete --ignore-not-found=true -f samples/httpbin/httpbin.yaml
```

## 6.Egress

查看 istio 系统服务，确认 egress gateway 组件正常运行

```Bash
kubectl get svc -n istio-system | grep egress
root@node1:~/istio-1.26.0# kubectl get svc -n istio-system | grep egress
istio-egressgateway    ClusterIP      10.105.229.85    <none>        80/TCP,443/TCP   
```

查看 istio 系统 pod

```Bash
kubectl get pod -n istio-system | grep egress
root@node1:~/istio-1.26.0# kubectl get pod -n istio-system | grep egress
istio-egressgateway-d84b5f89f-558m4    1/1     Running   0          12m
```

安装 sleep 应用

```Bash
kubectl apply -f samples/sleep/sleep.yaml
```

设置 source_pod 变量

```Bash
export SOURCE_POD=$(kubectl get pod -l app=sleep -o jsonpath={.items..metadata.name})
```

> 这个命令是用于在 Kubernetes 环境中找到并获取一个名为`sleep`的应用程序的 pod 名称，然后将名称保存在名为`SOURCE_POD`的环境变量中。
>
> 让我逐步解释命令组件：
>
> - `kubectl get pod -l app=sleep`： 这部分命令使用`kubectl`工具从 Kubernetes 集群中获取 pod 信息，并使用`-l app=sleep`参数来过滤仅具有标签`app=sleep`的 pod。
> - `-o jsonpath={.items..metadata.name}`： 这部分是一个输出格式化选项，它要求`kubectl`以 JSON 格式返回结果，并使用 jsonpath 表达式`{.items..metadata.name}`提取名称字段。这将返回一个名称列表，其中包括所有匹配的 pod。
> - `export SOURCE_POD=$(…)`： 此部分将前面提到的命令的输出（即找到的 pod 名称）保存到名为`SOURCE_POD`的环境变量中，以便于以后在其他命令中使用。
>
> 此命令在某些场景中很有用，例如，您需要确定与特定应用程序关联的 pod 中的一个以便向其发送请求以进行测试。

为外部 httpbin 服务创建 service entry

```Bash
kubectl  apply -f  istiolabmanual/egressse.yaml 
```

查看 egress service entry 配置文件

```Bash
nano istiolabmanual/egressse.yaml 
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: httpbin
spec:
  hosts:
  - httpbin.org
  ports:
  - number: 80
    name: http-port
    protocol: HTTP
  resolution: DNS
```

> 这是一个 Istio 的配置文件，定义了一个`ServiceEntry`资源。`ServiceEntry`资源用于为集群外提供的服务在 Istio 服务网格中创建条目，以便将这些外部服务视为网格中的服务。这使得 Istio 可以将流量路由到这些外部服务，并对其施加策略。
>
> 以下是对这个配置文件的详细解析：
>
> - `apiVersion: networking.istio.io/v1alpha3` ：这指定了 Istio 资源的版本以及使用的 Istio API 子集。
> - `kind: ServiceEntry` ：此字段指定这是一个`ServiceEntry`资源。
> - `metadata` ：包含此资源的元数据。
>   - `name: httpbin`： 为资源分配的名称为`httpbin`。
> - `spec` ：定义`ServiceEntry`的具体配置。
>   - `hosts`： 列出与此资源关联的外部服务主机。
>     - `httpbin.org` ：此`ServiceEntry`将包含`httpbin.org`域。
>   - `ports`： 定义与外部服务关联的端口。
>     - `number: 80`：端口号为 80。
>     - `name: http-port`：为端口分配的名称为`http-port`。
>     - `protocol: HTTP`：使用的通信协议为 HTTP。
>   - `resolution: DNS`：指定如何解析外部服务的地址。此字段中使用的`DNS`值表示 Istio 将使用 DNS 解析来发现外部服务的 IP。
>
> 这个`ServiceEntry`配置允许 Istio 网格内的服务访问`httpbin.org`。这表示 Istio 将对`http://httpbin.org`的请求执行类似于集群内部服务的操作。因此，该配置可以帮助在 Istio 服务网格中实现请求的平滑路由、熔断、故障注入等功能。

检查 Service Entry

```Bash
kubectl get se
root@node1:~/istio-1.26.0# kubectl get se
NAME      HOSTS             LOCATION   RESOLUTION   AGE
httpbin   ["httpbin.org"]              DNS          104s
```

从 sleep 上访问外网

```Bash
kubectl exec -it $SOURCE_POD -c sleep -- curl http://httpbin.org/ip
root@node1:~/istio-1.26.0# kubectl exec -it $SOURCE_POD -c sleep -- curl http://httpbin.org/ip
{
  "origin": "20.205.45.50"
}
```

检查 sidecar 里的 proxy 日志

```Bash
kubectl logs $SOURCE_POD -c istio-proxy | tail
root@node1:~/istio-1.26.0# kubectl logs $SOURCE_POD -c istio-proxy | tail
2024-03-21T07:52:39.719206Z     info    cache   returned workload certificate from cache        ttl=23h59m59.280795108s
2024-03-21T07:52:39.719382Z     info    cache   returned workload trust anchor from cache       ttl=23h59m59.280627706s
2024-03-21T07:52:39.719501Z     info    ads     SDS: PUSH request for node:sleep-75bbc86479-8dv7p.default resources:1 size:4.0kB resource:default
2024-03-21T07:52:39.719557Z     info    ads     SDS: PUSH request for node:sleep-75bbc86479-8dv7p.default resources:1 size:1.1kB resource:ROOTCA
2024-03-21T07:52:39.719666Z     info    cache   returned workload trust anchor from cache       ttl=23h59m59.280338602s
2024-03-21T07:52:40.051272Z     info    Readiness succeeded in 5.666123916s
2024-03-21T07:52:40.051666Z     info    Envoy proxy is ready
2024-03-21T07:52:44.407739Z     warn    HTTP request failed: Get "http://169.254.169.254/metadata/instance?api-version=2019-08-15": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
2024-03-21T07:52:44.407773Z     warn    Could not unmarshal response: unexpected end of JSON input:
[2024-03-21T07:55:06.694Z] "GET /ip HTTP/1.1" 200 - via_upstream - "-" 0 31 608 607 "-" "curl/7.81.0-DEV" "e839b6e0-060c-9c78-9f97-26dda40d94d0" "httpbin.org" "54.166.148.227:80" outbound|80||httpbin.org 10.244.104.16:55424 3.215.37.86:80 10.244.104.16:43594 - default
```

> 这是从名为 sleep 的 Kubernetes 容器内的 istio-proxy （Envoy 代理）日志中提取的一段日志信息。我会为你逐条解释这些日志：
>
> 1. `2024-03-21T07:52:39.719206Z` 和 `2024-03-21T07:52:39.719382Z` 分别表示返回了请求的 workload 证书和 trust anchor 的缓存，这是与 Istio 的证书管理和安全通信相关的条目。
> 2. `2024-03-21T07:52:39.719501Z` 和 `2024-03-21T07:52:39.719557Z` 是与 Envoy 代理更新密钥和证书相关的信息（SDS 即 Secret Discovery Service 的推送）。
> 3. `2024-03-21T07:52:39.719666Z` 表示又一次返回了请求的 workload trust anchor 的缓存。
> 4. `2024-03-21T07:52:40.051272Z` 和 `2024-03-21T07:52:40.051666Z` 说明 Envoy 代理已经通过准备就绪检查，现在可以正常提供服务了。
> 5. `2024-03-21T07:52:44.407739Z` 和 `2024-03-21T07:52:44.407773Z` 是警告信息，表示发送到 `http://169.254.169.254/metadata/instance` 的 HTTP 请求因超时而失败。这可能是因为在请求元数据时出现了问题，或者请求的目标服务不可用。
> 6. 最后一条日志 `[2024-03-21T07:55:06.694Z] "GET /ip HTTP/1.1" 200 - via_upstream - "-" 0 31 608 607 "-" "curl/7.81.0-DEV" "e839b6e0-060c-9c78-9f97-26dda40d94d0" "httpbin.org" "54.166.148.227:80" outbound|80||httpbin.org 10.244.104.16:55424 3.215.37.86:80 10.244.104.16:43594 - default` 是一个访问 httpbin.org 的 GET 请求，请求的路径是 `/ip`。HTTP 响应状态为 200，表示请求成功。这个请求是通过 `curl/7.81.0-DEV` 客户端发起的，并在 Istio 服务网格中通过 Envoy 代理完成了路由。
>
> 这个日志的内容主要与 Envoy 代理的启动、证书更新以及通过代理处理的 HTTP 请求有关。
>
> 注意观察，此处的`upstream_cluster："outbound|80||httpbin.org"`

查看 Virtual Service 和 Destination Rule 信息

```Bash
kubectl get vs

kubectl get dr
root@node1:~/istio-1.26.0# kubectl get vs
NAME       GATEWAYS               HOSTS   AGE
bookinfo   ["bookinfo-gateway"]   ["*"]   55m
root@node1:~/istio-1.26.0#
root@node1:~/istio-1.26.0# kubectl get dr
NAME          HOST          AGE
details       details       54m
productpage   productpage   54m
ratings       ratings       54m
reviews       reviews       54m
```

创建 egress gateway

```Bash
kubectl  apply -f  istiolabmanual/egressgw.yaml 
```

查看 egress gateway 的配置文件

```Bash
nano istiolabmanual/egressgw.yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: istio-egressgateway
spec:
  selector:
    istio: egressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - httpbin.org
```

> 这是一个 Istio Gateway 配置文件。Gateway 是一个用于加载入网格或从网格中注入流量的特殊 Istio 资源。在本例中，这是一个关于 Istio egress 网关的配置。Istio 的 egress 网关允许你对出站流量进行安全、受控和观察能力的管理。
>
> 让我们详细解析一下这个配置文件：
>
> - `apiVersion`： 使用的 Istio API 版本是`networking.istio.io/v1alpha3`。
> - `kind`： 此资源类型是`Gateway`。
> - `metadata`： 定义了资源的元数据。
>   - `name`： 资源的名称是`istio-egressgateway`。
> - `spec`： 描述了 Gateway 资源的配置：
>   - `selector`： 定义了哪些部署工作负载应与此 Gateway 关联。
>     - `istio: egressgateway`： 已选择 Istio egressgateway 工作负载。
>   - `servers`： 定义了与此 Gateway 关联的端口、协议和允许的主机。
>     - `port`： 定义了端口，协议和名称。
>       - `number: 80`： 端口号为 80。
>       - `name: http`： 端口名称为 http。
>       - `protocol: HTTP`： 协议是 HTTP。
>     - `hosts`： 列出哪些主机可以使用这个 Gateway。
>       - `httpbin.org`： 允许从这个网关访问`httpbin.org`。
>
> 总结一下，这个 Istio Gateway 配置允许从网格的工作负载访问外部服务`httpbin.org`，通过 Istio egress 网关并使用 HTTP 协议在端口 80 上发起请求。

查看 gateway

```Bash
kubectl get gw
root@node1:~/istio-1.26.0# kubectl get gw
NAME                  AGE
bookinfo-gateway      56m
istio-egressgateway   83s
```

创建 virtual service，将流量引导到 egress gateway

```Bash
kubectl  apply -f  istiolabmanual/egressvs.yaml 
```

查看 egress virtual service 配置文件

```Plaintext
nano istiolabmanual/egressvs.yaml 
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: vs-for-egressgateway
spec:
  hosts:
  - httpbin.org
  gateways:
  - istio-egressgateway
  - mesh
  http:
  - match:
    - gateways:
      - mesh
      port: 80
    route:
    - destination:
        host: istio-egressgateway.istio-system.svc.cluster.local
        subset: httpbin
        port:
          number: 80
      weight: 100
  - match:
    - gateways:
      - istio-egressgateway
      port: 80
    route:
    - destination:
        host: httpbin.org
        port:
          number: 80
      weight: 100
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: dr-for-egressgateway
spec:
  host: istio-egressgateway.istio-system.svc.cluster.local
  subsets:
  - name: httpbin
```

> 这个配置文件的目的是通过 Istio Egress 网关来控制和监控 Kubernetes 集群内部服务对外部服务（在这个例子中是`httpbin.org`）的访问。
>
> 通过 Istio 的 Egress 网关，集群中的服务能够：
>
> 1. 通过 Istio 管理和安全地访问外部服务：所有出站流量都会被引导到 Istio Egress 网关，从而实现了统一的流量控制和安全策略。
> 2. 对访问外部服务的流量进行监控和跟踪：因为流量都经过 Egress 网关，所以 Istio 可以采集外部服务调用的指标数据，如延迟、错误率等。
> 3. 灵活地通过配置应用路由规则：例如，在处理过程中对请求或响应进行修改，或者将流量引导到一个备用的外部服务上。
> 4. 实施细粒度的访问控制策略：通过定义 DestinationRule 和 VirtualService 规则，可以有效地实施安全策略，例如为不同的服务设置不同的超时、重试、断路器策略等。
>
> 具体到这个配置文件，有两个 VirtualService 和一个 DestinationRule：
>
> 1. 第一个 VirtualService 将集群内部服务发往`httpbin.org`的请求定向到 Istio Egress 网关。
> 2. 第二个 VirtualService 将在 Istio Egress 网关收到的请求继续定向到`httpbin.org`。
> 3. DestinationRule 定义了访问 Istio Egress 网关的子集策略，包括负载平衡和连接池设置。这些设置可以为向外部服务发起的请求提供更好的性能和稳定性。
>
> 这个配置文件定义了一个 Istio VirtualService 和一个 DestinationRule，在 Kubernetes 集群中使用 Istio egress 网关访问外部服务`httpbin.org`。
>
> **VirtualService 解析**
>
> 1. `apiVersion`： 表示使用了 Istio 配置的版本，这里是`networking.istio.io/v1alpha3`。
> 2. `kind`： 表示配置文件的类型，这里是 VirtualService。
> 3. `metadata`： 配置的元数据，其中定义了 VirtualService 的名称，这里是`vs-for-egressgateway`。
> 4. `spec.hosts`： 定义了使用这个 VirtualService 的 hosts，这里是`httpbin.org`。
> 5. `spec.gateways`： 定义了应用此 VirtualService 的网关，这里包括`istio-egressgateway`（表示 Istio egress 网关）和`mesh`（表示 Istio 服务网格）。
> 6. `spec.http`： 定义了 HTTP 路由规则。
>
> `spec.http`中有两个路由规则：
>
> - 第一个规则：
>   - 当入口网关是 mesh，端口是 80 时，匹配该规则。
>   - 将请求路由到`istio-egressgateway.istio-system.svc.cluster.local`的 httpbin 子集，端口为 80。
>   - 路由权重为 100
> - 第二个规则：
>   - 当入口网关是 istio-egressgateway，端口是 80 时，匹配该规则。
>   - 将请求路由到`httpbin.org`，端口为 80。
>   - 路由权重为 100
>
> 首先， 请求从`mesh`中的服务提交至`istio-egressgateway`网关，然后，从网关转发至`httpbin.org`。
>
> **DestinationRule 解析**
>
> 1. `apiVersion`： 表示使用了 Istio 配置的版本，这里是`networking.istio.io/v1alpha3`。
> 2. `kind`： 表示配置文件的类型，这里是 DestinationRule。
> 3. `metadata`： 配置的元数据，其中定义了 DestinationRule 的名称，这里是`dr-for-egressgateway`。
> 4. `spec.host`： 定义了 DestinationRule 应用的主机，这里是`istio-egressgateway.istio-system.svc.cluster.local`（表示 Istio egress 网关的服务地址）。
> 5. `spec.subsets`： 定义了主机的子集，这里有一个名为`httpbin`的子集。
>
> 这个 DestinationRule 用于设置访问 Istio egress 网关时使用的子集策略，可以用于设置连接池、负载均衡等。

用一个简化的 mermaid 序列图来表示配置文件中的路由过程。

```Plaintext
sequenceDiagram
participant Client as Client
participant IstioMesh as Istio Mesh
participant EgressGateway as Istio Egress Gateway
participant HttpbinOrg as httpbin.org

Client->>IstioMesh: HTTP Request (httpbin.org)
IstioMesh->>EgressGateway: Route to Egress Gateway (subset: httpbin, port: 80)
EgressGateway->>HttpbinOrg: Forward request (host: httpbin.org, port: 80)
HttpbinOrg->>EgressGateway: FORWARD_RESPONSE
EgressGateway->>IstioMesh: FORWARD_RESPONSE
IstioMesh->>Client: HTTP Response (httpbin.org)
```

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=MzI3YWE2NGUwZmU5MGIwYzlhOTdjM2JiYTY1M2E4ZjJfSmhQeHVPOE9mTlBjbFNxOFlBSTJHZFdKMk1rbXNRNk5fVG9rZW46WkYxa2JNNE9jb1R3Y0d4ZGJKdGNoM2VpbnBkXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

在这个图中，

1. 首先，客户端发送一个请求到 Istio 服务网格，目标是 httpbin.org。
2. 服务网格将请求路由到 Istio egress 网关（使用 httpbin 子集和端口 80）。
3. 接下来，Istio egress 网关将请求转发给 httpbin.org（使用端口 80）。
4. 来自 httpbin.org 的响应经过 Istio egress 网关和 Istio 服务网格，最后被发送给客户端。

查看 Virtual Service 和 Destination Rule 信息

```Bash
kubectl get vs
kubectl get dr
root@node1:~/istio-1.26.0# kubectl get vs
NAME                   GATEWAYS                         HOSTS             AGE
bookinfo               ["bookinfo-gateway"]             ["*"]             59m
vs-for-egressgateway   ["istio-egressgateway","mesh"]   ["httpbin.org"]   89s
root@node1:~/istio-1.26.0# kubectl get dr
NAME                   HOST                                                 AGE
details                details                                              59m
dr-for-egressgateway   istio-egressgateway.istio-system.svc.cluster.local   93s
productpage            productpage                                          59m
ratings                ratings                                              59m
reviews                reviews                                              59m
```

从 sleep 上访问外网

```Bash
kubectl exec -it $SOURCE_POD -c sleep -- curl http://httpbin.org/ip
root@node1:~/istio-1.26.0# kubectl exec -it $SOURCE_POD -c sleep -- curl http://httpbin.org/ip
{
  "origin": "10.244.104.16, 20.205.45.50"
}
注意：此处的ip地址发生了变化
```

检查 sidecar 里的 proxy 日志，观察新的条目

```Bash
kubectl logs $SOURCE_POD -c istio-proxy | tail
root@node1:~/istio-1.26.0# kubectl logs $SOURCE_POD -c istio-proxy | tail
2024-03-21T07:52:39.719382Z     info    cache   returned workload trust anchor from cache       ttl=23h59m59.280627706s
2024-03-21T07:52:39.719501Z     info    ads     SDS: PUSH request for node:sleep-75bbc86479-8dv7p.default resources:1 size:4.0kB resource:default
2024-03-21T07:52:39.719557Z     info    ads     SDS: PUSH request for node:sleep-75bbc86479-8dv7p.default resources:1 size:1.1kB resource:ROOTCA
2024-03-21T07:52:39.719666Z     info    cache   returned workload trust anchor from cache       ttl=23h59m59.280338602s
2024-03-21T07:52:40.051272Z     info    Readiness succeeded in 5.666123916s
2024-03-21T07:52:40.051666Z     info    Envoy proxy is ready
2024-03-21T07:52:44.407739Z     warn    HTTP request failed: Get "http://169.254.169.254/metadata/instance?api-version=2019-08-15": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
2024-03-21T07:52:44.407773Z     warn    Could not unmarshal response: unexpected end of JSON input:
[2024-03-21T07:55:06.694Z] "GET /ip HTTP/1.1" 200 - via_upstream - "-" 0 31 608 607 "-" "curl/7.81.0-DEV" "e839b6e0-060c-9c78-9f97-26dda40d94d0" "httpbin.org" "54.166.148.227:80" outbound|80||httpbin.org 10.244.104.16:55424 3.215.37.86:80 10.244.104.16:43594 - default
[2024-03-21T08:01:34.332Z] "GET /ip HTTP/1.1" 200 - via_upstream - "-" 0 46 614 614 "-" "curl/7.81.0-DEV" "81f48ace-057e-978a-85b4-1bcecdb48e27" "httpbin.org" "10.244.135.13:8080" outbound|80|httpbin|istio-egressgateway.istio-system.svc.cluster.local 10.244.104.16:47570 54.166.148.227:80 10.244.104.16:57680 - -
```

> 注意观察，启用了`egress gateway`之后此处的`upstream_cluster："outbound|80|httpbin|istio-egressgateway.istio-system.svc.cluster.local"`
>
> 在`2024-03-21T07:55:06.694Z`和`2024-03-21T08:01:34.332Z`，本地服务向`httpbin.org`发起了两个请求，请求路径为`/ip`。在这两个请求中，都成功返回了 HTTP 状态码 200，表示请求已成功处理。
>
> 注意，在第一个请求中，响应直接从`httpbin.org`（IP 地址： 54.166.148.227）返回，在第二个请求中，响应先经过了我们之前配置的 Istio Egress 网关（Istio Egress Gateway 地址：10.244.135.13：8080），然后再路由至外部服务`httpbin.org`。
>
> 这说明之前的 Istio 配置文件已成功生效，将流量导向了正确的路径。

清理

```Bash
kubectl delete -f samples/sleep/sleep.yaml
kubectl delete -f  istiolabmanual/egressse.yaml 
kubectl delete -f istiolabmanual/egressgw.yaml
kubectl delete -f istiolabmanual/egressvs.yaml
```

# Lab 4 弹性能力

## 1.超时重试

（可选）加载 default destination rules。

```Bash
kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
```

将 review 指向 v2 版本

```Bash
kubectl apply -f istiolabmanual/reviewsv2.yaml 
```

查看配置文件

```YAML
nano istiolabmanual/reviewsv2.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v2
```

> 这个 Istio VirtualService 资源将所有发送到`reviews`服务的流量路由至版本为 v2 的`reviews`服务实例。这对于实现流量管理，如蓝绿部署、金丝雀发布等策略非常有用。

查看 bookinfo 页面，看黑星星

给 ratings 服务添加延迟

```Bash
kubectl apply -f istiolabmanual/delay.yaml 
```

查看配置文件

```YAML
nano istiolabmanual/delay.yaml 
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - fault:
      delay:
        percent: 100
        fixedDelay: 2s
    route:
    - destination:
        host: ratings
        subset: v1
```

> 这是一个 Istio VirtualService 资源配置文件。它定义了一个名为"ratings"的 VirtualService，用于在服务网格内管理流量。以下是各部分的详细解释：
>
> - `apiVersion`： 指定 Istio API 的版本，这里是`networking.istio.io/v1alpha3`。
> - `kind`： 资源类型为`VirtualService`，表明这是一个 Istio VirtualService 资源。
> - `metadata`： 定义了资源的元数据。
>   - `name`： 名称为`ratings`的资源。
> - `spec`： 描述 VirtualService 的具体规范。
>   - `hosts`： 指定 VirtualService 所管理的主机或服务，这里是`ratings`。
>   - `http`： 定义 HTTP 流量规则。
>     - `fault`： 故障注入规则，用于模拟服务异常，如延迟和故障，帮助测试服务的弹性。
>       - `delay`： 延迟注入，表示所有调用"ratings"服务的请求将被延迟。
>         - `percent`： 注入延迟的请求百分比。这里是 100%，表示所有请求都会受到延迟影响。
>         - `fixedDelay`： 固定延迟时间，这里是 2 秒。表明流量将被延迟 2 秒。
>     - `route`： 定义路由规则。
>       - `destination`： 目标服务的信息。
>         - `host`： 目标服务的主机名，这里是"ratings"。
>         - `subset`： 目标服务的子集（例如版本），这里是"v1"。表示流量将路由至"ratings"服务的"v1"版本。
>
> 总结：这个配置文件表示了一个名为"`ratings`"的 VirtualService 作用于"`ratings`"服务，实现了故障注入，对全部请求进行延迟`2`秒的操作，并将流量路由至"`ratings`"服务的"`v1`"版本。这种配置方式在模拟故障以测试服务的弹性时非常有用。

查看 bookinfo 页面观察延迟，会观察到页面需要大约 2s 才能加载完成

给 reviews 服务添加超时策略

```Bash
kubectl apply -f istiolabmanual/timeout.yaml 
```

查看 bookinfo 页面观察快速失败，延时设置为 2s，但是我们的超时是 1s，所以就可耻地失败了

给 ratings 服务添加重试策略

```Bash
kubectl apply -f istiolabmanual/retry.yaml 
```

查看超时配置文件

```Bash
nano istiolabmanual/retry.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v2
    timeout: 1s
```

> 这是一个 Istio VirtualService 资源的配置文件，用于在 Istio 服务网格内管理服务"reviews"的流量。
>
> 详细解析如下：
>
> - `apiVersion`： 这个参数表示配置文件使用的 API 版本，本例中为"networking.istio.io/v1alpha3"。
> - `kind`： 表示资源类型，在这里类型是"VirtualService"。
> - `metadata`： 配置文件的元数据信息，包括资源名称等。
>   - `name`： 这个参数表示资源的名字，这里为"reviews"。
> - `spec`： 资源配置的具体规格。
>   - `hosts`： 表示与此配置相关联的主机。这里的主机是"reviews"。
>   - `http`： HTTP 路由规则的配置列表。
>     - `route`： 定义流量路由规则。
>       - `destination`： 定义流量的目的地。
>         - `host`： 流量要发送到的目的主机，这里指定为"reviews"。
>         - `subset`： 流量要路由到的目标服务的子集，在本例中为"v2"版本。
>     - `timeout`： 设定请求超时时间，在这个例子中，超时时间为 1 秒。
>
> 总结：该配置文件定义了一个 Istio VirtualService 资源，为在服务网格内名为"`reviews`"的服务，将所有流量路由至"`v2`"版本，并设置了`1`秒的请求超时时间。

从 bookinfo 页面上刷新一次

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=ZjU5MTEwYmUzNjc3MGJlNzNjM2MyOGIzMzQ5YTc5MTVfQU14TTM2NTFZY1lqaEpacWtBaUtmUUJyU096a2tEMlhfVG9rZW46Q0dNUmJMdlZZb2J4SkZ4c2RsWmNtdUt3bmZiXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

查看日志看是否有两次重试

```Bash
kubectl logs -f ratings-v1-xxxxx -c istio-proxy
[2024-03-21T08:53:38.675Z] "GET /ratings/0 HTTP/1.1" 200 - via_upstream - "-" 0 48 1 0 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:107.0) Gecko/20100101 Firefox/107.0" "7b7596ea-aa34-98f9-9d7a-e17fbd58cdf3" "ratings:9080" "10.244.135.9:9080" inbound|9080|| 127.0.0.6:35157 10.244.135.9:9080 10.244.135.10:50698 outbound_.9080_.v1_.ratings.default.svc.cluster.local default
[2024-03-21T08:53:39.688Z] "GET /ratings/0 HTTP/1.1" 200 - via_upstream - "-" 0 48 1 0 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:107.0) Gecko/20100101 Firefox/107.0" "7b7596ea-aa34-98f9-9d7a-e17fbd58cdf3" "ratings:9080" "10.244.135.9:9080" inbound|9080|| 127.0.0.6:58597 10.244.135.9:9080 10.244.135.10:49704 outbound_.9080_.v1_.ratings.default.svc.cluster.local default
```

> 注意观察日志中的两个条目的`path`和`start_time`
>
> 这是两条 Istio Envoy 代理的访问日志，记录了服务之间 HTTP 请求和响应的信息。每条记录包含了许多字段，分别代表请求处理中的不同信息，我会逐一解释：
>
> 1. 日期和时间：日志发生的时间，第一条记录的时间是"`2024-03-21T08:53:38.675Z`"，第二条记录是"`2024-03-21T08:53:39.688Z`"。
> 2. 请求方法和 URL："GET /ratings/0 HTTP/1.1"表示使用 HTTP/1.1 协议发送一个 GET 请求到"/ratings/0"这个 URL。
> 3. 响应状态码：200 表示请求成功，"-"表示没有额外的状态标志。
> 4. 是否通过上游服务："via_upstream"表示请求是经过上游服务处理的。
> 5. 后续的"-"表示该字段没有额外信息。
> 6. 请求的字节数：0 表示请求头中没有"Content-Length"字段。
> 7. 响应的字节数：48 表示响应的正文部分包含 48 个字节。
> 8. 请求持续时间：1 表示请求处理耗时 1 毫秒。
> 9. 后续的"0"和"-"表示该字段没有额外信息。
> 10. 用户代理："Mozilla/5.0 （Windows NT 10.0； Win64; x64; rv:107.0） Gecko/20100101 Firefox/107.0"表示请求者使用的浏览器和操作系统信息，即 Windows 10 系统上的 Firefox 107.0 版本。
> 11. 请求 ID："7b7596ea-aa34-98f9-9d7a-e17fbd58cdf3"表示该请求的唯一标识符。
> 12. 目标主机："ratings:9080"表示目标服务的主机名和端口号。
> 13. 被请求的主机 IP 和端口："10.244.135.9：9080"表示处理请求的主机 IP 地址和监听的端口号。
> 14. 其他相关信息，如路由设置、源 IP 地址等。
>
> 总之，这两条日志记录了发送到"ratings"服务 URL "/ratings/0"的两个 GET 请求。请求方法是 HTTP/1.1，请求成功（状态码为 200），请求处理时间为 1 毫秒，请求者使用的浏览器是 Windows 10 系统上的 Firefox 107.0。

根据提供的日志信息，我们可以使用 Mermaid 绘制访问过程。下面是一个示例：

```Plain
sequenceDiagram
    participant Client
    participant Gateway
    participant ProductPage
    participant RatingsService

    Client->>Gateway: GET /ratings/0
    Gateway->>ProductPage: GET /ratings/0
    ProductPage->>RatingsService: GET /ratings/0
    RatingsService-->>ProductPage: 200 OK (ratings data)
    ProductPage-->>Gateway: 200 OK (product page with ratings)
    Gateway-->>Client: 200 OK (product page with ratings)

    Client->>Gateway: GET /ratings/0
    Gateway->>ProductPage: GET /ratings/0
    ProductPage->>RatingsService: GET /ratings/0
    RatingsService-->>ProductPage: 200 OK (ratings data)
    ProductPage-->>Gateway: 200 OK (product page with ratings)
    Gateway-->>Client: 200 OK (product page with ratings)
```

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=MzI2M2YzNTNhY2QyOWU0YjE3NWJhNjlhYTZjNDI3YmVfTXpYU09qaElsTTY4bnNOaGVEaEJhV1BUODdHQUVud25fVG9rZW46TzNnZ2JDTmd6bzA2aHR4d1lrc2NPVXJwbldkXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

在这个序列图中：

- `Client` 表示发送请求的客户端（例如浏览器）。
- `Gateway` 表示 Istio 网关，它接收外部请求并将其转发到内部服务。
- `ProductPage` 表示产品页面服务，它需要从 `RatingsService` 获取评分数据。
- `RatingsService` 表示评分服务，它提供评分数据。

根据日志信息，我们可以看到有两个 GET /ratings/0 的请求被发送。每个请求都遵循以下流程：

1. 客户端向网关发送 GET /ratings/0 请求。
2. 网关将请求转发给产品页面服务。
3. 产品页面服务向评分服务发送 GET /ratings/0 请求以获取评分数据。
4. 评分服务响应评分数据。
5. 产品页面服务将评分数据合并到响应中，并将响应发送回网关。
6. 网关将响应发送回客户端。

这个序列图清晰地展示了请求在 Istio 服务网格中的流动过程，以及不同服务之间的交互。

清理

```Bash
kubectl delete -f samples/bookinfo/networking/virtual-service-all-v1.yaml
```

## 2.熔断

部署 httpin 服务

```Bash
kubectl apply -f samples/httpbin/httpbin.yaml
```

在服务的 DestinationRule 中添加熔断设置

```Bash
kubectl apply -f istiolabmanual/circuitbreaking.yaml 
```

查看熔断配置文件

```Bash
nano istiolabmanual/circuitbreaking.yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: httpbin
spec:
  host: httpbin
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 1
      http:
        http1MaxPendingRequests: 1
        maxRequestsPerConnection: 1
    outlierDetection:
      consecutive5xxErrors: 1
      interval: 1s
      baseEjectionTime: 3m
      maxEjectionPercent: 100
```

> 这是一个 Istio 的`DestinationRule`资源配置。Istio 用于管理服务网格中的网络行为。这个特定的资源配置用于定义和调整名为`httpbin`的服务的流量行为。
>
> 资源的各部分如下所述：
>
> - `apiVersion`：指定 Istio API 的版本，本例中为"networking.istio.io/v1alpha3"。
> - `kind`： 指定要创建的资源类型，本例中为`DestinationRule`。
> - `metadata`：设置资源的元数据，如名称等。在这个案例中，`DestinationRule`的名称是" httpbin "。
> - `spec`：资源的具体规格和配置。
>   - `host`： 指定受此规则影响的服务名称，这里是`httpbin`。
>   - `trafficPolicy`： 定义与该服务相关的各种流量策略。
>     - `connectionPool`： 配置用于连接服务的连接池策略。
>       - `tcp`： 针对 TCP 连接的配置。
>         - `maxConnections`： 允许的最大 TCP 连接数，本例中设置为 1。
>       - `http`： 针对 HTTP 连接的配置。
>         - `http1MaxPendingRequests`： 允许的最大待处理的 HTTP1.x 请求，本例中设为 1。
>         - `maxRequestsPerConnection`： 每个 TCP 连接允许的最大 HTTP 请求数，这里设置为 1。
>     - `outlierDetection`： 配置用于检测异常服务实例的策略。
>       - `consecutive5xxErrors`： 连续 5xx 错误数阈值，达到该数目后服务实例会被短暂逐出。这里设置为 1。
>       - `interval`： 异常检测的时间间隔，本例中为 1 秒。
>       - `baseEjectionTime`： 逐出服务实例的基础时间，达到该时间后可能会恢复。这里设置为 3 分钟。
>       - `maxEjectionPercent`： 允许最大逐出实例的百分比，这里设置为 100。
>
> 总之，这个配置定义了一个针对`httpbin`服务的`DestinationRule`，限制其连接池中 TCP 最大连接为 1，HTTP 请求的最大等待数量为 1，每个 TCP 连接上的最大 HTTP 请求数为 1。另外，异常检测策略产生效果时，异常的服务实例迅速被逐出。注意这里的资源限制非常严格，实际生产环境中根据需要进行调整。

以下是一个简化的 Mermaid 图，用于表示熔断器控制循环流程图。

```Plaintext
graph TD
  A(客户端请求) --> B(熔断器)
  B --> C1(连接池配置)
  C1 --> C1a(TCP: 最大连接数 - 1)
  C1 --> C1b(HTTP: 最大挂起请求数 - 1)
  C1 --> C1c(HTTP: 每个连接的最大请求数 - 1)
  B --> D1(异常检测)
  D1 --> D1a(基于错误的Ejection)
  D1a --> E1a1(连续5xx错误 - 1次)
  E1a1 --> f1(熔断开启)
  D1 --> E1b(检测间隔 - 1s)
  D1 --> E1c(基本驱逐时间 - 3m)
  D1 --> E1d(最大驱逐百分比 - 100%)
  B --> F(熔断状态)
  f1 --> F
  F --> G(请求被拒绝)
  F --> H(请求通过)
  G --> I1(熔断结束)
  H --> I2(熔断结束)
  I1 --> J(恢复请求)
  I2 --> J(恢复请求)
  J --> A(客户端请求)
```

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=NWU4MjY5OTBhNDg2NDkxMmIzM2UxZDc2M2FhMzkyMTdfa1RYZ21PVVJuaDFsMzNSNE01WjBqVzlLclBLb2pMUDZfVG9rZW46R1ZjSmJjMDFPb1cxSHZ4M2pNc2NwbkZDbmtjXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=ZjFiZjQ1NzhiY2I4NTQwODQ2MDc0NTJjNjkzZmVhZDlfOURsOVREVTlmWWZQYkZHbzNtMEVrSTZaZ2MxeE1uNTNfVG9rZW46QUxDamJuUGdjb3cxUHd4MndrY2NCUTFSbnZlXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

这个流程图展示了熔断器的控制循环，包括客户端请求、熔断器、连接池设置、异常检测、熔断状态和请求恢复。熔断状态可能处于开启或关闭状态。在开启状态下，请求会被拒绝；在关闭状态下，请求会通过。根据您提供的熔断器配置，触发熔断开启的条件是连续 5xx 错误达到 1 次。熔断器的连接池配置与异常检测中的其他参数也在流程图中标注出来。

安装测试工具

```Bash
kubectl apply -f samples/httpbin/sample-client/fortio-deploy.yaml
```

查看正常访问结果

```Bash
FORTIO_POD=$(kubectl get pods -lapp=fortio -o 'jsonpath={.items[0].metadata.name}')
kubectl exec -it "$FORTIO_POD"  -c fortio -- /usr/bin/fortio load -curl http://httpbin:8000/get
```

> 这段命令的目的是使用`fortio`工具向`httpbin`服务发送一个 HTTP GET 请求，并输出结果。让我们逐行分解这段命令。
>
> - `FORTIO_POD=$(kubectl get pods -lapp=fortio -o 'jsonpath={.items[0].metadata.name}')`
>   - 这一行将`FORTIO_POD`变量设置为运行`fortio`应用的 Kubernetes pod 的名称。该命令通过`-lapp=fortio`标签选择器来查找匹配的 pods，然后使用`-o 'jsonpath={.items[0].metadata.name}'`选项提取第一个匹配的 pod 的名称。
> - `kubectl exec -it "$FORTIO_POD" -c fortio -- /usr/bin/fortio load -curl http://httpbin:8000/get`
>   - 这一行使用`kubectl exec`命令在运行`fortio`应用的 pod 中执行一个命令。`-it`选项表示交互式地运行该命令。`-c fortio`选项表示在`fortio`容器中执行命令。
> - 命令执行的实际内容是：`/usr/bin/fortio load -curl http://httpbin:8000/get`。`fortio`工具位于`/usr/bin/fortio`路径，我们使用`load`子命令向指定的 URL 发送 HTTP 请求。`-curl`选项表示只发送一个请求（而不是持续发送请求以进行性能测试）并以 curl-like 的格式显示输出。`http://httpbin:8000/get`是`httpbin`服务的 URL 和端口，我们要向其发送 GET 请求。
>
> 总结起来，这段命令首先找到运行`fortio`应用的 Kubernetes pod，然后在该 pod 的`fortio`容器中使用`fortio`工具向`httpbin`服务发送一个 HTTP GET 请求，并输出结果。

```Bash
root@node1:~/istio-1.26.0# FORTIO_POD=$(kubectl get pods -lapp=fortio -o 'jsonpath={.items[0].metadata.name}')
root@node1:~/istio-1.26.0# kubectl exec -it "$FORTIO_POD"  -c fortio -- /usr/bin/fortio load -curl http://httpbin:8000/get
HTTP/1.1 200 OK
server: envoy
date: Thu, 01 Dec 2022 09:03:34 GMT
content-type: application/json
content-length: 594
access-control-allow-origin: *
access-control-allow-credentials: true
x-envoy-upstream-service-time: 27

{
  "args": {},
  "headers": {
    "Host": "httpbin:8000",
    "User-Agent": "fortio.org/fortio-1.17.1",
    "X-B3-Parentspanid": "9d922efa9abce5eb",
    "X-B3-Sampled": "1",
    "X-B3-Spanid": "7ac5bf604849d79b",
    "X-B3-Traceid": "8295319ca4c94cc09d922efa9abce5eb",
    "X-Envoy-Attempt-Count": "1",
    "X-Forwarded-Client-Cert": "By=spiffe://cluster.local/ns/default/sa/httpbin;Hash=53b68132bb6f26d82a22d779c2506fb18000d56a56f308d67e6756570004b994;Subject=\"\";URI=spiffe://cluster.local/ns/default/sa/default"
  },
  "origin": "127.0.0.6",
  "url": "http://httpbin:8000/get"
}
```

> 这个输出展示了一个来自 `fortio` 客户端向 `httpbin` 服务发送 HTTP GET 请求的结果。以下是输出中各部分的解释：
>
> - 请求：`kubectl exec -it "$FORTIO_POD" -c fortio -- /usr/bin/fortio load -curl http://httpbin:8000/get`。这个命令在名为 "FORTIO_POD" 的 Pod 中运行 `fortio` 容器，并向 `http://httpbin:8000/get` 发送 HTTP GET 请求。
> - 响应状态行：`HTTP/1.1 200 OK`。这表示请求成功，服务器返回了 200 OK 状态码。
> - 响应头部信息：
>   - server: envoy - 表明响应由 Envoy 代理服务器生成。
>   - date: Thu， 01 Dec 2022 09:03:34 GMT - 响应生成的日期和时间。
>   - content-type: application/json - 表明响应内容类型为 JSON。
>   - content-length: 594 - 响应内容的长度为 594 字节。
>   - access-control-allow-origin: * - 跨域资源共享（CORS）相关的头部，允许任何来源访问此资源。
>   - access-control-allow-credentials: true - 表明响应可以在浏览器中暴露凭据。
>   - x-envoy-upstream-service-time: 27 - Envoy 代理请求上游服务所消耗的时间，单位为毫秒。
> - 响应内容：一个 JSON 对象，包含以下字段：
>   - args: 一个空对象，表示没有传递任何查询参数。
>   - headers: 包含传递给请求的所有头部信息的对象。
>   - origin: 发送请求的来源 IP 地址。
>   - url: 请求的完整 URL。
>
> 这个输出表明，当 `fortio` 客户端向 `httpbin` 服务发送单个 HTTP GET 请求时，请求已成功完成并返回了预期的响应。在这个实例中，可以了解请求和响应的详细信息，以验证服务是否按预期工作或调试潜在问题。

触发熔断 2 个并发，执行 20 次

```Bash
kubectl exec -it "$FORTIO_POD"  -c fortio -- /usr/bin/fortio load -c 2 -qps 0 -n 20 -loglevel Warning http://httpbin:8000/get
```

> 这个命令与先前的命令相比，主要区别在于并发数、请求数量和日志记录级别。这个命令将以最大速度并发发送 2 个请求，共 20 个请求，同时日志记录级别设为 "Warning"。

```Bash
...
Sockets used: 11 (for perfect keepalive, would be 2)
Jitter: false
Code 200 : 10 (50.0 %)
Code 503 : 10 (50.0 %)
Response Header Sizes : count 20 avg 115.25 +/- 115.3 min 0 max 231 sum 2305
Response Body/Total Sizes : count 20 avg 532.75 +/- 291.8 min 241 max 825 sum 10655
All done 20 calls (plus 0 warmup) 10.408 ms avg, 185.8 qps
```

> 这是使用 `fortio` 命令发送请求时得到的统计输出。我们可以从以下几个方面对输出进行分析：
>
> - **Sockets used**：用于发送请求的套接字数量为 11 个。在理想的 keepalive 状态下，只需要 2 个套接字。这里使用了更多的套接字，可能因为请求是分批进行的，或者部分连接没有被完全重用。
> - **Jitter**：抖动为 false，表示在同一时间发送请求时没有延迟差异。
> - **响应代码**：
>   - Code 200：成功的响应数为 10 个，占比 50.0%。
>   - Code 503：服务不可用的响应数为 10 个，占比 50.0%。
> - 这些结果表明，一半的请求已正常处理，另一半遇到了服务不可用的问题，可能是目标服务器接收请求的能力有限，或者负载均衡器将部分流量引向了异常状态的实例。
> - **Response Header Sizes**: 
>   - count: 20 次响应。
>   - avg: 平均头部大小为 115.25 字节。
>   - +/- 115.3： 平均头部大小的标准差。
>   - min: 最小头部大小为 0 字节。
>   - max: 最大头部大小为 231 字节。
>   - sum: 总计头部大小为 2305 字节。
> - 这部分统计信息显示了所收到的响应头部大小的分布。
> - **Response Body/Total Sizes**: 
>   - count: 20 次响应。
>   - avg: 平均响应正文大小为 532.75 字节，包含响应头部和正文的总大小。
>   - +/- 291.8： 平均大小的标准差。
>   - min: 最小正文大小为 241 字节。
>   - max: 最大正文大小为 825 字节。
>   - sum: 总计正文大小为 10655 字节。
> - 这部分统计信息显示了收到的响应正文以及包含响应头部和正文的总大小的分布。
> - **All done**：总共发送了 20 个请求（plus 0 warmup，没有进行预热请求）。
>   - 平均每个请求耗时为 10.408 毫秒。
>   - 请求发送速率为 185.8 qps（每秒请求数）。
>
> 这些统计数据帮助我们了解请求在目标服务器上的执行情况，以及系统的延迟和吞吐量。根据这些信息，我们可以深入探究服务的性能表现，优化请求配置并解决潜在问题。

触发熔断 again 3 个并发，执行 30 次

```Bash
kubectl exec -it "$FORTIO_POD"  -c fortio -- /usr/bin/fortio load -c 3 -qps 0 -n 30 -loglevel Warning http://httpbin:8000/get
```

> 这个命令的核心区别在于并发数从 2 增加到了 3，同时请求数量从 20 增加到了 30。这意味着这个命令将以最大速度并发发送 3 个请求，共计 30 个请求。日志记录级别依然设为 "Warning"。

```Bash
...
Sockets used: 23 (for perfect keepalive, would be 3)
Jitter: false
Code 200 : 8 (26.7 %)
Code 503 : 22 (73.3 %)
Response Header Sizes : count 30 avg 61.5 +/- 102 min 0 max 231 sum 1845
Response Body/Total Sizes : count 30 avg 396.63333 +/- 258.1 min 241 max 825 sum 11899
All done 30 calls (plus 0 warmup) 5.330 ms avg, 452.0 qps
```

查看熔断指标

```Bash
kubectl exec "$FORTIO_POD" -c istio-proxy -- pilot-agent request GET stats | grep httpbin | grep pending
root@node1:~/istio-1.26.0# kubectl exec "$FORTIO_POD" -c istio-proxy -- pilot-agent request GET stats | grep httpbin | grep pending
cluster.outbound|8000||httpbin.default.svc.cluster.local.circuit_breakers.default.remaining_pending: 1
cluster.outbound|8000||httpbin.default.svc.cluster.local.circuit_breakers.default.rq_pending_open: 0
cluster.outbound|8000||httpbin.default.svc.cluster.local.circuit_breakers.high.rq_pending_open: 0
cluster.outbound|8000||httpbin.default.svc.cluster.local.upstream_rq_pending_active: 0
cluster.outbound|8000||httpbin.default.svc.cluster.local.upstream_rq_pending_failure_eject: 0
cluster.outbound|8000||httpbin.default.svc.cluster.local.upstream_rq_pending_overflow: 42
cluster.outbound|8000||httpbin.default.svc.cluster.local.upstream_rq_pending_total: 29
```

> `overflow`即是被熔断的访问次数
>
> 这个输出是通过在 Fortio Pod 的 Istio-Proxy 容器上执行 `pilot-agent request GET stats` 命令来收集有关 `httpbin` 服务的 Envoy 代理统计信息。然后使用 `grep` 命令过滤特定的统计数据，关注于 httpbin 服务的 pending 请求。根据输出中的统计信息，可以总结出以下关于熔断效果的观察结果：
>
> 1. 在 `httpbin.default.svc.cluster.local` 服务上，熔断器成功阻止了一些请求进入系统。可以从 `upstream_rq_pending_overflow` 的值（42）看出，有 42 个请求因为达到最大挂起请求阈值而未能进入系统。
> 2. 至于剩余挂起的请求数量（`remaining_pending`）为 1，这意味着当前允许挂起的请求仅剩下 1 个。
> 3. 关于 `rq_pending_open` 的值为 0，表示没有触发熔断开关的情况，这意味着熔断器仍然处于关闭状态。
> 4. 对于 `upstream_rq_pending_active` 的值为 0，意味着目前系统内没有积压的挂起请求。
> 5. 关于 `upstream_rq_pending_failure_eject` 的值为 0，表示没有触发失败的请求。
> 6. 最后，`upstream_rq_pending_total` 的值为 29，这表示总共有 29 个请求处于挂起状态。
>
> 总体来说，熔断器对 `httpbin.default.svc.cluster.local` 服务产生了作用，有限制地允许请求进入系统，并防止系统过载。在这种配置下，熔断器成功地保护了服务，确保不会因为达到最大负载而崩溃。

清理

```Bash
kubectl delete destinationrule httpbin
kubectl delete deploy httpbin fortio-deploy
kubectl delete svc httpbin fortio
```

 

# Lab 5 调试

## 1.故障注入

启用路由策略

```Bash
kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
```

> 分别使用匿名用户和 jason 查看 bookinfo 界面，请大家脑补效果 Jason 同学黑星星 普通群众无星星

注入延时故障

```Bash
kubectl apply -f samples/bookinfo/networking/virtual-service-ratings-test-delay.yaml
```

查看延时故障配置文件

```Bash
nano samples/bookinfo/networking/virtual-service-ratings-test-delay.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    fault:
      delay:
        percentage:
          value: 100.0
        fixedDelay: 7s
    route:
    - destination:
        host: ratings
        subset: v1
  - route:
    - destination:
        host: ratings
        subset: v1
```

> 这是一个 Istio VirtualService 配置文件，用于定义“ratings”服务的流量路由规则。让我们一一解析配置文件的内容：
>
> - `apiVersion`： 定义了配置使用的 Istio API 版本，这里为 networking.istio.io/v1alpha3。
> - `kind`： 定义了资源类型，这里是 VirtualService。
> - `metadata` - 包含该资源的元数据：
>   - `name`： 资源名称，这里为 ratings。
> - `spec` - 描述了 VirtualService 的详细配置：
>   - `hosts`： 定义一个或多个主机名，这里是一个单一的主机名“ratings”，表示仅对“ratings”服务应用。
>   - `http`： 定义一系列的 HTTP 配置：
>     - 第一个 `match` 对象用于以下规则：
>       - 如果 HTTP 请求头包含 `end-user` 键，并且其值为`jason`，则应用此规则。
>       - 配置故障注入：`fault` 中，为此类请求添加延迟。`fixedDelay` 设置为 7 秒，`percentage` 设置为 100.0%，表示匹配的请求将会有 100% 的概率经历 7 秒延迟。
>       - `route`： 描述匹配的请求将被发送到的目标服务。这里，目标是“ratings”服务的“v1”子集（即 v1 版本）。
>     - 第二个规则没有指定条件，是默认行为：
>       - `route`： 默认请求将被发送到目标服务。这里，目标是“ratings”服务的“v1”子集（即 v1 版本）。
>
> 综上所述，此配置文件执行以下操作：当 HTTP 请求头中的 `end-user` 为 `jason` 时，为请求注入 7 秒延迟，然后将其路由到“`ratings`”服务的 `v1` 子集；对于所有其他请求，则直接路由到 `v1 `子集，而不注入延迟。

分别使用匿名用户和 jason 查看 bookinfo 界面 Jason 踩坑了

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=YjFlNDAyMzY3MmUxOGZhNjQ3MWEyMDIzMTEyOTNmYjFfcFFpT2VlOVROV1hFMUFDSXhVM2N3OGU5Vk5iU0dyVVVfVG9rZW46WmN4U2JCbjRtb1lOVkV4a0JiTGMzQkN4bnBlXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

普通群众情绪稳定

注入异常中断故障

```Bash
kubectl apply -f samples/bookinfo/networking/virtual-service-ratings-test-abort.yaml
```

查看注入故障配置文件

```Bash
nano samples/bookinfo/networking/virtual-service-ratings-test-abort.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    fault:
      abort:
        percentage:
          value: 100.0
        httpStatus: 500
    route:
    - destination:
        host: ratings
        subset: v1
  - route:
    - destination:
        host: ratings
        subset: v1
```

> 这个 VirtualService 配置文件定义了以下流量路由规则：如果请求头中的 `end-user` 为 `jason`，则注入故障（HTTP 500 错误，100.0% 概率）并将流量路由到 `ratings` 服务的 `v1` 子集；对于其他请求，直接路由到 `ratings` 服务的 `v1` 子集，而不注入故障。

分别使用匿名用户和 jason 查看 bookinfo 界面 Jason 中招

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=ZDhlZDZhZGQxMGIzOTJhMDdiZjk4YWFhNGI1ZGUwMWRfYVNudjJ3eUl2clp0U2tXRkhXaktXVTBrSU1Namd4cWlfVG9rZW46QXZ6c2JOSERhb3RwZHh4ajN6NmNaRWI4bjdBXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

普通群众没事

清理环境

```Bash
kubectl delete -f samples/bookinfo/networking/virtual-service-all-v1.yaml
```

## 2.流量镜像

创建`httpbin-v1` 和 `httpbin-v2`

```Bash
kubectl apply -f istiolabmanual/httpbin-v1.yaml  
kubectl apply -f istiolabmanual/httpbin-v2.yaml 
```

查看 v1 的定义文件

```Bash
nano istiolabmanual/httpbin-v1.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpbin-v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: httpbin
      version: v1
  template:
    metadata:
      labels:
        app: httpbin
        version: v1
    spec:
      containers:
      - image: docker.io/kennethreitz/httpbin
        imagePullPolicy: IfNotPresent
        name: httpbin
        command: ["gunicorn", "--access-logfile", "-", "-b", "0.0.0.0:80", "httpbin:app"]
        ports:
        - containerPort: 80
```

> 这个配置文件定义了一个名为 `httpbin-v1 `的 Deployment 资源，部署一个包含单个容器的 Pod，容器运行 `kennethreitz/httpbin `镜像，使用 `gunicorn `监听在 `80` 端口。标签选择器确保 Deployment 仅管理应用名为 `httpbin`，版本为 `v1 `的 Pod。

发布服务

```Bash
kubectl apply -f istiolabmanual/httpbinsvc.yaml
```

查看服务定义文件

```Bash
nano istiolabmanual/httpbinsvc.yaml
apiVersion: v1
kind: Service
metadata:
  name: httpbin
  labels:
    app: httpbin
spec:
  ports:
  - name: http
    port: 8000
    targetPort: 80
  selector:
    app: httpbin
```

> 这个配置文件定义了一个名为`httpbin`的 Service，它监听`8000`端口并将请求转发到后端标签为'`app: httpbin`'的 Pod 的`80`端口。

启动 sleep 服务

```Bash
kubectl apply -f samples/sleep/sleep.yaml
```

设置路由规则

```Bash
kubectl apply -f istiolabmanual/httpbinvs.yaml 
```

查看路由规则文件

```Bash
nano stiolabmanual/httpbinvs.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: httpbin
spec:
  hosts:
    - httpbin
  http:
  - route:
    - destination:
        host: httpbin
        subset: v1
      weight: 100
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: httpbin
spec:
  host: httpbin
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

> 这个配置文件定义了一个 Istio 的 VirtualService 和 DestinationRule。VirtualService 负责将`100%`的流量路由到`httpbin`服务的`v1`子集。DestinationRule 为`httpbin`服务定义了两个子集，分别是带有不同版本标签的服务实例（`v1`和`v2`）。

使用 sleep 访问服务

```Bash
export SLEEP_POD=$(kubectl get pod -l app=sleep -o jsonpath={.items..metadata.name})
kubectl exec -it $SLEEP_POD -c sleep -- sh -c 'curl  http://httpbin:8000/headers' | python3 -m json.tool
root@node1:~/istio-1.26.0# export SLEEP_POD=$(kubectl get pod -l app=sleep -o jsonpath={.items..metadata.name})
root@node1:~/istio-1.26.0# kubectl exec -it $SLEEP_POD -c sleep -- sh -c 'curl  http://httpbin:8000/headers' | python3 -m json.tool
{
    "headers": {
        "Accept": "*/*",
        "Host": "httpbin:8000",
        "User-Agent": "curl/7.81.0-DEV",
        "X-B3-Parentspanid": "9d5d287f47ecd220",
        "X-B3-Sampled": "1",
        "X-B3-Spanid": "d6be1d30ff475d60",
        "X-B3-Traceid": "7599f42a01151eab9d5d287f47ecd220",
        "X-Envoy-Attempt-Count": "1",
        "X-Forwarded-Client-Cert": "By=spiffe://cluster.local/ns/default/sa/default;Hash=0eba4506484820359556030d829573051b0f525ef9b6205fa5d88eed040e131b;Subject=\"\";URI=spiffe://cluster.local/ns/default/sa/sleep"
    }
}
```

> 输出展示了'httpbin'服务对'/headers'请求的响应结果。主要包括以下 HTTP 头：
>
> - Accept：接受的内容类型，这里是`*/*`，表示任何类型。
> - Host：目标主机和端口，这里是`httpbin:8000`。
> - User-Agent：发出请求的用户代理，这里是`curl/7.81.0-DEV`。
> - X-B3-*：Zipkin 分布式追踪系统用于追踪请求的标头。（如 X-B3-Parentspanid，X-B3-Sampled，X-B3-Spanid，X-B3-Traceid）
> - X-Envoy-Attempt-Count：请求已经尝试了几次，默认为 1。
> - X-Forwarded-Client-Cert：包含有关用于双向 TLS 的客户端证书身份信息的标头。
>
> 从输出中可以看到，请求成功到达'`httpbin`'服务，并通过 Istio 组件进行了透明处理，因为 X-B3-* 和 X-Envoy-Attempt-Count 等 Envoy 特定的标头已经添加到了响应中。

查看 v1 和 v2 的日志

```Bash
export V1_POD=$(kubectl get pod -l app=httpbin,version=v1 -o jsonpath={.items..metadata.name})
kubectl logs -f $V1_POD -c httpbin
root@node1:~/istio-1.26.0# export V1_POD=$(kubectl get pod -l app=httpbin,version=v1 -o jsonpath={.items..metadata.name})
root@node1:~/istio-1.26.0# kubectl logs -f $V1_POD -c httpbin
[2024-03-21 09:16:59 +0000] [1] [INFO] Starting gunicorn 19.9.0
[2024-03-21 09:16:59 +0000] [1] [INFO] Listening at: http://0.0.0.0:80 (1)
[2024-03-21 09:16:59 +0000] [1] [INFO] Using worker: sync
[2024-03-21 09:16:59 +0000] [9] [INFO] Booting worker with pid: 9
127.0.0.6 - - [01/Dec/2022:09:29:11 +0000] "GET /headers HTTP/1.1" 200 527 "-" "curl/7.81.0-DEV"
```

> 这个输出是来自于 Kubernetes 集群中一个名为 `httpbin` 的应用程序的 Pod 的日志。让我们逐步解析这个输出：
>
> - `export V1_POD=$(kubectl get pod -l app=httpbin,version=v1 -o jsonpath={.items..metadata.name})` 这一行命令获取了具有标签 `app=httpbin` 和 `version=v1` 的 Pod 的名称，并将其赋值给环境变量 `V1_POD`。
> - `kubectl logs -f $V1_POD -c httpbin` 这一行命令获取了上一步获取的 Pod 中名为 `httpbin` 的容器的日志，并使用 `-f` 标志持续跟踪日志输出。
> - 接下来的几行是 `httpbin` 容器的启动日志：
>   - `[2024-03-21 09:16:59 +0000] [1] [INFO] Starting gunicorn 19.9.0` 表示正在启动 Gunicorn Web 服务器，版本为 19.9.0。
>   - `[2024-03-21 09:16:59 +0000] [1] [INFO] Listening at: http://0.0.0.0:80 (1)` 表示 Gunicorn 正在监听 0.0.0.0：80 端口，即容器的 80 端口。
>   - `[2024-03-21 09:16:59 +0000] [1] [INFO] Using worker: sync` 表示 Gunicorn 使用的工作模式是 `sync`。
>   - `[2024-03-21 09:16:59 +0000] [9] [INFO] Booting worker with pid: 9` 表示一个工作进程已启动，进程 ID 为 9。
> - 最后一行 `127.0.0.6 - - [01/Dec/2022:09:29:11 +0000] "GET /headers HTTP/1.1" 200 527 "-" "curl/7.81.0-DEV"` 是一条访问日志，表示有一个来自 IP 地址 `127.0.0.6` 的 HTTP 请求访问了 `/headers` 路径，使用的 HTTP 方法是 `GET`。响应状态码是 `200`，响应体长度为 `527` 字节。请求没有 `Referer` 头（表示为 `-`）。请求使用的用户代理是 `curl/7.81.0-DEV`。
>
> 这个输出显示了 `httpbin` 应用程序的 Pod 已经成功启动，并且可以接收 HTTP 请求。最后一行日志记录了一个实际的请求，可以用于调试和监控目的。从这里我们可以看出，之前执行的 curl 命令已经成功地访问了`httpbin`服务的`v1`子集。

```Bash
export V2_POD=$(kubectl get pod -l app=httpbin,version=v2 -o jsonpath={.items..metadata.name})
kubectl logs -f $V2_POD -c httpbin
root@node1:~/istio-1.26.0# export V2_POD=$(kubectl get pod -l app=httpbin,version=v2 -o jsonpath={.items..metadata.name})
root@node1:~/istio-1.26.0# kubectl logs -f $V2_POD -c httpbin
[2024-03-21 09:17:18 +0000] [1] [INFO] Starting gunicorn 19.9.0
[2024-03-21 09:17:18 +0000] [1] [INFO] Listening at: http://0.0.0.0:80 (1)
[2024-03-21 09:17:18 +0000] [1] [INFO] Using worker: sync
[2024-03-21 09:17:18 +0000] [10] [INFO] Booting worker with pid: 10
```

> 在这个输出中，缺少最后一行访问日志 `127.0.0.6 - - [01/Dec/2022:09:29:11 +0000] "GET /headers HTTP/1.1" 200 527 "-" "curl/7.81.0-DEV"`。这是因为在查看新 Pod 的日志时，还没有收到任何 HTTP 请求。

设置镜像规则

```Bash
kubectl apply -f istiolabmanual/mirror.yaml 
```

查看镜像规则文件

```Bash
nano istiolabmanual/mirror.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: httpbin
spec:
  hosts:
    - httpbin
  http:
  - route:
    - destination:
        host: httpbin
        subset: v1
      weight: 100
    mirror:
      host: httpbin
      subset: v2
    mirrorPercentage:
      value: 100
```

> 这个配置文件定义了一个 Istio VirtualService 资源，用于控制对 httpbin 服务的流量路由。让我们逐步解释每个部分：
>
> - `apiVersion` 和 `kind` 分别指定了资源的 API 版本和类型。
> - `metadata.name` 指定了 VirtualService 的名称为 "httpbin"。
> - `spec.hosts` 列出了应用此 VirtualService 的主机名，在这种情况下是 "httpbin"。
> - `spec.http` 部分定义了应用于 HTTP 流量的规则。
> - `spec.http[0].route` 定义了主要的路由规则。
>   - `route[0].destination.host` 指定目标服务的主机名为 "httpbin"。
>   - `route[0].destination.subset` 指定目标服务的子集为 "v1"。
>   - `route[0].weight` 设置为 100，表示将所有入站流量路由到 "v1" 子集。
> - `spec.http[0].mirror` 定义了流量镜像的规则。
>   - `mirror.host` 指定镜像流量的目标主机为 "httpbin"。
>   - `mirror.subset` 指定镜像流量的目标子集为 "v2"。
> - `spec.http[0].mirrorPercentage` 指定了需要镜像的流量百分比。
>   - `mirrorPercentage.value` 设置为 100，表示镜像所有流量。
>
> 这个 VirtualService 配置将所有流量路由到 httpbin 服务的 `v1` 子集同时将 100% 的流量镜像到` v2` 子集。这种设置通常用于逐步向新版本迁移流量，同时监控新版本的行为。

使用 sleep 访问服务

```Bash
export SLEEP_POD=$(kubectl get pod -l app=sleep -o jsonpath={.items..metadata.name})
kubectl exec -it $SLEEP_POD -c sleep -- sh -c 'curl  http://httpbin:8000/headers' | python3 -m json.tool
root@node1:~/istio-1.26.0# export SLEEP_POD=$(kubectl get pod -l app=sleep -o jsonpath={.items..metadata.name})
root@node1:~/istio-1.26.0# kubectl exec -it $SLEEP_POD -c sleep -- sh -c 'curl  http://httpbin:8000/headers' | python3 -m json.tool
{
    "headers": {
        "Accept": "*/*",
        "Host": "httpbin:8000",
        "User-Agent": "curl/7.81.0-DEV",
        "X-B3-Parentspanid": "f5000d3241a11cbb",
        "X-B3-Sampled": "1",
        "X-B3-Spanid": "b0a32f1cbca43d0d",
        "X-B3-Traceid": "0bf35512fb2b57abf5000d3241a11cbb",
        "X-Envoy-Attempt-Count": "1",
        "X-Forwarded-Client-Cert": "By=spiffe://cluster.local/ns/default/sa/default;Hash=0eba4506484820359556030d829573051b0f525ef9b6205fa5d88eed040e131b;Subject=\"\";URI=spiffe://cluster.local/ns/default/sa/sleep"
    }
}
```

查看 v1 和 v2 的日志

```Bash
export V1_POD=$(kubectl get pod -l app=httpbin,version=v1 -o jsonpath={.items..metadata.name})
kubectl logs -f $V1_POD -c httpbin
root@node1:~/istio-1.26.0# export V1_POD=$(kubectl get pod -l app=httpbin,version=v1 -o jsonpath={.items..metadata.name})
root@node1:~/istio-1.26.0# kubectl logs -f $V1_POD -c httpbin
[2024-03-21 09:16:59 +0000] [1] [INFO] Starting gunicorn 19.9.0
[2024-03-21 09:16:59 +0000] [1] [INFO] Listening at: http://0.0.0.0:80 (1)
[2024-03-21 09:16:59 +0000] [1] [INFO] Using worker: sync
[2024-03-21 09:16:59 +0000] [9] [INFO] Booting worker with pid: 9
127.0.0.6 - - [01/Dec/2022:09:29:11 +0000] "GET /headers HTTP/1.1" 200 527 "-" "curl/7.81.0-DEV"
127.0.0.6 - - [01/Dec/2022:09:32:03 +0000] "GET /headers HTTP/1.1" 200 527 "-" "curl/7.81.0-DEV"
export V2_POD=$(kubectl get pod -l app=httpbin,version=v2 -o jsonpath={.items..metadata.name})
kubectl logs -f $V2_POD -c httpbin
root@node1:~/istio-1.26.0# export V2_POD=$(kubectl get pod -l app=httpbin,version=v2 -o jsonpath={.items..metadata.name})
root@node1:~/istio-1.26.0# kubectl logs -f $V2_POD -c httpbin
[2024-03-21 09:17:18 +0000] [1] [INFO] Starting gunicorn 19.9.0
[2024-03-21 09:17:18 +0000] [1] [INFO] Listening at: http://0.0.0.0:80 (1)
[2024-03-21 09:17:18 +0000] [1] [INFO] Using worker: sync
[2024-03-21 09:17:18 +0000] [10] [INFO] Booting worker with pid: 10
127.0.0.6 - - [01/Dec/2022:09:32:03 +0000] "GET /headers HTTP/1.1" 200 567 "-" "curl/7.81.0-DEV"
```

清理

```Bash
kubectl delete virtualservice httpbin
kubectl delete destinationrule httpbin
kubectl delete deploy httpbin-v1 httpbin-v2 sleep
kubectl delete svc httpbin
```

# Lab 6 验证：配置 TLS 安全网关 

模拟网格内的应用和外部应用之间的安全访问，证书管理需要一定的手动。

## 1.为单 Host 配置 TLS ingress gateway

创建根证书和为证书签名的私钥

```Bash
openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -subj '/O=example Inc./CN=example.com' -keyout example.com.key -out example.com.crt
```

> 这是一个使用`openssl`命令生成自签名 SSL 证书的示例。下面简要解释这个命令的各个部分：
>
> - `openssl req`：`openssl`是一个强大的安全工具，`req`是用于创建和处理证书签名请求（CSR）的命令。
> - `-x509`：用于生成自签名的 X.509 证书，而不是生成证书签名请求（CSR）。
> - `-sha256`：指定使用 SHA-256 哈希算法。
> - `-nodes`：这意味着私钥不用密码进行加密。如果省略此选项，会提示您输入密码来保护私钥。
> - `-days 365`：证书的有效期是 365 天。
> - `-newkey rsa:2048`：要为证书和私钥创建一个新的 RSA 密钥对，密钥长度为 2048 位。
> - `-subj '/O=example Inc./CN=example.com'`：设置证书的主题字段。其中，`O`代表组织名称，`CN`代表通用名称（即域名）。
> - `-keyout example.com.key`：将生成的私钥存储到名为`example.com.key`的文件中。
> - `-out example.com.crt`：将生成的自签名证书存储到名为`example.com.crt`的文件中。
>
> 通过运行这个命令，您将在当前目录下创建两个文件：`example.com.key`（私钥）和`example.com.crt`（自签名证书）。这些文件可以用于部署到 Web 服务器，以便支持 HTTPS 连接。但是，自签名证书在浏览器中通常会引发安全警告，因为它们没有受到可信认证机构（CA）的签署。在生产环境中，建议使用由可信任的认证机构签署的证书。

为 httpbin.example.com 创建证书和私钥：

```Bash
openssl req -out httpbin.example.com.csr -newkey rsa:2048 -nodes -keyout httpbin.example.com.key -subj "/CN=httpbin.example.com/O=httpbin organization"
openssl x509 -req -sha256 -days 365 -CA example.com.crt -CAkey example.com.key -set_serial 0 -in httpbin.example.com.csr -out httpbin.example.com.crt
```

> 这些命令用于为`httpbin.example.com`域名创建一个 SSL 证书，并用先前生成的自签名根证书（`example.com.key`和`example.com.crt`）对其进行签名。以下是对这两个命令的逐项解释：
>
> 第一个命令：
>
> - `openssl req`：`openssl`是一个强大的安全工具，`req`是用于创建和处理证书签名请求（CSR）的命令。
> - `-out httpbin.example.com.csr`：将生成的证书签名请求存储到名为`httpbin.example.com.csr`的文件中。
> - `-newkey rsa:2048`：要为证书和私钥创建一个新的 RSA 密钥对，密钥长度为 2048 位。
> - `-nodes`：这表示私钥不进行密码保护。如果省略此选项，会提示您输入密码以保护私钥。
> - `-keyout httpbin.example.com.key`：将生成的私钥存储在名为`httpbin.example.com.key`的文件中。
> - `-subj "/CN=httpbin.example.com/O=httpbin organization"`：设置证书的主题字段。其中，`CN`代表通用名称（即域名），`O`代表组织名称。
>
> 第二个命令：
>
> - `openssl x509`：命令用于处理 x509 证书。
> - `-req`：表示下一个输入文件是一个证书签名请求（CSR）。
> - `-sha256`：指定使用 SHA-256 哈希算法。
> - `-days 365`：证书的有效期为 365 天。
> - `-CA example.com.crt`：使用名为`example.com.crt`的自签名根证书对 CSR 进行签名。
> - `-CAkey example.com.key`：指定与根证书关联的私钥文件`example.com.key`。
> - `-set_serial 0`：为新签名的证书设置序列号。在这种情况下是 0，但建议使用唯一的序列号。
> - `-in httpbin.example.com.csr`：从名为`httpbin.example.com.csr`的 CSR 文件读取。
> - `-out httpbin.example.com.crt`：将生成的已签名证书存储在名为`httpbin.example.com.crt`的文件中。
>
> 运行这两个命令后，您将创建以下文件：
>
> - `httpbin.example.com.key`：私钥
> - `httpbin.example.com.csr`：证书签名请求
> - `httpbin.example.com.crt`：已签名证书
>
> 这些文件可部署到 Web 服务器以支持 HTTPS 连接（使用自签名根证书对证书进行签名）。然而，在生产环境中，建议使用经可信认证机构（CA）签署的证书，以避免浏览器安全警告。

启动 httpbin 用例

```Bash
kubectl apply -f istiolabmanual/sdshttpbin.yaml 
```

为 ingress gateway 创建 secret

```Bash
kubectl create -n istio-system secret tls httpbin-credential --key=httpbin.example.com.key --cert=httpbin.example.com.crt
```

> 这个命令行使用 `kubectl`（Kubernetes 命令行工具）在特定的命名空间（`istio-system`）下创建一个新的 TLS secret。它用于将您已经创建的 SSL 证书（httpbin.example.com。crt）和私钥（httpbin.example.com。key）导入 Kubernetes 集群。下面我们来详细解释这个命令的各个部分。
>
> - `kubectl`： Kubernetes 命令行工具，用于与集群进行交互。
> - `create`： 创建一个新的 Kubernetes 资源。
> - `-n istio-system`： 指定要在其中创建 TLS secret 的命名空间。在这里，命名空间是`istio-system`。
> - `secret`：指定要创建的 Kubernetes 资源类型，即 secret（密钥）。
> - `tls`：指定创建的是一个 TLS 类型的 secret，用于包含 SSL 证书和私钥。
> - `httpbin-credential`：这是您将创建的 TLS secret 的名称。可以根据需要为它们指定名称，但是这里我们已经给出了一个示例名称。
> - `--key=httpbin.example.com.key`：指定私钥文件。这里是先前生成的 httpbin.example.com。key 文件。
> - `--cert=httpbin.example.com.crt`：指定证书文件。这里是先前生成的 httpbin.example.com。crt 文件。
>
> 当您运行这个命令时，Kubernetes 集群将在`istio-system`命名空间下创建一个名为`httpbin-credential`的 secret，然后您可以将其用于在 Kubernetes 上部署带有 SSL 的服务。需要注意的是，在使用任何与 TLS 相关的 Kubernetes 资源之前，请确保您已通过 OpenSSL 或其他受信任的认证机构生成了相应的证书和私钥。

创建 Gateway ，可以打开 yaml 文件重点关注 servers 以及 TLS 部分的定义

```Bash
kubectl apply -f istiolabmanual/sdsgateway.yaml
```

查看 gateway 配置文件

```Bash
nano istiolabmanual/sdsgateway.yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: mygateway
spec:
  selector:
    istio: ingressgateway # use istio default ingress gateway
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: httpbin-credential # must be the same as secret
    hosts:
    - httpbin.example.com
```

> 这是一个 Istio Gateway 资源的 YAML 配置文件。Gateway 用于配置入口流量网关，以便处理进入 Istio 服务网格的流量。下面我们分别解释此配置文件的各个部分：
>
> - `apiVersion: networking.istio.io/v1alpha3`： 定义了我们正在使用的 Istio API 版本，这里使用的是 v1alpha3 版本。
> - `kind: Gateway`： 表示我们要创建的 Kubernetes 资源类型是一个 Istio Gateway。
> - `metadata:`：包含有关 Kubernetes 资源的元数据。
>   - `name: mygateway`： 指定 Gateway 资源的名称，这里使用了名称`mygateway`。
> - `spec:`：包含 Gateway 资源的规范定义。
>   - `selector:`：用于指定此 Gateway 应用于哪些工作负载。在这里，我们选择 Istio 的默认 Ingress 网关。
>     - `istio: ingressgateway`： 使用 Istio 默认的 ingressgateway。
>   - `servers:`： 在 Gateway 中定义了一个或多个服务器。
>     - `- port:`： 配置服务器监听的端口。
>       - `number: 443`： 监听的端口号，这里实例使用了 HTTPS 的默认端口 443。
>       - `name: https`： 给端口一个名称。在这里，我们使用`https`作为名称。
>       - `protocol: HTTPS`： 指定服务器使用的协议。这里设置为 HTTPS。
>     - `tls:`： 配置 TLS，定义 HTTPS 的 TLS 设置。
>       - `mode: SIMPLE`： 设置 TLS 模式为 SIMPLE，即一个普通的单向 TLS，客户端与服务器之间建立加密通信。
>       - `credentialName: httpbin-credential`： 指定要使用的 Kubernetes secret 的名称，其中包含证书和私钥。这里的名称`httpbin-credential`必须与先前创建的 secret 名称相同。
>     - `hosts:`：指定监听哪些主机名。可以使用特定的域名或通配符匹配多个域名。
>       - `- httpbin.example.com`： 在这个示例中，Gateway 配置为使用`httpbin.example.com`这个域名。
>
> 此配置文件创建了一个名为`mygateway`的 Istio Gateway，该 Gateway 将处理到`httpbin.example.com`的 HTTPS 流量。流量通过 Istio 默认的 ingressgateway，在端口 443 上监听，启用具有普通模式的 TLS，并使用`httpbin-credential`secret 下载证书和私钥。

配置网关的 ingress traffic routes 定义相应的虚拟服务

```Bash
kubectl apply -f istiolabmanual/sdsvirtualserver.yaml 
```

查看 virtual server 配置文件

```Bash
nano istiolabmanual/sdsvirtualserver.yaml 
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: httpbin
spec:
  hosts:
  - "httpbin.example.com"
  gateways:
  - mygateway
  http:
  - match:
    - uri:
        prefix: /status
    - uri:
        prefix: /delay
    route:
    - destination:
        port:
          number: 8000
        host: httpbin
```

> 此配置定义了一个 VirtualService，用于将 httpbin.example.com 的流量路由至特定路径（/status 和/delay）的 httpbin 服务实例上。这个资源必须与一个 Gateway 资源（在本例中是 "mygateway"）协同工作，才能正确地处理入口流量。

设置 ingress 主机和端口变量

```Bash
export INGRESS_HOST=$(kubectl get po -l istio=ingressgateway -n istio-system -o jsonpath='{.items[0].status.hostIP}')
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
export TCP_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="tcp")].nodePort}')
```

> 这些命令提取了与 Istio Ingress Gateway 相关的信息，包括 Host IP 地址，HTTP2 端口，HTTPS 端口和 TCP 端口。这些值将存储在相应的环境变量中，以便在后续的操作中使用。

发送 HTTPS 请求访问 httpbin 服务

```Bash
curl -v -HHost:httpbin.example.com --resolve "httpbin.example.com:$SECURE_INGRESS_PORT:$INGRESS_HOST" \
--cacert example.com.crt "https://httpbin.example.com:$SECURE_INGRESS_PORT/status/418"
```

 此处应该有茶壶

```Bash
root@node1:~/istio-1.26.0# curl -v -HHost:httpbin.example.com --resolve "httpbin.example.com:$SECURE_INGRESS_PORT:$INGRESS_HOST" \
> --cacert example.com.crt "https://httpbin.example.com:$SECURE_INGRESS_PORT/status/418"
* Added httpbin.example.com:32696:192.168.1.233 to DNS cache
* Hostname httpbin.example.com was found in DNS cache
*   Trying 192.168.1.233:32696...
* TCP_NODELAY set
* Connected to httpbin.example.com (192.168.1.233) port 32696 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: example.com.crt
  CApath: /etc/ssl/certs
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* ALPN, server accepted to use h2
* Server certificate:
*  subject: CN=httpbin.example.com; O=httpbin organization
*  start date: Dec  2 01:06:59 2022 GMT
*  expire date: Dec  2 01:06:59 2023 GMT
*  common name: httpbin.example.com (matched)
*  issuer: O=example Inc.; CN=example.com
*  SSL certificate verify ok.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x562ada683e30)
> GET /status/418 HTTP/2
> Host:httpbin.example.com
> user-agent: curl/7.68.0
> accept: */*
>
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* old SSL session ID is stale, removing
* Connection state changed (MAX_CONCURRENT_STREAMS == 2147483647)!
< HTTP/2 418
< server: istio-envoy
< date: Fri, 02 Dec 2022 01:13:11 GMT
< access-control-allow-credentials: true
< x-more-info: http://tools.ietf.org/html/rfc2324
< access-control-allow-origin: *
< content-length: 135
< x-envoy-upstream-service-time: 18
<

    -=[ teapot ]=-

       _...._
     .'  _ _ `.
    | ."` ^ `". _,
    \_;`"---"`|//
      |       ;/
      \_     _/
        `"""`
* Connection #0 to host httpbin.example.com left intact
```

> 这个命令行是用`curl`工具向 Istio Ingress Gateway 发起一个使用自定义主机头（Host: httpbin.example.com）的 HTTP 请求。请求的目标是 https 协议下的/status/418 端点。由于 Istio 使用 TLS，所以需要提供一个根证书文件（example.com。crt）来验证服务器证书。
>
> 在命令行中：
>
> 1. `-v`参数表示输出详细的调试信息。
> 2. `-HHost:httpbin.example.com`设置了一个自定义的主机头（Host header）。
> 3. `--resolve "httpbin.example.com:$SECURE_INGRESS_PORT:$INGRESS_HOST"`参数是告诉 curl 使用指定的 IP 地址和端口号，而不是通过 DNS 解析 httpbin.example.com。
> 4. `--cacert`参数提供了根证书文件（example.com。crt），以正确验证服务器证书。
>
> 输出中，首先显示了请求相关的信息，例如 DNS 解析、连接建立、TLS 握手、协议协商等。然后在输出中看到发送了一个 GET 请求到/status/418 端点，使用 HTTP/2 协议并设置了 Host 头为`httpbin.example.com`。
>
> 服务器响应的状态码为 418，表示"I'm a teapot"。这个状态码是在 RFC 2324（超文本咖啡壶控制协议，HTCPCP）中定义的，用于在服务器不是一个咖啡壶的情况下返回。响应中还包含了一个 ASCII 艺术作品，表现了一个茶壶的形象。
>
> 最后，`* Connection #0 to host httpbin.example.com left intact`表明与服务器的连接完好无损。
>
> 回滚日志查看 TSL 握手过程，在这个 TLS 握手的例子里，客户端（curl 命令）访问了一个通过 Istio Envoy 服务的 HTTP 服务器。这是一个典型的 TLS 1.3 握手过程。根据提供的 curl 输出，我们可以用 Mermaid 表示这个完整的过程。

以下是用 Mermaid 编写的时序图代码及对应步骤的解释：

```Plaintext
sequenceDiagram
    Participant Client
    Participant Server
    Client->>Server: Client Hello (Offering TLSv1.3, ALPN: h2, http/1.1)
    Server->>Client: Server Hello (TLSv1.3, ALPN: h2)
    Server->>Client: Encrypted Extensions
    Server->>Client: Server Certificate
    Server->>Client: CERT Verify (with server's public key)
    Server->>Client: Finished
    Client->>Server: Change Cipher Spec
    Client->>Server: Finished
    Client->>Server: GET /status/418 HTTP/2 (encrypted with negotiated cipher)
    Server->>Client: HTTP/2 418 Response (encrypted with negotiated cipher)
```

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=NWU1NmM2NjgyNDRlNWRmYmM5N2U3ZjkzYjk4Yzg5YTdfSWI1dmpaeHk1YXh5YW80YnVXNnVEd25OYjJ4dGw5dEVfVG9rZW46SHg1UWJncmI4b1pHVGN4ak5QcWNxZlhRbkJjXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

> 步骤解释：
>
> 1. 客户端（Client）发送 "Client Hello" 消息，提供支持的 TLS 版本（在这里是 TLSv1.3），同时携带一个 ALPN（Application-Layer Protocol Negotiation）协议列表（h2 和 http/1.1）。
> 2. 服务器（Server）回应 "Server Hello" 消息，确认采用的 TLS 版本（TLSv1.3）和选择的 ALPN 协议（在这里是 h2）。
> 3. 服务器发送 "Encrypted Extensions" 消息。这是 TLS 1.3 的新特性，将所有握手扩展都置于加密的握手消息中，以提高安全性。
> 4. 服务器提供其数字证书，其中包含服务器的公钥信息。
> 5. 服务器发送 "CERT Verify" 消息（带有服务器的公钥），客户端需要使用这个消息验证服务器的身份。
> 6. 服务器发送 "Finished" 消息，表明其握手部分已完成。
> 7. 客户端发送 "Change Cipher Spec" 消息，告知服务器将切换到协商好的加密套件进行通信。
> 8. 客户端发送 "Finished" 消息，表明其握手部分已完成。
> 9. 客户端使用协商好的加密套件发送加密的请求（GET /status/418 HTTP/2）。
> 10. 服务器使用同样的加密套件发送加密的 HTTP 响应（HTTP/2 418）。
>
> 这个示例展示了典型的 TLSv1.3 握手过程，包括版本协商、证书验证、加密套件选择以及加密通信。在这个过程中，客户端和服务器建立了安全的通信环境。

删除网关的 secret 并创建一个新 secret 以更改 ingress gateway 的凭据

```Bash
kubectl -n istio-system delete secret httpbin-credential
mkdir new_certificates
openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -subj '/O=example Inc./CN=example.com' -keyout new_certificates/example.com.key -out new_certificates/example.com.crt
openssl req -out new_certificates/httpbin.example.com.csr -newkey rsa:2048 -nodes -keyout new_certificates/httpbin.example.com.key -subj "/CN=httpbin.example.com/O=httpbin organization"
openssl x509 -req -sha256 -days 365 -CA new_certificates/example.com.crt -CAkey new_certificates/example.com.key -set_serial 0 -in new_certificates/httpbin.example.com.csr -out new_certificates/httpbin.example.com.crt
kubectl create -n istio-system secret tls httpbin-credential \
--key=new_certificates/httpbin.example.com.key \
--cert=new_certificates/httpbin.example.com.crt
```

使用新证书进行访问

```Bash
curl -v -HHost:httpbin.example.com --resolve "httpbin.example.com:$SECURE_INGRESS_PORT:$INGRESS_HOST" \
--cacert new_certificates/example.com.crt "https://httpbin.example.com:$SECURE_INGRESS_PORT/status/418"
```

 此处还是有茶壶

```Bash
< HTTP/2 418
< server: istio-envoy
< date: Fri, 02 Dec 2022 01:14:48 GMT
< access-control-allow-credentials: true
< x-more-info: http://tools.ietf.org/html/rfc2324
< access-control-allow-origin: *
< content-length: 135
< x-envoy-upstream-service-time: 8
<

    -=[ teapot ]=-

       _...._
     .'  _ _ `.
    | ."` ^ `". _,
    \_;`"---"`|//
      |       ;/
      \_     _/
        `"""`
* Connection #0 to host httpbin.example.com left intact
```

尝试使用旧证书访问

```Bash
curl -v -HHost:httpbin.example.com --resolve "httpbin.example.com:$SECURE_INGRESS_PORT:$INGRESS_HOST" \
--cacert example.com.crt "https://httpbin.example.com:$SECURE_INGRESS_PORT/status/418"
```

 吃瘪

```Bash
root@node1:~/istio-1.26.0# curl -v -HHost:httpbin.example.com --resolve "httpbin.example.com:$SECURE_INGRESS_PORT:$INGRESS_HOST" \
> --cacert example.com.crt "https://httpbin.example.com:$SECURE_INGRESS_PORT/status/418"
* Added httpbin.example.com:32696:192.168.1.233 to DNS cache
* Hostname httpbin.example.com was found in DNS cache
*   Trying 192.168.1.233:32696...
* TCP_NODELAY set
* Connected to httpbin.example.com (192.168.1.233) port 32696 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: example.com.crt
  CApath: /etc/ssl/certs
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (OUT), TLS alert, decrypt error (563):
* error:0407008A:rsa routines:RSA_padding_check_PKCS1_type_1:invalid padding
* Closing connection 0
curl: (35) error:0407008A:rsa routines:RSA_padding_check_PKCS1_type_1:invalid padding
```

> 在这个输出中，我们的 curl 命令试图向 Istio Ingress Gateway（httpbin.example.com）发送一个 HTTPS 请求，访问/status/418 端点，同时提供 Host 头和自定义证书。这是逐步解析各个部分：
>
> - `* Added httpbin.example.com:32696:192.168.1.233 to DNS cache` - Curl 将 httpbin.example.com 及其对应的端口和 IP 地址添加到 DNS 缓存中。
> - `* Hostname httpbin.example.com was found in DNS cache` - Curl 在 DNS 缓存中找到了 httpbin.example.com 域名。
> - `* Trying 192.168.1.233:32696...` - Curl 尝试连接到给定的 IP 地址和端口。
> - `* TCP_NODELAY set` - Curl 设置 TCP_NODELAY，禁用 Nagle 算法以提高传输速度。
> - `* Connected to httpbin.example.com (192.168.1.233) port 32696 (#0)` - Curl 成功地连接到了目标地址。
> - 下面一系列端展示了 curl 与服务器进行 TLS 握手的过程。
>   - `* ALPN, offering h2`
>   - `* ALPN, offering http/1.1`
>   - `* successfully set certificate verify locations:` 等。
> - `* TLSv1.3 (OUT), TLS alert, decrypt error (563):` - 在与服务器完成握手过程的某个时候，TLS 连接遇到了解密错误。
> - `* error:0407008A:rsa routines:RSA_padding_check_PKCS1_type_1:invalid padding` - 连接中出现了一个错误，使用 RSA 填充检查方法时返回了无效的填充错误。这可能是由于证书的错误导致的。
> - `* Closing connection 0` - 由于错误，Curl 关闭了连接 ID 为 0 的连接。
> - `curl: (35) error:0407008A:rsa routines:RSA_padding_check_PKCS1_type_1:invalid padding` - 最后，curl 命令由于错误而退出，并将 RSA 例程错误的详细信息显示给用户。
>
> 总之，在这里的输出中，Curl 试图连接到 Istio Ingress Gateway 并访问指定的端点。但是，在 TLS 握手过程中，遇到了一个无效填充的错误，导致连接无法成功建立。这可能是由于证书错误。要解决此问题，请检查提供的证书是否正确。

## 2.为多 Host 配置 TLS ingress gateway

重新创建 httpbin 凭据

```Bash
kubectl -n istio-system delete secret httpbin-credential
kubectl create -n istio-system secret tls httpbin-credential \
--key=httpbin.example.com.key \
--cert=httpbin.example.com.crt
```

启用 helloworld-v1 样例

```Bash
kubectl apply -f istiolabmanual/helloworld-v1.yaml
```

为 helloworld-v1.example.com 创建证书和私钥：

```Bash
openssl req -out helloworld-v1.example.com.csr -newkey rsa:2048 -nodes -keyout helloworld-v1.example.com.key -subj "/CN=helloworld-v1.example.com/O=helloworld organization"
openssl x509 -req -sha256 -days 365 -CA example.com.crt -CAkey example.com.key -set_serial 1 -in helloworld-v1.example.com.csr -out helloworld-v1.example.com.crt
```

创建 helloworld-credential secret

```Bash
kubectl create -n istio-system secret tls helloworld-credential --key=helloworld-v1.example.com.key --cert=helloworld-v1.example.com.crt
```

创建指向两个服务的 gateway

```Bash
kubectl apply -f istiolabmanual/mygatewayv1.yaml
```

查看 gateway 配置文件

```Bash
nano istiolabmanual/mygatewayv1.yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: mygateway
spec:
  selector:
    istio: ingressgateway # use istio default ingress gateway
  servers:
  - port:
      number: 443
      name: https-httpbin
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: httpbin-credential
    hosts:
    - httpbin.example.com
  - port:
      number: 443
      name: https-helloworld
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: helloworld-credential
    hosts:
    - helloworld-v1.example.com
```

> 这是一个 Istio 的`Gateway`资源配置文件。它指定了如何将外部流量路由到在 Istio 服务网格里的服务。让我解释一下具体的配置内容：
>
> - `apiVersion`： 这是 API 版本，这里使用的是`networking.istio.io/v1alpha3`。
> - `kind`： Kubernetes 资源类型，这里的类型是`Gateway`。
> - `metadata`：资源的配置元数据。
>   - `name`：指定 Gateway 的名称，此处为`mygateway`。
>
> `spec`字段描述了`Gateway`的配置内容：
>
> - `selector`：选择器，定义要使用的 Istio 网关。
>   - `istio: ingressgateway`：这里选择了 Istio 默认的 ingress gateway。
>
> `servers`字段定义了不同的服务器配置。这个示例包括两个服务器配置：
>
> - 第一个服务器（httpbin）:
>   - `port`：端口配置。
>     - `number`： 443 - 使用 HTTPS 协议的默认端口。
>     - `name`： https-httpbin - 为端口命名。
>     - `protocol`： HTTPS - 使用 HTTPS 协议。
>   - `tls`： TLS 配置。
>     - `mode`： SIMPLE - 简单的 TLS 模式，即客户端和 Gateway 之间的单向加密。
>     - `credentialName`： httpbin-credential - TLS 证书的密钥名称。
>   - `hosts`： 路由规则支持的主机名列表。
>     - httpbin.example.com - 当请求的 Host header 匹配此域名时，流量将路由到这个服务器配置。
> - 第二个服务器（helloworld）:
>   - `port`：端口配置。
>     - `number`： 443 - 使用 HTTPS 协议的默认端口。
>     - `name`： https-helloworld - 为端口命名。
>     - `protocol`： HTTPS - 使用 HTTPS 协议。
>   - `tls`： TLS 配置。
>     - `mode`： SIMPLE - 简单的 TLS 模式，即客户端和 Gateway 之间的单向加密。
>     - `credentialName`： helloworld-credential - TLS 证书的密钥名称。
>   - `hosts`： 路由规则支持的主机名列表。
>     - helloworld-v1.example.com - 当请求的 Host header 匹配此域名时，流量将路由到这个服务器配置。
>
> 总之，这个 Gateway 配置定义了两个使用 HTTPS 协议的服务器。其中，一个处理域名为`httpbin.example.com`的流量，并使用`httpbin-credential`证书；另一个处理域名为`helloworld-v1.example.com`的流量，并使用`helloworld-credential`证书。这些服务器都使用`SIMPLE`的 TLS 模式，即客户端和 Gateway 之间的单向加密。

创建 helloworld-v1 的 vs

```Bash
kubectl apply -f istiolabmanual/helloworld-v1vs.yaml
```

查看 virtual server 配置文件

```Bash
nano istiolabmanual/helloworld-v1vs.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: helloworld-v1
spec:
  hosts:
  - helloworld-v1.example.com
  gateways:
  - mygateway
  http:
  - match:
    - uri:
        exact: /hello
    route:
    - destination:
        host: helloworld-v1
        port:
          number: 5000
```

> 该配置针对向`helloworld-v1.example.com`发送的请求，将匹配 URI 为`/hello`的流量，并将这些流量路由到`helloworld-v1`服务的 5000 端口。

向 v1 发起访问

```Bash
curl -v -HHost:helloworld-v1.example.com --resolve "helloworld-v1.example.com:$SECURE_INGRESS_PORT:$INGRESS_HOST" \
--cacert example.com.crt "https://helloworld-v1.example.com:$SECURE_INGRESS_PORT/hello"
```

 此处收获 HTTP/2 200

```YAML
...
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* old SSL session ID is stale, removing
* Connection state changed (MAX_CONCURRENT_STREAMS == 2147483647)!
< HTTP/2 200
< content-type: text/html; charset=utf-8
< content-length: 60
< server: istio-envoy
< date: Fri, 02 Dec 2022 01:20:51 GMT
< x-envoy-upstream-service-time: 113
<
Hello version: v1, instance: helloworld-v1-5f44dd8565-zplmv
* Connection #0 to host helloworld-v1.example.com left intact
```

向 httpbin.example.com 发起访问

```Bash
curl -v -HHost:httpbin.example.com --resolve "httpbin.example.com:$SECURE_INGRESS_PORT:$INGRESS_HOST" \
--cacert example.com.crt "https://httpbin.example.com:$SECURE_INGRESS_PORT/status/418"
```

 此处还是有茶壶

```Bash
< HTTP/2 418
< server: istio-envoy
< date: Fri, 02 Dec 2022 01:21:55 GMT
< access-control-allow-credentials: true
< x-more-info: http://tools.ietf.org/html/rfc2324
< access-control-allow-origin: *
< content-length: 135
< x-envoy-upstream-service-time: 3
<

    -=[ teapot ]=-

       _...._
     .'  _ _ `.
    | ."` ^ `". _,
    \_;`"---"`|//
      |       ;/
      \_     _/
        `"""`
* Connection #0 to host httpbin.example.com left intact
```

## 3.配置交互 TLS ingress gateway

更新证书

```Bash
kubectl -n istio-system delete secret httpbin-credential
kubectl create -n istio-system secret generic httpbin-credential --from-file=tls.key=httpbin.example.com.key \
--from-file=tls.crt=httpbin.example.com.crt --from-file=ca.crt=example.com.crt
```

把 mygateway 切换到`mutual`模式

```Bash
kubectl apply -f istiolabmanual/mygatewayv2.yaml
```

查看 gateway 配置文件

```Bash
nano istiolabmanual/mygatewayv2.yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
 name: mygateway
spec:
 selector:
   istio: ingressgateway # use istio default ingress gateway
 servers:
 - port:
     number: 443
     name: https
     protocol: HTTPS
   tls:
     mode: MUTUAL
     credentialName: httpbin-credential # must be the same as secret
   hosts:
   - httpbin.example.com
```

> 这个配置文件定义了一个 Istio Gateway 资源，它用来配置 Istio 代理（Envoy）以执行基于 L7 协议层（如 HTTP/HTTPS）的流量入口。以下是关于这个具体配置文件的解释：
>
> - `apiVersion: networking.istio.io/v1alpha3`：定义了资源的 API 版本，使用 Istio 网络 API 库中的`v1alpha3`版本。
> - `kind: Gateway`：指定这是一个 Gateway 资源。
> - `metadata`字段包含以下信息：
>   - `name: mygateway`：为这个 Gateway 定义一个名为`mygateway`的具体名称。
> - `spec`字段描述了 Gateway 的配置内容，包括：
>   - `selector`字段定义了应用于此 Gateway 的代理（Envoy）工作负载标签。当前的选择器是：
>     - `istio: ingressgateway`：使用默认的 Istio ingress gateway。
>   - `servers`字段定义了一组服务器配置，每个服务器为一个或多个主机名启用不同协议。当前的 Gateway 有一个服务器配置：
>     - `port`字段定义了服务器运行的端口和协议类型：
>       - `number: 443`：监听端口号为 443，通常用于 HTTPS。
>       - `name: https`：给监听端口命名为“https”。
>       - `protocol: HTTPS`：指定这个服务器用于 HTTPS 流量。
>     - `tls`字段包含 TLS 相关设置，如模式和证书：
>       - `mode: MUTUAL`：指定服务器为双向 TLS（mTLS）模式。要求客户端和服务器端都提供证书以进行相互认证。
>       - `credentialName: httpbin-credential`：指定使用名为`httpbin-credential`的证书与密钥。该名称应与相应的 Kubernetes Secret 名称相同。
>     - `hosts`字段包含此服务器应接受的主机名列表：
>       - `httpbin.example.com`：此服务器应处理发往`httpbin.example.com`的流量。
>
> 总之，这个 Gateway 定义了一个监听 443 端口的 HTTPS 服务器，处理发往`httpbin.example.com`的流量。它使用`MUTUAL`（需要客户端和服务器证书）进行连接，并使用名为`httpbin-credential`的证书与密钥。此外，它使用 Istio 的默认 ingress gateway。

尝试使用之前的方式进行访问

```Bash
curl -v -HHost:httpbin.example.com --resolve "httpbin.example.com:$SECURE_INGRESS_PORT:$INGRESS_HOST" \
--cacert example.com.crt "https://httpbin.example.com:$SECURE_INGRESS_PORT/status/418"
```

 吃瘪

```Bash
root@node1:~/istio-1.26.0# curl -v -HHost:httpbin.example.com --resolve "httpbin.example.com:$SECURE_INGRESS_PORT:$INGRESS_HOST" \
> --cacert example.com.crt "https://httpbin.example.com:$SECURE_INGRESS_PORT/status/418"
* Added httpbin.example.com:31811:192.168.14.233 to DNS cache
* Hostname httpbin.example.com was found in DNS cache
*   Trying 192.168.14.233:31811...
* TCP_NODELAY set
* Connected to httpbin.example.com (192.168.14.233) port 31811 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: example.com.crt
  CApath: /etc/ssl/certs
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, Request CERT (13):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (OUT), TLS handshake, Certificate (11):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* ALPN, server accepted to use h2
* Server certificate:
*  subject: CN=httpbin.example.com; O=httpbin organization
*  start date: Mar 21 07:39:20 2024 GMT
*  expire date: Mar 21 07:39:20 2025 GMT
*  common name: httpbin.example.com (matched)
*  issuer: O=example Inc.; CN=example.com
*  SSL certificate verify ok.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x55873a3bae30)
> GET /status/418 HTTP/2
> Host:httpbin.example.com
> user-agent: curl/7.68.0
> accept: */*
>
* TLSv1.3 (IN), TLS alert, unknown (628):
* OpenSSL SSL_read: error:1409445C:SSL routines:ssl3_read_bytes:tlsv13 alert certificate required, errno 0
* Failed receiving HTTP2 data
* OpenSSL SSL_write: SSL_ERROR_ZERO_RETURN, errno 0
* Failed sending HTTP2 data
* Connection #0 to host httpbin.example.com left intact
curl: (56) OpenSSL SSL_read: error:1409445C:SSL routines:ssl3_read_bytes:tlsv13 alert certificate required, errno 0
```

> 根据输出信息，访问失败的原因是服务器要求客户端提供证书进行双向 TLS 认证，但是客户端（curl）没有提供证书。
>
> 具体分析如下：
>
> - 连接建立成功，进行 TLS 握手：
>
> ```Plain
> * TLSv1.3 (OUT), TLS handshake, Certificate (11):
> * TLSv1.3 (OUT), TLS handshake, Finished (20):
> * SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
> ```
>
> - 服务器验证客户端证书失败，发送 `certificate_required` 警告：
>
> ```Plain
> * TLSv1.3 (IN), TLS alert, unknown (628):
> * OpenSSL SSL_read: error:1409445C:SSL routines:ssl3_read_bytes:tlsv13 alert certificate required, errno 0
> ```
>
> - 客户端没有提供证书，连接被终止：
>
> ```Plain
> * Failed receiving HTTP2 data
> * OpenSSL SSL_write: SSL_ERROR_ZERO_RETURN, errno 0
> * Failed sending HTTP2 data
> ```
>
> 要解决这个问题，需要为客户端（curl）提供一个合法的客户端证书，并在请求时使用`--cert`和`--key`参数指定证书和密钥文件。或者可以尝试不使用客户端证书，看服务器是否允许匿名访问。

生成新的客户端证书和私钥

```Bash
openssl req -out client.example.com.csr -newkey rsa:2048 -nodes -keyout client.example.com.key -subj "/CN=client.example.com/O=client organization"
openssl x509 -req -sha256 -days 365 -CA example.com.crt -CAkey example.com.key -set_serial 1 -in client.example.com.csr -out client.example.com.crt
```

向 httpbin.example.com 发起访问

```Bash
curl -v -HHost:httpbin.example.com --resolve "httpbin.example.com:$SECURE_INGRESS_PORT:$INGRESS_HOST" \
--cacert example.com.crt --cert client.example.com.crt --key client.example.com.key \
"https://httpbin.example.com:$SECURE_INGRESS_PORT/status/418"
```

 此处还是有茶壶

```Bash
root@node1:~/istio-1.26.0# curl -v -HHost:httpbin.example.com --resolve "httpbin.example.com:$SECURE_INGRESS_PORT:$INGRESS_HOST" \
> --cacert example.com.crt --cert client.example.com.crt --key client.example.com.key \
> "https://httpbin.example.com:$SECURE_INGRESS_PORT/status/418"
* Added httpbin.example.com:32696:192.168.1.233 to DNS cache
* Hostname httpbin.example.com was found in DNS cache
*   Trying 192.168.1.233:32696...
* TCP_NODELAY set
* Connected to httpbin.example.com (192.168.1.233) port 32696 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: example.com.crt
  CApath: /etc/ssl/certs
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, Request CERT (13):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (OUT), TLS handshake, Certificate (11):
* TLSv1.3 (OUT), TLS handshake, CERT verify (15):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* ALPN, server accepted to use h2
* Server certificate:
*  subject: CN=httpbin.example.com; O=httpbin organization
*  start date: Dec  2 01:06:59 2022 GMT
*  expire date: Dec  2 01:06:59 2023 GMT
*  common name: httpbin.example.com (matched)
*  issuer: O=example Inc.; CN=example.com
*  SSL certificate verify ok.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x563d03e87e30)
> GET /status/418 HTTP/2
> Host:httpbin.example.com
> user-agent: curl/7.68.0
> accept: */*
>
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* old SSL session ID is stale, removing
* Connection state changed (MAX_CONCURRENT_STREAMS == 2147483647)!
< HTTP/2 418
< server: istio-envoy
< date: Fri, 02 Dec 2022 01:27:09 GMT
< access-control-allow-credentials: true
< x-more-info: http://tools.ietf.org/html/rfc2324
< access-control-allow-origin: *
< content-length: 135
< x-envoy-upstream-service-time: 1
<

    -=[ teapot ]=-

       _...._
     .'  _ _ `.
    | ."` ^ `". _,
    \_;`"---"`|//
      |       ;/
      \_     _/
        `"""`
* Connection #0 to host httpbin.example.com left intact
```

> 在这个输出中，客户端提供了客户端证书和私钥，成功通过了服务器的双向 TLS 认证，最终获得了预期的 HTTP `418` 响应。通过提供合法的客户端证书，客户端成功通过了服务器的双向 TLS 认证，获得了预期的响应。这种双向认证可以增强系统的安全性。

对比此前的握手验证过程，这一次明显增多，我们可以用 Mermaid 时序图描述 TLS 握手过程如下：

```Plaintext
sequenceDiagram
    participant C as Client
    participant S as Server

    C->>S: ClientHello (TLSv1.3)
    S->>C: ServerHello (TLSv1.3)
    S->>C: Encrypted Extensions
    S->>C: Request CERT
    S->>C: 提供服务器证书
    S->>C: CERT verify
    S->>C: Finished
    C->>S: ChangeCipherSpec
    C->>S: 提供客户端证书
    C->>S: CERT verify
    C->>S: Finished
    C->>S: 使用HTTP/2发起请求
    S->>C: HTTP/2 响应 (418)
```

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=MDhiOTIwNmYzMDExM2VmZDA3NjIyYzFkN2I0M2VkOGNfM0g2azUwbEtKd2NXSjhteU9JMGViWU9HS3EwN3VZYVdfVG9rZW46Rk5idmJqektZbzdtYTd4U1V6NWNBUGdzbmhlXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

> 1. 客户端向服务器发送 ClientHello 消息，指明使用 TLSv1.3。
> 2. 服务器回复 ServerHello 消息，确认使用 TLSv1.3。
> 3. 服务器发送 Encrypted Extensions 消息。
> 4. 服务器请求客户端证书（Request CERT 消息）。
> 5. 服务器提供服务器证书。
> 6. 服务器发送 CERT verify 消息，要求客户端验证服务器证书。
> 7. 服务器发送 Finished 消息。
> 8. 客户端发送 ChangeCipherSpec 消息，告知服务器将开始加密所有后续的通信。
> 9. 客户端提供客户端证书。
> 10. 客户端发送 CERT verify 消息，证明客户端证书有效。
> 11. 客户端发送 Finished 消息。
> 12. 之后，客户端使用 HTTP/2 向服务器发起请求。
> 13. 服务器通过 HTTP/2 发回响应（418 代码，表示“我是一个茶壶”）。

清理环境

```Bash
kubectl delete gateway mygateway
kubectl delete virtualservice httpbin
kubectl delete --ignore-not-found=true -n istio-system secret httpbin-credential \
helloworld-credential
kubectl delete --ignore-not-found=true virtualservice helloworld-v1
rm -rf example.com.crt example.com.key httpbin.example.com.crt httpbin.example.com.key httpbin.example.com.csr helloworld-v1.example.com.crt helloworld-v1.example.com.key helloworld-v1.example.com.csr client.example.com.crt client.example.com.csr client.example.com.key ./new_certificates
kubectl delete deployment --ignore-not-found=true httpbin helloworld-v1
kubectl delete service --ignore-not-found=true httpbin helloworld-v1
```

# Lab 7 认证：为应用生成双向 TLS

模拟网格内部的应用通信，证书管理全部自动化。

可选，重新声明 istioctl 路径

```Bash
  export PATH=$PWD/bin:$PATH
```

创建两个名称空间 foo 和 bar，并在它们两个上部署 httpbin 和 sleep 并启用 sidecar 注入：

```Bash
kubectl create ns foo
kubectl apply -f <(istioctl kube-inject -f samples/httpbin/httpbin.yaml) -n foo
kubectl apply -f <(istioctl kube-inject -f samples/sleep/sleep.yaml) -n foo
kubectl create ns bar
kubectl apply -f <(istioctl kube-inject -f samples/httpbin/httpbin.yaml) -n bar
kubectl apply -f <(istioctl kube-inject -f samples/sleep/sleep.yaml) -n bar
```

创建另一个 legacy 名称空间，不启用 sidecar 注入的情况下部署 sleep：

```Bash
kubectl create ns legacy
kubectl apply -f samples/httpbin/httpbin.yaml -n legacy
kubectl apply -f samples/sleep/sleep.yaml -n legacy
```

查看这三个名称空间的相互访问情况

```Bash
for from in "foo" "bar" "legacy"; do for to in "foo" "bar"; do kubectl exec $(kubectl get pod -l app=sleep -n ${from} -o jsonpath={.items..metadata.name}) -c sleep -n ${from} -- curl http://httpbin.${to}:8000/ip -s -o /dev/null -w "sleep.${from} to httpbin.${to}: %{http_code}\n"; done; done
```

> 这个命令的目的是检查在不同命名空间中的`sleep` Pod 与其他命名空间中的`httpbin`服务之间的连接情况。最终，每个请求的源和目标以及 HTTP 状态代码将被输出。

两两互通，和谐

```Bash
root@node1:~/istio-1.26.0# for from in "foo" "bar" "legacy"; do for to in "foo" "bar"; do kubectl exec $(kubectl get pod -l app=sleep -n ${from} -o jsonpath={.items..metadata.name}) -c sleep -n ${from} -- curl http://httpbin.${to}:8000/ip -s -o /dev/null -w "sleep.${from} to httpbin.${to}: %{http_code}\n"; done; done
sleep.foo to httpbin.foo: 200
sleep.foo to httpbin.bar: 200
sleep.bar to httpbin.foo: 200
sleep.bar to httpbin.bar: 200
sleep.legacy to httpbin.foo: 200
sleep.legacy to httpbin.bar: 200
```

检查 authentication policies 和 destination rules

```Bash
kubectl get peerauthentication --all-namespaces

kubectl get destinationrule --all-namespaces
root@node1:~/istio-1.26.0# kubectl get peerauthentication --all-namespaces
No resources found
root@node1:~/istio-1.26.0#
root@node1:~/istio-1.26.0# kubectl get destinationrule --all-namespaces
NAMESPACE   NAME          HOST          AGE
default     details       details       18h
default     productpage   productpage   18h
default     ratings       ratings       18h
default     reviews       reviews       18h
```

在整个网格上启用`PERMISSIVE`模式的认证策略

```Bash
kubectl apply -n istio-system -f istiolabmanual/mtlspermissive.yaml 
```

查看认证策略配置文件

```Bash
nano istiolabmanual/mtlspermissive.yaml
apiVersion: "security.istio.io/v1beta1"
kind: "PeerAuthentication"
metadata:
  name: "default"
spec:
  mtls:
    mode: PERMISSIVE
```

> 这是一个 Istio 的`PeerAuthentication`资源配置文件。它用于定义服务之间的认证策略，主要用于配置 mTLS（双向 TLS）行为。下面是关于这个配置文件的详细解释：
>
> - `apiVersion: "security.istio.io/v1beta1"`：指定资源对象使用的 API 版本，这里用的是 Istio Security API 的 v1beta1 版本。
> - `kind: "PeerAuthentication"`：资源对象的 Kind（类型），表示这是一个`PeerAuthentication`资源。
> - `metadata`：元数据部分，其中包含资源对象的名称和其他相关信息。
>   - `name: "default"`：资源对象的名称为"default"。
> - `spec`：配置的具体细节：
>   - `mtls`：定义 mTLS 的配置。
>     - `mode: PERMISSIVE`：设置 mTLS 模式为"PERMISSIVE"。在这种模式下，服务可以同时接收使用 TLS 加密或者未加密的流量。这将允许任何能与服务通信的来源发送请求，而无论其是否使用 TLS。
>
> 总之，这个配置文件定义了一个名为"default"的`PeerAuthentication`资源，该资源将服务的 mTLS 模式设置为"`PERMISSIVE`"，允许服务接收来自其他使用或不使用 TLS 加密的服务的请求。这通常用于平滑升级和过渡，但请注意，这样的设置可能会降低安全性。在需要保证较高安全性的场景下，可以考虑使用更严格的模式，如"`STRICT`"。

查看这三个名称空间的相互访问情况

```Bash
for from in "foo" "bar" "legacy"; do for to in "foo" "bar"; do kubectl exec $(kubectl get pod -l app=sleep -n ${from} -o jsonpath={.items..metadata.name}) -c sleep -n ${from} -- curl http://httpbin.${to}:8000/ip -s -o /dev/null -w "sleep.${from} to httpbin.${to}: %{http_code}\n"; done; done
```

还是很和谐，因为策略比较宽松

```Bash
root@node1:~/istio-1.26.0# for from in "foo" "bar" "legacy"; do for to in "foo" "bar"; do kubectl exec $(kubectl get pod -l app=sleep -n ${from} -o jsonpath={.items..metadata.name}) -c sleep -n ${from} -- curl http://httpbin.${to}:8000/ip -s -o /dev/null -w "sleep.${from} to httpbin.${to}: %{http_code}\n"; done; done
sleep.foo to httpbin.foo: 200
sleep.foo to httpbin.bar: 200
sleep.bar to httpbin.foo: 200
sleep.bar to httpbin.bar: 200
sleep.legacy to httpbin.foo: 200
sleep.legacy to httpbin.bar: 200
```

在整个网格上启用`STRICT`模式的认证策略

```Bash
kubectl  apply -n istio-system -f istiolabmanual/mtlsstrict.yaml 
```

查看认证策略配置文件

```Bash
nano istiolabmanual/mtlsstrict.yaml
apiVersion: "security.istio.io/v1beta1"
kind: "PeerAuthentication"
metadata:
  name: "default"
spec:
  mtls:
    mode: STRICT
```

> 在这个配置文件中，唯一不同的部分是`spec`下的`mtls`设置。`mode`的值从"`PERMISSIVE`"变为了"`STRICT`"。
>
> 这意味着在 Istio 服务网格中，mTLS（双向 TLS）策略已经升级为"`STRICT`"模式。在此模式下，只有使用 TLS 加密的流量才能通过认证。与刚才的"`PERMISSIVE`"模式相比，"`STRICT`"模式提供了更高的安全性，因为它要求所有流量都进行加密。在"`STRICT`"模式下，不允许未加密的流量进入服务。

查看这三个名称空间的相互访问情况

```Bash
for from in "foo" "bar" "legacy"; do for to in "foo" "bar"; do kubectl exec $(kubectl get pod -l app=sleep -n ${from} -o jsonpath={.items..metadata.name}) -c sleep -n ${from} -- curl http://httpbin.${to}:8000/ip -s -o /dev/null -w "sleep.${from} to httpbin.${to}: %{http_code}\n"; done; done
```

策略一旦收紧，legacy 被发现是裸泳的了，悲剧

```Bash
root@node1:~/istio-1.26.0# for from in "foo" "bar" "legacy"; do for to in "foo" "bar"; do kubectl exec $(kubectl get pod -l app=sleep -n ${from} -o jsonpath={.items..metadata.name}) -c sleep -n ${from} -- curl http://httpbin.${to}:8000/ip -s -o /dev/null -w "sleep.${from} to httpbin.${to}: %{http_code}\n"; done; done
sleep.foo to httpbin.foo: 200
sleep.foo to httpbin.bar: 200
sleep.bar to httpbin.foo: 200
sleep.bar to httpbin.bar: 200
sleep.legacy to httpbin.foo: 000
command terminated with exit code 56
sleep.legacy to httpbin.bar: 000
command terminated with exit code 56
```

重建 legacy 名称空间的服务，请启用 sidecar 注入

```Bash
kubectl apply -f <(istioctl kube-inject -f samples/httpbin/httpbin.yaml) -n legacy
kubectl apply -f <(istioctl kube-inject -f samples/sleep/sleep.yaml) -n legacy
```

查看 legacy 名称空间 pod 重建情况

```Bash
kubectl get pod -n legacy
root@node1:~/istio-1.26.0# kubectl get pod -n legacy
NAME                      READY   STATUS    RESTARTS   AGE
httpbin-dddb978d5-wg7pq   2/2     Running   0          26s
sleep-7786869f6c-56q75    2/2     Running   0          24s
```

查看这三个名称空间的相互访问情况

```Bash
for from in "foo" "bar" "legacy"; do for to in "foo" "bar"; do kubectl exec $(kubectl get pod -l app=sleep -n ${from} -o jsonpath={.items..metadata.name}) -c sleep -n ${from} -- curl http://httpbin.${to}:8000/ip -s -o /dev/null -w "sleep.${from} to httpbin.${to}: %{http_code}\n"; done; done
```

注入 sidecar 之后，legacy 又可以和小伙伴们一起玩耍了

```Bash
root@node1:~/istio-1.26.0# for from in "foo" "bar" "legacy"; do for to in "foo" "bar"; do kubectl exec $(kubectl get pod -l app=sleep -n ${from} -o jsonpath={.items..metadata.name}) -c sleep -n ${from} -- curl http://httpbin.${to}:8000/ip -s -o /dev/null -w "sleep.${from} to httpbin.${to}: %{http_code}\n"; done; done
sleep.foo to httpbin.foo: 200
sleep.foo to httpbin.bar: 200
sleep.bar to httpbin.foo: 200
sleep.bar to httpbin.bar: 200
sleep.legacy to httpbin.foo: 200
sleep.legacy to httpbin.bar: 200
```

查看验证策略

```Bash
kubectl get peerauthentication --all-namespaces
root@node1:~/istio-1.26.0# kubectl get peerauthentication --all-namespaces
NAMESPACE      NAME      MODE     AGE
istio-system   default   STRICT   6m22s
```

清理

```Bash
kubectl delete peerauthentication --all-namespaces --all
kubectl delete ns foo bar legacy
```

# Lab 8 授权：实现 JWT 身份的认证和授权

创建包含 httpbin 和 sleep 样例的名称空间 foo

```Bash
kubectl create ns foo
kubectl apply -f <(istioctl kube-inject -f samples/httpbin/httpbin.yaml) -n foo
kubectl apply -f <(istioctl kube-inject -f samples/sleep/sleep.yaml) -n foo
```

检查 httpbin 和 sleep 的通讯情况

```Bash
kubectl exec $(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name}) -c sleep -n foo -- curl http://httpbin.foo:8000/ip -s -o /dev/null -w "%{http_code}\n"
root@node1:~/istio-1.26.0# kubectl exec $(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name}) -c sleep -n foo -- curl http://httpbin.foo:8000/ip -s -o /dev/null -w "%{http_code}\n"
200
```

> 这个命令行包含了几个子命令，组合在一起完成了在命名空间`foo`下的`sleep` Pod 内部检查`http://httpbin.foo:8000/ip`的 HTTP 响应状态的操作。输出：`200`这表示请求成功。

为 httpbin 创建 request authentication policy

```Bash
kubectl apply -f istiolabmanual/jwtra.yaml
```

查看验证策略

```Bash
nano istiolabmanual/jwtra.yaml
apiVersion: "security.istio.io/v1beta1"
kind: "RequestAuthentication"
metadata:
  name: "jwt-example"
  namespace: foo
spec:
  selector:
    matchLabels:
      app: httpbin
  jwtRules:
  - issuer: "testing@secure.istio.io"
    jwksUri: "https://raw.githubusercontent.com/istio/istio/release-1.5/security/tools/jwt/samples/jwks.json"
```

> 这是一个 Istio RequestAuthentication 资源的 YAML 配置文件。RequestAuthentication 资源用于配置请求的认证策略，对于符合策略的请求，Istio 将要求客户端提供有效的 JWT（JSON Web Token）令牌。以下是配置文件的详细解释：
>
> - apiVersion: 配置文件版本，security.istio.io/v1beta1 表示这是 Istio security 的 v1beta1 版本。
> - kind: 资源类型，这里是 RequestAuthentication 用于配置请求认证策略。
> - metadata: 包括资源的名称和命名空间。
>   - name: 资源名称，这里是 "jwt-example"
>   - namespace: 资源所在的命名空间，这里是 "foo"
> - spec: RequestAuthentication 资源的具体配置信息。
>   - selector: 定义策略适用的目标工作负载，通过指定匹配标签来选择目标。
>     - matchLabels: 目标工作负载的标签选择器，这里需要标签 "app: httpbin" 的工作负载才能匹配。
>   - jwtRules: 定义处理 JWT 的规则。
>     - issuer: JWT 发行人，这里是 "testing@secure.istio.io"
>     - jwksUri: JWT 签名密钥集的 URL，这里是 "https://raw.githubusercontent.com/istio/istio/release-1.5/security/tools/jwt/samples/jwks.json"
>
> 这个配置文件定义了一个请求认证策略，要求具有标签 `"app: httpbin`" 的工作负载的请求提供有效的 JWT，发行人为 "`testing@secure.istio.io`"，同时通过提供的`jwksUri`验证 JWT 的签名。

检查持有无效 JWT 的访问

```Bash
kubectl exec $(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name}) -c sleep -n foo -- curl "http://httpbin.foo:8000/headers" -s -o /dev/null -H "Authorization: Bearer invalidToken" -w "%{http_code}\n"
```

> 这个命令是在 Kubernetes 集群中执行的，它的作用是从 `foo` 命名空间中获取一个标签为 `app=sleep` 的 Pod，在该 Pod 的 `sleep` 容器中执行 `curl` 命令，向 `httpbin.foo:8000/headers` 发送一个 HTTP 请求，并打印响应的 HTTP 状态码。
>
> 具体解释如下：
>
> - `kubectl exec $(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name})` 获取 `foo` 命名空间中标签为 `app=sleep` 的 Pod 名称，并执行后面的命令。
> - `-c sleep` 指定在该 Pod 的 `sleep` 容器中执行命令。
> - `-n foo` 指定在 `foo` 命名空间中执行命令。
> - `curl "``http://httpbin.foo:8000/headers``"` 向 `httpbin.foo:8000/headers` 发送 HTTP 请求。
> - `-s` 静默模式，不输出进度信息。
> - `-o /dev/null` 将响应体输出到 `/dev/null`，即丢弃响应体。
> - `-H "Authorization: Bearer invalidToken"` 在请求头中添加一个无效的 Bearer 令牌。
> - `-w "%{http_code}\n"` 打印响应的 HTTP 状态码，并换行。
>
> 这个命令是在 Kubernetes 集群中的一个 Pod 容器中，向一个服务发送带有无效令牌的 HTTP 请求，并打印响应的 HTTP 状态码，用于测试或调试目的。

理应不能访问，401 没毛病

```Bash
root@node1:~/istio-1.26.0# kubectl exec $(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name}) -c sleep -n foo -- curl "http://httpbin.foo:8000/headers" -s -o /dev/null -H "Authorization: Bearer invalidToken" -w "%{http_code}\n"
401
```

检查不持有 JWT 的访问，

```Bash
kubectl exec $(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name}) -c sleep -n foo -- curl "http://httpbin.foo:8000/headers" -s -o /dev/null -w "%{http_code}\n"
```

> 与之前的命令相比，这个命令的差异在于：
>
> 移除了 `-H "Authorization: Bearer invalidToken"` 部分。
>
> 之前的命令在发送 HTTP 请求时，使用了一个无效的 Bearer 令牌 `invalidToken`。而这个命令则没有在请求头中添加任何授权相关的头字段。
>
> 因此，这个命令只是简单地向 `httpbin.foo:8000/headers` 发送一个 HTTP 请求，而不携带任何授权信息。其余部分，如获取 Pod 名称、在指定容器中执行命令、丢弃响应体、打印 HTTP 状态码等，与之前的命令相同。

这居然也能成功，尴尬

```Bash
root@node1:~/istio-1.26.0# kubectl exec $(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name}) -c sleep -n foo -- curl "http://httpbin.foo:8000/headers" -s -o /dev/null -w "%{http_code}\n"
200
```

在 foo 上启用 authorization policy

```Bash
kubectl apply -f istiolabmanual/jwtap.yaml 
```

查看验证策略配置文件

```Bash
nano istiolabmanual/jwtap.yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: require-jwt
  namespace: foo
spec:
  selector:
    matchLabels:
      app: httpbin
  action: ALLOW
  rules:
  - from:
    - source:
       requestPrincipals: ["testing@secure.istio.io/testing@secure.istio.io"]
```

> 这个 YAML 文件定义了一个 Istio 的 `AuthorizationPolicy` 资源对象。它用于配置 Istio 服务网格中的授权策略。
>
> 结合之前关于令牌的问题，我们来解释一下这个授权策略的作用。
>
> 之前的问题是，使用无效令牌访问失败，而不携带任何令牌反而能够成功访问。这种情况看起来有些反常，可能是由于服务本身的配置或授权机制存在问题导致的。
>
> 而这个 `AuthorizationPolicy` 的作用，就是为了明确定义服务的授权规则，规避这种不一致的情况。具体来说：
>
> 1. 它将授权策略应用于标签为 `app=httpbin` 的工作负载，也就是之前提到的 `httpbin.foo:8000/headers` 这个服务。
> 2. 授权策略的行为是 `ALLOW`，即只允许符合规则的请求访问。
> 3. 规则要求请求必须携带一个有效的 JWT 令牌，且令牌的 `issuer` 和 `subject` 都必须是 `testing@secure.istio.io`。
> 4. 对于不携带有效令牌的请求，由于不符合规则，将被拒绝访问。
> 5. 对于携带无效令牌（issuer 或 subject 不正确）的请求，也将被拒绝访问。
> 6. 只有携带满足规则要求的有效 JWT 令牌的请求，才能被允许访问该服务。
>
> 通过这个明确的授权策略，Istio 服务网格可以强制执行统一的访问控制规则，避免出现之前那种不一致的情况。它确保了只有合法的请求才能访问服务，提高了系统的安全性。

设置指向 JWT 的 Token 变量

```Bash
TOKEN=$(curl https://raw.githubusercontent.com/istio/istio/release-1.5/security/tools/jwt/samples/demo.jwt -s) && echo $TOKEN | cut -d '.' -f2 - | base64 --decode -
```

> 这个命令是一种在 Unix 或 Linux 终端中执行的多步骤过程，旨在从 Istio 的 GitHub 仓库获取一个示例的 JWT（JSON Web Token），并对其进行解码。让我们逐步解析这个命令：
>
> 1. `TOKEN=$(curl https://raw.githubusercontent.com/istio/istio/release-1.5/security/tools/jwt/samples/demo.jwt -s)`： 使用 curl 命令从 Istio 的 GitHub 仓库下载一个示例的 JWT，并将其内容保存在名为`TOKEN`的变量中。`-s`参数使 curl 操作在静默模式下进行，这意味着它不会显示进度或错误消息。
> 2. `echo $TOKEN`： 输出`TOKEN`变量的值。这将打印下载的 JWT 字符串。
> 3. `cut -d '.' -f2 -`： 使用 cut 命令从 JWT 字符串中提取第二部分（以'。'分隔）。一个典型的 JWT 由三部分组成：header、payload 和 signature，它们用点号分隔。这个命令将 JWT 解析为其净荷部分。
> 4. `base64 --decode -`： 使用 base64 命令对提取后的净荷部分进行解码。这将把 Base64 编码的净荷部分解码为一个 JSON 对象。
>
> 整个命令通过管道将这些步骤连接起来，使输出能够顺利流转。最后，你会看到 JWT 的净荷部分作为一个易读的 JSON 对象。

使用有效 JWT 进行访问

```Bash
kubectl exec $(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name}) -c sleep -n foo -- curl "http://httpbin.foo:8000/headers" -s -o /dev/null -H "Authorization: Bearer $TOKEN" -w "%{http_code}\n"
```

> 这个命令行是一个用于发送请求到"httpbin"服务的复合命令。它在名为 "foo" 的命名空间中执行，并向"httpbin"服务发送带有 JWT 令牌的请求。让我们逐步解析它：
>
> 1. `kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name}`：这个子命令用于获取在 "foo" 命名空间中标签为 "app=sleep" 的 pod 的名称。使用 `-o jsonpath={.items..metadata.name}` 选择输出仅包含 pod 名称的子集。
> 2. `kubectl exec $(...) -c sleep -n foo --`： 这一部分的命令使用 `kubectl exec` 在 "foo" 命名空间中找到具有 "app=sleep" 标签的 Pod，并在其中的 sleep 容器中执行后续的命令。`$(...)` 部分会被之前子命令的结果（即对应的 Pod 的名称）替换。
> 3. `curl "http://httpbin.foo:8000/headers" -s -o /dev/null -H "Authorization: Bearer $TOKEN" -w "%{http_code}\n"`：这一部分的命令在 sleep 容器内使用 curl 向"httpbin"服务发送请求。它请求"httpbin"服务的 "/headers" 接口，使用之前获取的 TOKEN 作为发送请求的 JWT 令牌。`-s` 参数表示静默模式，`-o /dev/null` 将响应主体发送到/dev/null， 然后 `-w "%{http_code}\n"` 参数输出 HTTP 状态码并以换行符结尾。
>
> 这个命令与之前的命令有关。之前的命令获取并解码了 JWT 令牌的净荷部分。而这个命令使用获取的 JWT 令牌（保存在 TOKEN 变量中）作为身份验证信息向"httpbin"服务发送请求。结合之前分析的 Istio 授权策略，这意味着只有请求包含有效的 JWT 令牌且 JWT 的请求主体合法时，请求才会被 "httpbin" 服务接受，并返回 HTTP 状态码。

正常，200

```Bash
root@node1:~/istio-1.26.0# kubectl exec $(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name}) -c sleep -n foo -- curl "http://httpbin.foo:8000/headers" -s -o /dev/null -H "Authorization: Bearer $TOKEN" -w "%{http_code}\n"
200
```

再次验证不持有 JWT 的访问

```Bash
kubectl exec $(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name}) -c sleep -n foo -- curl "http://httpbin.foo:8000/headers" -s -o /dev/null -w "%{http_code}\n"
```

授权策略启用之后，就没办法浑水摸鱼了，403 了

```Bash
root@node1:~/istio-1.26.0# kubectl exec $(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name}) -c sleep -n foo -- curl "http://httpbin.foo:8000/headers" -s -o /dev/null -w "%{http_code}\n"
403
```

清理环境

```Bash
kubectl delete namespace foo
```

# Lab 9 服务可观测性

## 1.日志

模拟一次页面访问，为了防止被不必要的 katacode 页面元素“污染“，我们最好从内部用命令行执行一次 curl 访问

```Bash
kubectl get svc

curl http://10.108.91.118:9080/productpage
root@node1:~# kubectl get svc
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
details       ClusterIP   10.97.72.135    <none>        9080/TCP   19h
kubernetes    ClusterIP   10.96.0.1       <none>        443/TCP    222d
productpage   ClusterIP   10.108.91.118   <none>        9080/TCP   19h
ratings       ClusterIP   10.96.212.103   <none>        9080/TCP   19h
reviews       ClusterIP   10.102.110.0    <none>        9080/TCP   19h
root@node1:~# curl http://10.108.91.118:9080/productpage
<!DOCTYPE html>
<html>
  <head>
    <title>Simple Bookstore App</title>
<meta charset="utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<!-- Latest compiled and minified CSS -->
<link rel="stylesheet" href="static/bootstrap/css/bootstrap.min.css">

<!-- Optional theme -->
<link rel="stylesheet" href="static/bootstrap/css/bootstrap-theme.min.css">

  </head>
  <body>
...
```

查看 istio proxy 日志

```Plaintext
kubectl get pods
kubectl logs -f productpage-xxx istio-proxy
```

观察到两个 outbond 条目，分别指向 details 和 reviews，还有一个 inbound 条目，指向 productpage

```Bash
2024-03-21T07:08:05.712268Z     info    xdsproxy        connected to upstream XDS server: istiod.istio-system.svc:15012
[2024-03-21T02:36:05.715Z] "GET /details/0 HTTP/1.1" 200 - via_upstream - "-" 0 178 2 2 "-" "curl/7.68.0" "7b923129-a124-9cf9-9c32-8fbe97cb3f57" "details:9080" "10.244.135.8:9080" outbound|9080||details.default.svc.cluster.local 10.244.135.11:49730 10.97.72.135:9080 10.244.135.11:34060 - default
[2024-03-21T02:36:05.721Z] "GET /reviews/0 HTTP/1.1" 200 - via_upstream - "-" 0 438 791 791 "-" "curl/7.68.0" "7b923129-a124-9cf9-9c32-8fbe97cb3f57" "reviews:9080" "10.244.104.11:9080" outbound|9080||reviews.default.svc.cluster.local 10.244.135.11:48624 10.102.110.0:9080 10.244.135.11:47568 - default
```

> 这个输出文件展示了 Istio 服务网格中的通信过程。具体来说：
>
> 1. 第一行是 Istio sidecar 代理连接到 istiod 控制平面的日志。istiod 是 Istio 的核心组件，负责服务发现、配置分发等功能。
> 2. 接下来的两行是访问应用服务的请求日志。
>
> 第一个请求日志：
>
> - 发起了一个 HTTP GET 请求到 `/details/0` 路径
> - 该请求经过 Istio sidecar 代理转发，最终由 `details` 服务的 9080 端口处理
> - 请求来自于源 IP `10.244.135.11:49730`，目的地是 `10.97.72.135:9080`
> - 请求通过 `outbound|9080||details.default.svc.cluster.local` 这个出站集群发出
>
> 第二个请求日志：
>
> - 发起了一个 HTTP GET 请求到 `/reviews/0` 路径
> - 该请求经过 sidecar 代理转发，最终由 `reviews` 服务的 9080 端口处理
> - 请求来自 `10.244.135.11:48624`，目的地是 `10.102.110.0:9080`
> - 请求通过 `outbound|9080||reviews.default.svc.cluster.local` 这个出站集群发出
>
> 这些日志反映了 Istio 处理东西向流量的情况。应用发起的请求首先被 sidecar 代理接管，然后通过配置的出站路由规则转发到实际的服务端点。整个过程对应用是透明的，应用只需要像访问本地服务一样访问即可，而底层的负载均衡、服务发现等都由 Istio 处理。

将使用 Mermaid 绘制第二个请求的访问流程图：

```Plain
sequenceDiagram
    participant Client
    participant Sidecar
    participant OutboundCluster
    participant ReviewsService

    Client->>Sidecar: GET /reviews/0 (2024-03-21T02:36:05.720Z)
    Sidecar->>OutboundCluster: Forward request (2024-03-21T02:36:05.720Z)
    OutboundCluster->>ReviewsService: GET /reviews/0 (2024-03-21T02:36:05.721Z)
    ReviewsService-->>OutboundCluster: HTTP 200 Response (2024-03-21T02:36:05.722Z)
    OutboundCluster-->>Sidecar: Forward response (2024-03-21T02:36:05.723Z)
    Sidecar-->>Client: HTTP 200 Response (2024-03-21T02:36:05.724Z)

    Note right of OutboundCluster: outbound|9080||reviews.default.svc.cluster.local
    Note right of ReviewsService: reviews:9080 (10.102.110.0:9080)
    Note left of Client: Source: 10.244.135.11:48624
```

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=ODcyMjkzZTI3MDJhZmJmZjVlMmE4OTE1YzM0OWI5OThfdnlxbHdoazBTYkNwWExwVno2WXVBS2FMQWRHVjZVMWhfVG9rZW46RzBDVWJ6TG05b1F6VnB4NGtadWM3WGxCbkZmXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

> 在这个序列图中：
>
> 1. 客户端发起一个 `GET /reviews/0` 的 HTTP 请求到 Sidecar 代理。
> 2. Sidecar 代理将请求转发到 `outbound|9080||reviews.default.svc.cluster.local` 这个出站集群。
> 3. 出站集群将请求转发到实际的 `reviews` 服务实例 （10.102.110.0：9080）。
> 4. `reviews` 服务处理请求并返回 HTTP 200 响应。
> 5. 响应通过出站集群和 Sidecar 代理传递回客户端。
>
> 图中还包含了一些关键信息的注释：
>
> - 出站集群的名称
> - `reviews` 服务的实际端点地址
> - 客户端的源 IP 和端口

为获取更多信息，设置日志以 JSON 格式显式

```Bash
istioctl manifest apply --set profile=demo --set values.meshConfig.accessLogFile="/dev/stdout" --set values.meshConfig.accessLogEncoding=JSON
```

（可选）查看日志设置

```Bash
kubectl describe configmap istio -n istio-system | less
```

配置文件有以下输出

```Bash
Data
====
mesh:
----
accessLogEncoding: JSON #格式
accessLogFile: /dev/stdout #日志输入位置
defaultConfig:
  discoveryAddress: istiod.istio-system.svc:15012
  proxyMetadata: {}
  tracing:
    zipkin:
      address: zipkin.istio-system:9411
enablePrometheusMerge: true
extensionProviders:
- envoyOtelAls:
    port: 4317
    service: opentelemetry-collector.istio-system.svc.cluster.local
  name: otel
rootNamespace: istio-system
trustDomain: cluster.local
meshNetworks:
----
networks: {}
```

> 这个配置文件是 Istio 服务网格的配置文件。它定义了 Istio 控制平面和数据平面的一些关键设置。
>
> 在 `Data` 部分下的 `mesh` 字段中包含了 Istio 服务网格的全局配置。其中：
>
> - `accessLogEncoding` 和 `accessLogFile` 分别定义了访问日志的编码格式和输出位置。
> - `defaultConfig` 字段定义了 Istio 代理的默认配置，包括发现地址、元数据和追踪设置。
> - `enablePrometheusMerge` 启用了 Prometheus 指标的合并。
> - `extensionProviders` 定义了扩展提供程序，这里配置了 OpenTelemetry 收集器。
> - `rootNamespace` 和 `trustDomain` 分别指定了 Istio 控制平面的命名空间和信任域。
>
> 在 `meshNetworks` 部分下的 `networks` 字段用于定义服务网格中的网络，但在这个配置文件中它是空的。
>
> 这个配置文件定义了 Istio 服务网格的一些全局设置，包括日志、发现、追踪、指标收集等方面的配置。它为整个服务网格提供了一个统一的配置基础。

再次查看 istio proxy 日志

```Bash
kubectl logs -f productpage-xxx istio-proxy
```

分别指向 details 和 reviews 的 outbound 条目

```JSON
{"downstream_remote_address":"10.40.0.12:41654","authority":"details:9080","path":"/details/0","protocol":"HTTP/1.1","upstream_service_time":"9","upstream_local_address":"10.40.0.12:48508","duration":"15","upstream_transport_failure_reason":"-","route_name":"default","downstream_local_address":"10.99.18.67:9080","user_agent":"curl/7.47.0","response_code":"200","response_flags":"-","start_time":"2020-05-15T06:30:26.459Z","method":"GET","request_id":"e2de9f03-38ac-924d-ae2b-176d868c56ab","upstream_host":"10.40.0.8:9080","x_forwarded_for":"-","requested_server_name":"-","bytes_received":"0","istio_policy_status":"-","bytes_sent":"178","upstream_cluster":"outbound|9080||details.default.svc.cluster.local"}

{"upstream_cluster":"outbound|9080||reviews.default.svc.cluster.local","downstream_remote_address":"10.40.0.12:43866","authority":"reviews:9080","path":"/reviews/0","protocol":"HTTP/1.1","upstream_service_time":"1786","upstream_local_address":"10.40.0.12:36048","duration":"1787","upstream_transport_failure_reason":"-","route_name":"default","downstream_local_address":"10.98.163.167:9080","user_agent":"curl/7.47.0","response_code":"200","response_flags":"-","start_time":"2020-05-15T06:30:26.480Z","method":"GET","request_id":"e2de9f03-38ac-924d-ae2b-176d868c56ab","upstream_host":"10.40.0.11:9080","x_forwarded_for":"-","requested_server_name":"-","bytes_received":"0","istio_policy_status":"-","bytes_sent":"379"}
```

> 这两个 JSON 对象是 Istio 代理生成的访问日志条目。它们记录了两个服务之间的 HTTP 请求和响应的详细信息。
>
> 第一个 JSON 对象描述了一个从 `productpage` 服务发送到 `details` 服务的请求：
>
> - `downstream_remote_address` 是发送请求的客户端 IP 和端口
> - `authority` 是请求的目标主机和端口
> - `path` 是请求路径
> - `upstream_service_time` 是上游服务处理请求的时间（9 毫秒）
> - `upstream_cluster` 是上游集群的名称
>
> 第二个 JSON 对象描述了一个从 `productpage` 服务发送到 `reviews` 服务的请求：
>
> - `upstream_service_time` 是上游服务处理请求的时间（1786 毫秒）
> - `upstream_cluster` 是上游集群的名称
>
> 这些日志条目提供了请求的详细元数据，如源 IP、目标服务、请求路径、响应代码、字节数等。它们对于监控服务性能、诊断问题和了解服务之间的通信模式非常有用。

指向 productpage 的 inbound 条目

```JSON
{"upstream_cluster":"inbound|9080|http|productpage.default.svc.cluster.local","downstream_remote_address":"10.32.0.1:40938","authority":"10.109.209.72:9080","path":"/productpage","protocol":"HTTP/1.1","upstream_service_time":"1839","upstream_local_address":"127.0.0.1:51264","duration":"1840","upstream_transport_failure_reason":"-","route_name":"default","downstream_local_address":"10.40.0.12:9080","user_agent":"curl/7.47.0","response_code":"200","response_flags":"-","start_time":"2020-05-15T06:30:26.446Z","method":"GET","request_id":"e2de9f03-38ac-924d-ae2b-176d868c56ab","upstream_host":"127.0.0.1:9080","x_forwarded_for":"-","requested_server_name":"-","bytes_received":"0","istio_policy_status":"-","bytes_sent":"5183"}
```

> 好的，这个 JSON 日志条目代表了一个被 Istio 服务网格中的 `productpage` 服务接收到的请求。以下是关键字段的解释：
>
> - `upstream_cluster`： 这是接收请求的上游集群的名称。在这种情况下，它是 `inbound|9080|http|productpage.default.svc.cluster.local`，表示请求被 `productpage` 服务在 9080 端口上使用 HTTP 协议接收。
> - `downstream_remote_address`： 这是发起请求的客户端的 IP 地址和端口 （10.32.0.1：40938）。
> - `authority`： 这是客户端在请求中指定的授权主机和端口 （10.109.209.72：9080）。
> - `path`： 这是请求 URL 的路径部分 （"/productpage"）。
> - `upstream_service_time`： 这是上游服务 （`productpage`） 处理请求所花费的时间（以毫秒为单位，这里是 1839 毫秒）。
> - `upstream_local_address`： 这是 `productpage` 服务监听的本地 IP 地址和端口 （127.0.0.1：51264）。
> - `response_code`： 这是 `productpage` 服务返回的 HTTP 响应代码 （200，表示响应成功）。
> - `bytes_sent`： 这是 `productpage` 服务发送的响应字节数 （5183 字节）。
>
> 该日志条目提供了 `productpage` 服务处理的特定请求的详细信息，包括发起请求的客户端、请求详情、响应详情以及服务处理该请求所花费的时间。这些信息对于监控、调试和分析 Istio 服务网格中服务的行为非常有用。

使用 JSON Handler 查看详细日志尤其是五元组信息和 Flag

```Bash
`"outbound|9080||details.default.svc.cluster.local"`

`"outbound|9080||reviews.default.svc.cluster.local"`

`"inbound|9080|http|productpage.default.svc.cluster.local"
```

## 2.指标和可视化

安装仪表板：

```Bash
kubectl apply -f samples/addons
```

开放监控工具的 NodePort 端口

```Bash
kubectl patch svc -n istio-system prometheus -p '{"spec":{"type": "NodePort"}}'
kubectl patch service prometheus --namespace=istio-system --type='json' --patch='[{"op": "replace", "path": "/spec/ports/0/nodePort", "value":31120}]'

kubectl patch svc -n istio-system grafana  -p '{"spec":{"type": "NodePort"}}'
kubectl patch service grafana --namespace=istio-system --type='json' --patch='[{"op": "replace", "path": "/spec/ports/0/nodePort", "value":31121}]'

kubectl patch svc -n istio-system tracing -p '{"spec":{"type": "NodePort"}}'
kubectl patch service tracing --namespace=istio-system --type='json' --patch='[{"op": "replace", "path": "/spec/ports/0/nodePort", "value":31122}]'

kubectl patch svc -n istio-system kiali -p '{"spec":{"type": "NodePort"}}'
kubectl patch service kiali --namespace=istio-system --type='json' --patch='[{"op": "replace", "path": "/spec/ports/0/nodePort", "value":31123}]'
```

> 这几段命令的作用是将 Istio 系统命名空间中的几个服务的类型从默认的 ClusterIP 改为 NodePort 类型，并且指定了每个服务的节点端口号。具体来说：
>
> 1. 第一对命令将 Prometheus 服务的类型改为 NodePort，并将其节点端口设置为 31120。
> 2. 第二对命令将 Grafana 服务的类型改为 NodePort，并将其节点端口设置为 31121。
> 3. 第三对命令将 Tracing 服务的类型改为 NodePort，并将其节点端口设置为 31122。
> 4. 第四对命令将 Kiali 服务的类型改为 NodePort，并将其节点端口设置为 31123。
>
> 通过这些更改，这些服务将可以通过集群中任何节点的指定端口从集群外部访问。这对于监控、可视化和跟踪 Istio 服务网格非常有用。

仪表板组件打开方式

- prometheus: http://node1:31120/
- grafana: http://node1:31121/
- tracing: http://node1:31122/
- kiali: [http://node1:31123/

查看 ingress gateway 的端口号

```Bash
 kubectl get svc -n istio-system | grep ingress
root@node1:~/istio-1.26.0# kubectl get svc -n istio-system | grep ingress
istio-ingressgateway   LoadBalancer   10.105.85.107    <pending>     15021:31215/TCP,80:30193/TCP,443:32696/TCP,31400:30573/TCP,15443:30572/TCP   23h
```

在另外一个窗口执行压测脚本

```Plaintext
while true; do curl http://node3:30193/productpage; done  
```

查看 Prometheus 监控指标：服务指标

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=MmYzODg4OGIxZmNiNTJiMDA4MjkzMDQ4MTc4OTg2Y2Ffa1l5TVZBdHpoVVMyakljajNuOEZTbDg3U2lFbjhWU0hfVG9rZW46UGtLZGJ5QUN1b2R1R3J4ckRPVGNzOVhqbmJiXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

查看 Prometheus 监控指标： Envoy 代理指标

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=ZmI0YmNjOWIyOGRjMzAzZjQzOGIwMTczODdkM2E5NjZfckFwQTEwSGJ6VHZpZWpPS0hpTmpwS0JuUW1UZDhQU2lfVG9rZW46TmloVGJVQkNvbzJzN0N4aFZzS2Njdmo3blRiXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

查看 Prometheus 监控指标： 控制平面指标，Status：runtime and build

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=OGQ0ODIyMTU2NzZlOGM3MzgxYzIzNDUwMWU0NzQ4MjNfRU9iVzhGWG9hTFpIZEVvakc4bUY4aENiRWJmWGdIWlBfVG9rZW46VkQ5ZGJLRUl2b0N0S0Z4QkFSeGNQcW10bjNnXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

查看 Prometheus 监控指标： Status：configuration

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=YTNkZGRkYmRhNjQwOWJmMDU1MDQ0NGY0MzM2OTAxYjVfWXVjTWZyZ3dGemlIMUc5RjUwc3JuNGNnN1M4M3RwZ01fVG9rZW46S295NmJFRFJmb1BnSm94cmNOc2N1b0F6bjBmXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

使用 Grafana 查看系统整体状态

打开 Grafana 页面，从报表列表中可以看到 istio 相关的报表，这里最关键的两个报表分别是

- 查看应用（服务）数据的 Mesh Dashboard
- 查看 Istio 自身（各组件）数据的 Performance Dashboard

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=ZmYzZjJkYmI3ZGM2ZTljOTA1OGRjOTllOWVmZDhhZWZfZUJBVG5qcEhpalI1aWw1MzY3T1R0cTdXU0lxblpuWEVfVG9rZW46R24xUGIzMnl1bzlVaXh4bXhOWmNDSHdJbndkXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

打开 mesh dashboard 我们可以观察到网格数据总览

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=MDA1Y2FmMWFiNTkzYjA3ZjJiNmE4ZTM2NzBlNGVhYjdfNTJ0ZzVoZ2pvVzVhS3p2cElvVXhQQm1yZTlQYmtvZjJfVG9rZW46T0NLdGIyZEExb3hPUTh4VWJuUWNBdGgzblRmXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

如果需要查看应用层面信息可以从 service 列表下面进行选择进入相关的服务视图（Service Dashboard）

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=ZGI5YjIzYTBlMjNkMzBmOWU4ZWI5ZjBlYTkzNWZmNmJfMUJ0QUM3OVViWUV4SDV2TEdCbnI1RWJpRWhINTB4alBfVG9rZW46WkJZdGJqUW1mbzh6enZ4NjA1MmN2bXBoblRYXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

如果需要查看部署层面信息可以从 workload 列表下面进行选择进入相关的工作负载（workload dashboard）视图

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=ODJkZjFmZGNkZDY5NDNhMTFmMmFhNTcwMjg4OTgxN2JfcEdDZXdZS1hUR0dHMUU2Vk0xTW9jQVQ5Z0dSMk1vSEFfVG9rZW46V0ZOeGJOYmllb2t6d0R4V1lITGMyYk9rbm1nXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

打开 Performance Dashboard 可以查看 Istio 自身以及各个组件的数据

Istio 系统总览

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=NzRhMjlhNjFmODI5NTgwOWM3NDFlNGUyZGI4YTRjMWRfZ1plVUZadTd5MXR6RmlRTmUzeUdkcVBHOHd4cVlrQm9fVG9rZW46TmJmSGJoeGsxbzMzNEV4dnc5OWMyTVhRbkw5XzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

控制平面视图

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=N2VkOTAxODgyZDYzYjk3ZmJkMDllMjNiZDg1N2IxNzdfWnBJV3JqZmdvRUtPQUt1eVU0bkN0NzhGU0RPN2FoSmpfVG9rZW46UWEzRmJWdWRpb1pNU1F4a1AzUWNwRE5NbnVmXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

## 3.查看应用拓扑信息

Overview 界面

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=NTZhNDdkYTczZTQ1MDg5ZDE1ZmRmYjQ1MTdjZmQ2ZGVfSk8yRTNjRkNEWjQ5RGFwdHRjMjNHWUMycVk4RHZEbk5fVG9rZW46SE1hbGIyWmc2b0k1eHN4UXNRVGNKWTFpbkVkXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

Graph 页面：查看应用拓扑

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=NTkwODdiYjM3MjNlOGEzZjFjNDQ5ZThiZDI4OGRkNTZfVFdOV1FGSkRZa1h1amxkdHVuSWFGeWxySzh0bG80RG1fVG9rZW46RGFnWGJ4M1pqbzMyS2F4bUw2SmNkRVlxbkY1XzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

尝试选择不同的构图和指标查看拓扑信息，比如在下图中可以观察流量在不同版本的 reviews 服务之间的分配及延时参数

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=ZTgzNmZmYmY3MTM3MzkzN2M0YmNmNmI5ZmJlMThmZDBfSzlKaE9QaHFJcXVPWUh6MmJCY21jS3d4bm9HWElCaVZfVG9rZW46SnhPTmJjTkZ2b3dpMkF4eDJxSmNNTWx6blFkXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

Application 以拓扑和出入站流量为监控核心

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=MGY1ODI5NWIxNzYxNWZkMTZmMmFkNjlmMTgwMmRlZDJfU2lZRWpkVm4xUkROWllabEhRYmo0Mk5TeW9uRXNYWmxfVG9rZW46UHkwMWIzT3cwbzg0WWx4eVBjVGNJc0dMbkFjXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

Service：侧重观察服务定义

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=OTM1ZDhlMjJkNTEwODMyMTU2ZjcxMjkyY2ZmYmQ0YTNfS3V5cmg3bWVsOWxMcXczdFJWcTlCT3lrTDZZV213UDdfVG9rZW46VUg4SWJ5T2xlb09UMkR4V0tBUGNGcVNpbmNkXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

Workload： 侧重观察部署和基础架构：

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=Yjg1YWEzYmM4MTAzMDVlMmEwOWQ4NzI4N2VhNWUxM2FfaXpwU3JaUGozcXM3V08xek0yYmlYdjRIeGMxc3NHTXZfVG9rZW46QUdhd2JNcGpubzU0b294RmFwamNsNU1TbmVmXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

## 4.分布式追踪

在 Jeager 页面中，从 search 中选择服务和相关的参数

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=NzY3MmE3M2Y3NzQ4YjBiNzQ4NTc0NzNjZmM0ZjY3MDFfSTRvNVVHdXpGdVVPYXRoUGtTd0ZKV25COE9pU01jSDhfVG9rZW46RE9TV2JzNHVJbzJVTmp4NmRRZ2NmSnF0bklkXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

从右侧选择一个 trace 进行分析，（建议选择饱满一点的，比如有 8 个 span）

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=ODVjMDg2ODg5ODZkMmQzNjNlZjUxM2I5NDc3ZDZjZTJfQmRYYndVdnVLUzZZQ1c3Z1o4UkZINjJnVjY0NmpxNENfVG9rZW46WkdwYWJDM25zb1o4Njh4WnU0NmNTdXpmbmliXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

对比 bookinfo 的访问过程进行访问链的调查

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=OWVkZTRhMDQ5YmM0ODI3NjlhNzYyZDEwZTgwMjAxN2Nfd3ZVeXZNT1ZtdmVTbFNWdDZYS2N0TDkwQTd5b0dmdVNfVG9rZW46Q1dqS2JGd3ZqbzZqSjJ4b3REaGNyQ3hGbmtiXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

展开某个 span 查看访问过程，可以看到 envoy 日志中的关键信息 比如 response_flag quest-id 和五元组信息

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=MGI3NzU1ZDA3YzRlNjZkMTA1OTI4OWEwMGIxZWIxMWNfTmE0UVlIVEFaS29OV1gzYW5vVTkwbHhlenBYdlM5STNfVG9rZW46SXlRTmJmRzFjb3NubGd4ZlNBcWNqVTVobkNjXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

亦可使用 Trace Graph 方式查看

![img](https://zsyhjtnsa5.feishu.cn/space/api/box/stream/download/asynccode/?code=OGNhOTZiMGU0N2Q0ZDI5OGQ3YmJiZTJjNDIzNjU3NDdfOUJFYTgwSmVrNjNrV3ZzSXVaTnBLamxGbzM2TmdYVDJfVG9rZW46Q2lhY2JjM09Mb3ZES3Z4WnpUVGMxUmhJblNiXzE3NzA1MTMzNjU6MTc3MDUxNjk2NV9WNA)

# 备注

## 国内安装方法

如果直接下载 istio 失败，可以使用这个步骤下载 1.26.0 安装包并解压：

```Bash
wget https://chengzhstor.blob.core.windows.net/k8slab/istio-1.26.0-linux-amd64.tar.gz
tar xf istio-1.26.0-linux-amd64.tar.gz
```

进入下载目录，随着产品的迭代，此处的版本号可能不同，请大家依据屏幕提示进行后两步操作

```Bash
cd istio-1.26.0/
```

设置环境变量

```Bash
export PATH="$PATH:/root/istio-1.26.0/bin"
```

安装

```Bash
istioctl manifest apply --set profile=demo -y
```

## 清理整个环境

清理 Bookinfo

```Bash
samples/bookinfo/platform/kube/cleanup.sh
```

卸载 istio

```Bash
kubectl delete -f samples/addons
istioctl manifest generate --set profile=demo | kubectl delete --ignore-not-found=true -f -
```

 Istio 卸载程序按照层次结构逐级的从 istio-system 命令空间中删除 RBAC 权限和所有资源。对于不存在的资源报错，可以安全的忽略掉，毕竟他们已经被分层的删除了。

删除命名空间 istio-system

```Bash
kubectl delete namespace istio-system
```

指示 Istio 自动注入 Envoy 边车代理的标签默认也不删除。 不需要的时候，使用下面命令删掉它。

```Bash
kubectl label namespace default istio-injection-
```

暂时无法在飞书文档外展示此内容