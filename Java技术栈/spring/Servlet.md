### 一、Tomcat项目部署及访问静态资源

Tomcat是web服务器，我们的项目应用是部署在webapps下，然后通过特定的URL访问

#### 1.1创建项目

- 在webapps中建立文件夹（项目应用），比如：myweb
  - 创建WEB-INF文件夹，用于存放项目的核心内容
    - 创建classes，用于存放.class文件
    - 创建lib，用于存放jar文件
    - 创建web.xml，项目配置文件（到ROOT项目下的WEB-INF复制即可）
  - 把网页hello.html复制到myweb文件夹中，与WEB-INF在同级目录



### 二、Servlet

#### 2.1 概念

- Servlet：Server Applet的简称，是服务器端的程序（代码、功能实现），可交互式的处理客户端发送到服务端的请求，并完成操作响应
- 动态网页技术





#### 2.2 Servlet作用

- 接收客户端请求，完成操作
- 动态生成网页（页面数据可变）
- 将包含操作结果的动态网页响应给客户端



#### 2.3 Servlet开发步骤



##### 2.3.1 搭建开发环境

将Servlet相关jar包（lib\servlet-api.jar）配置到classpath中



##### 2.3.2 编写servlet

- 实现javax.servlet.Servlet

- 重写5个主要方法

- 在核心的service()方法中编写输出语句，打印访问结果

  ```java
  import javax.servlet.Servlet;
  import javax.servlet.ServletConfig;
  import javax.servlet.ServletException;
  import javax.servlet.ServletResponse;
  import javax.servlet.ServletRequest;
  import java.io.IOException;
  
  public class MyServlet implements Servlet{
      public void init(ServletConfig servletConfig) throws ServletException{
  
      }
  
      public void service(ServletRequest request, ServletResponse response) throws ServletException,IOException{
  
      }
  
      public void destroy(){
  
      }
  
      public ServletConfig getServletConfig(){
          return null;
      }
  
      public String getServletInfo(){
          return null;
      }
  
  }
  
  ```

##### 2.3.3 部署Servlet

编译MyServlet后，将生成的.class文件放在WEB-INF/classes文件中



##### 2.2.4 配置Servlet

编写WEB-INF下项目配置文件web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<web-app xmlns="https://jakarta.ee/xml/ns/jakartaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="https://jakarta.ee/xml/ns/jakartaee
                      https://jakarta.ee/xml/ns/jakartaee/web-app_5_0.xsd"
  version="5.0"
  metadata-complete="true">

  <servlet>
    <servlet-name>my</servlet-name>
    <servlet-class>MyServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>my</servlet-name>
    <!-- url-pattern 配置的内容就是浏览器地址栏输入的URL中项目名称后资源的内容 -->
    <url-pattern>/myservlet</url-pattern>
  </servlet-mapping>

</web-app>

```



### 三、IDEA创建Web项目

#### 3.1 IDEA创建Web项目

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211115100607114.png" alt="image-20211115100607114" style="zoom:50%;" />

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211115100607114.png" style="zoom:50%;" />

#### 3.2 IDEA部署Web项目

##### 3.2.1 IDEA集成Tomcat

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211114112249221.png" alt="image-20211114112249221" style="zoom:30%;" />

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211114112503327.png" alt="image-20211114112503327" style="zoom:50%;" />

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211114112648761.png" alt="image-20211114112648761" style="zoom:30%;" />

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211114112812134.png" alt="image-20211114112812134" style="zoom:30%;" />

##### 3.2.2 IDEA部署Web项目

IDEA测试的时候，并没有在WebApps文件夹中保存Web项目，当部署Web项目的时候，才会在WebApps文件夹中创建项目文件

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211115101050314.png" alt="image-20211115101050314" style="zoom:50%;" />

将out项目下的war包复制到Apache服务器中的webapps文件夹下

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211115102837369.png" alt="image-20211115102837369" style="zoom:50%;" />

#### 3.3 其他操作

 ##### 3.3.1 关联第三方jar包

步骤：

1.  在【WEB-INF】目录下新建lib包
2. 复制jar包到lib目录下
3. 点击lib目录，选择Add as Library
4. 选择Project Library完成
   - Global Library 表示所有工程都可以使用
   - Project Library表示当前工程中所有模块都可以使用
   - Module Library表示当前模块可以使用





### 四、Servlet详解

---

#### 4.1 Servlet核心接口和类

​	在Servlet体系结构中，除了实现Servlet接口，还可以通过继承GenericServlet或HttpServlet类，完成编写



##### 4.1.1 Servlet接口

在Servlet API中最重要的是Servlet接口，所有Servlet都会直接或间接的与该接口发生联系，或是直接实现该接口，或间接继承实现了该接口的类

该接口包括以下五种方法：

```java
 public void init(ServletConfig servletConfig) throws ServletException;

 public void service(ServletRequest request, ServletResponse response) throws ServletException,IOException;

 public void destroy();

 public ServletConfig getServletConfig();

 public String getServletInfo();
