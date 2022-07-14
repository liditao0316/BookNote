## Spring Boot的常见注解

### 1. @SpringBootApplication

- Spring Boot的核心注解

  ```java
  @SpringBootConfiguration
  @EnableAutoConfiguration
  @ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
  		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
  public @interface SpringBootApplication {
  }
  ```

- `@EnableAutoConfiguration`：启动Spring Boot的自动配置机制
- `@ComponentScan`： 扫描被`@Component` (`@Service`,`@Controller`)注解的 bean，注解默认会扫描该类所在的包下所有的类。
- `@SpringBootConfiguration`：允许在 Spring 上下文中注册额外的 bean 或导入其他配置类





### 2.Spring Bean相关

#### 2.1 @Autowired

自动导入对象到类中，被注入的类必须在Spring容器管理



#### 2.2 @component,@Repository,@Service,@Controller

- @component：通用的注解，可标注任意类为Spring组件。
- @Repository：持久层即Dao层，主要用于数据库相关操作
- @Service：服务层，主要涉及一些复杂的逻辑
- @Controller：对应Spring MVC控制层



#### 2.3. @RestController

- @RestController注解是@Controller和@ResponseBody的集合，表示这是个控制器bean，并且是将函数的返回值直接填入http响应体中，是REST风格的控制器

#### 2.4. @Scope

- 声明Spring Bean的作用域
- 四种常用的Spring bean的作用域：
  - singLeton：单例模式
  - prototype：每次请求都会创建一个新的bean的实例
  - request
  - session

#### 2.5. @Configuration

- 声明配置类

#### 

### 3. 读取配置类

#### 3.1. @ConfigurationProperties

- 通过@ConfigurationProperties读取配置信息并与bean绑定



### 4.其他

#### 4.1. @conditional

- 条件装配，复合条件的配置类，才创建bean

#### 4.2. @ImportResource

- 导入原生配置文件，比如application.xml

#### 4.3. @PathVariable

- 获取路径中的参数

### 5.加不加@RequestBody的区别

不加@RequestBody：可以接收Content-Type为application/x-www-form-urlencoded类型的请求所提交的数据，数据格式：aaa=111&bbb=222

加@RequestBody：用于接收Content-Type为application/json类型的请求,数据类型是JSON：{"aaa":"111","bbb":"222"}