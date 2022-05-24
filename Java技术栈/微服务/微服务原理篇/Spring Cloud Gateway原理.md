## 问题描述

在微服务架构中，一个系统往往由多个微服务组成，而这些服务可能部署在不同机房、不同地区、不同域名下。这种情况下，客户端（例如浏览器、手机、软件工具等）想要直接请求这些服务，就需要知道它们具体的地址信息，例如 IP 地址、端口号等。

>这种客户端直接请求服务的方式存在以下问题：
>
>- 当服务数量众多时，客户端需要维护大量的服务地址，这对于客户端来说，是非常繁琐复杂的。
>- 在某些场景下可能会存在跨域请求的问题
>- 身份认证难度大，每个微服务需要独立认证



## API网关

API网关是一个搭建在客户端和微服务之间的服务，**我们可以在API网关中处理一些非业务功能的逻辑，例如权限验证、监控、缓存、请求路由等**。

API网关类似于设计模式中的门面设计，是系统对外的唯一接口。有了它，客户端会先将请求发送到API网关，然后由API网关根据请求的标识信息将请求转发到微服务实例。

![img](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/101P46212-0.png)

>对于服务数量众多、复杂度较高、规模比较大的系统来说，使用 API 网关具有以下好处：
>
>- 客户端通过 API 网关与微服务交互时，客户端只需要知道 API 网关地址即可，而不需要维护大量的服务地址，简化了客户端的开发。
>- 客户端直接与 API 网关通信，能够减少客户端与各个服务的交互次数。
>- 客户端与后端的服务耦合度降低。
>- 节省流量，提高性能，提升用户体验。
>- API 网关还提供了安全、流控、过滤、缓存、计费以及监控等 API 管理功能。





## Spring Cloud Gateway

Spring Cloud Gateway是Spring Cloud团队基于Spring 5.0、Spring Boot 2.0和 Project Reactor等技术开发的高性能APi网关组件。

Spring Cloud Gateway旨在提供一种简单而有效的途径来发送API，并为它们提供横切关注点，例如：安全性、监控/指标和弹性。

>spring Cloud Gateway是基于WebFlux框架实现的，而WebFlux框架底层则使用了高性能的Reactor模式通信框架Netty。

### Spring Cloud Gateway核心概念

Spring Cloud Gateway最主要的功能就是路由转发，而在定义转发规则时主要涉及了三个核心概念，如下表：

| 核心概念          | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| Route（路由）     | 网关最基本的模块。它由一个ID、一个目标URI、一组断言（Predicate）和一组过滤器（Filter）组成 |
| Predicate（断言） | 路由转发的判断条件，**我们可以通过Predicate对HTTP请求进行匹配，例如请求方式、请求路径、请求头、参数等**，如果请求与断言匹配成功，则将请求转发到相应的服务。 |
| Filter（过滤器）  | 过滤器，我们可以使用它对请求进行拦截和修改，还可以使用它对上文的相应进行**再处理** |

> 注意：其中Route和Predicate必须同时声明



### Spring Cloud Gateway的特征

Spring Cloud Gateway具有以下特征：

- 基于`spring Framework 5`、`Project Reactor `和`Spring Boot 2.0`构建
- 能够在任意请求属性上匹配路由
- predicates和filters是特定于路由的
- **集成了Hystrix熔断器**
- **集成了Spring Cloud DiscoveryClient（服务发现客户端）**
- 能够限制请求频率
- 能够重写请求路径



### Spring Cloud Gateway的工作流程

![Spring Cloud Gateway 工作流程](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/101P45T2-1.png)

**Spring Cloud Gateway工作流程说明如下：**

1. 客户端将请求发送到Spring Cloud Gateway上。
2. Spring Cloud Gateway通过`Gateway Handler Mapping`找到与请求相匹配的路由，将其发送到`Gateway Web handler`。
3. `Gateway Web handler`通过指定的过滤器链（Filter Chain），将请求转发到实际的服务节点中，执行业务逻辑返回响应结果。
4. 过滤器之间用虚线分开是因为过滤器可能会在转发请求之前（pre）或之后（post）执行业务逻辑。
5. 过滤器（Filter）可以在请求被转发到服务端前，对请求进行拦截和修改，例如参数校验、权限校验、流量控制、日志输出以及协议转换等
6. 过滤器可以在响应返回客户端之前，对响应进行拦截和再处理，例如修改响应内容或响应头、日志输出、流量控制等。
7. 响应原路返回给客户端

