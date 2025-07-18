---
title: Kubernetes Gateway API
description: 描述在 Istio 中如何配置 Kubernetes Gateway API。
weight: 50
aliases:
    - /zh/docs/tasks/traffic-management/ingress/service-apis/
    - /latest/zh/docs/tasks/traffic-management/ingress/service-apis/
keywords: [traffic-management,ingress, gateway-api]
owner: istio/wg-networking-maintainers
test: yes
---

除了它自己的流量管理 API 之外，
{{< boilerplate gateway-api-future >}}
本文描述 Istio 和 Kubernetes API 之间的差异，并提供了一个简单的例子，
向您演示如何配置 Istio 以使用 Gateway API 在服务网格集群外部暴露服务。
请注意，这些 API 是 Kubernetes [Service](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/)
和 [Ingress](https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress/) API 的积极发展演进。

{{< tip >}}
许多 Istio 流量管理文档均囊括了 Istio 或 Kubernetes API 的使用说明
（例如请参阅[控制入站流量](/zh/docs/tasks/traffic-management/ingress/ingress-control)）。
您可以按照[入门指南](/zh/docs/setup/getting-started/)从一开始就使用 Gateway API。
{{< /tip >}}

## 设置 {#setup}

1. 在大多数 Kubernetes 集群中，默认情况下不会安装 Gateway API。如果 Gateway API CRD 不存在，请安装：

    {{< text bash >}}
    $ kubectl get crd gateways.gateway.networking.k8s.io &> /dev/null || \
      { kubectl kustomize "github.com/kubernetes-sigs/gateway-api/config/crd?ref={{< k8s_gateway_api_version >}}" | kubectl apply -f -; }
    {{< /text >}}

1. 使用 `minimal` 配置安装 Istio：

    {{< text bash >}}
    $ istioctl install --set profile=minimal -y
    {{< /text >}}

## 与 Istio API 的区别 {#differences-from-istio-apis}

Gateway API 与 Istio API（如 Gateway 和 VirtualService）有很多相似之处。
主资源使用相同的 `Gateway` 名称，并且这些资源服务于相类似的目标。

新的 Gateway API 致力于从 Kubernetes 的各种 Ingress 实现（包括 Istio）中吸取经验，
以构建标准化的，独立于供应商的 API。
这些 API 通常与 Istio Gateway 和 VirtualService 具有相同的用途，但有一些关键的区别：

