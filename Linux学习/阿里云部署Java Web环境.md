# 手动部署Java Web环境（CentOS 7）

更新时间：2021-10-26 13:57

[我的收藏](https://help.aliyun.com/my_favorites.html)

本篇教程介绍如何手动在ECS实例上部署Java web项目，适用于刚开始使用阿里云进行建站的个人用户。



## 背景信息

本篇教程在示例步骤中使用了以下实例规格和软件版本。操作时，请您以实际软件版本为准。 

- 实例规格：ecs.c6.large
- 操作系统：CentOS 7.4
- Tomcat 版本：Tomcat 8.5.53
- JDK 版本：JDK 1.8.0_241
- FTP工具：Winscp





## 步骤一：下载源代码

1. [下载Apache Tomcat](https://mirrors.aliyun.com/apache/tomcat/tomcat-8/)。

2. 下载JDK。

   1. [下载JDK安装压缩包](https://www.oracle.com/cn/java/technologies/javase-downloads.html)。 

      **说明** 如果在实例中执行**wget**命令下载JDK安装压缩包，但在解压时报错，您可以下载JDK安装压缩包，再上传到ECS实例上。

   2. 登录[ECS管理控制台](https://ecs.console.aliyun.com/)。

   3. 在左侧导航栏，选择***\*实例与镜像\** > \**实例\****。

   4. 选择已购ECS实例所在的地域。

   5. 在**实例列表**页面，找到已购ECS实例，在**IP 地址**列获取该实例的公网IP地址。

   6. 在Winscp工具里使用公网IP地址连接Linux实例。

   7. 将下载好的Apache Tomcat和JDK安装压缩包上传到Linux实例的根目录下。



## 步骤二：安装前准备

1. 在安全组入方向添加规则放行所需端口。具体步骤，请参见[添加安全组规则](https://help.aliyun.com/document_detail/25471.htm#concept-sm5-2wz-xdb)。

   例如本示例中，SSH协议的22端口和HTTP协议的8080端口。

2. 远程连接Linux实例。具体步骤，请参见[通过密码认证登录Linux实例](https://help.aliyun.com/document_detail/25433.htm#concept-sdk-1jx-wdb)。

3. 关闭防火墙。

   1. 运行**systemctl status firewalld**命令查看当前防火墙的状态。

      ![查看防火墙状态](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/7712359951/p32172.png)

      - 如果防火墙的状态参数是inactive，则防火墙为关闭状态。
      - 如果防火墙的状态参数是active，则防火墙为开启状态。本示例中防火墙为开启状态，因此需要关闭防火墙。

   2. 关闭防火墙。如果防火墙为关闭状态可以忽略此步骤。

      - 如果您想临时关闭防火墙，运行以下命令。

        ```shell
        systemctl stop firewalld
        ```

        **说明** 该操作只是暂时关闭防火墙，下次重启Linux后，防火墙还会开启。

      - 如果您想永久关闭防火墙，需要依次运行以下命令。

        1. 关闭当前运行中的防火墙。

           ```shell
           systemctl stop firewalld
           ```

        2. 关闭防火墙服务，在下次重启实例后，防火墙服务将不会开机自启动。

           ```shell
           systemctl disable firewalld
           ```

        **说明** 该操作会关闭防火墙服务，当您重新启动实例后，防火墙将会默认保持关闭状态。 如果您想重新开启防火墙，具体操作，请参见[firewalld官网信息](https://firewalld.org/)。

4. 关闭SELinux。

   1. 运行命令以下命令查看SELinux的当前状态。

      ```shell
      getenforce
      ```

      查看结果示例，如下图所示：![查看SELinux状态](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/7712359951/p21065.png)

      - 如果SELinux状态参数是Disabled， 则SELinux为关闭状态。
      - 如果SELinux状态参数是Enforcing，则SELinux为开启状态。本示例中SELinux为开启状态，因此需要关闭SELinux。

   2. 关闭SELinux。如果SELinux为关闭状态可以忽略此步骤。

      - 如果您想临时关闭SELinux，运行以下命令。

        ```shell
        setenforce 0
        ```

        **说明** 该操作只是暂时关闭SELinux，下次重启Linux后，SELinux还会开启。

      - 如果您想永久关闭SELinux，运行以下命令打开SELinux配置文件。

        ```shell
        vi /etc/selinux/config
        ```

        在/etc/selinux/config文件内，将光标移动到`SELINUX=enforcing`一行，按i键进入编辑模式，修改为`SELINUX=disabled`，然后按Esc键，再输入`:wq`并回车，保存关闭SELinux配置文件。

        **说明** 如果您想重新开启SELinux，具体操作，请参见[开启或关闭SELinux](https://help.aliyun.com/document_detail/157022.htm#task-2385075)。

   3. <font style="color:#DAA520">重启系统使设置生效</font>

5. 为保证系统安全性，建议创建一般用户来运行Tomcat。

   例如，本示例中创建一般用户www。 

   ```bash
   useradd www
   ```

6. 运行以下命令创建网站根目录。 

   ```bash
   mkdir -p /data/wwwroot/default
   ```

7. 将需要部署的Java Web项目文件WAR包上传到网站根目录下，然后将网站根目录下文件所属用户改为www。

   本示例中，将依次运行以下命令直接在网站根目录下新建一个Tomcat测试页面，并将网站根目录下文件所属用户改为www。 

   ```bash
   echo Tomcat test > /data/wwwroot/default/index.jsp
   ```

   ```
   chown -R www.www /data/wwwroot
   ```

## 步骤三：安装JDK

1. 运行以下命令新建一个目录。

   ```bash
   mkdir /usr/java
   ```

2. 依次运行以下命令为jdk-8u241-linux-x64.tar.gz添加可执行权限并解压到/usr/java。 

   ```bash
   chmod +x jdk-8u241-linux-x64.tar.gz
   ```

   ```bash
   tar xzf jdk-8u241-linux-x64.tar.gz -C /usr/java
   ```

3. 设置环境变量。

   1. 运行命令`vi /etc/profile`打开/etc/profile文件。

   2. 按下`i`键，添加以下内容。 

      ```bash
      # set java environment
      export JAVA_HOME=/usr/java/jdk1.8.0_241
      export CLASSPATH=$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib
      export PATH=$JAVA_HOME/bin:$PATH
      ```

   3. 按下Esc键，输入`:wq`并回车以保存并关闭文件。

4. 运行以下命令加载环境变量。

   ```shell
   source /etc/profile
   ```

5. 运行以下命令显示JDK版本信息。

   ```shell
   java -version
   ```

   返回结果如图所示，表示JDK已经安装成功。![jdk180241](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/5012649951/p97228.png)

## 步骤四：安装Apache Tomcat

1. 依次运行以下命令。

   1. 解压

      apache-tomcat-8.5.53.tar.gz 

      ```shell
      tar xzf apache-tomcat-8.5.53.tar.gz
      ```

   2. 重命名Tomcat目录 

      ```bash
      mv apache-tomcat-8.5.53 /usr/local/tomcat/
      ```

   3. 设置文件的所属用户

      ```bash
      chown -R www.www /usr/local/tomcat/
      ```

   在/usr/local/tomcat/目录下： 

   - bin：存放Tomcat的一些脚本文件，包含启动和关闭Tomcat服务脚本。
   - conf：存放Tomcat服务器的各种全局配置文件，其中最重要的是server.xml和web.xml。
   - webapps：Tomcat的主要Web发布目录，默认情况下把Web应用文件放于此目录。
   - logs：存放Tomcat执行时的日志文件。

2. 配置server.xml文件。 

   1. 运行以下命令切换到/usr/local/tomcat/conf/目录。

      ```shell
      cd /usr/local/tomcat/conf/
      ```

   2. 运行以下命令重命名server.xml文件。 

      ```shell
      mv server.xml server.xml_bk
      ```

   3. 新建一个server.xml文件。

      1. 运行以下命令创建并打开server.xml文件。

         ```
         vi server.xml
         ```

      2. 按下i键，添加以下内容。 

         ```xml
         <?xml version="1.0" encoding="UTF-8"?>
         <Server port="8006" shutdown="SHUTDOWN">
         <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener"/>
         <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener"/>
         <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener"/>
         <Listener className="org.apache.catalina.core.AprLifecycleListener"/>
         <GlobalNamingResources>
         <Resource name="UserDatabase" auth="Container"
          type="org.apache.catalina.UserDatabase"
          description="User database that can be updated and saved"
          factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
          pathname="conf/tomcat-users.xml"/>
         </GlobalNamingResources>
         <Service name="Catalina">
         <Connector port="8080"
          protocol="HTTP/1.1"
          connectionTimeout="20000"
          redirectPort="8443"
          maxThreads="1000"
          minSpareThreads="20"
          acceptCount="1000"
          maxHttpHeaderSize="65536"
          debug="0"
          disableUploadTimeout="true"
          useBodyEncodingForURI="true"
          enableLookups="false"
          URIEncoding="UTF-8"/>
         <Engine name="Catalina" defaultHost="localhost">
         <Realm className="org.apache.catalina.realm.LockOutRealm">
         <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
           resourceName="UserDatabase"/>
         </Realm>
         <Host name="localhost" appBase="/data/wwwroot/default" unpackWARs="true" autoDeploy="true">
         <Context path="" docBase="/data/wwwroot/default" debug="0" reloadable="false" crossContext="true"/>
         <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
         prefix="localhost_access_log." suffix=".txt" pattern="%h %l %u %t &quot;%r&quot; %s %b" />
         </Host>
         </Engine>
         </Service>
         </Server>
         ```

      3. 按Esc键，输入`:wq`并回车以保存并关闭文件。

3. 设置JVM内存参数。

   1. 运行以下命令创建并打开/usr/local/tomcat/bin/setenv.sh文件。

      ```shell
      vi /usr/local/tomcat/bin/setenv.sh
      ```

   2. 按下i键，添加以下内容。 

      指定`JAVA_OPTS`参数，用于设置JVM的内存信息以及编码格式。

      ```bash
      JAVA_OPTS='-Djava.security.egd=file:/dev/./urandom -server -Xms256m -Xmx496m -Dfile.encoding=UTF-8'
      ```

   3. 按下Esc键，输入`:wq`并回车以保存并关闭文件。

4. 设置Tomcat自启动脚本。

   1. 运行以下命令下载Tomcat自启动脚本。

      **说明** 该脚本来源于社区，仅供参考。阿里云对其可靠性以及操作可能带来的潜在影响，不做任何暗示或其他形式的承诺。如果您运行**wget**命令下载失败，您可以通过浏览器访问`https://raw.githubusercontent.com/oneinstack/oneinstack/master/init.d/Tomcat-init`直接获取脚本内容。

      ```shell
      wget https://raw.githubusercontent.com/oneinstack/oneinstack/master/init.d/Tomcat-init
      ```

   2. 运行以下命令移动并重命名Tomcat-init。

      ```shell
      mv Tomcat-init /etc/init.d/tomcat
      ```

   3. 运行以下命令为/etc/init.d/tomcat添加可执行权限。 

      ```shell
      chmod +x /etc/init.d/tomcat
      ```

   4. 运行以下命令设置启动脚本JAVA_HOME。 

      **注意** 脚本中JDK的版本信息必须与您安装的JDK版本信息一致，否则Tomcat会启动失败。

      ```shell
      sed -i 's@^export JAVA_HOME=.*@export JAVA_HOME=/usr/java/jdk1.8.0_241@' /etc/init.d/tomcat                  
      ```

5. 依次运行以下命令设置Tomcat开机自启动。 

   1. ```bash
      chkconfig --add tomcat
      ```

   2. ```
      chkconfig tomcat on
      ```

6. 运行以下命令启动Tomcat。

   ```bash
   service tomcat start             
   ```

7. 在浏览器地址栏中输入`http://公网IP:8080`进行访问。

   返回页面如下图所示，表示安装成功。

## 总结

- 首先要关闭防火墙
- wget命令的URL地址如何获取
- 注意版本的兼容性问题
- 注意端口是否开启