```



##### 4.1.2 GenericServlet抽象类

它提供生命周期方法init和destroy的实现，要编写一般的Servlet，只需重写抽象service方法。



##### 4.1.3 HttpServlet类

HttpServlet是继承GenericService的基础上进一步的拓展。提供将要被子类化以创建适用于Web站点的Http servlet的抽象类。

HttpServlet的子类至少必须重写一个方法，该方法通常是以下这些方法之一：

|   方法   |            作用             |
| :------: | :-------------------------: |
|  dopost  |      用于HTTP POST请求      |
|  doGet   | 如果servlet支持HTTP GET请求 |
|  doPut   |      用于HTTP PUT请求       |
| doDelete |     用于HTTP DELETE请求     |



#### 4.2 Servlet两种创建方式

```java
/*
* Servlet 创建的第一种方式，实现接口Servlet
* 继承上面的抽象类，建议继承HttpServlet
*/
```



#### 4.3 Servlet两种配置方式

##### 4.3.1 使用web.xml（Servlet2.5之前使用）

```xml
<?xml version="1.0" encoding="UTF-8"?>

<web-app xmlns="https://jakarta.ee/xml/ns/jakartaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="https://jakarta.ee/xml/ns/jakartaee
                      https://jakarta.ee/xml/ns/jakartaee/web-app_5_0.xsd"
  version="5.0"
  metadata-complete="true">

  <servlet>
    <servlet-name>my</servlet-name>
    <servlet-class>MyServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>my</servlet-name>
    <!-- url-pattern 配置的内容就是浏览器地址栏输入的URL中项目名称后资源的内容 -->
    <url-pattern>/myservlet</url-pattern>
  </servlet-mapping>

</web-app>
```

url-pattern定义匹配规则，取值说明：

|  匹配类型  |    定义     |                     说明                     |
| :--------: | :---------: | :------------------------------------------: |
|  精确匹配  | /具体的名称 | 只有url路径是具体的名称的时候才会触发Servlet |
|  后缀匹配  |    *.xxx    |      只要是以xxx结尾的就匹配触发Servlet      |
| 通配符匹配 |     /*      |             触发服务器的所有资源             |
| 通配符匹配 |      /      |       触发服务器的所有资源，不包括.jsp       |

​    

##### 4.3.2 使用注解（Servlet3.0以后支持）

```java
@WebServlet(name = "helloServlet", value = "/hello-servlet")
public class HelloServlet extends HttpServlet {
    private String message;

    public void init() {
        message = "Hello World!";
    }

    public void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        response.setContentType("text/html");

