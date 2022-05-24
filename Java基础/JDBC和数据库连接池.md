## 基础部分 - JDBC API

### 一、jdbc原理

1、模拟jabc接口

- jdbc接口

  ```java
  package work.jdbc;
  
  public interface JdbcInterface {
  
      public Object getConnection();
  
      public void crud();
  
      public void close();
  }
  
  ```

- MysqlJdbcImpl

  ```java
  public class MysqlJdbcImpl implements JdbcInterface{
      @Override
      public Object getConnection() {
          System.out.println("得到 Mysql 的连接");
          return null;
      }
  
      @Override
      public void crud() {
          System.out.println("完成 mysql 增删改查");
      }
  
      @Override
      public void close() {
          System.out.println("关闭 mysql 的连接");
      }
  }
  ```

- OracleJdbcImpl

  ```java
  package work.jdbc;
  
  public class OracleJdbcImpl implements JdbcInterface{
      @Override
      public Object getConnection() {
          System.out.println("得到 Oracle 的连接");
          return null;
      }
  
      @Override
      public void crud() {
          System.out.println("完成 Oracle 增删改查");
      }
  
      @Override
      public void close() {
          System.out.println("关闭 Oracle 的连接");
      }
  }
  
  ```

- 下面是jdbc的原理图，JDBC是Java提供的一套用于数据库操作的接口API，Java程序员只需要面向这套接口编程即可。不同的数据库厂商，需要针对这套接口，提供不同的实现。

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211203185145198.png" alt="image-20211203185145198" style="zoom:50%;" />



- JDBC API

  JDBC API是一系列的接口，它统一和规范了应用程序与数据库的连接、执行SQL语句，并得到返回结果等各类操作，相关类和接口在java.sql和javax.sql包中

  <img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211203190638717.png" alt="image-20211203190638717" style="zoom:50%;" />

- JDBC快速入门

  ```java
  package work.jdbc;
  
  import com.mysql.jdbc.Driver;
  
  import java.sql.Connection;
  import java.sql.ResultSet;
  import java.sql.SQLException;
  import java.sql.Statement;
  import java.util.Properties;
  
  public class Jdbc01 {
  
      public static void main(String[] args) throws SQLException {
  
          //加载驱动
          Driver driver = new Driver();
  
          //创建连接
          String url= "jdbc:mysql://localhost:3306/user";
          //配置相关的信息
          Properties properties = new Properties();
          properties.setProperty("user","root");
          properties.setProperty("password","12345678");
          //获得mysql连接
          Connection connect = driver.connect(url, properties);
  
          //查询语句
          String sql = "select * from user";
          String sql2 = "insert into user values('xiaoqiang','123')";
          //用于执行静态SQL语句并返回其生成的结果对象
          Statement statement = connect.createStatement();
          statement.executeUpdate(sql2);
          //关闭连接
          statement.close();
          connect.close();
      }
  }
  
  ```



### 二、数据库连接方式

- 推荐使用DriverManager方法

```java
//方式一
    @Test
    public void connect1(){
        //加载驱动
        Driver driver = new Driver();

        //创建连接
        String url= "jdbc:mysql://localhost:3306/user";
        //配置相关的信息
        Properties properties = new Properties();
        properties.setProperty("user","root");
        properties.setProperty("password","12345678");
        //获得mysql连接
        Connection connect = driver.connect(url, properties);
    }

//方式二
		@Test
    public void conNet2() throws Exception{
        
        //创建驱动类
        Class<?> driverCls = Class.forName("com.mysql.jdbc.Driver");
        Driver driver = (Driver) driverCls.getDeclaredConstructor().newInstance();
        
        //配置路径
        String url = "jdbc:mysql://localhost:3306/user";
        
        //配置相关信息
        Properties properties = new Properties();
        properties.setProperty("user","root");
        properties.setProperty("password","12345678");
        
        //创建连接
        driver.connect(url,properties);
    }

//方式三
 		@Test
    public void connect3() throws Exception{
        //创建驱动类
        Class<?> driverCls = Class.forName("com.mysql.jdbc.Driver");
        Driver driver = (Driver) driverCls.getDeclaredConstructor().newInstance();

        //配置路径
        String url = "jdbc:mysql://localhost:3306/user";
        String user = "root";
        String password = "12345678";
        //注册driver驱动
        DriverManager.registerDriver(driver);
        Connection connection = DriverManager.getConnection(url, user, password);

    }

//方式四

	  /**
     * 在加载Driver类时，会自动完成Driver对象注册工作
     * static {
     *         try {
     *             DriverManager.registerDriver(new Driver());
     *         } catch (SQLException var1) {
     *             throw new RuntimeException("Can't register driver!");
     *         }
     *     }
     */
    @Test
    public void connect4() throws Exception{
        Class.forName("com.mysql.jdbc.Driver");
        
        //配置路径
        String url = "jdbc:mysql://localhost:3306/user";
        String user = "root";
        String password = "12345678";
        Connection connection = DriverManager.getConnection(url, user, password);
    }

//方式五
		
 		@Test
		//通过读写配置文件，获取连接数据库的相关信息
    public void connect5() throws Exception{
        Class.forName("com.mysql.jdbc.Driver");

        //读取配置文件
        Properties properties = new Properties();
        properties.load(new FileInputStream("src/re.properties"));
        String user = properties.getProperty("user");
        String password = properties.getProperty("password");
        String url = properties.getProperty("url");
        Connection connection = DriverManager.getConnection(url,user,password);
        System.out.println(connection);
    }
```