>总而言之，客户端发送到 Spring Cloud Gateway 的请求需要通过一定的匹配条件，才能定位到真正的服务节点。在将请求转发到服务进行处理的过程前后（pre 和 post），我们还可以对请求和响应进行一些精细化控制。
>
>Predicate 就是路由的匹配条件，而 Filter 就是对请求和响应进行精细化控制的工具。有了这两个元素，再加上目标 URI，就可以实现一个具体的路由了







## Predicate断言

Spring Cloud Gateway 通过 Predicate 断言来实现 Route 路由的匹配规则。简单点说，Predicate 是路由转发的判断条件，请求只有满足了 Predicate 的条件，才会被转发到指定的服务上进行处理。

使用 Predicate 断言需要注意以下 3 点：

- Route 路由与 Predicate 断言的对应关系为“一对多”，一个路由可以包含多个不同断言。
- 一个请求想要转发到指定的路由上，就必须同时匹配路由上的所有断言。
- 当一个请求同时满足多个路由的断言条件时，请求只会被首个成功匹配的路由转发。



![img](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/101P42B6-2.png )



常见的 Predicate 断言如下表（假设转发的 URI 为 http://localhost:8001）。



| 断言    | 示例                                                         | 说明                                                         |
| ------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Path    | - Path=/dept/list/**                                         | 当请求路径与 /dept/list/** 匹配时，该请求才能被转发到 http://localhost:8001 上。 |
| Before  | - Before=2021-10-20T11:47:34.255+08:00[Asia/Shanghai]        | 在 2021 年 10 月 20 日 11 时 47 分 34.255 秒之前的请求，才会被转发到 http://localhost:8001 上。 |
| After   | - After=2021-10-20T11:47:34.255+08:00[Asia/Shanghai]         | 在 2021 年 10 月 20 日 11 时 47 分 34.255 秒之后的请求，才会被转发到 http://localhost:8001 上。 |
| Between | - Between=2021-10-20T15:18:33.226+08:00[Asia/Shanghai],2021-10-20T15:23:33.226+08:00[Asia/Shanghai] | 在 2021 年 10 月 20 日 15 时 18 分 33.226 秒 到 2021 年 10 月 20 日 15 时 23 分 33.226 秒之间的请求，才会被转发到 http://localhost:8001 服务器上。 |
| Cookie  | - Cookie=name,c.biancheng.net                                | 携带 Cookie 且 Cookie 的内容为 name=c.biancheng.net 的请求，才会被转发到 http://localhost:8001 上。 |
| Header  | - Header=X-Request-Id,\d+                                    | 请求头上携带属性 X-Request-Id 且属性值为整数的请求，才会被转发到 http://localhost:8001 上。 |
| Method  | - Method=GET                                                 | 只有 GET 请求才会被转发到 http://localhost:8001 上。         |







## Spring Cloud Gateway动态路由

默认情况下，Spring Cloud Gateway 会根据服务注册中心（例如 Eureka Server）中维护的服务列表，以服务名（spring.application.name）作为路径创建动态路由进行转发，从而实现动态路由功能。

我们可以在配置文件中，将 Route 的 uri 地址修改为以下形式。

```yaml
lb://service-name
以上配置说明如下：
```

- lb：uri 的协议，表示开启 Spring Cloud Gateway 的负载均衡功能。
- service-name：服务名，Spring Cloud Gateway 会根据它获取到具体的微服务地址。







## Filter过滤器

通常情况下，出于安全方面的考虑，服务端提供的服务往往都会有一定的校验逻辑，例如用户登陆状态校验、签名校验等。

在微服务架构中，系统由多个微服务组成，所有这些服务都需要这些校验逻辑，此时我们就可以将这些校验逻辑写到 Spring Cloud Gateway 的 Filter 过滤器中。

### Filter 的分类

Spring Cloud Gateway 提供了以下两种类型的过滤器，可以对请求和响应进行精细化控制。

| 过滤器类型 | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| Pre 类型   | 这种过滤器在请求被转发到微服务之前可以对请求进行拦截和修改，例如参数校验、权限校验、流量监控、日志输出以及协议转换等操作。 |
| Post 类型  | 这种过滤器在微服务对请求做出响应后可以对响应进行拦截和再处理，例如修改响应内容或响应头、日志输出、流量监控等。 |


按照作用范围划分，Spring Cloud gateway 的 Filter 可以分为 2 类：

- `GatewayFilter`：应用在单个路由或者一组路由上的过滤器。
- `GlobalFilter`：应用在所有的路由上的过滤器。

### GatewayFilter 网关过滤器

`GatewayFilter` 是 Spring Cloud Gateway 网关中提供的一种应用在单个或一组路由上的过滤器。它可以对单个路由或者一组路由上传入的请求和传出响应进行拦截，并实现一些与业务无关的功能，比如登陆状态校验、签名校验、权限校验、日志输出、流量监控等。



Spring Cloud Gateway 内置了多达 31 种 `GatewayFilter`，下表中列举了几种常用的网关过滤器及其使用示例。

| 路由过滤器             | 描述                                                         | 参数                                                         | 使用示例                                               |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------ |
| AddRequestHeader       | 拦截传入的请求，并在请求上添加一个指定的请求头参数。         | name：需要添加的请求头参数的 key； value：需要添加的请求头参数的 value。 | - AddRequestHeader=my-request-header,1024              |
| AddRequestParameter    | 拦截传入的请求，并在请求上添加一个指定的请求参数。           | name：需要添加的请求参数的 key； value：需要添加的请求参数的 value。 | - AddRequestParameter=my-request-param,c.biancheng.net |
| AddResponseHeader      | 拦截响应，并在响应上添加一个指定的响应头参数。               | name：需要添加的响应头的 key； value：需要添加的响应头的 value。 | - AddResponseHeader=my-response-header,c.biancheng.net |
| PrefixPath             | 拦截传入的请求，并在请求路径增加一个指定的前缀。             | prefix：需要增加的路径前缀。                                 | - PrefixPath=/consumer                                 |
| PreserveHostHeader     | 转发请求时，保持客户端的 Host 信息不变，然后将它传递到提供具体服务的微服务中。 | 无                                                           | - PreserveHostHeader                                   |
| RemoveRequestHeader    | 移除请求头中指定的参数。                                     | name：需要移除的请求头的 key。                               | - RemoveRequestHeader=my-request-header                |
| RemoveResponseHeader   | 移除响应头中指定的参数。                                     | name：需要移除的响应头。                                     | - RemoveResponseHeader=my-response-header              |
| RemoveRequestParameter | 移除指定的请求参数。                                         | name：需要移除的请求参数。                                   | - RemoveRequestParameter=my-request-param              |
| RequestSize            | 配置请求体的大小，当请求体过大时，将会返回 413 Payload Too Large。 | maxSize：请求体的大小。                                      | - name: RequestSize   args:    maxSize: 5000000        |

### GlobalFilter 全局过滤器

`GlobalFilter` 是一种作用于所有的路由上的全局过滤器，通过它，我们可以实现一些统一化的业务功能，例如权限认证、IP 访问限制等。当某个请求被路由匹配时，那么所有的 GlobalFilter 会和该路由自身配置的 GatewayFilter 组合成一个过滤器链。

Spring Cloud Gateway 为我们提供了多种默认的 GlobalFilter，例如与转发、路由、负载均衡等相关的全局过滤器。但在实际的项目开发中，通常我们都会自定义一些自己的 GlobalFilter 全局过滤器以满足我们自身的业务需求，而很少直接使用 Spring Cloud Config 提供这些默认的 GlobalFilter。