        // Hello
        PrintWriter out = response.getWriter();
        out.println("<html><body>");
        out.println("<h1>" + message + "</h1>");
        out.println("</body></html>");
    }

    public void destroy() {
    }
}
```



### 五、Servlet应用

---

#### 5.1 request对象

在Servlet中用来处理客户端请求需要用doGet或doPost方法的request对象



##### 5.1.1 get和post请求

   <font style="color:orange">get请求</font>

- get提交的数据回放在URL之后，以？分割URL和传输数据，参数之间以&相连
- get方式以明文传递，数据量少，不安全
- 效率高

<font style="color:orange">post请求</font>

- post方法是把提交的数据放在HTTP包的Body中
- 数据量大、安全



##### 5.1.2 request主要方法

|                  方法名                   |              说明              |
| :---------------------------------------: | :----------------------------: |
|     String setParameter(String name)      | 根据表单组件名称获取提交的数据 |
| void setCharacterEncoding(String charset) |       指定每个请求的编码       |



##### 5.1.3 post中文乱码问题

由于客户端是以UTF-8字符编码将表单数据传输到服务器端的，因此服务器也需要设置以UTF-8字符编码进行接收。

解决方案：使用从SerlvetRequest接口继承而来的setCharacterEncoding(charset)方法进行统一的编码设置

```java

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.setCharacterEncoding("UTF-8");
        String username = req.getParameter("username");
        String password = req.getParameter("password");
        System.out.println("提交的数据"+username+"\t"+password);
    }
