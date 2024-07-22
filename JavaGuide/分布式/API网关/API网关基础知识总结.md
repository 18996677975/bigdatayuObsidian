# 什么是网关？
微服务背景下，一个系统被拆分为多个服务，但是像安全认证，流量控制，日志，监控等功能是每个服务都需要的，没有网关的话，我们就需要在每个服务中单独实现，这使得我们做了很多重复的事情并且没有一个全局的视图来统一管理这些功能。

![[Pasted image 20240723000812.png]]

一般情况下，网关可以为我们提供请求转发、安全认证（身份/权限认证）、流量控制、负载均衡、降级熔断、日志、监控、参数校验、协议转换等功能。

上面介绍了这么多功能，实际上，网关主要做了两件事情：**请求转发** + **请求过滤**。

由于引入网关之后，会多一步网络转发，因此性能会有一点影响（几乎可以忽略不计，尤其是内网访问的情况下）。 另外，我们需要保障网关服务的高可用，避免单点风险。

如下图所示，网关服务外层通过 Nginx（其他负载均衡设备/软件也行） 进行负载转发以达到高可用。Nginx 在部署的时候，尽量也要考虑高可用，避免单点风险。

![[Pasted image 20240723000905.png]]

# 网关能提供哪些功能？
绝大部分网关可以提供下面这些功能（有一些功能需要借助其他框架或者中间件）：
- **请求转发**：将请求转发到目标微服务。
- **负载均衡**：根据各个微服务实例的负载情况或者具体的负载均衡策略配置对请求实现动态的负载均衡。
- **安全认证**：对用户请求进行身份验证并仅允许可信客户端访问 API，并且还能够使用类似 RBAC 等方式来授权。
- **参数校验**：支持参数映射与校验逻辑。
- **日志记录**：记录所有请求的行为日志供后续使用。
- **监控告警**：从业务指标、机器指标、JVM 指标等方面进行监控并提供配套的告警机制。
- **流量控制**：对请求的流量进行控制，也就是限制某一时刻内的请求数。
- **熔断降级**：实时监控请求的统计信息，达到配置的失败阈值后，自动熔断，返回默认值。
- **响应缓存**：当用户请求获取的是一些静态的或更新不频繁的数据时，一段时间内多次请求获取到的数据很可能是一样的。对于这种情况可以将响应缓存起来。这样用户请求可以直接在网关层得到响应数据，无需再去访问业务服务，减轻业务服务的负担。
- **响应聚合**：某些情况下用户请求要获取的响应内容可能会来自于多个业务服务。网关作为业务服务的调用方，可以把多个服务的响应整合起来，再一并返回给用户。
- **灰度发布**：将请求动态分流到不同的服务版本（最基本的一种灰度发布）。
- **异常处理**：对于业务服务返回的异常响应，可以在网关层在返回给用户之前做转换处理。这样可以把一些业务侧返回的异常细节隐藏，转换成用户友好的错误提示返回。
- **API 文档：** 如果计划将 API 暴露给组织以外的开发人员，那么必须考虑使用 API 文档，例如 Swagger 或 OpenAPI。
- **协议转换**：通过协议转换整合后台基于 REST、AMQP、Dubbo 等不同风格和实现技术的微服务，面向 Web Mobile、开放平台等特定客户端提供统一服务。
- **证书管理**：将 SSL 证书部署到 API 网关，由一个统一的入口管理接口，降低了证书更换时的复杂度。

![[Pasted image 20240723000948.png]]

# 有哪些常见的网关系统？
## Netflix Zuul
## Spring Cloud Gateway
SpringCloud Gateway 属于 Spring Cloud 生态系统中的网关，其诞生的目标是为了替代老牌网关 **Zuul**。准确点来说，应该是 Zuul 1.x。SpringCloud Gateway 起步要比 Zuul 2.x 更早。

