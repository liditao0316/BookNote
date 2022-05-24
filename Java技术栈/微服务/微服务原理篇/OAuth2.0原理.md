# 基础篇01

[OAuth 2.0是要通过什么方式解决什么问题？](https://time.geekbang.org/column/article/254565)

## OAuth 2.0 是什么？

>OAuth2.0授权的核心就是颁发访问令牌、使用访问令牌。OAuth解决的是用token来代替用户名和密码传输

### OAuth2.0运转流程

- 流程中核心的是颁发访问令牌和使用访问令牌

- 授权码模式只是其中一个运用场景。

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/77197844a8f41a33cb68947b1dc9ee79.png" alt="img" style="zoom:50%;" />

### OAuth2.0使用场景

>除了**授权码许可类型**外，OAuth 2.0 针对不同的使用场景，还有 3 种基础的许可类型，分别是:
>
>- 隐式许可（Implicit）
>- 客户端凭据许可（Client Credentials）
>- 资源拥有者凭据许可（Resource Owner Password Credentials）





# 基础篇02

[授权码许可类型中，为什么一定要有授权码？](https://time.geekbang.org/column/article/256196)

>在 OAuth 2.0 的体系里面有 4 种角色，按照官方的称呼它们分别是资源拥有者、客户端(第三方软件)、授权服务和受保护资源

## 授权码的作用

- 如果没有第二次重定向，那么即使完成了授权操作，小明也不能跳回到小兔软件上。
- 如果直接将token暴露到浏览器上，即进行第二次重定向，并附带token。显然这是不被允许的。
- 授权服务界面是授权服务平台生成的。
- **通过授权码的方式，可以让用户小明在授权服务上给小兔授权之后，还能重新回到小兔的操作页面上。**

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/96973a6f5637fb3d1049f6d456702932.png" alt="img" style="zoom:50%;" />



## 授权码许可类型的通信过程

### 间接通信

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/9e4f51f1f77840bd0b8f756be40d42bf.jpg" alt="img" style="zoom:30%;" />

### 直接通信

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/84dc2d6f578b6968b782a0280a73be9b.png" alt="img" style="zoom:40%;" />

### 两个 “一伙”

OAuth 2.0 中的 4 个角色是 “两两站队” 的：资源拥有者和第三方软件“站在一起”，因为第三方软件要代表资源拥有者去访问受保护资源；授权服务和受保护资源“站在一起”，因为授权服务负责颁发访问令牌，受保护资源负责接收并验证访问令牌。

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/1c86e21496882894d7f03b35a01972ff.jpg" alt="img" style="zoom:30%;" />

## 总结

今天，我从为什么需要授权码这个问题开始讲起，并通过授权码把授权码许可流程整体的通信过程串了一遍，提到了授权码这种方式解决的问题，也提到了授权码流程的通信方式。总结来说，我需要你记住以下两点。

授权码许可流程有两种通信方式。

- 一种是前端通信，因为它通过浏览器促成了授权码的交互流程，比如京东商家开放平台的授权服务生成授权码发送到浏览器，第三方软件小兔从浏览器获取授权码。**正因为获取授权码的时候小兔软件和授权服务并没有发生直接的联系，也叫做间接通信**。
- 另外一种是后端通信，在小兔软件获取到授权码之后，**在后端服务直接发起换取访问令牌的请求，也叫做直接通信**。
- 在 OAuth 2.0 中，访问令牌被要求有极高的安全保密性，因此我们不能让它暴露在浏览器上面，只能通过第三方软件（比如小兔）的后端服务来获取和使用，以最大限度地保障访问令牌的安全性。正因为访问令牌的这种安全要求特性，当需要前端通信，比如浏览器上面的流转的时候，OAuth 2.0 才又提供了一个临时的凭证：授权码。**通过授权码的方式，可以让用户小明在授权服务上给小兔授权之后，还能重新回到小兔的操作页面上**。这样，在保障安全性的情况下，提升了小明在小兔上的体验。

从授权码许可流程中就可以看出来，它完美地将 OAuth 2.0 的 4 个角色组织了起来，并保证了它们之间的顺畅通信。**它提出的这种结构和思想都可以被迁移到其他环境或者协议上，比如在微信小程序中使用授权码许可。**

不过，也正是因为有了授权码的参与，才使得授权码许可要比其他授权许可类型，在授权的流程上多出了好多步骤，让授权码许可类型成为了 OAuth 2.0 体系中迄今流程最完备、安全性最高的授权流程。在接下来的两讲中，我还会为你重点讲解授权码许可类型下的授权服务。







# 基础篇03

[授权服务：授权码和访问令牌的颁发流程是怎样的？](https://time.geekbang.org/column/article/257101)

![a5d231c5b356ecf2031yy7d17207c011](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/a5d231c5b356ecf2031yy7d17207c011-20220519172333777.png)

>以京东开放平台为例
>
>- 如果要需要授权登陆京东，那么该第三方平台（客户端）需要在京东开放平台上**注册**。注册完后，京东商家开放平台就会给小兔软件 **app_id** 和 **app_secret** 等信息，以方便后面授权时的各种身份校验。
>
>- 同时，注册的时候，第三方软件也会请求受保护资源的可访问范围。比如，小兔能否获取小明店铺 3 个月以前的订单，能否获取每条订单的所有字段信息等等。这个权限范围，就是 scope。

```xml
力扣调用微信授权接口
https://open.weixin.qq.com/connect/qrconnect?appid=wx7598c387d69bfc45&redirect_uri=https%3A%2F%2Fleetcode-cn.com%2Faccounts%2Fweixin%2Flogin%2Fcallback%3Ffrom%3Dleetcode.cn&response_type=code&scope=snsapi_login&state=rBnsM2OJ
```

>颁发授权码code和颁发访问令牌都是在授权服务上完成的

## 过程一：颁发授权码 code

### 为什么要有两次权限验证

- 第一次权限验证：验证的是客户端注册时申请的权限
- 第二次权限验证：验证的是资源拥有者选择的权限

### 为什么知道app_id、app_secret以及code也不会存在安全问题

- 因为第二次重定向操作是服务端完成的，确保code重定向到一个合法的地址上。同时，当客户端申请访问令牌时需要携带app_secret(类似于密码)发送给授权服务器，这样就能够保证。
- app_secret是客户端后台管理的，如果泄露出去就是客户端相关人员的责任了。
- app_secret在申请访问令牌的时候才会用到。

## 过程二：颁发访问令牌

## 刷新令牌

刷新令牌也是给第三方软件使用的，同样需要遵循先颁发再使用的原则。刷新令牌存在的初衷是，在访问令牌失效的情况下，为了不让用户频繁手动授权，用来通过系统重新请求**生成一个新的访问令牌**。

### 颁发刷新令牌

>其实，颁发刷新令牌和颁发访问令牌是一起实现的，都是在过程二的步骤三生成访问令牌 access_token 中生成的。也就是说，第三方软件得到一个访问令牌的同时，也会得到一个刷新令牌。

### 使用刷新令牌

说到刷新令牌的使用，我们需要先明白一点。在 OAuth 2.0 规范中，刷新令牌是一种特殊的授权许可类型，是嵌入在授权码许可类型下的一种特殊许可类型。在授权服务的代码里，当我们接收到这种授权许可请求的时候，会先比较 grant_type 和 refresh_token 的值，然后做下一步处理。

#### 第一步，接收刷新令牌请求，验证基本信息。

此时请求中的 grant_type 值为 refresh_token。

```java
String grantType = request.getParameter("grant_type");
if("refresh_token".equals(grantType)){
  
}
```

和颁发访问令牌前的验证流程一样，这里我们也需要验证第三方软件是否存在。需要注意的是，这里需要同时验证刷新令牌是否存在，目的就是要保证传过来的刷新令牌的合法性。

```java
String refresh_token = request.getParameter("refresh_token");

if(!refreshTokenMap.containsKey(refresh_token)){
    //该refresh_token值不存在
}
```

另外，我们还需要验证刷新令牌是否属于该第三方软件。授权服务是将颁发的刷新令牌与第三方软件、当时的授权用户绑定在一起的，因此这里需要判断该刷新令牌的归属合法性。

```java
String appStr = refreshTokenMap.get("refresh_token");
if(!appStr.startsWith(appId+"|"+"USERTEST")){
    //该refresh_token值不是颁发给该第三方软件的
}
```



**需要注意，一个刷新令牌被使用以后，授权服务需要将其废弃，并重新颁发一个刷新令牌。**

#### 第二步，重新生成访问令牌。

生成访问令牌的处理流程，与颁发访问令牌环节的生成流程是一致的。授权服务会将新的访问令牌和新的刷新令牌，一起返回给第三方软件。这里就不再赘述了。



# 基础篇04

[在OAuth2.0中，如何使用JWT结构化令牌](https://time.geekbang.org/column/article/257747)

## JWT结构化令牌组成

>- HEADER（头部）
>- PAYLOAD（数据体）
>- SIGNATURE（签名）
>
>**JWT 的整体结构，是被句点符号分割的三段内容，结构为**	` header.payload.signature`

### HEADER

`HEADER` 表示装载令牌类型和算法等信息，是 JWT 的头部。其中，typ 表示第二部分 PAYLOAD 是 JWT 类型，alg 表示使用 HS256 对称签名的算法。

### PAYLOAD

`PAYLOAD` 表示是 JWT 的数据体，代表了一组数据。其中，sub（令牌的主体，一般设为资源拥有者的唯一标识）、exp（令牌的过期时间戳）、iat（令牌颁发的时间戳）是 JWT 规范性的声明，代表的是常规性操作。更多的通用声明，你可以参考RFC 7519 开放标准。不过，在一个 JWT 内可以包含一切合法的 JSON 格式的数据，也就是说，PAYLOAD 表示的一组数据允许我们自定义声明。

### SIGNATURE

`SIGNATURE` 表示对 JWT 信息的签名。那么，它有什么作用呢？我们可能认为，有了 HEADER 和 PAYLOAD 两部分内容后，就可以让令牌携带信息了，似乎就可以在网络中传输了，但是在网络中传输这样的信息体是不安全的，因为你在“裸奔”啊。所以，我们还需要对其进行加密签名处理，而 SIGNATURE 就是对信息的签名结果，当受保护资源接收到第三方软件的签名后需要验证令牌的签名是否合法。**（相当于网络传输中的数字签名）**

## 令牌内检

**受保护资源来调用授权服务提供的检验令牌的服务，我们把这种校验令牌的方式称为令牌内检。**

在如今已经成熟的分布式以及微服务的环境下，不同的系统之间是依靠服务而不是数据库来通信了，比如授权服务给受保护资源服务提供一个 RPC 服务。

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/963bb5dfc504c700fce24c8aac0dd2bf.png" alt="img" style="zoom:50%;" />

## 使用JWT

**授权服务“扔出”一个令牌，受保护资源服务“接住”这个令牌，然后自己开始解析令牌本身所包含的信息就可以了，而不需要再去查询数据库或者请求RPC服务。**

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/1a4cf53349aeb5d588e27c608e06d539.png" alt="img" style="zoom:50%;" />

> 在传输过程中，JWT 令牌需要进行 Base64 编码以防止乱码，同时还需要进行**签名**及**加密处理**来防止数据信息泄露。 而这些工作可以通过JJWT工具来解决。

JJWT 是目前 Java 开源的、比较方便的 JWT 工具，封装了 Base64URL 编码和对称 HMAC、非对称 RSA 的一系列签名算法。



## 为什么使用JWT令牌

1. **JWT 的核心思想，就是用计算代替存储，有些 “时间换空间” 的 “味道”**。当然，这种经过计算并结构化封装的方式，也减少了“共享数据库” 因远程调用而带来的网络传输消耗，所以也有可能是节省时间的。
2. 也是一个重要特性，是加密。因为 JWT 令牌内部已经包含了重要的信息，所以在整个传输过程中都必须被要求是密文传输的，**这样被强制要求了加密也就保障了传输过程中的安全性**。这里的加密算法，既可以是对称加密，也可以是非对称加密。**注意这里加密和解密的双方分别是授权服务和受保护资源服务。**
3. **使用 JWT 格式的令牌，有助于增强系统的可用性和可伸缩性**。这一点要怎么理解呢？我们前面讲到了，这种 JWT 格式的令牌，通过“自编码”的方式包含了身份验证需要的信息，不再需要服务端进行额外的存储，所以每次的请求都是无状态会话。这就符合了我们尽可能遵循无状态架构设计的原则，也就是增强了系统的可用性和伸缩性。



## JWT令牌失效的问题

### 问题描述：

- 小明在使用小兔软件的时候，是不是有可能因为某种原因修改了在京东的密码，或者是不是有可能突然取消了给小兔的授权？这时候，令牌的状态是不是就要有相应的变更，将原来对应的令牌置为无效。

- 但，使用 JWT 格式令牌时，每次颁发的令牌都不会在服务端存储，这样我们要改变令牌状态的时候，就无能为力了。因为服务端并没有存储这个 JWT 格式的令牌。这就意味着，JWT 令牌在有效期内，是可以“横行无止”的。

>解决这个问题的关键还是在**加密密钥**上。
>
>1. 将**每次生成 JWT 令牌时的秘钥粒度缩小到用户级别**，也就是一个用户一个秘钥。这样，当用户取消授权或者修改密码后，就可以让这个密钥一起修改。一般情况下，这种方案需要配套一个单独的密钥管理服务。
>2. 在不提供用户主动取消授权的环境里面，如果只考虑到修改密码的情况，那么我们就可以**把用户密码作为 JWT 的密钥**。当然，这也是用户粒度级别的。这样一来，用户修改密码也就相当于修改了密钥。





# 基础篇05

[如何安全、快速地接入OAuth2.0](https://time.geekbang.org/column/article/257767)

## 构建第三方软件应用

> 开发第三方软件应用的过程：`注册信息`、`引导授权`、`使用访问令牌`、`使用刷新令牌`。

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/ee18ea7aab4fbee26cf23d7613801078.png" alt="img" style="zoom:50%;" />

### 第一点，注册信息

首先，小兔软件只有先有了身份，才可以参与到 OAuth 2.0 的流程中去。也就是说，小兔软件需要先拥有自己的 app_id 和 app_serect 等信息，同时还要填写自己的回调地址 redirect_uri、申请权限等信息。

这种方式的注册呢，我们有时候也称它为**静态注册**，也就是小兔软件的研发人员提前登录到京东商家开放平台进行手动注册，以便后续使用这些注册的相关信息来请求访问令牌。

### 第二点，引导授权

当用户需要使用第三方软件，来操作其在受保护资源上的数据，就需要第三方软件来引导授权。说白了就是发送一个请求，授权服务器会返回一个授权界面

`https://open.weixin.qq.com/connect/qrconnect?appid=wx7598c387d69bfc45&redirect_uri=https%3A%2F%2Fleetcode-cn.com%2Faccounts%2Fweixin%2Flogin%2Fcallback%3Ffrom%3Dleetcode.cn&response_type=code&scope=snsapi_login&state=QH5FTPvx`

### 第三点，使用访问令牌

拿到令牌后去使用令牌，才是第三方软件的最终目的。然后我们看看如何使用令牌。目前 OAuth 2.0 的令牌只支持一种类型，那就是 bearer 令牌，也就是我之前讲到的可以是任意字符串格式的令牌。官方规范给出的使用访问令牌请求的方式，有三种:

#### Form-Encoded Body Parameter（表单参数）

```http
POST /resource HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded
access_token=b1a64d5c-5e0c-4a70-9711-7af6568a61fb
```

#### URI Query Parameter（URI 查询参数）

```http
GET /resource?access_token=b1a64d5c-5e0c-4a70-9711-7af6568a61fb HTTP/1.1
Host: server.example.com
```

#### Authorization Request Header Field（授权请求头部字段）

```http
GET /resource HTTP/1.1
Host: server.example.com
Authorization: Bearer b1a64d5c-5e0c-4a70-9711-7af6568a61fb
```



### 第四点，使用刷新令牌

使用刷新令牌的方式跟使用访问令牌是一样的，具体可以参照上面我们讲的访问令牌的方式。关于刷新令牌的使用，你最需要关心的是，什么时候你会来决定使用刷新令牌。

在小兔打单软件收到访问令牌的同时，也会收到访问令牌的过期时间 expires_in。一个设计良好的第三方应用，**应该将 expires_in 值保存下来并定时检测**；如果发现 expires_in 即将过期，则需要利用 refresh_token 去重新请求授权服务，以便获取新的、有效的访问令牌。

这种定时检测的方法可以提前发现访问令牌是否即将过期。此外，还有一种方法是“现场”发现。也就是说，比如小兔软件访问小明店铺订单的时候，突然收到一个访问令牌失效的响应，此时小兔软件立即使用 refresh_token 来请求一个访问令牌，以便继续代表小明使用他的数据。

综合来看的话，**定时检测的方式**，需要我们额外开发一个定时任务；而**“现场”发现**，就没有这种额外的工作量啦。具体采用哪一种方式，你可以结合自己的实际情况。不过呢，我还是建议你采用定时检测这种方式，因为它可以带来“提前量”，以便让我们有更好的主动性，而现场发现就有点被动了。

## 构建受保护资源服务

权限的分类主要分为以下几类：

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/e7b134686b9f2e824ffa8410e20f59f6.jpg" alt="img" style="zoom:50%;" />

### 不同的权限对应的不用的操作

这里的操作，其实对应的是 Web API，比如目前京东商家开放平台提供有查询商品 API、新增商品 API、删除商品 API 这三种。如果小兔软件请求过来的一个访问令牌 access_token 的 scope 权限范围只对应了查询商品 API、新增商品 API，那么包含这个 access_token 值的请求，就不能执行删除商品 API 的操作。

```java

//不同的权限对应不同的操作
String[] scope = OauthServlet.tokenScopeMap.get(accessToken);

StringBuffer sbuf = new StringBuffer();
for(int i=0;i<scope.length;i++){
    sbuf.append(scope[i]).append("|");
}

if(sbuf.toString().indexOf("query")>0){
    queryGoods("");
}

if(sbuf.toString().indexOf("add")>0){
    addGoods("");
}

if(sbuf.toString().indexOf("del")>0){
    delGoods("");
}
```

### 不同的权限对应不同的数据



### 不同的用户对应不同的数据

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/a5175438e76411c808dd5e72d3d3dbb0.png" alt="img" style="zoom:50%;" />

# 基础篇06

## 资源拥有者凭证许可

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/cd596cfd73a42449a39342f951c5cce9.png" alt="img" style="zoom:40%;" />

**步骤 1**：当用户访问第三方软件小兔时，会提示输入用户名和密码。索要用户名和密码，就是资源拥有者凭据许可类型的特点。

**步骤 2**：这里的 grant_type 的值为 password，告诉授权服务使用资源拥有者凭据许可凭据的方式去请求访问。

```java
Map<String, String> params = new HashMap<String, String>();
params.put("grant_type","password");
params.put("app_id","APPIDTEST");
params.put("app_secret","APPSECRETTEST");
params.put("name","NAMETEST");
params.put("password","PASSWORDTEST");

String accessToken = HttpURLClient.doPost(oauthURl,HttpURLClient.mapToStr(params));
```

**步骤 3**：授权服务在验证用户名和密码之后，生成 access_token 的值并返回给第三方软件。

```java
if("password".equals(grantType)){
    String appSecret = request.getParameter("app_secret");
    String username = request.getParameter("username");
    String password = request.getParameter("password");

    if(!"APPSECRETTEST".equals(appSecret)){
        response.getWriter().write("app_secret is not available");
        return;
    }
    if(!"USERNAMETEST".equals(username)){
        response.getWriter().write("username is not available");
        return;
    }
    if(!"PASSWORDTEST".equals(password)){
        response.getWriter().write("password is not available");
        return;
    }
    String accessToken = generateAccessToken(appId,"USERTEST");//生成访问令牌access_token的值
    response.getWriter().write(accessToken);
}
```



## 客户端凭证许可

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/cbc8cc1e03cb1d0a2f945ffd9dbb37ff.png" alt="img" style="zoom:40%;" />

另外一点呢，因为授权过程没有了资源拥有者小明的参与，小兔软件的后端服务可以随时发起 access_token 的请求，所以这种授权许可也不需要刷新令牌。

步骤 1：第三方软件小兔通过后端服务向授权服务发送请求，这里 grant_type 的值为 client_credentials，告诉授权服务要使用第三方软件凭据的方式去请求访问。

```java
Map<String, String> params = new HashMap<String, String>();
params.put("grant_type","client_credentials");
params.put("app_id","APPIDTEST");
params.put("app_secret","APPSECRETTEST");

String accessToken = HttpURLClient.doPost(oauthURl,HttpURLClient.mapToStr(params));
```

步骤 2：在验证 app_id 和 app_secret 的合法性之后，生成 access_token 的值并返回。

```java
String grantType = request.getParameter("grant_type");
String appId = request.getParameter("app_id");

if(!"APPIDTEST".equals(appId)){
    response.getWriter().write("app_id is not available");
    return;
}
if("client_credentials".equals(grantType)){
    String appSecret = request.getParameter("app_secret");
    if(!"APPSECRETTEST".equals(appSecret)){
        response.getWriter().write("app_secret is not available");
        return;
    }
    String accessToken = generateAccessToken(appId,"USERTEST");//生成访问令牌access_token的值
    response.getWriter().write(accessToken);
}
```



## 隐式许可



## 如何选择

这里，我给你的建议是，在对接 OAuth 2.0 的时候先考虑授权码许可类型，其次再结合现实生产环境来选择：

- 如果小兔软件是**官方出品**，那么可以直接**使用资源拥有者凭据许可**；
- 如果小兔软件就是只**嵌入到浏览器端的应用**且没有服务端，那就只能**选择隐式许可**；
- 如果小兔软件获取的信息**不属于任何一个第三方用户**，那可以直接**使用客户端凭据许可类型**。

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/3ee0ceff6c543157a51aae985756454d.jpg" alt="img" style="zoom:30%;" />