```



#### 5.2 response对象

##### 5.2.1 response主要方法

|           方法名称           |                作用                |
| :--------------------------: | :--------------------------------: |
|    setHeader(name, value)    |           设置相应信息头           |
|    setContentType(String)    | 设置相应文件类型，响应式的编码格式 |
| setCharacterEncoding(String) |     设置服务端响应内容编码格式     |
|         getWriter()          |           获取字符输出流           |

##### 5.2.2 解决输出中文乱码

- 设置服务器响应的编码格式

- 设置客户端响应内容的头内容的文件类型及编码格式

  ```JAVA
  //方法一
  resp.setCharacterEncoding("utf-8");
  resp.setHeader("Content-Type","text/html;charset=utf-8");
  ```

  ```java
  //方法二
  resp.setContentType("text/html;charset=utf-8");
  ```

##### 5.2.3 综合案例（servlet+JDBC）

要求：实现注册功能，Servlet结合JDBC

- 项目结构

  dao、pojo、servcie层的代码和Spring的代码类似，这里就不再赘述

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211116163836429.png" alt="image-20211116163836429" style="zoom:50%;" />

- controller

  - ServletController

    ```java
    package work.controller;
    
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    import work.pojo.User;
    
    import javax.servlet.ServletException;
    import javax.servlet.annotation.WebServlet;
    import javax.servlet.http.HttpServlet;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import java.io.IOException;
    import java.util.List;
    
    @WebServlet("/selectAll")
    public class ServletController extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            ApplicationContext context = new ClassPathXmlApplicationContext("Application.xml");
            UserController userController = (UserController) context.getBean("userController");
            List<User> users = userController.selectAll();
            req.setAttribute("users",users);
            req.getRequestDispatcher("/responseImpl").forward(req,resp);
        }
    
    }
    ```

  - ResponseController

    ```java
    package work.controller;
    
    import work.pojo.User;
    
    import javax.servlet.ServletException;
    import javax.servlet.annotation.WebServlet;
    import javax.servlet.http.HttpServlet;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import java.io.IOException;
    import java.io.PrintWriter;
    import java.util.List;
    
    @WebServlet("/responseImpl")
    public class ResponseController extends HttpServlet {
    
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            resp.setContentType("text/html;charset=utf-8");
            List<User> users = (List<User>) req.getAttribute("users");
            PrintWriter printWriter = resp.getWriter();
    
            if(users!=null){
                printWriter.println("<html>");
                printWriter.println("<head>");
                printWriter.println("<meta charset='UTF-8'>");
                printWriter.println("</head>");
                printWriter.println("<body>");
                printWriter.println("<table>");
                printWriter.println("<tr>");
                printWriter.println("<td>id</td>");
                printWriter.println("<td>name</td>");
                printWriter.println("<td>account</td>");
                printWriter.println("</tr>");
                for(User user:users){
                    printWriter.println("<tr>");
                    printWriter.println("<td>"+user.getId()+"</td>");
                    printWriter.println("<td>"+user.getName()+"</td>");
                    printWriter.println("<td>"+user.getAccount()+"</td>");
                    printWriter.println("</tr>");
                }
                printWriter.println("</table>");
                printWriter.println("</body>");
                printWriter.println("</html>");
            }
        }
    }
    ```

  - UserController

    ```java
    package work.controller;
    
    import work.pojo.User;
    
    import java.util.List;
    
    public interface UserController {
    
        public List<User> selectAll();
        public User selectOne(int id);
        public void update(User user);
        public void insert(User user);
        public void delete(int id);
    }
    ```

  - UserControllerImpl

    ```java
    package work.controller;
    
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Controller;
    import work.dao.UserDao;
    import work.dao.UserDaoImpl;
    import work.pojo.User;
    import work.service.UserService;
    
    import java.util.List;
    
    @Controller("userController")
    public class UserControllerImpl implements UserController {
    
        @Autowired
        private UserService userService;
    
        public void setUserService(UserService userService) {
            this.userService = userService;
        }
    
        @Override
        public List<User> selectAll() {
            return userService.selectAll();
        }
    
        @Override
        public User selectOne(int id) {
            return null;
        }
    
        @Override
        public void update(User user) {
    
        }
    
        @Override
        public void insert(User user) {
    
        }
    
        @Override
        public void delete(int id) {
    
        }
    }
    ```

- pom.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>
  
      <groupId>com.example</groupId>
      <artifactId>demo5</artifactId>
      <version>1.0-SNAPSHOT</version>
      <name>demo5</name>
      <packaging>war</packaging>
  
      <properties>
          <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
          <maven.compiler.target>1.8</maven.compiler.target>
          <maven.compiler.source>1.8</maven.compiler.source>
          <junit.version>5.7.1</junit.version>
      </properties>
  
      <dependencies>
          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-webmvc</artifactId>
              <version>5.3.12</version>
          </dependency>
          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-jdbc</artifactId>
              <version>5.2.18.RELEASE</version>
          </dependency>
          <dependency>
              <groupId>org.mybatis</groupId>
              <artifactId>mybatis-spring</artifactId>
              <version>2.0.6</version>
          </dependency>
          <dependency>
              <groupId>mysql</groupId>
              <artifactId>mysql-connector-java</artifactId>
              <version>5.1.47</version>
          </dependency>
          <dependency>
              <groupId>org.mybatis</groupId>
              <artifactId>mybatis</artifactId>
              <version>3.5.2</version>
          </dependency>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot</artifactId>
              <version>2.5.6</version>
          </dependency>
          <dependency>
              <groupId>javax.servlet</groupId>
              <artifactId>javax.servlet-api</artifactId>
              <version>4.0.1</version>
              <scope>provided</scope>
          </dependency>
          <dependency>
              <groupId>org.junit.jupiter</groupId>
              <artifactId>junit-jupiter-api</artifactId>
              <version>${junit.version}</version>
              <scope>test</scope>
          </dependency>
          <dependency>
              <groupId>org.junit.jupiter</groupId>
              <artifactId>junit-jupiter-engine</artifactId>
              <version>${junit.version}</version>
              <scope>test</scope>
          </dependency>
          <dependency>
              <groupId>junit</groupId>
              <artifactId>junit</artifactId>
              <version>4.12</version>
              <scope>compile</scope>
          </dependency>
      </dependencies>
  
      <build>
          <plugins>
              <plugin>
                  <groupId>org.apache.maven.plugins</groupId>
                  <artifactId>maven-war-plugin</artifactId>
                  <version>3.3.1</version>
              </plugin>
          </plugins>
          <resources>
              <resource>
                  <directory>src/main/java</directory>
                  <includes>
                      <include>**/*.xml</include>
                  </includes>
                  <filtering>true</filtering>
              </resource>
          </resources>
      </build>
  </project>
  ```

  

### 六、转发和重定向

#### 6.1 现有问题

在之前的案例中，调用业务逻辑和显示结果页面都在同一个Servlet里，就会产生设计问题

