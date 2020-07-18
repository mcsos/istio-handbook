---
authors: ["mcsos"]
reviewers: [""]
---

# Ingress/Egress

## Ingress

### 概述

Istio 通过 Istio Ingress 来将集群内部的服务暴露到集群外部，Istio Ingress 相对于 Kubernetes Ingress 的功能做了很多增强，包括四至七层的负载均衡器、TLS配置等等，并且可以将 Istio 在集群内部的流量控制和安全等功能按照同样的方式用于对外提供的服务中，而且可以将 Ingress 与内部的对象统一进行管理。

Istio Ingress 功能会提供一个 Gateway 的 CRD 对象来让用户对对外暴露的服务进行配置，同时，可以将 Virtual Service 和 Destination Rule 等对应结合起来使用。需要说明的是 Gateway 除了可以用于配置从集群外部访问集群内的服务之外，还可以用于配置从集群内部访问集群外部服务的情况。

下面是一个 Gateway 的例子：

``` yaml
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

这个例子中配置了用户可以使用 `httpbin.example.com` 这个 host 来访问集群内部的 `httpbin` 服务，使用的端口号为 80。

Gateway 对象本身只负责在 Istio Ingress 上的配置， 使 Istio Ingress 可以识别这个请求并且将其放行，但是从 Istio Ingress 到集群内部的服务之间还需要一个 Virtual Service 对象来搭配使用，配置如下：

``` yaml
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
  - route:
    - destination:
        host: httpbin
```

这个 Virtual Service 使用 `gateways` 字段与刚才配置的 Istio Ingress 对象关联到一起，将到达 Istio Ingress 上的请求转发到集群内部的 `httpbin` 服务上。
