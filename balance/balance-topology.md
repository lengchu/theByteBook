# 4.3 负载均衡拓扑类型

前面介绍了负载均衡高层概览、四层负载均衡与七层负载均衡的区别，接下来介绍它们的分布式部署拓扑，建立对负载均衡的全局性的认识。

:::tip 注意
下面介绍的每种拓扑都适用于四层和七层负载均衡器。
:::

第一种类型为中间代理拓扑，如图 4-2 所示。这应该是大家最熟悉的负载均衡方式，负载均衡器位于客户端和后端服务器之间，负责分发流量到多个后端实例（realserver）。

典型的中间代理拓扑的方案有：

- 硬件设备：Cisco、Juniper、F5 Networks 等公司的产品。
- 云软件解决方案：阿里云的 SLB（Server Load Balancer），AWS 的 ELB（Elastic Load Balancer）、Azure 的 Load Balancer 和 Google Cloud 的 Cloud Load Balancing 等。
- 纯软件方案：Nginx、HAProxy、Envoy 等。

总结中间代理模式的优缺点是：
 - 优点是，配置简单，**用户只需通过 DNS 连接到负载均衡器，无需关注其他细节**。
 - 缺点是，**中间代理可能存在单点故障风险**，尤其是负载均衡这种集中式的设计，如果负载均衡器出现问题，会导致整个系统的访问中断。

:::center
  ![](../assets/balancer.svg)<br/>
 图 4-2 中间代理负载均衡拓扑
:::


第二种边缘代理拓扑实际上是中间代理拓扑的一个变种。

一个具体的边缘代理例子是：本书第二章 2.7 节提到的的动态请求“加速”技术。Aakamai 的服务部署在全球多个数据中心，用户的请求首先到达最近的 Aakamai 边缘节点，Aakamai 边缘节点接收请求后，进行安全性检查（如 DDoS 防护），并根据缓存策略决定是否直接提供缓存内容或将请求转发到源服务器。

总结边缘代理模式的优缺点是：
 - 优点是，将负载均衡逻辑（还有缓存、安全策略等）集中在网络边缘，能够显著减少延迟，提高网站的响应速度，增强安全性（预防 DDos）；
 - 缺点是，虽然边缘代理可以减少单点故障，但如果某个边缘节点出现问题，仍会影响到该节点所服务的用户。

:::center
  ![](../assets/balancer-edge-proxy.svg)<br/>
 图 4-3 网络边缘负载均衡拓扑
:::

为了解决中间代理拓扑模式的单点故障和扩展问题，出现了一些更复杂的解决方案。如将负载均衡器以 SDK 库的形式嵌入到客户端（如图 4-4 所示）。熟悉微服务开发的读者肯定知晓 Finagle、Eureka、Ribbon 和 Hystrix 等微服务治理方案。虽然这些库特性各异，但还是能总结出它们的共同优缺点：
- 优点是，将负载均衡器的功能完全下沉至客户端，从而避免了单点故障和扩展问题；
- 缺点是，必须为项目使用的每种开发语言实现相应的 SDK ，当项目非常复杂时，处理版本依赖等兼容问题时将非常棘手（本书第八章 8.2 节详细阐述了微服务框架的痛点，可阅读本节进一步了解）。

:::center
  ![](../assets/balancer-sdk.svg)<br/>
 图 4-4 客户端内嵌库实现负载均衡
:::

客户端内嵌库拓扑的一个变种是**Sidecar 拓扑**。近年来这种拓扑非常流行，被称为服务网格（service mesh）。Sidecar 代理模式背后的思想是：将流量导到另一个进程，牺牲一点（延迟）性能，实现客户端内嵌库模式的所有好处，而无任何语言绑定。当下，最流行的 Sidecar 代理有 Envoy、Linkerd，笔者将第八章详细阐述服务网格的模式的原理。

:::center
  ![](../assets/balancer-sidecar.svg)<br/>
 图 4-5 Sidecar 代理实现负载均衡
:::

总体上讲，中间代理类型正在演变为功能更为强大的“网关”，所有请求通过唯一的网关进入系统。在这种架构中，负载均衡器不仅需要执行基本的流量分发功能，还需提供额外的“API 网关”功能，如 TLS 卸载、流量限制、身份验证，以及复杂的流量路由等。另一方面，在处理东西向流量（服务间通信）的场景中，Sidecar 模式通过将代理部署在每个服务实例旁边，实现透明的流量管理、监控和安全功能，正逐渐取代其他所有的拓扑类型。