- 不符合单一职能原则、各司其职的思想
- 不符合后续的维护

应该将业务逻辑和显示结果分离开

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211116141333392.png" alt="image-20211116141333392" style="zoom:50%;" />

##### 6.1.1 业务、显示分离

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211116141730159.png" alt="image-20211116141730159" style="zoom:50%;" />

#### 6.2 转发

转发的作用在服务器端，将请求发送给服务器上的其他资源，以共同完成一次请求的处理。



##### 6.2.1 页面跳转

在调用业务逻辑的Servlet中，编写以下代码

- request.getRequestDispatcher("/目标URL-pattern").forward(request,response);
- 注意：使用forward跳转时，是在服务器内部跳转，地址栏不发生变化，属于同一次请求

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211116142803613.png" alt="image-20211116142803613" style="zoom:50%;" />

##### 6.2.2 数据传递

forward表示一次请求，是在服务器内部跳转，可以共享同一次request作用域中的数据

- request作用域：拥有存储数据的空间，作用范围是一次请求有效（一次请求可以经过多次转发）
  - 可以将数据存入request后，在一次请求过程中的任何位置进行获取（任何位置指转发到的类中）
  - 可传递任何数据（基本数据类型、对象、数组、集合等）

- 存数据：request.setAttribute(key,value);
  - 以键值对形式存储在request作用域中。key为String类型，value为Object类型
- 取数据：request.getAttribute(key);
  - 通过String类型的key访问Object类型的value



##### 6.2.3 转发特点

- 转发是服务器行为
- 转发是浏览器只做了一次访问请求
- 转发浏览器地址不变
- 转发两次跳转之间传输信息不会丢失，所以可以通过request进行数据的传递

#### 6.3 重定向

重定向作用在客户端，客户端将请求发送给服务器后，服务器响应给客户端一个新的请求地址，客户端重新发送新请求。



##### 6.3.1 页面跳转

在调用业务逻辑的Servlet中，编写以下代码

- response.sendRedirect("目标URI");
- URI：统一资源标识符，用来表示服务器中定位一个资源，资源在web项目中的路径（/project/source）

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211116185828516.png" alt="image-20211116185828516" style="zoom:50%;" />



##### 6.3.2 数据传递

sendRedirect跳转时，地址栏改变，代表客户端重新发送的请求，属于两次请求

- response没有作用域，两次request请求中的数据无法共享
- 传递数据：通过URI的拼接进行数据传递("/WebProject/b?username=tom");
- 获取数据：request.getParammeter("username");



### 七、Servlet生命周期

#### 7.1 生命周期四个阶段

-  实例化
- 初始化
- 服务
- 销毁



### 八、Servlet特性

#### 8.1 线程安全问题

Servlet在被访问之后，会执行实例化操作，创建一个Servlet对象，而我们Tomcat容器可以同时多个线程并发访问同一个Servlet，如果在方法中对成员变量做修改操作，就会有线程安全问题



#### 8.2 如何保证线程安全

- synchronized
  - 将存在线程安全问题的代码放在同步代码块中
- 尽可能使用局部变量



### 九、状态管理

#### 11.1 现有问题

- HTTP协议是无状态的，不能保存每次提交的信息
- 如果用户发来一个新的请求，服务器无法知道它是否与上次的请求有联系
- 对于那些需要多次提交数据才能完成的Web操作，比如在登陆情况下，多次提交请求就成问题了



#### 11.2 概念

将浏览器与Web服务器之间多次交互当作一个整体来处理，并且将多次交互涉及的数据（即状态）保存下来



#### 11.3 状态管理分类

- 客户端状态管理技术：将状态保存在客户端，代表性的是Cookie技术
- 服务器状态管理技术：将状态保存在服务器端，代表性的是session技术





### 十、Cookie的使用

#### 10.1 什么是Cookie

- Cookie是在浏览器访问Web服务器的某个资源时，由Web服务器在Http响应信息头中附带传送给浏览器的一小段数据