为了提升网关的性能，SpringCloud Gateway 基于 Spring WebFlux 。Spring WebFlux 使用 Reactor 库来实现响应式编程模型，底层基于 Netty 实现同步非阻塞的 I/O。

![[Pasted image 20240723001052.png]]

Spring Cloud Gateway 不仅提供统一的路由方式，并且基于 Filter 链的方式提供了网关基本的功能，例如：安全，监控/指标，限流。

Spring Cloud Gateway 和 Zuul 2.x 的差别不大，也是通过过滤器来处理请求。不过，目前更加推荐使用 Spring Cloud Gateway 而非 Zuul，Spring Cloud 生态对其支持更加友好。

## OpenResty
## Kong
Kong 是一款基于 [OpenRestyopen in new window](https://github.com/openresty/) （Nginx + Lua）的高性能、云原生、可扩展、生态丰富的网关系统，主要由 3 个组件组成：
- Kong Server：基于 Nginx 的服务器，用来接收 API 请求。
- Apache Cassandra/PostgreSQL：用来存储操作数据。
- Kong Dashboard：官方推荐 UI 管理工具，当然，也可以使用 RESTful 方式 管理 Admin api。

![[Pasted image 20240723001129.png]]

由于默认使用 Apache Cassandra/PostgreSQL 存储数据，Kong 的整个架构比较臃肿，并且会带来高可用的问题。

Kong 提供了插件机制来扩展其功能，插件在 API 请求响应循环的生命周期中被执行。比如在服务上启用 Zipkin 插件：
```sh
curl -X POST http://kong:8001/services/{service}/plugins \
--data "name=zipkin"  \
--data "config.http_endpoint=http://your.zipkin.collector:9411/api/v2/spans" \
--data "config.sample_ratio=0.001"
```

Kong 本身就是一个 Lua 应用程序，并且是在 Openresty 的基础之上做了一层封装的应用。归根结底就是利用 Lua 嵌入 Nginx 的方式，赋予了 Nginx 可编程的能力，这样以插件的形式在 Nginx 这一层能够做到无限想象的事情。例如限流、安全访问策略、路由、负载均衡等等。编写一个 Kong 插件，就是按照 Kong 插件编写规范，写一个自己自定义的 Lua 脚本，然后加载到 Kong 中，最后引用即可。除了 Lua，Kong 还可以基于 Go 、JavaScript、Python 等语言开发插件，得益于对应的 PDK（插件开发工具包）。

![[Pasted image 20240723001210.png]]

## APISIX
## Shenyu


# 如何选择？
上面介绍的几个常见的网关系统，最常用的是 Spring Cloud Gateway、Kong、APISIX 这三个。

对于公司业务以 Java 为主要开发语言的情况下，Spring Cloud Gateway 通常是个不错的选择，其优点有：简单易用、成熟稳定、与 Spring Cloud 生态系统兼容、Spring 社区成熟等等。不过，Spring Cloud Gateway 也有一些局限性和不足之处， 一般还需要结合其他网关一起使用比如 OpenResty。并且，其性能相比较于 Kong 和 APISIX，还是差一些。如果对性能要求比较高的话，Spring Cloud Gateway 不是一个好的选择。

Kong 和 APISIX 功能更丰富，性能更强大，技术架构更贴合云原生。Kong 是开源 API 网关的鼻祖，生态丰富，用户群体庞大。APISIX 属于后来者，更优秀一些，根据 APISIX 官网介绍：“APISIX 已经生产可用，功能、性能、架构全面优于 Kong”。

下面简单对比一下二者：
- APISIX 基于 etcd 来做配置中心，不存在单点问题，云原生友好；而 Kong 基于 Apache Cassandra/PostgreSQL ，存在单点风险，需要额外的基础设施保障做高可用。
- APISIX 支持热更新，并且实现了毫秒级别的热更新响应；而 Kong 不支持热更新。
- APISIX 的性能要优于 Kong 。
- APISIX 支持的插件更多，功能更丰富。
