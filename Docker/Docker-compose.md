### 模板文件

Compose允许用户通过一个docker-compose.yaml模板文件来定义一组相关联的应用容器为一个项目。

Docker-compose标准模板文件应该包含version、services、networks三大部分，最关键的是services和networks两个部分。

#### 1、image

image指定服务的镜像名称和镜像ID。

#### 2、build

build用于构建镜像

#### 3、context

context选项可以是Dockerfile的文件路径，也可以是链接到git仓库的url。当提供的值是相对路径时，被解析为相对于撰写文件的路径，此目录也是发送到Docker守护进程的context

#### 4、dockerfile

使用dockerfile文件来构建，必须指定构建路径

```yaml
build:
  context: .
  dockerfile: Dockerfile-alternate 
```

#### 5、command

使用command可以覆盖容器启动后默认执行的命令。`command: bundle exec thin -p 3000`

#### 6、container_name

指定容器的名字

#### 7、depends_on

depends_on标签用于解决容器的依赖、启动先后的问题

```yaml
version: '2'
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

#### 8、pid

`pid:host`将pid模式设置为主机PID模式，跟宿主机系统共享进程命名空间。容器使用pid标签将能够访问和操纵其他容器和宿主机的名称空间。

#### 9、ports

ports用于映射端口的标签。

### 整合FastDFS

```yaml
version: "3.8"

networks:
  bookshop:

volumes:
  data:

services:

  tracker:
    image: season/fastdfs:1.2
    container_name: tracker
    restart: always
    volumes:
      - "./tracker_data:/fastdfs/tracker/data"
    ports:
      - "22122:22122"
    command: "tracker"

  storage:
    image: season/fastdfs:1.2
    container_name: storage
    links:
      - tracker
    restart: always
    volumes:
      - "./storage.conf:/fdfs_conf/storage.conf"
      - "./storage_base_path:/fastdfs/storage/data"
      - "./store_path0:/fastdfs/store_path"
    ports:
      - "23000:23000"
    environment:
      TRACKER_SERVER: "tracker:22122"
    command: "storage"

  nginx:
    image: season/fastdfs:1.2
    container_name: fastdfs-nginx
    restart: always
    volumes:
      - "./nginx.conf:/etc/nginx/conf/nginx.conf"
      - "./store_path0:/fastdfs/store_path"
    links:
      - tracker
    ports:
      - "8088:8088"
    environment:
      TRACKER_SERVER: "tracker:22122"
    command: "nginx"
```



### 整合微服务

```yaml
version: "3.8"

networks:
  bookshop:

volumes:
  data:

services:

  tracker:
    image: season/fastdfs:1.2
    container_name: tracker
    restart: always
    volumes:
      - "./tracker_data:/fastdfs/tracker/data"
    ports:
      - "22122:22122"
    command: "tracker"

  storage:
    image: season/fastdfs:1.2
    container_name: storage
    links:
      - tracker
    restart: always
    volumes:
      - "./storage.conf:/fdfs_conf/storage.conf"
      - "./storage_base_path:/fastdfs/storage/data"
      - "./store_path0:/fastdfs/store_path"
    ports:
      - "23000:23000"
    environment:
      TRACKER_SERVER: "tracker:22122"
    command: "storage"

  nginx:
    image: season/fastdfs:1.2
    container_name: fastdfs-nginx
    restart: always
    volumes:
      - "./nginx.conf:/etc/nginx/conf/nginx.conf"
      - "./store_path0:/fastdfs/store_path"
    links:
      - tracker
    ports:
      - "8088:8088"
    environment:
      TRACKER_SERVER: "tracker:22122"
    command: "nginx"

  bookshop-oauth2-server:
    image: book-oauth:latest
    container_name: book-oauth
    ports:
      - "8001:8001"
    networks:
      - bookshop
    depends_on:
      - mysql
      - nacos
      - sentinel

  bookshop-gateway:
    image: bookshop-gateway:latest
    container_name: book-gateway
    ports:
      - "8002:8002"
    networks:
      - bookshop
    depends_on:
      - mysql
      - nacos
      - sentinel

  bookshop-module-system:
    image: bookshop-module-system:latest
    container_name: book-module-system
    ports:
      - "8004:8004"
    networks:
      - bookshop
    depends_on:
      - mysql
      - nacos
      - sentinel

  bookshop-service-user:
    image: bookshop-service-user:latest
    container_name: bookshop-service-user
    ports:
      - "9501:9501"
    networks:
      - bookshop
    depends_on:
      - mysql
      - nacos
      - sentinel

  sentinel:
    image: sentinel:latest
    container_name: sentinel
    ports:
      - "8003:8080"
    networks:
      - bookshop


  nacos:
    image: nacos/nacos-server:latest
    container_name: nacos
    ports:
      - "8848:8848"
    environment:
      - "MODE=standalone"
    networks:
      - bookshop


  mysql:
    image: mysql:5.7
    container_name: mysql
    ports:
      - "3307:3306" #只写一个端口随机使用宿主机一个端口进行容器端口映射
    environment:
      - "MYSQL_ROOT_PASSWORD=123456"
      - "MYSQL_DATABASE=bookshop"
    volumes:
      - data:/var/lib/mysql
      #- ./ems.sql:/docker-entrypoint-initdb.d/ems.sql
    networks:
      - bookshop
    command:
      --default-authentication-plugin=mysql_native_password #解决外部无法访问

```