- 一旦Web浏览器保存了某个Cookie，那么以后每次访问该Web服务器时，都应在HTTP请求头中将这个Cookie回传给Web服务器

- 一个Cookie主要由标识该信息的名称（name）和值（value）组成

  <img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211116203523719.png" alt="image-20211116203523719" style="zoom:50%;" />

#### 10.2 创建Cookie

```java
//服务端创建Cookie对象
Cookie cookie = new Cookie("username","liditao");
//设置能够访问cookie对象的路径
cookie.setPath("/project_name/可以访问该cookie的servlet对象路径");
//设置cookie的有效期。>0有效期，单位为秒；=0浏览器关闭；<0内存存储，默认-1
cookie.setMaxAge(-1);
//将Cookie响应给客户端
response.addCookie(cookie);
```



#### 10.3 获取Cookie

```java
//通过request对象获取所有的cookie
Cookie[] cookies = request.getCookies();
//通过循环遍历Cookie
for(Cookie cookie:cookies){
  cookie.getName();
  cookie.getValue();
}
```



#### 10.4 修改Cookie

只需要保证Cookie的名和路径一致即可修改，其他操作和创建Cookie一样，当浏览器发现一个名称和路径一致的Cookie时，将会覆盖原来的Cookie

```java
//服务端创建Cookie对象
Cookie cookie = new Cookie("username","liditao");
//设置能够访问cookie对象的路径
cookie.setPath("/project_name/可以访问该cookie的servlet对象路径");
//设置cookie的有效期。>0有效期，单位为秒；=0浏览器关闭；<0内存存储，默认-1
cookie.setMaxAge(-1);
//将Cookie响应给客户端
response.addCookie(cookie);
```



### 十一、Session对象

#### 11.1 Session概述

- Session用于记录用户的状态。Session指的是在一段时间内，单个客户端与Web服务器的一连串相关的交互过程。
- 在一个Session中，客户可能会多次请求访问同一个资源，也可能请求访问各种不同的服务器资源



#### 11.2 Session原理

- 服务器会为每一次会话分配一个Session对象
- 同一个浏览器发起的多次请求，同属于一次会话(Session)
- 首次使用Session时，服务器会自动创建Session，并创建Cookie存储Sessionid发送给客户端
- 注意：Session是由服务端创建的



#### 11.3 Session原理

- Session作用域：拥有存储数据的空间，作用范围是一次会话有效
  - 一次会话是使用同一浏览器发送的多次请求，一旦浏览器关闭，则结束会话
  - 可以将数据存入Session中，在一次会话的任意位置进行获取
  - 可传递任何数据(基本数据类型，对象、集合、数组)



##### 11.3.1 获取Session

```java
//获取Session对象
HttpSession session = request.getSession();
session.getId();
```

##### 11.3.2 Session保存数据

```java
session.setAttribute("key",value);//以键值对形式存储在session作用域中
```

##### 11.3.3 Session获取数据

```java
session.getAttribute("key");
```

##### 11.3.4 Session移除数据

```java
session.removeAttribute("key");
```





#### 11.4浏览器禁用Cookies解决方案

服务器在默认情况下，会使用Cookie将sessionID发送给浏览器，如果用户禁用cookie，则sessionID不会被浏览器保存，此时，服务器可以使用如URL重写这样的方式来发送sessionID



##### 11.4.1 URL重写

浏览器在访问服务器上的某个地址时，不再使用原来的那个地址，而是使用经过改写的地址（即在原来的地址后面加上了sessionID）



##### 11.4.2 实现URL重写

response.encodeRedirectURL(String url)生成重写的URL

```java
HttpSession session = request.getSession();
String newUrl = response.encodeRedirectURL("/project_name/具体路径名");
response.sendRedirect(newUrl2);
```



#### 11.5 Session实战权限验证

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211117200315151.png" alt="image-20211117200315151" style="zoom:50%;" />

#### 11.6 Session实战保存验证码



### 十二、ServletContext对象

#### 12.1 ServletContext概述

