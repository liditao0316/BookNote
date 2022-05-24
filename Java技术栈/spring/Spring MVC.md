## Spring MVC 入门

### MVC是什么

MVC是Model、view、Controller的简写，是一种软件设计规范。



### 一、Spring MVC是什么

Spring MVC 是Spring提供的一个基于MVC设计模式的轻量级Web开发框架，本质上相当于Servlet。

Spring MVC 是 结构最清晰的 Servlet+JSP+JavaBean 的实现，是一个典型的教科书式的MVC架构，不像 Struts 等其它框架都是变种或者不是完全基于MVC系统的架构。

Spring MVC角色划分清晰，分工明细，并且和Spring框架无缝结合。

在Spring MVC框架中，Controller代替Servlet担任控制器的职责，用于接收请求，调用相应的Model进行处理，处理器完成业务处理后返回处理结果。Controller调用相应的View并对处理结果进行视图渲染，最终客户端得到响应信息。

Spring MVC架构采用松耦合可拔插的组件结构，具有高度可配置性，比起其它MVC框架更具有扩展性和灵活性。

Spring MVC 的注解驱动和对REST风格的支持，也是它最具特色的功能。

#### 1、Spring MVC的核心依赖包

```xml
	<dependency>
       <groupId>javax.servlet</groupId>
       <artifactId>servlet-api</artifactId>
       <version>2.5</version>
   </dependency>
   <dependency>
       <groupId>javax.servlet.jsp</groupId>
       <artifactId>jsp-api</artifactId>
       <version>2.2</version>
   </dependency>
   <dependency>
       <groupId>javax.servlet</groupId>
       <artifactId>jstl</artifactId>
       <version>1.2</version>
   </dependency>
```



### 二、第一个Spring MVC程序

---

 搭建步骤如下：

1. 创建 Web 应用并引入 JAR 包，本教程 Spring 使用版本为 5.2.3
2. Spring MVC 配置：在 web.xml 中配置 Servlet，创建 Spring MVC 的配置文件
3. 创建 Controller（处理请求的控制器）
4. 创建 View（本教程使用 JSP 作为视图）
5. 部署运行

#### 1.创建Web应用并引入JAR包

创建 Web 应用 springmvcDemo，在 springmvcDemo 的 lib 目录中添加 Spring MVC 所依赖的 JAR 包。

> Spring MVC 依赖 JAR 文件包括 Spring 的核心 JAR 包和 commons-logging 的 JAR 包。

Maven 项目在 pom.xml 文件中添加以下内容：

```xml
<!--测试-->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.11</version>
    <scope>test</scope>
</dependency>
<!--日志-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.21</version>
</dependency>
<!--J2EE-->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
</dependency>
<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>jsp-api</artifactId>
    <version>2.2</version>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>
<!--mysql驱动包-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.35</version>
</dependency>
<!--springframework-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>5.2.3.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.2.3.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.3.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.2.3.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.2.3.RELEASE</version>
</dependency>
<dependency>
    <groupId>com.github.stefanbirkner</groupId>
    <artifactId>system-rules</artifactId>
    <version>1.16.1</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.8.9</version>
</dependency>
<!--其他需要的包-->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.4</version>
</dependency>
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.1</version>
</dependency>
```

#### 2. Spring MVC配置

Spring MVC 是基于 Servlet 的，DispatcherServlet 是整个 Spring MVC 框架的核心，主要负责截获请求并将其分派给相应的处理器处理。所以配置 Spring MVC，首先要定义 DispatcherServlet。跟所有 Servlet 一样，用户必须在 web.xml 中进行配置。

##### 1) 定义DispatcherServlet

在开发 Spring MVC 应用时需要在 web.xml 中部署 DispatcherServlet，代码如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0">
    <display-name>springMVC</display-name>
    <!-- 部署 DispatcherServlet -->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 表示容器再启动时立即加载servlet -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <!-- 处理所有URL -->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

Spring MVC 初始化时将在应用程序的 WEB-INF 目录下查找配置文件，该配置文件的命名规则是“servletName-servlet.xml”，例如 springmvc-servlet.xml。

也可以将 Spring MVC 的配置文件存放在应用程序目录中的任何地方，但需要使用 servlet 的 init-param 元素加载配置文件，通过 contextConfigLocation 参数来指定 Spring MVC 配置文件的位置，示例代码如下。

```xml
<!-- 部署 DispatcherServlet -->
<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet </servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc-servlet.xml</param-value>
    </init-param>
    <!-- 表示容器再启动时立即加载servlet -->
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

此处使用 Spring 资源路径的方式进行指定，即 `classpath:springmvc-servlet.xml`。

上述代码配置了一个名为“springmvc”的 Servlet。该 Servlet 是 DispatcherServlet 类型，它就是 Spring MVC 的入口，并通过 `<load-on-startup>1</load-on-startup>` 配置标记容器在启动时就加载此 DispatcherServlet，即自动启动。然后通过 servlet-mapping 映射到“/”，即 DispatcherServlet 需要截获并处理该项目的所有 URL 请求。

##### 2）创建Spring MVC配置文件

在 WEB-INF 目录下创建 springmvc-servlet.xml 文件，如下所示。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- LoginController控制器类，映射到"/login" -->
    <bean name="/login"
          class="net.biancheng.controller.LoginController"/>
    <!-- LoginController控制器类，映射到"/register" -->
    <bean name="/register"
          class="net.biancheng.controller.RegisterController"/>
</beans>
```



#### 3. 创建Controller