### 三、ResultSet底层

- ResultSet是一个接口
- 表示数据库结果集的数据表，通常通过执行查询数据库的语句生成。 
- `ResultSet` 对象具有指向其当前数据行的光标。最初，光标被置于第一行之前。`next` 方法将光标移动到下一行；因为该方法在 `ResultSet` 对象没有下一行时返回 `false`，所以可以在 `while` 循环中使用它来迭代结果集。 
- 默认的 `ResultSet` 对象不可更新，仅有一个向前移动的光标。因此，只能迭代它一次，并且只能按从第一行到最后一行的顺序进行。可以生成可滚动和/或可更新的 `ResultSet` 对象

```java
//查询语句
String sql = "select * from user";
String sql2 = "insert into user values('xiaoqiang123','123')";
//用于执行静态SQL语句并返回其生成的结果对象
Statement statement = connect.createStatement();
ResultSet resultSet = statement.executeQuery(sql);
while(resultSet.next()){//让光标向后移动，如果没有更多行，则返回false
  String name = resultSet.getString(1);//获取该行的第1列
  String password = resultSet.getString(2);//获取该行的第2列
  System.out.println(name+password);
}
```

- Result结果集的数据情况展示

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211203213007664.png" alt="image-20211203213007664" style="zoom:50%;" />



### 四、Statement（SQL注入风险）

#### 1、基本介绍

- Statement对象用于执行静态SQL语句并返回其生成的结果的对象
- 在连接建立之后，需要对数据库进行访问，执行 命名或是SQL 语句，可以通过
  - Statement  [存在SQL注入问题]
  - PreparedStatement [预处理]
  - CallableStatement [存储过程]
- Statement对象执行SQL语句，存在SQL注入风险
- SQL注入是利用某些系统没有对用户输入的数据进行充分的检查，而在用户输入数据中注入非法的SQL语句段或命令，恶意攻击数据库
- 要防范SQL注入，用PreparedStatement取代Statement就可以了

#### 2、SQL注入风险例子

```java
String admin_name = "1' or";
String admin_pwd = "'1'='1";
String sql = "select * from user where name ='"+admin_name+"'and pwd='"+admin_pwd+"'";
//经过字符串连接之后，sql = "select * from user where name ='1' or'and pwd='or'1'='1'"
//可知where后面的语句是永远为true的，因此这条语句无论什么情况下都可以执行，这种情况下，对数据库系统而言是一个潜在的风险
```



### 五、PreparedStatement

- PreparedStatement继承自Statement，而且是一个接口

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211204193927542.png" alt="image-20211204193927542" style="zoom:50%;" />

- PreparedStatement使用实例

  ```java
  String sql = "select * from user where name = ? ";
  String name0 = "liditao";
  //用于执行静态SQL语句并返回其生成的结果对象
  PreparedStatement preparedStatement = connect.prepareStatement(sql);
  preparedStatement.setString(1,name0);
  //一定要注意，executeQuery()中是没有参数的，因为预处理的时候，已经执行语句加入到PreparedStatement对象中
  ResultSet resultSet = preparedStatement.executeQuery();
  while(resultSet.next()){//让光标向后移动，如果没有更多行，则返回false
    String name = resultSet.getString(1);//获取该行的第1列
    String password = resultSet.getString(2);//获取该行的第2列
    System.out.println(name+password);
  }
  ```

### 六、总结

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211204203850774.png" alt="image-20211204203850774" style="zoom:50%;" />



## 提升部分

### 一、JDBCUtils开发

- 现在有一个想法，由于数据库的连接和数据库连接的断开，都是常规且重复性的操作，为了提高代码的重用性，我们将这两部分提取成一个工具类。由于数据库的连接和断开应该在加载的时候就被创建，同时避免连接的重复创建，因此将数据库的连接和断开操作设置为Static方法。

  <img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20211204203917480.png" alt="image-20211204203917480" style="zoom:50%;" />

- 实现代码

  ```java
  public class JDBCUtils {
  
      private static String username;
      private static String password;
      private static String url;
      private static String driver;
  
      static {
          try {
              Properties properties = new Properties();
              properties.load(new FileInputStream("src/re.properties"));
              //读取相关的属性值
              username = properties.getProperty("user");
              password = properties.getProperty("password");
              url = properties.getProperty("url");
              driver = properties.getProperty("driver");
          } catch (Exception e){
              //在实际开发中，我们这样处理
              //1、将编译异常转成 运行异常抛出
              //2、这样，调用者可以选择捕获该异常，也可以选择默认方式处理该异常，比较方便
              throw new RuntimeException(e);
          }
      }
  
      public static Connection getConnection(){
          try{
              return DriverManager.getConnection(url,username,password);
          }catch (SQLException e){
              throw new RuntimeException(e);
          }
      }
      /**
       * 关闭资源
       * 1、Result结果集
       * 2、Statement 或者 Preparedment
       * 3、connection
       */
      public static void close(ResultSet resultSet, Statement statement,Connection connection){
          //判断是否为null
          try{
              if(resultSet !=null)
                  resultSet.close();
              if(statement != null)
                  statement.close();
              if(connection != null)
                  connection.close();
          }catch (SQLException e){
              throw new RuntimeException(e);
          }
  
      }
  }
  
  ```



### 二、事务