- 全局对象，拥有作用域，对应一个Tomcat中的Web应用
- 当Web服务器启动时，会为每一个Web应用程序创建一块共享的存储区域（ServletContext）
- ServletContext在Web服务器启动时创建，服务器关闭时销毁



#### 12.2 获取ServletContext对象

- GenericServlet提供了getServletContext()方法，使用this.getServletContext()获取
- HttpServletRequest提供了getServletContext()方法
- Httpsession提供了getServletContext()方法



#### 12.3 ServletContext作用

##### 12.3.1 获取项目真实路径

获取当前项目子啊服务器发布的真实路径

```java
String realpath = servletContext.getRealPath("/");
```



##### 12.3.2 获取项目上下文路径（应用程序名称）

```java
servletContext.getContextPath();//上下文路径（应用程序名称）
reqeust.getContextPath();
```



##### 12.3.3 全局容器

ServletContext拥有作用域，可以存储数据在全局容器中

- 存储数据：servletContext.setAttribute("name",value);
- 获取数据：servletContext.getAttribute("name",value);
- 移除数据：servletContext.removeAttribute("name",value);

应用场景：当系统应用需要统计应用被访问的情况时



#### 12.4 作用域总结

- HttpServletRequest：一次请求周期内有效
- HttpSession：一次会话周期内有效
- servletContext：服务器运行周期内有效



### 十三、过滤器

#### 13.1 概念

过滤器（Filter）是处于客户端与服务器目标资源之间的一道过滤技术

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211117200523343.png" alt="image-20211117200523343" style="zoom:50%;" />

### 13.2 过滤器的作用

- 执行地位在Servlet之前，客户端发送请求时，要先经过Filter，再到达目标Servlet中；响应时，会根据执行流程再次反向执行Filter
- 可以解决多个Servlet共性代码的冗余问题（例如：乱码处理、登陆验证）



#### 13.3 编写过滤器

Servlet API中提供了一个Filter接口，开发人员编写一个Java类实现了这个接口即可，这个Java类称之为过滤器



#### 13.4 实现过程

- 编写Java类实现Filter接口

- 在doFilter方法中编写拦截逻辑

- 设置拦截路径

  ```java
  package work.filter;
  
  import javax.servlet.*;
  import javax.servlet.annotation.WebFilter;
  import java.io.IOException;
  
  @WebFilter("/LoginManager")//@WebFilter(value="/过滤目标资源")
  public class MyFilter implements Filter {
      @Override
      public void init(FilterConfig filterConfig) throws ServletException {
          Filter.super.init(filterConfig);
      }
  
      @Override
      public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
          System.out.println("--begin--");
          chain.doFilter(request,response);
          System.out.println("--end--");
      }
  
      @Override
      public void destroy() {
          Filter.super.destroy();
      }
  }
  ```

#### 13.5 过滤器配置

##### 13.5.1 注解配置

@WebFilter(value="/过滤目标资源")



##### 13.5.2 xml配置

在web.xml中进行过滤器的配置

```xml
<filter>
    <filter-name>xml</filter-name>
    <filter-class>work.filter.MyFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>xml</filter-name>
    <url-pattern>/LoginManager</url-pattern>
</filter-mapping>
```



##### 13.5.3 关于拦截路径

过滤器的拦截路径通常有三种形式：

精确拦截匹配，比如/index.jsp    /myserlvet

后缀拦截匹配，比如*.jsp   *.html    *.jpg



#### 13.6 过滤器链和优先级

##### 13.6.1 过滤器链

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211117203654520.png" alt="image-20211117203654520" style="zoom:50%;" />

##### 13.6.2 过滤器优先级

在一个Web应用中，可以开发编写多个Filter，这些Filter组合起来称之为一个Filter链

优先级：

- 如果为注解的话，是按照类全名称的字符串顺序决定作用顺序
- 如果web.xml，按照filter-mapping注册顺序，从上往下
- web.xml配置高于注解方式
- 如果注解和Web.xml同时配置，会创建多个过滤器对象，造成过滤多次