在 src 目录下创建 net.biancheng.controller 包，并在该包中创建 RegisterController 和 LoginController 两个传统风格的控制器类（实现 Controller 接口），分别处理首页中“注册”和“登录”超链接的请求。

> Controller 是控制器接口，接口中只有一个方法 handleRequest，用于处理请求和返回 ModelAndView。

RegisterController 的具体代码如下。

```java
package controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

public class LoginController implements Controller {
    public ModelAndView handleRequest(HttpServletRequest arg0,
            HttpServletResponse arg1) throws Exception {
        return new ModelAndView("/WEB-INF/jsp/register.jsp");
    }
}
```

LoginController 的具体代码如下。

```java
package controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

public class RegisterController implements Controller {

    public ModelAndView handleRequest(HttpServletRequest arg0,
            HttpServletResponse arg1) throws Exception {
        return new ModelAndView("/WEB-INF/jsp/login.jsp");
    }
}
```

#### 4. 创建View

index.jsp 代码如下。

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
    未注册的用户，请
    <a href="${pageContext.request.contextPath }/register"> 注册</a>！
    <br /> 已注册的用户，去
    <a href="${pageContext.request.contextPath }/login"> 登录</a>！
</body>
</html>
```

在 WEB-INF 下创建 jsp 文件夹，将 login.jsp 和 register.jsp 放到 jsp 文件夹下。login.jsp 代码如下。

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
    登录页面！
</body>
</html>
```

register.jsp 代码如下。

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
         pageEncoding="UTF-8" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Insert title here</title>
<body>
    注册页面！