* Istio API 中的 `Gateway` **仅配置**[已部署](/zh/docs/setup/additional-setup/gateway/)的现有网关 Deployment/Service，
  而在 Gateway API 中的 `Gateway` 资源**不仅配置也会部署**网关。
  有关更多信息，请参阅具体[部署方法](#deployment-methods)。
* 在 Istio `VirtualService` 中，所有协议都在单一的资源中配置，
  而在 Gateway API 中，每种协议类型都有自己的资源，例如 `HTTPRoute` 和 `TCPRoute`。
* 虽然 Gateway API 提供了大量丰富的路由功能，但它还没有涵盖 Istio 的全部特性。
  因此，正在进行的工作是扩展 API 以覆盖这些用例，以及利用 API
  的[可拓展性](https://gateway-api.sigs.k8s.io/#gateway-api-concepts)
  来更好地暴露 Istio 的功能。

## 配置网关 {#configuring-a-gateway}

有关 API 的信息，请参阅 [Gateway API](https://gateway-api.sigs.k8s.io/) 文档。

在本例中，我们将部署一个简单的应用，并使用 `Gateway` 将其暴露到外部。

1. 首先部署一个 `httpbin` 测试应用：

    {{< text bash >}}
    $ kubectl apply -f @samples/httpbin/httpbin.yaml@
    {{< /text >}}

1. 部署 Gateway API 配置，包括单个暴露的路由（即 `/get`）：

    {{< text bash >}}
    $ kubectl create namespace istio-ingress
    $ kubectl apply -f - <<EOF
    apiVersion: gateway.networking.k8s.io/v1
    kind: Gateway
    metadata:
      name: gateway
      namespace: istio-ingress
    spec:
      gatewayClassName: istio
      listeners:
      - name: default
        hostname: "*.example.com"
        port: 80
        protocol: HTTP
        allowedRoutes:
          namespaces:
            from: All
    ---
    apiVersion: gateway.networking.k8s.io/v1
    kind: HTTPRoute
    metadata:
      name: http
      namespace: default
    spec:
      parentRefs:
      - name: gateway
        namespace: istio-ingress
      hostnames: ["httpbin.example.com"]
      rules:
      - matches:
        - path:
            type: PathPrefix
            value: /get
        backendRefs:
        - name: httpbin
          port: 8000
    EOF
    {{< /text >}}

1.  设置主机 Ingress 环境变量：

    {{< text bash >}}
    $ kubectl wait -n istio-ingress --for=condition=programmed gateways.gateway.networking.k8s.io gateway
    $ export INGRESS_HOST=$(kubectl get gateways.gateway.networking.k8s.io gateway -n istio-ingress -ojsonpath='{.status.addresses[0].value}')
    {{< /text >}}

1.  使用 **curl** 访问 `httpbin` 服务：

    {{< text bash >}}
    $ curl -s -I -HHost:httpbin.example.com "http://$INGRESS_HOST/get"
    ...
    HTTP/1.1 200 OK
    ...
    server: istio-envoy
    ...
    {{< /text >}}

    请注意，使用 `-H` 标志可以将 **Host** HTTP 标头设置为
    "httpbin.example.com"。这一步是必需的，因为 `HTTPRoute` 已配置为处理 "httpbin.example.com" 的请求，
    但是在测试环境中，该主机没有 DNS 绑定，只是将请求发送到入口 IP。

1.  访问其他没有被显式暴露的 URL 时，将看到 HTTP 404 错误：

    {{< text bash >}}
    $ curl -s -I -HHost:httpbin.example.com "http://$INGRESS_HOST/headers"
    HTTP/1.1 404 Not Found
    ...
    {{< /text >}}

1.  更新路由规则也会暴露 `/headers` 并为请求添加标头：

    {{< text bash >}}
    $ kubectl apply -f - <<EOF
    apiVersion: gateway.networking.k8s.io/v1
    kind: HTTPRoute
    metadata:
      name: http
      namespace: default
    spec:
      parentRefs:
      - name: gateway
        namespace: istio-ingress
      hostnames: ["httpbin.example.com"]
      rules:
      - matches:
        - path:
            type: PathPrefix
            value: /get
        - path:
            type: PathPrefix
            value: /headers
        filters:
        - type: RequestHeaderModifier
          requestHeaderModifier:
            add:
            - name: my-added-header
              value: added-value
        backendRefs:
        - name: httpbin
          port: 8000
    EOF
    {{< /text >}}

1.  再次访问 `/headers`，注意到 `My-Added-Header` 标头已被添加到请求：

    {{< text bash >}}
    $ curl -s -HHost:httpbin.example.com "http://$INGRESS_HOST/headers" | jq '.headers["My-Added-Header"][0]'
    ...
    "added-value"
    ...
    {{< /text >}}

## 部署方法 {#deployment-methods}

在上面的示例中，在配置网关之前，您不需要安装 Ingress 网关 `Deployment`。
因为在默认配置中会根据 `Gateway` 配置自动分发网关 `Deployment` 和 `Service`。
但是对于高级别的用例，仍然允许手动部署。

### 自动部署 {#automated-deployment}

默认情况下，每个 `Gateway` 将被自动​​配置一个 `Service` 和 `Deployment`。
它们将被命名为 `<Gateway name>-<GatewayClass name>`
（`istio-waypoint` `GatewayClass` 除外，它不被添加后缀）。
如果 `Gateway` 发生变化（例如，如果被添加了新端口），这些配置将被自动更新。

可以使用 `infrastructure` 字段定制这些资源：

{{< text yaml >}}
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: gateway
spec:
  infrastructure:
    annotations:
      some-key: some-value
    labels:
      key: value
    parametersRef:
      group: ""
      kind: ConfigMap
      name: gw-options
  gatewayClassName: istio
{{< /text >}}

`labels` 和 `annotations` 下的键值对将被复制到生成的资源上。
`parametersRef` 可用于完全自定义生成的资源。
这必须引用与 `Gateway` 位于同一命名空间中的 `ConfigMap`。

示例配置：

{{< text yaml >}}
apiVersion: v1
kind: ConfigMap
metadata:
  name: gw-options
data:
  horizontalPodAutoscaler: |
    spec:
      minReplicas: 2
      maxReplicas: 2

  deployment: |
    metadata:
      annotations:
        additional-annotation: some-value
    spec:
      replicas: 4
      template:
        spec:
          containers:
          - name: istio-proxy
            resources:
              requests:
                cpu: 1234m

  service: |
    spec:
      ports:
      - "\$patch": delete
        port: 15021
{{< /text >}}

这些配置将使用[策略合并补丁](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-api-machinery/strategic-merge-patch.md)策略覆盖在生成的资源之上。
以下键有效：
* `service`
* `deployment`
* `serviceAccount`
* `horizontalPodAutoscaler`
* `podDisruptionBudget`

{{< tip >}}
默认情况下不会创建 `Horizo​​ntalPodAutoscaler` 和 `PodDisruptionBudget`。
但是，如果自定义中存在相应字段，则会创建它们。
{{< /tip >}}

#### GatewayClass 默认值 {#gatewayclass-defaults}

可以为每个 `GatewayClass` 配置所有 `Gateway` 的默认值。
这通过带有标签 `gateway.istio.io/defaults-for-class: <gateway class name>`
的 `ConfigMap` 完成。此 `ConfigMap`
必须位于[根命名空间](/zh/docs/reference/config/istio.mesh.v1alpha1/#MeshConfig-root_namespace)
（通常为 `istio-system`）中。每个 `GatewayClass` 只允许一个 `ConfigMap`。
此 `ConfigMap` 采用与 `Gateway` 的 `ConfigMap` 相同的格式。

自定义可能同时存在于 `GatewayClass` 和 `Gateway` 上。
如果两者都存在，则 `Gateway` 自定义在 `GatewayClass` 自定义之后应用。

这个 `ConfigMap` 也可以在安装时被创建。例如：

{{< text yaml >}}
kind: IstioOperator
spec:
  values:
    gatewayClasses:
      istio:
        deployment:
          spec:
            replicas: 2
{{< /text >}}

#### 资源附加和扩缩 {#resource-attachment-and-scaling}

资源可以附加到 `Gateway` 进行自定义。
然而，大多数 Kubernetes 资源目前不支持直接附加到 `Gateway`，
但这些资源可以转为直接被附加到相应生成的 `Deployment` 和 `Service`。
这很容易做到，因为[资源是用众所周知的标签](https://gateway-api.sigs.k8s.io/geps/gep-1762/#resource-attachment)
（`gateway.networking.k8s.io/gateway-name: <gateway name>`）和名称生成的：

* Gateway: `<gateway name>-<gateway class name>`
* waypoint: `<gateway name>`

例如，参照以下部署类别为 `HorizontalPodAutoscaler` 和 `PodDisruptionBudget` 的 `Gateway`：

{{< text yaml >}}
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: gateway
spec:
  gatewayClassName: istio
  listeners:
  - name: default
    hostname: "*.example.com"
    port: 80
    protocol: HTTP
    allowedRoutes:
      namespaces:
        from: All
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: gateway
spec:
  # 通过引用与生成的 Deployment 匹配
  # 注意不要使用 `kind: Gateway`
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: gateway-istio
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: gateway
spec:
  minAvailable: 1
  selector:
    # 通过标签匹配生成的 Deployment
    matchLabels:
      gateway.networking.k8s.io/gateway-name: gateway
{{< /text >}}

### 手动部署 {#manual-deployment}

如果您不希望使用自动部署，可以进行[手动配置](/zh/docs/setup/additional-setup/gateway/)
`Deployment` 和 `Service`。

完成此选项后，您将需要手动将 `Gateway` 链接到 `Service`，并保持它们的端口配置同步。

为了支持策略附件，例如当您在 AuthorizationPolicy 上使用
[`targetRef`](/zh/docs/reference/config/type/workload-selector/#PolicyTargetReference) 字段时，
您还需要通过向网关 Pod 添加以下标签来引用 `Gateway` 的名称：
`gateway.networking.k8s.io/gateway-name: <gateway name>`。

要将 `Gateway` 链接到 `Service`，需要将 `addresses` 字段配置为指向**单个** `Hostname`。

{{< tip >}}
如果 `Service` 与 `Gateway` 位于不同的命名空间中，Istio 的控制器将不会配置 `Service`。
{{< /tip >}}

{{< text yaml >}}
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: gateway
spec:
  addresses:
  - value: ingress.istio-gateways.svc.cluster.local
    type: Hostname
...
{{< /text >}}

## 网格流量 {#mesh-traffic}

Gateway API 也可以用来配置网格流量。
具体做法是配置 `parentRef` 指向一个服务，而不是指向一个 Gateway。

例如，要将所有调用的头部添加到一个名为 `example` 的集群内 `Service`：

{{< text yaml >}}
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: mesh
spec:
  parentRefs:
  - group: ""
    kind: Service
    name: example
  rules:
  - filters:
    - type: RequestHeaderModifier
      requestHeaderModifier:
        add:
        - name: my-added-header
          value: added-value
    backendRefs:
    - name: example
      port: 80
{{< /text >}}

有关更多详情和示例，请参阅其他[流量管理](/zh/docs/tasks/traffic-management/)。

## 清理 {#cleanup}

1. 删除 `httpbin` 示例和网关：

    {{< text bash >}}
    $ kubectl delete -f @samples/httpbin/httpbin.yaml@
    $ kubectl delete httproute http
    $ kubectl delete gateways.gateway.networking.k8s.io gateway -n istio-ingress
    $ kubectl delete ns istio-ingress
    {{< /text >}}

1. 卸载 Istio：

    {{< text bash >}}
    $ istioctl uninstall -y --purge
    $ kubectl delete ns istio-system
    $ kubectl delete ns istio-ingress
    {{< /text >}}

1. 如果不再需要这些 Gateway API CRD 资源，请移除：

    {{< text bash >}}
    $ kubectl kustomize "github.com/kubernetes-sigs/gateway-api/config/crd?ref={{< k8s_gateway_api_version >}}" | kubectl delete -f -
    {{< /text >}}
