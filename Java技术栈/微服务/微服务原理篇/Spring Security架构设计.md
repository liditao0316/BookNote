# 认证架构

>`SecurityContextHolder`：Spring Security存储身份验证者详细信息的位置。
>
>`SecurityContext`：从`SecurityContextHandler`中获取，并包含当前授权用户的`Authentication`
>
>`Authentication`：可以是AuthenticationManager的输入，以提供用户提供的凭证以进行身份验证，也可以是SecurityContext中的当前用户
>
>`GrantedAuthority`：在身份验证上授予主体的权限（即角色、范围等）
>
>`AuthenticationManager`：定义 Spring Security 的过滤器如何执行身份验证的 API。
>
>`ProviderManager`：AuthenticationManager最常见的实现
>
>`AuthenticationProvider`：被ProviderManager用于执行特定类型的身份验证
>
>`AbstractAuthenticationProcessingFilter`：用于身份验证的基本过滤器。这也很好地了解了身份验证的高级流程以及各个部分如何协同工作。



## SecurityContextHolder

- Spring Security身份认证模型的核心是SecurityContextHolder，它包含SecurityContext。
- SecurityContextHolder是Spring Security存储身份验证详细信息的地方。

![](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/securitycontextholder.png)

- 最简单声明一个被授权用户的方式是直接设置`SecurityContextHolder`。



### Setting SecurityContextHolder

```java
SecurityContext context = SecurityContextHolder.createEmptyContext();
//TestingAuthenticationToken是最为简单的使用方式
Authentication authentication =
    new TestingAuthenticationToken("username", "password", "ROLE_USER");
context.setAuthentication(authentication);
SecurityContextHolder.setContext(context);
```

### Access Currently Authenticated User

```java
SecurityContext context = SecurityContextHolder.getContext();
Authentication authentication = context.getAuthentication();
String username = authentication.getName();
Object principal = authentication.getPrincipal();
Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
```

## Authentication

- 在Spring Security体系中，Authentication有两个主要目的：
  - 作为AuthenticationManager 的输入，用于提供用户提供的身份验证凭据。在这种情况下使用时，isAuthenticated() 返回 false。（未授权）
  - 表示当前经过身份验证的用户。当前的Authentication可以从SecurityContext中获取
- Authentication包含的部分：
  - `principal`：识别用户。使用用户名/密码进行身份验证时，这通常是UserDetails的一个实例
  - `credentials`：通常是密码。在许多情况下，这将在用户通过身份验证后被清除，以确保它不被泄漏。
  - `authorities`：`GrantedAuthority`s是授予用户的高级权限。一些示例是角色（roles）或范围（scopes）。

 



## ProviderManager

- ProviderManager委托一个AuthenticationProvider列表。每一个AuthenticationProvider都有机会指示身份验证成功、失败或表明它无法做出决定并允许下游AuthenticationProvider决定。
- 如果没有配置AuthenticationProvider，则会抛出异常。
- 每一个AuthenticationProvider可以执行特定类型的身份验证，并且支持多种类型的身份验证而只需要暴露一个AuthenticationManager bean即可。

<img src="https://docs.spring.io/spring-security/reference/_images/servlet/authentication/architecture/providermanager.png" alt="providermanager" style="zoom:70%;" />



## AbstractAuthenticationProcessingFilter

![abstractauthenticationprocessingfilter](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/abstractauthenticationprocessingfilter.png)





# 授权架构