</body>
</html>
</head>
```

#### 5. 部署运行

将 springmvcDemo 项目部署到 Tomcat 服务器，首先访问 index.jsp 页面，如下图所示。



![img](http://c.biancheng.net/uploads/allimg/210705/1133302422-0.png)
index.jsp页面

![img](http://c.biancheng.net/uploads/allimg/210705/1133302624-1.png)
register.jsp页面

![img](http://c.biancheng.net/uploads/allimg/210705/1133301V7-2.png)
login.jsp页面



### 三、Spring MVC的执行流程

---

Spring MVC 框架是高度可配置的，包含多种视图技术，例如 JSP、FreeMarker、Tiles、iText 和 POI。Spring MVC 框架并不关心使用的视图技术，也不会强迫开发者只使用 JSP。

#### 1、Spring MVC 执行流程

Spring MVC 执行流程如图 1 所示。



![Spring MVC执行流程](http://c.biancheng.net/uploads/allimg/210705/1139441444-0.png)


SpringMVC 的执行流程如下。

1. 用户点击某个请求路径，发起一个 HTTP request 请求，该请求会被提交到 DispatcherServlet（前端控制器）；
2. 由 DispatcherServlet 请求一个或多个 HandlerMapping（处理器映射器），并返回一个执行链（HandlerExecutionChain）。
3. DispatcherServlet 将执行链返回的 Handler 信息发送给 HandlerAdapter（处理器适配器）；
4. HandlerAdapter 根据 Handler 信息找到并执行相应的 Handler（常称为 Controller）；
5. Handler 执行完毕后会返回给 HandlerAdapter 一个 ModelAndView 对象（Spring MVC的底层对象，包括 Model 数据模型和 View 视图信息）；
6. HandlerAdapter 接收到 ModelAndView 对象后，将其返回给 DispatcherServlet ；
7. DispatcherServlet 接收到 ModelAndView 对象后，会请求 ViewResolver（视图解析器）对视图进行解析；
8. ViewResolver 根据 View 信息匹配到相应的视图结果，并返回给 DispatcherServlet；
9. DispatcherServlet 接收到具体的 View 视图后，进行视图渲染，将 Model 中的模型数据填充到 View 视图中的 request 域，生成最终的 View（视图）；
10. 视图负责将结果显示到浏览器（客户端）。

#### 2、Spring MVC接口

Spring MVC 涉及到的组件有 DispatcherServlet（前端控制器）、HandlerMapping（处理器映射器）、HandlerAdapter（处理器适配器）、Handler（处理器）、ViewResolver（视图解析器）和 View（视图）。下面对各个组件的功能说明如下。

##### 1）DispatcherServlet

DispatcherServlet 是前端控制器，从图 1 可以看出，Spring MVC 的所有请求都要经过 DispatcherServlet 来统一分发。DispatcherServlet 相当于一个转发器或中央处理器，控制整个流程的执行，对各个组件进行统一调度，以降低组件之间的耦合性，有利于组件之间的拓展。

##### 2）HandlerMapping

HandlerMapping 是处理器映射器，其作用是根据请求的 URL 路径，通过注解或者 XML 配置，寻找匹配的处理器（Handler）信息。

##### 3）HandlerAdapter

HandlerAdapter 是处理器适配器，其作用是根据映射器找到的处理器（Handler）信息，按照特定规则执行相关的处理器（Handler）。

##### 4）Handler

Handler 是处理器，和 Java Servlet 扮演的角色一致。其作用是执行相关的请求处理逻辑，并返回相应的数据和视图信息，将其封装至 ModelAndView 对象中。

##### 5）View Resolver

View Resolver 是视图解析器，其作用是进行解析操作，通过 ModelAndView 对象中的 View 信息将逻辑视图名解析成真正的视图 View（如通过一个 JSP 路径返回一个真正的 JSP 页面）。

##### 6）View

View 是视图，其本身是一个接口，实现类支持不同的 View 类型（JSP、FreeMarker、Excel 等）。

以上组件中，需要开发人员进行开发的是处理器（Handler，常称Controller）和视图（View）。通俗的说，要开发处理该请求的具体代码逻辑，以及最终展示给用户的界面。



### 四、@RequestMapping注解

---

使用基于注解的控制器具有一下2个优点：

- 在基于注解的控制器类中可以编写多个处理方法，进而可以处理多个请求，这就允许将相关的操作编写在同一个控制器类中，进而减少控制器类的数量，方便以后维护。
- 基于注解的控制器不需要在配置文件中部署映射，仅需使用@RequestMapping注解一个方法进行请求处理即可

注意：使用RequestMapping注解时，必须要进行包扫描

#### 1、RequestMapping注解

一个控制器内有多个处理请求的方法，如 UserController 里通常有增加用户、修改用户信息、删除指定用户、根据条件获取用户列表等。每个方法负责不同的请求操作，而 @RequestMapping 就负责将请求映射到对应的控制器方法上。

在基于注解的控制器类中可以为每个请求编写对应的处理方法。使用 @RequestMapping 注解将请求与处理方法一 一对应即可。

@RequestMapping 注解可用于类或方法上。用于类上，表示类中的所有响应请求的方法都以该地址作为父路径。

@RequestMapping 注解常用属性如下。

##### 1) value 属性

value 属性是 @RequestMapping 注解的默认属性，因此如果只有 value 属性时，可以省略该属性名，如果有其它属性，则必须写上 value 属性名称。如下。

```java
@RequestMapping(value="toUser")
@RequestMapping("toUser")
```

value 属性支持通配符匹配，如 @RequestMapping(value="toUser/*")

##### 2) path属性

path 属性和 value 属性都用来作为映射使用。即 @RequestMapping(value="toUser") 和 @RequestMapping(path="toUser") 都能访问 toUser() 方法。

path 属性支持通配符匹配，如 @RequestMapping(path="toUser/*") 

##### 3) name属性

name属性相当于方法的注释，使方法更易理解。如 @RequestMapping(value = "toUser",name = "获取用户信息")。

##### 4) method属性

method 属性用于表示该方法支持哪些 HTTP 请求。如果省略 method 属性，则说明该方法支持全部的 HTTP 请求。

@RequestMapping(value = "toUser",method = RequestMethod.GET) 表示该方法只支持 GET 请求。也可指定多个 HTTP 请求，如 @RequestMapping(value = "toUser",method = {RequestMethod.GET,RequestMethod.POST})，说明该方法同时支持 GET 和 POST 请求。

##### 5) params属性

params 属性用于指定请求中规定的参数，代码如下。

```java
@RequestMapping(value = "toUser",params = "type")
public String toUser() {        
	return "showUser";
}
```

以上代码表示请求中必须包含 type 参数时才能执行该请求。即 http ://localhost:8080/toUser?type=xxx 能够正常访问 toUser() 方法，而 http ://localhost:8080/toUser 则不能正常访问 toUser() 方法。

```java
@RequestMapping(value = "toUser",params = "type=1")
public String toUser() {        
	return "showUser";
}
```

以上代码表示请求中必须包含 type 参数，且 type 参数为 1 时才能够执行该请求。即 http: //localhost:8080/toUser?type=1 能够正常访问 toUser() 方法，而 http ://localhost:8080/toUser?type=2 则不能正常访问 toUser() 方法。

##### 6) header属性

header 属性表示请求中必须包含某些指定的 header 值。

@RequestMapping(value = "toUser",headers = "Referer=http ://www. xxx.com") 表示请求的 header 中必须包含了指定的“Referer”请求头，以及值为“http: //www .xxx.com”时，才能执行该请求。

##### 7) consumers属性

consumers 属性用于指定处理请求的提交内容类型（Content-Type），例如：application/json、text/html。如 
@RequestMapping(value = "toUser",consumes = "application/json")。

##### 8) produces属性

produces 属性用于指定返回的内容类型，返回的内容类型必须是 request 请求头（Accept）中所包含的类型。如 @RequestMapping(value = "toUser",produces = "application/json")。

除此之外，produces 属性还可以指定返回值的编码。如 @RequestMapping(value = "toUser",produces = "application/json,charset=utf-8")，表示返回 utf-8 编码。

使用 @RequestMapping 来完成映射，具体包括 4 个方面的信息项：请求 URL、请求参数、请求方法和请求头。



<font style="color:orange">使用 @RequestMapping 来完成映射，具体包括 4 个方面的信息项：请求 URL、请求参数、请求方法和请求头。</font>

#### 2、通过请求URL进行映射

##### 1）方法级别注解

方法级别注解的示例代码如下。

```java
package net.biancheng.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class IndexController {
    @RequestMapping(value = "/index/login")
    public String login() {
        return "login";
    }

    @RequestMapping(value = "/index/register")
    public String register() {
        return "register";
    }
}
```

上述示例中有两个 RequestMapping 注解语句，它们都作用在处理方法上。在整个 Web 项目中，@RequestMapping 映射的请求信息必须保证全局唯一。

用户可以使用如下 URL 访问 login 方法（请求处理方法），在访问 login 方法之前需要事先在 /WEB-INF/jsp/ 目录下创建 login.jsp。

http ://localhost:8080/springmvcDemo/index/login

##### 2）类级别注解

类级别注解的示例代码如下：

```java
package net.biancheng.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
@Controller
@RequestMapping("/index")
public class IndexController {
    @RequestMapping("/login")
    public String login() {
        return "login";
    }

    @RequestMapping("/register")
    public String register() {
        return "register";
    }
}
```

在类级别注解的情况下，控制器类中的所有方法都将映射为类级别的请求。用户可以使用如下 URL 访问 login 方法。

http ://localhost:8080/springmvcDemo/index/login

为了方便维护程序，建议开发者采用类级别注解，将相关处理放在同一个控制器类中。例如，对用户的增、删、改、查等处理方法都可以放在 UserController 控制类中。



#### 3、通过请求参数、请求方法进行映射

@RequestMapping 除了可以使用请求 URL 映射请求之外，还可以使用请求参数、请求方法来映射请求，通过多个条件可以让请求映射更加精确。

```java
package net.biancheng.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class IndexController {
    @RequestMapping(value = "/index/success" method=RequestMethod.GET, Params="username")
    public String success(@RequestParam String username) {
        
        return "index";
}
```

上述代码中，@RequestMapping 的 value 表示请求的 URL；method 表示请求方法，此处设置为 GET 请求，若是 POST 请求，则无法进入 success 这个处理方法中。params 表示请求参数，此处参数名为 username。



#### 4、编写请求处理方法

在控制类中每个请求处理方法可以有多个不同类型的参数，以及一个多种类型的返回结果。

##### 1）请求处理方法中常出现的参数类型

如果需要在请求处理方法中使用 Servlet API 类型，那么可以将这些类型作为请求处理方法的参数类型。Servlet API 参数类型的示例代码如下：

```java
package net.biancheng.controller;
import javax.servlet.http.HttpSession;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
@Controller
@RequestMapping("/index")
public class IndexController {
    @RequestMapping("/login")
    public String login(HttpSession session,HttpServletRequest request) {
        session.setAttribute("skey", "session范围的值");
        session.setAttribute("rkey", "request范围的值");
        return "login";
    }
}
```

除了 Servlet API 参数类型以外，还有输入输出流、表单实体类、注解类型、与 Spring 框架相关的类型等，这些类型在后续章节中使用时再详细介绍。

其中特别重要的类型是 org.springframework.ui.Model 类型，该类型是一个包含 Map 的 Spring MVC类型。在每次调用请求处理方法时 Spring MVC 都将创建 org.springframework.ui.Model 对象。Model 参数类型的示例代码如下：

```java
package net.biancheng.controller;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
@Controller
@RequestMapping("/index")
public class IndexController {
    @RequestMapping("/register")
    public String register(Model model) {
        /*在视图中可以使用EL表达式${success}取出model中的值*/
        model.addAttribute("success", "注册成功");
        return "register";
    }
}
```

##### 2）请求处理方法常见的返回类型

请求处理方法可以返回如下类型的对象：

- ModelAndView
- Model
- 包含模型属性的 Map
- View
- 代表逻辑视图名的 String
- void
- 其它任意Java类型

最常见的返回类型就是代表逻辑视图名称的 String 类型，例如前面几节中的请求处理方法。



### 五、视图解析器 ViewResolver

视图解析器（ViewResolver）是 Spring MVC 的重要组成部分，负责将逻辑视图名解析为具体的视图对象。

Spring MVC 提供了很多视图解析类，其中每一项都对应 Java Web 应用中特定的某些视图技术。下面介绍一些常用的视图解析类。

#### 1、URLBasedViewResolver

UrlBasedViewResolver 是对 ViewResolver 的一种简单实现，主要提供了一种拼接 URL 的方式来解析视图。

UrlBasedViewResolver 通过 prefix 属性指定前缀，suffix 属性指定后缀。当 ModelAndView 对象返回具体的 View 名称时，它会将前缀 prefix 和后缀 suffix 与具体的视图名称拼接，得到一个视图资源文件的具体加载路径，从而加载真正的视图文件并反馈给用户。

使用 UrlBasedViewResolver 除了要配置前缀和后缀属性之外，还需要配置“viewClass”，表示解析成哪种视图。示例代码如下。

```xml
<bean id="viewResolver" class="org.springframework.web.servlet.view.UrlBasedViewResolver">            
    <property name="viewClass" value="org.springframework.web.servlet.view.InternalResourceViewResolver"/> <!--不能省略-->
    <!--前缀-->
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <!--后缀-->
    <property name="suffix" value=".jsp"/>  
 </bean>
```

上述视图解析器配置了前缀和后缀两个属性，这样缩短了 view 路径。因此《[第一个Spring MVC应用](http://c.biancheng.net/spring_mvc/first-program.html)》一节中的 RegisterController 和 LoginController 控制器类的视图路径仅需提供 register 和 login，视图解析器将会自动添加前缀和后缀，此处解析为 /WEB-INF/jsp/register.jsp 和 /WEB-INF/jsp/login.jsp。

上述 viewClass 值为 InternalResourceViewResolver，它用来展示 JSP 页面。如果需要使用 jstl 标签展示数据，将 viewClass 属性值指定为 JstlView 即可。

另外，存放在 /WEB-INF/ 目录下的内容不能直接通过 request 请求得到，所以为了安全性考虑，通常把 jsp 文件放在 WEB-INF 目录下。

#### 2、InternalResourceViewResolver

InternalResourceViewResolver 为“内部资源视图解析器”，是日常开发中最常用的视图解析器类型。它是 URLBasedViewResolver 的子类，拥有 URLBasedViewResolver 的一切特性。

InternalResourceViewResolver 能自动将返回的视图名称解析为 InternalResourceView 类型的对象。InternalResourceView 会把 Controller 处理器方法返回的模型属性都存放到对应的 request 属性中，然后通过 RequestDispatcher 在服务器端把请求 forword 重定向到目标 URL。也就是说，使用 InternalResourceViewResolver 视图解析时，无需再单独指定 viewClass 属性。示例代码如下。

```xml
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="viewClass" value="org.springframework.web.servlet.view.InternalResourceViewResolver"/> <!--可以省略-->
    <!--前缀-->
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <!--后缀-->
    <property name="suffix" value=".jsp"/>  
 </bean>
```

<font style="color:red">如果没有配置该bean，则要返回一个完整的路径</font>

#### 3、FreeMarkerViewResolver

FreeMarkerViewResolver 是 UrlBasedViewResolver 的子类，可以通过 prefix 属性指定前缀，通过 suffix 属性指定后缀。

FreeMarkerViewResolver 最终会解析逻辑视图配置，返回 freemarker 模板。不需要指定 viewClass 属性。

FreeMarkerViewResolver 配置如下。

```xml
<bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
    <property name="prefix" value="fm_"/>
    <property name="suffix" value=".ftl"/>
</bean>
```

下面指定 FreeMarkerView 类型最终生成的实体视图（模板文件）的路径以及其他配置。需要给 FreeMarkerViewResolver 设置一个 FreeMarkerConfig 的 bean 对象来定义 FreeMarker 的配置信息，代码如下。

```xml
<bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
    <property name="templateLoaderPath" value="/WEB-INF/ftl" />
</bean>
```

定义了 templateLoaderPath 属性后，Spring 可以通过该属性找到 FreeMarker 模板文件的具体位置。当有模板位于不同的路径时，可以配置 templateLoaderPath 属性，来指定多个资源路径。

然后定义一个 Controller，让其返回 ModelAndView，同时定义一些返回参数和视图信息。

```java
@Controller
@RequestMapping("viewtest")
public class ViewController {
    @RequestMapping("freemarker")
    public ModelAndView freemarker() {
        ModelAndView mv = new ModelAndView();
        mv.addObject("username", "BianChengBang");
        mv.setViewName("freemarker");
        return mv;
    }
}
```

当 FreeMarkerViewResolver 解析逻辑视图信息时，会生成一个 URL 为“前缀+视图名+后缀”（这里即“fm_freemarker.ftl”）的 FreeMarkerView 对象，然后通过 FreeMarkerConfigurer 的配置找到 templateLoaderPath 对应文本文件的路径，在该路径下找到该文本文件，从而 FreeMarkerView 就可以利用该模板文件进行视图的渲染，并将 model 数据封装到即将要显示的页面上，最终展示给用户。

在 /WEB-INF/ftl 文件夹下创建 fm_freemarker.ftl，代码如下。

```jsp
<html>
<head>
<title>FreeMarker</title>
</head>
<body>
    <b>Welcome!</b>
    <i>${username }</i>
</body>
</html>
```



### 六、传递参数

Spring MVC Controller 接收请求参数的方式有很多种，有的适合 get 请求方式，有的适合 post 请求方式，有的两者都适合。主要有以下几种方式：

- 通过实体 Bean 接收请求参数
- 通过处理方法的形参接收请求参数
- 通过 HttpServletRequest 接收请求参数
- 通过 @PathVariable 接收 URL 中的请求参数
- 通过 @RequestParam 接收请求参数
- 通过 @ModelAttribute 接收请求参数


下面分别介绍这些方式，读者可以根据实际情况选择合适的接收方式。

#### 1、通过实体Bean

实体 Bean 接收请求参数适用于 get 和 post 提交请求方式。需要注意，Bean 的属性名称必须与请求参数名称相同。示例代码如下。

```java
@RequestMapping("/login")
public String login(User user, Model model) {
    if ("bianchengbang".equals(user.getName())
            && "123456".equals(user.getPwd())) {
       
        model.addAttribute("message", "登录成功");
        return "main"; // 登录成功，跳转到 main.jsp
    } else {
        model.addAttribute("message", "用户名或密码错误");
        return "login";
    }
}
```

#### 2、通过处理方法的形参

通过处理方法的形参接收请求参数就是直接把表单参数写在控制器类相应方法的形参中，即形参名称与请求参数名称完全相同。该接收参数方式适用于 get 和 post 提交请求方式。示例代码如下：

```java
@RequestMapping("/login")
public String login(String name, String pwd, Model model) {
    if ("bianchengbang".equals(user.getName())
            && "123456".equals(user.getPwd())) {
       
        model.addAttribute("message", "登录成功");
        return "main"; // 登录成功，跳转到 main.jsp
    } else {
        model.addAttribute("message", "用户名或密码错误");
        return "login";
    }
}
```

#### 3、通过HttpServletRequest

通过 HttpServletRequest 接收请求参数适用于 get 和 post 提交请求方式，示例代码如下：

```java
@RequestMapping("/login")
public String login(HttpServletRequest request, Model model) {
    String name = request.getParameter("name");
    String pwd = request.getParameter("pwd");
   
    if ("bianchengbang".equals(name)
            && "123456".equals(pwd)) {
       
        model.addAttribute("message", "登录成功");
        return "main"; // 登录成功，跳转到 main.jsp
    } else {
        model.addAttribute("message", "用户名或密码错误");
        return "login";
    }
}
```

#### 4、通过@PathVariable

通过 @PathVariable 获取 URL 中的参数，示例代码如下。

```java
@RequestMapping("/login/{name}/{pwd}")
public String login(@PathVariable String name, @PathVariable String pwd, Model model) {
   
    if ("bianchengbang".equals(name)
            && "123456".equals(pwd)) {
       
        model.addAttribute("message", "登录成功");
        return "main"; // 登录成功，跳转到 main.jsp
    } else {
        model.addAttribute("message", "用户名或密码错误");
        return "login";
    }
}
```

在访问“http ://localhost:8080/springMVCDemo02/user/register/bianchengbang/123456”路径时，上述代码会自动将 URL 中的模板变量 {name} 和 {pwd} 绑定到通过 @PathVariable 注解的同名参数上，即 name=bianchengbang、pwd=123456。

#### 5、通过@RequestParam

在方法入参处使用 @RequestParam 注解指定其对应的请求参数。@RequestParam 有以下三个参数：

- value：参数名
- required：是否必须，默认为 true，表示请求中必须包含对应的参数名，若不存在将抛出异常
- defaultValue：参数默认值


通过 @RequestParam 接收请求参数适用于 get 和 post 提交请求方式，示例代码如下。

```java
@RequestMapping("/login")
public String login(@RequestParam String name, @RequestParam String pwd, Model model) {
   
    if ("bianchengbang".equals(name)
            && "123456".equals(pwd)) {
       
        model.addAttribute("message", "登录成功");
        return "main"; // 登录成功，跳转到 main.jsp
    } else {
        model.addAttribute("message", "用户名或密码错误");
        return "login";
    }
}
```

该方式与“通过处理方法的形参接收请求参数”部分的区别如下：当请求参数与接收参数名不一致时，“通过处理方法的形参接收请求参数”不会报 404 错误，而“通过 @RequestParam 接收请求参数”会报 404 错误。

#### 6、通过@ModelAttribute

@ModelAttribute 注解用于将多个请求参数封装到一个实体对象中，从而简化数据绑定流程，而且自动暴露为模型数据，在视图页面展示时使用。

而“通过实体 Bean 接收请求参数”中只是将多个请求参数封装到一个实体对象，并不能暴露为模型数据（需要使用 model.addAttribute 语句才能暴露为模型数据，数据绑定与模型数据展示后面教程中会讲解）。

通过 @ModelAttribute 注解接收请求参数适用于 get 和 post 提交请求方式，示例代码如下。

```java
格式化复制
@RequestMapping("/login")
public String login(@ModelAttribute("user") User user, Model model) {
   
    if ("bianchengbang".equals(name)
            && "123456".equals(pwd)) {
       
        model.addAttribute("message", "登录成功");
        return "main"; // 登录成功，跳转到 main.jsp
    } else {
        model.addAttribute("message", "用户名或密码错误");
        return "login";
    }
}
```

### 五、重定向和转发

Spring MVC 请求方式分为转发、重定向 2 种，分别使用 forward 和 redirect 关键字在 controller 层进行处理。

重定向是将用户从当前处理请求定向到另一个视图（例如 JSP）或处理请求，以前的请求（request）中存放的信息全部失效，并进入一个新的 request 作用域；转发是将用户对当前处理的请求转发给另一个视图或处理请求，以前的 request 中存放的信息不会失效。

转发是服务器行为，重定向是客户端行为。

#### 1、转发过程

客户浏览器发送 http 请求，Web 服务器接受此请求，调用内部的一个方法在容器内部完成请求处理和转发动作，将目标资源发送给客户；在这里转发的路径必须是同一个 Web 容器下的 URL，其不能转向到其他的 Web 路径上，中间传递的是自己的容器内的 request。

在客户浏览器的地址栏中显示的仍然是其第一次访问的路径，也就是说客户是感觉不到服务器做了转发的。转发行为是浏览器只做了一次访问请求。

#### 2、重定向过程

客户浏览器发送 http 请求，Web 服务器接受后发送 302 状态码响应及对应新的 location 给客户浏览器，客户浏览器发现是 302 响应，则自动再发送一个新的 http 请求，请求 URL 是新的 location 地址，服务器根据此请求寻找资源并发送给客户。

在这里 location 可以重定向到任意 URL，既然是浏览器重新发出了请求，那么就没有什么 request 传递的概念了。在客户浏览器的地址栏中显示的是其重定向的路径，客户可以观察到地址的变化。重定向行为是浏览器做了至少两次的访问请求。

在 Spring MVC 框架中，控制器类中处理方法的 return 语句默认就是转发实现，只不过实现的是转发到视图。示例代码如下：

```java
@RequestMapping("/register")
public String register() {
    return "register";  //转发到register.jsp
}
```

在 Spring MVC 框架中，重定向与转发的示例代码如下：

```java
package net.biancheng.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/index")
public class IndexController {
    @RequestMapping("/login")
    public String login() {
        //转发到一个请求方法（同一个控制器类可以省略/index/）
        return "forward:/index/isLogin";
    }

    @RequestMapping("/isLogin")
    public String isLogin() {
        //重定向到一个请求方法
        return "redirect:/index/isRegister";
    }

    @RequestMapping("/isRegister")
    public String isRegister() {
        //转发到一个视图
        return "register";
    }
}
```

在 Spring MVC 框架中，不管是重定向或转发，都需要符合视图解析器的配置，如果直接转发到一个不需要 DispatcherServlet 的资源，例如：

return "forward:/html/my.html";

即配置静态资源

则需要使用 mvc:resources 配置：

```xml 
<mvc:resources location="/html/" mapping="/html/**" />
```





### 五、JSON数据交互

Spring MVC 在数据绑定的过程中需要对传递数据的格式和类型进行转换，它既可以转换 String 等类型的数据，也可以转换 JSON 等其他类型的数据。本节将针对 Spring MVC 中 JSON 类型的数据交互进行讲解。

#### 1、JSON 概述

JSON（JavaScript Object Notation, JS 对象标记）是一种轻量级的数据交换格式。与 XML 一样，JSON 也是基于纯文本的数据格式。它有对象结构和数组结构两种数据结构。

##### 1）对象结构

对象结构以`{`开始、以`}`结束，中间部分由 0 个或多个以英文`,`分隔的 key/value 对构成，key 和 value 之间以英文`:`分隔。对象结构的语法结构如下：

```json
{
  key1:value1,
  key2:value2,
  ...
}
```

其中，key 必须为 String 类型，value 可以是 String、Number、Object、Array 等数据类型。例如，一个 person 对象包含姓名、密码、年龄等信息，使用 JSON 的表示形式如下：

```json
{
  "pname":"张三",
  "password":"123456",
  "page":40
}
```

##### 2）数组结构

数组结构以`[`开始、以`]`结束，中间部分由 0 个或多个以英文`,`分隔的值的列表组成。数组结构的语法结构如下：

```json
{
  value1,
  value2,
  ...
}
```

上述两种（对象、数组）数据结构也可以分别组合构成更加复杂的数据结构。例如，一个 student 对象包含 sno、sname、hobby 和 college 对象，其 JSON 的表示形式如下：

```json
{
  "sno":"201802228888",
  "sname":"张三",
  "hobby":["篮球","足球"]，
  "college":{
    "cname":"清华大学",
    "city":"北京"
  }
}
```



#### 2、JSON 数据转换

为实现浏览器与控制器类之间的 JSON 数据交互，Spring MVC 提供了 MappingJackson2HttpMessageConverter 实现类默认处理 JSON 格式请求响应。该实现类利用 Jackson 开源包读写 JSON 数据，将 Java 对象转换为 JSON 对象和 XML 文档，同时也可以将 JSON 对象和 XML 文档转换为 Java 对象。

在使用注解开发时需要用到两个重要的 JSON 格式转换注解，分别是 @RequestBody 和 @ResponseBody。

- @RequestBody：用于将请求体中的数据绑定到方法的形参中，该注解应用在方法的形参上。
- @ResponseBody：用于直接返回 return 对象，该注解应用在方法上。


需要注意的是，在该处理方法上，除了通过 @RequestMapping 指定请求的 URL，还有一个 @ResponseBody 注解。该注解的作用是将标注该注解的处理方法的返回结果直接写入 HTTP Response Body（Response 对象的 body 数据区）中。一般情况下，@ResponseBody 都会在异步获取数据时使用，被其标注的处理方法返回的数据都将输出到响应流中，客户端获取并显示数据。

#### 3、各个JSON技术比较

早期 JSON 的组装和解析都是通过手动编写代码来实现的，这种方式效率不高，所以后来有许多的关于组装和解析 JSON 格式信息的工具类出现，如 json-lib、Jackson、Gson 和 FastJson 等，可以解决 JSON 交互的开发效率。

##### 1）json-lib

json-lib 最早也是应用广泛的 JSON 解析工具，缺点是依赖很多的第三方包，如 commons-beanutils.jar、commons-collections-3.2.jar、commons-lang-2.6.jar、commons-logging-1.1.1.jar、ezmorph-1.0.6.jar 等。

对于复杂类型的转换，json-lib 在将 JSON 转换成 Bean 时还有缺陷，比如一个类里包含另一个类的 List 或者 Map 集合，json-lib 从 JSON 到 Bean 的转换就会出现问题。

所以 json-lib 在功能和性能上面都不能满足现在互联网化的需求。

##### 2）开源的Jackson

开源的 Jackson 是 Spring MVC 内置的 JSON 转换工具。相比 json-lib 框架，Jackson 所依赖 jar 文件较少，简单易用并且性能也要相对高些。并且 Jackson 社区相对比较活跃，更新速度也比较快。

但是 Jackson 对于复杂类型的 JSON 转换 Bean 会出现问题，一些集合 Map、List 的转换出现问题。而 Jackson 对于复杂类型的 Bean 转换 JSON，转换的 JSON 格式不是标准的 JSON 格式。

##### 3）Google的Gson

Gson 是目前功能最全的 JSON 解析神器，Gson 当初是应 Google 公司内部需求由 Google 自行研发。自从在 2008 年 5 月公开发布第一版后，Gson 就已经被许多公司或用户应用。

Gson 主要提供了 toJson 与 fromJson 两个转换函数，不需要依赖其它的 jar 文件，就能直接在 JDK 上运行。在使用这两个函数转换之前，需要先创建好对象的类型以及其成员才能成功的将 JSON 字符串转换为相对应的对象。

类里面只要有 get 和 set 方法，Gson 完全可以将复杂类型的 JSON 到 Bean 或 Bean 到 JSON 的转换，是 JSON 解析的神器。Gson 在功能上面无可挑剔，但性能比 FastJson 有所差距。

##### 4）阿里巴巴的FastJson

FastJson 是用 Java 语言编写的高性能 JSON 处理器，由阿里巴巴公司开发。

FastJson 不需要依赖其它的 jar 文件，就能直接在 JDK 上运行。

FastJson 在复杂类型的 Bean 转换 JSON 上会出现一些问题，可能会出现引用的类型，导致 JSON 转换出错，需要制定引用。

FastJson 采用独创的算法，将 parse 的速度提升到极致，超过所有 JSON 库。

综上 4 种 JSON 技术的比较，在项目选型的时候可以使用 Google 的 Gson 和阿里巴巴的 FastJson 两种并行使用，如果只是功能要求，没有性能要求，可以使用Google 的 Gson。如果有性能上面的要求可以使用 Gson 将 Bean 转换 JSON 确保数据的正确，使用 FastJson 将 JSON 转换 Bean。



#### 4、示例

本节示例基于阿里巴巴提供的 FastJson。下面结合具体需求演示 Spring MVC 如何处理 JSON 格式数据。（本节代码基于《》一节的 springmvcDemo2 实现）

##### 1)  导入jar文件

导入所需 jar 包 fastjson-1.2.62.jar，下载地址：https://github.com/alibaba/fastjson/releases。

Maven 项目在 pom.xml 文件中添加以下依赖。

```xml
<!-- fastjson -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.62</version>
</dependency>
```

##### 2)  配置Spring MVC核心配置文件

在 springmvc-servlet.xml 中添加以下代码。

```xml
<mvc:annotation-driven>
<!--配置@ResponseBody由fastjson解析 -->
    <mvc:message-converters>
        <bean
            class="org.springframework.http.converter.StringHttpMessageConverter">
            <property name="defaultCharset" value="UTF-8" />
        </bean>
        <bean
            class="com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter4" />
    </mvc:message-converters>
</mvc:annotation-driven>
<mvc:default-servlet-handler />
<bean id="fastJsonpResponseBodyAdvice"
    class="com.alibaba.fastjson.support.spring.FastJsonpResponseBodyAdvice">
    <constructor-arg>
        <list>
            <value>callback</value>
            <value>jsonp</value>
        </list>
    </constructor-arg>
</bean>
<!-- annotation-driven用于简化开发的配置，注解DefaultAnnotationHandlerMapping和AnnotationMethodHandlerAdapter -->
<!-- 使用resources过滤掉不需要dispatcherservlet的资源（即静态资源，例如css、js、html、images）。
    在使用resources时必须使用annotation-driven，否则resources元素会阻止任意控制器被调用 -->
<!-- 允许js目录下的所有文件可见 -->
<mvc:resources location="/" mapping="/**" />
```

##### 3)  创建POJO类

在 net.biancheng.pojo 包下创建 User 类，代码如下。

```java
package net.biancheng.po;

public class User {
    private String name;
    private String password;
    private Integer age;
    
    /**省略setter和getter方法*/
}
```

##### 4)  创建JSP页面

创建 index.jsp 页面测试 JSON 数据交互，代码如下。

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>测试JSON交互</title>
<script type="text/javaScript"
    src="${pageContext.request.contextPath }/js/jquery-3.2.1.min.js"></script>
</head>
<body>
    <form action="">
        用户名：<input type="text" name="name" id="name" />
        <br>
        密码：<input type="password" name="password" id="password" />
        <br>
        年龄：<input type="text" name="age" id="age">
        <br>
        <input type="button" value="测试" onclick="testJson()" />
    </form>
</body>
<script type="text/javaScript">
    function testJson() {
        var name = $("#name").val();
        var password = $("#password").val();
        var age = $("#age").val();
        $.ajax({
            //请求路径
            url : "${pageContext.request.contextPath}/testJson",
            //请求类型
            type : "post",

            //data表示发送的数据
            data : JSON.stringify({
                name : name,
                password : password,
                age : age
            }), //定义发送请求的数据格式为JSON字符串
            contentType : "application/json;charset=utf-8",
            //定义回调响应的数据格式为JSON字符串，该属性可以省略
            dataType : "json",
            //成功响应的结果
            success : function(data) {
                if (data != null) {
                    alert("输入的用户名：" + data.name + "，密码：" + data.password
                            + "， 年龄：" + data.age);
                }
            }
        });
    }
</script>
</body>
</html>
```

由于在 index.jsp 中使用的是 JQuery 的 AJAX 进行的 JSON 的数据提交和响应，所以还需要引入 jquery.js 文件。这里我们引入的是 webapp 目录下的 js 文件夹中的 jquery-3.2.1.min.js。

##### 5) 创建控制器

在 UserController 中添加以下代码。

```java
/**
* 接收页面请求的JSON参数，并返回JSON格式的结果
*/
@RequestMapping("/testJson")
@ResponseBody//告知SpringMVC框架，方法返回的字符串不是跳转是直接在http响应体中返回
public User testJson(@RequestBody User user) {
    // 打印接收的 JSON数据
    System.out.println("name=" + user.getName() + ",password=" + user.getPassword() + ",age=" + user.getAge());
    // 返回JSON格式的响应
    return user;
}
```

在上述控制器类中编写了接收和响应 JSON 格式数据的 testJson 方法，方法中的 @RequestBody 注解用于将前端请求体中的 JSON 格式数据绑定到形参 user 上，@ResponseBody 注解用于直接返回 Person 对象（当返回 POJO 对象时默认转换为 JSON 格式数据进行响应）。

##### 6)  测试JSON数据交互

访问地址：http://localhost:8080/springmvcDemo2/index.jsp，运行结果如下。



![index.jsp页面](images/1526434150-0.png)
index.jsp 页面

输入信息后点击”测试“按钮，将弹出显示输入信息的对话框，如下图所示。

![img](images/1526432K0-1.png)





### EL与JSTL





