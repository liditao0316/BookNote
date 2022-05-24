## 1. 两种模式

说到集群，小伙伴们可能第一个问题是，如果我有一个 RabbitMQ 集群，那么是不是我的消息集群中的每一个实例都保存一份呢？

这其实就涉及到 RabbitMQ 集群的两种模式：

- 普通集群
- 镜像集群

### 1.1 普通集群

普通集群模式，就是将 RabbitMQ 部署到多台服务器上，每个服务器启动一个 RabbitMQ 实例，多个实例之间进行消息通信。

此时我们创建的队列 Queue，它的元数据（主要就是 Queue 的一些配置信息）会在所有的 RabbitMQ 实例中进行同步，但是队列中的消息只会存在于一个 RabbitMQ 实例上，而不会同步到其他队列。

当我们消费消息的时候，如果连接到了另外一个实例，那么那个实例会通过元数据定位到 Queue 所在的位置，然后访问 Queue 所在的实例，拉取数据过来发送给消费者。

这种集群可以提高 RabbitMQ 的消息吞吐能力，但是无法保证高可用，因为一旦一个 RabbitMQ 实例挂了，消息就没法访问了，如果消息队列做了持久化，那么等 RabbitMQ 实例恢复后，就可以继续访问了；如果消息队列没做持久化，那么消息就丢了。

大致的流程图如下图：

![img](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/1460000041104412.png)

### 1.2 镜像集群

它和普通集群最大的区别在于 Queue 数据和原数据不再是单独存储在一台机器上，而是同时存储在多台机器上。也就是说每个 RabbitMQ 实例都有一份镜像数据（副本数据）。每次写入消息的时候都会自动把数据同步到多台实例上去，这样一旦其中一台机器发生故障，其他机器还有一份副本数据可以继续提供服务，也就实现了高可用。

大致流程图如下图：

![img](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/1460000041104413.png)

### 1.3 节点类型

RabbitMQ 中的节点类型有两种：

- RAM node：内存节点将所有的队列、交换机、绑定、用户、权限和 vhost 的元数据定义存储在内存中，好处是可以使得交换机和队列声明等操作速度更快。
- Disk node：将元数据存储在磁盘中，单节点系统只允许磁盘类型的节点，防止重启 RabbitMQ 的时候，丢失系统的配置信息

RabbitMQ 要求在集群中至少有一个磁盘节点，所有其他节点可以是内存节点，当节点加入或者离开集群时，必须要将该变更通知到至少一个磁盘节点。如果集群中唯一的一个磁盘节点崩溃的话，集群仍然可以保持运行，但是无法进行其他操作（增删改查），直到节点恢复。为了确保集群信息的可靠性，或者在不确定使用磁盘节点还是内存节点的时候，建议直接用磁盘节点。



## 2. 搭建普通集群

### 2.1 预备知识

大致的结构了解了，接下来我们就把集群给搭建起来。先从普通集群开始，我们就使用 docker 来搭建。

搭建之前，有两个预备知识需要大家了解：

1. 搭建集群时，节点中的 Erlang Cookie 值要一致，默认情况下，文件在 /var/lib/rabbitmq/.erlang.cookie，我们在用 docker 创建 RabbitMQ 容器时，可以为之设置相应的 Cookie 值。
2. RabbitMQ 是通过主机名来连接服务，必须保证各个主机名之间可以 ping 通。可以通过编辑 /etc/hosts 来手工添加主机名和 IP 对应关系。如果主机名 ping 不通，RabbitMQ 服务启动会失败（如果我们是在不同的服务器上搭建 RabbitMQ 集群，大家需要注意这一点，接下来的 2.2 小结，我们将通过 Docker 的容器连接 link 来实现容器之间的访问，略有不同）。

### 2.2 开始搭建

执行如下命令创建三个 RabbitMQ 容器：

```apache
docker run -d --hostname rabbit01 --name mq01 -p 5671:5672 -p 15671:15672 -e RABBITMQ_ERLANG_COOKIE="javaboy_rabbitmq_cookie" rabbitmq:3-management
docker run -d --hostname rabbit02 --name mq02 --link mq01:mylink01 -p 5672:5672 -p 15672:15672 -e RABBITMQ_ERLANG_COOKIE="javaboy_rabbitmq_cookie" rabbitmq:3-management
docker run -d --hostname rabbit03 --name mq03 --link mq01:mylink02 --link mq02:mylink03 -p 5673:5672 -p 15673:15672 -e RABBITMQ_ERLANG_COOKIE="javaboy_rabbitmq_cookie" rabbitmq:3-management
```

运行结果如下：

![img](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/1460000041104414.png)

三个节点现在就启动好了，注意在 mq02 和 mq03 中，分别使用了 `--link` 参数来实现容器连接，关于这个参数，如果大家不懂，可以在公众号江南一点雨后台回复 docker，由松哥写的 docker 入门教程，里边有讲这个。这里我就不啰嗦了。另外还需要注意，mq03 容器中要既能够连接 mq01 也能够连接 mq02。

接下来进入到 mq02 容器中，首先查看一下 hosts 文件，可以看到我们配置的容器连接已经生效了：

![img](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/1460000041104415.png)

将来在 mq02 容器中，就可以通过 mylink01 或者 rabbit01 访问到 mq01 容器了。

接下来我们开始集群的配置。

分别执行如下命令将 mq02 容器加入集群中：

```nginx
rabbitmqctl stop_app
rabbitmqctl join_cluster rabbit@rabbit01
rabbitmqctl start_app
```

![img](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/1460000041104416.png)

接下来输入如下命令我们可以查看集群的状态：

```ebnf
rabbitmqctl cluster_status
```

![img](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/1460000041104417.png)

可以看到，集群中已经有两个节点了。

接下来通过相同的方式将 mq03 也加入到集群中：

```nginx
rabbitmqctl stop_app
rabbitmqctl join_cluster rabbit@rabbit01
rabbitmqctl start_app
```

![img](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/1460000041104418.png)

接下来，我们可以查看集群信息：

![img](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/1460000041104419.png)

可以看到，此时集群中已经有三个节点了。

其实，这个时候，我们也可以通过网页来查看集群信息，在三个 RabbitMQ 实例的 Web 端首页，都可以看到如下内容：

![img](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/1460000041104420.png)



## 3. 构建镜像集群

### 主机资源

| 主机名    | 操作系统  | IP          | 备注               |
| --------- | --------- | ----------- | ------------------ |
| rabbitmq1 | centos7.4 | 192.168.1.1 | 磁盘节点，管理节点 |
| rabbitmq2 | centos7.4 | 192.168.1.2 | 内存节点           |
| rabbitmq3 | centos7.4 | 192.168.1.3 | 内存节点           |

### hosts设置

修改`/etc/hosts`文件

```accesslog
192.168.1.1 rabbitmq1
192.168.1.2 rabbitmq2
192.168.1.3 rabbitmq3
```

## 基础搭建

分别在以上三台主机上执行如下步骤：

### 1. erlang安装

```shell
# 下载erlang软件包源
wget https://packages.erlang-solutions.com/erlang-solutions-1.0-1.noarch.rpm
sudo yum install epel-release -y
# 安装erlang软件包源
sudo rpm -Uvh erlang-solutions-1.0-1.noarch.rpm
# 安装erlang环境
sudo yum install erlang -y
```

验证

```shell
[root@rabbitmq1 ~]# erl
Erlang/OTP 23 [erts-11.1] [source] [64-bit] [smp:8:8] [ds:8:8:10] [async-threads:1] [hipe]
Eshell V11.1  (abort with ^G)
```

### 2. rabbitmq安装

```shell
# 下载介质源
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.8/rabbitmq-server-3.8.8-1.el7.noarch.rpm
# 安装介质源
yum install -y rabbitmq-server-3.8.8-1.el7.noarch.rpm
# 打开开启动
systemctl enable rabbitmq-server
# 启动服务
systemctl start rabbitmq-server
# 查看服务状态
systemctl status rabbitmq-server
```

验证

```shell
[root@rabbitmq1 ~]$ systemctl status rabbitmq-server.service
● rabbitmq-server.service - RabbitMQ broker
   Loaded: loaded (/usr/lib/systemd/system/rabbitmq-server.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2020-12-05 15:34:50 UTC; 13h ago
 Main PID: 11663 (beam.smp)
   Status: "Initialized"
   CGroup: /system.slice/rabbitmq-server.service
           ├─11663 /usr/lib64/erlang/erts-11.1/bin/beam.smp -W w -K true -A 128 -MBas ageffcbf -MHas ageffcbf -MBlmbcs 512 -MHlmbcs 512 -MMmcs 30 -P 1048576 -t 5000000 -stbt db -zdbbl 128000 -- -root ...
           ├─11844 erl_child_setup 32768
           ├─11918 inet_gethost 4
           └─11919 inet_gethost 4
```

## 创建集群

将rabbitmq1作为群集的管理节点。

### 1.配置集群基础环境

copy erlang集群文件到rabbitmq2、rabbitmq3的`/var/lib/rabbitmq/`目录中，将重启`systemctl restart rabbitmq-server` rabbitmq2、rabbitmq3的服务。

```shell
# 查看erlang分布式集群文件
[root@rabbitmq1 ~] ls -al /var/lib/rabbitmq

# 查看.erlang.cookie内容
[root@rabbitmq1 ~] cat .erlang.cookie 

# copy .erlang.cookie 文件到其它主机对应的目录中
[root@rabbitmq1 ~] scp .erlang.cookie  root@rabbitmq2:/var/lib/rabbitmq/

[root@rabbitmq1 ~] scp .erlang.cookie  root@rabbitmq3:/var/lib/rabbitmq/

```

### 2.将节点加入集群中

将rabbitmq2、rabbitmq3加入群集

在**虚拟机结点rabbitmq2**中输入一下命令：

```shell
# 在rabbitmq1上看群集的状态
[root@rabbitmq2 ~] rabbitmqctl cluster_status
# 在rabbitmq2操作加入群集
# 1.停止rabbitmq2上的服务
[root@rabbitmq2 ~] rabbitmqctl stop_app
# 2.停止rabbitmq2上的服务
[root@rabbitmq2 ~] rabbitmqctl reset
# 2.将rabbitmq2加入集群，--ram是以内存方式加入
[root@rabbitmq2 ~] rabbitmqctl join_cluster --ram rabbit@rabbitmq1
# 3.启动rabbitmq2上的服务
[root@rabbitmq2 ~] rabbitmqctl  start_app

```

接着在**虚拟机结点rabbitmq3**中输入命令。

### 3.将集群设为镜像模式

 在我们使用 rabbitmq 作为消息服务时，在服务负载不是很大的情况下，一般我们只需要一个 rabbitmq 节点便能为我们提供服务，可这难免会发生单点故障，要解决这个问题，我们便需要配置 rabbitmq 的集群和镜像。

#### **镜像模式参数**

```vim
rabbitmqctl set_policy [-p Vhost] Name Pattern Definition [Priority]
 
-p Vhost：  可选参数，针对指定vhost下的queue进行设置
Name:       policy的名称
Pattern:    exchanges或queue的匹配模式(正则表达式)
Definition：镜像定义，包括三个部分ha-mode, ha-params, ha-sync-mode
    ha-mode:指明镜像队列的模式，有效值为 all/exactly/nodes
        all：表示在集群中所有的节点上进行镜像
        exactly：表示在指定个数的节点上进行镜像，节点的个数由ha-params指定
        nodes：表示在指定的节点上进行镜像，节点名称通过ha-params指定
    ha-params：ha-mode模式需要用到的参数
    ha-sync-mode：进行队列中消息的同步方式，有效值为automatic和manual。automatic:新增加节点自动同步全量数据。manual: 新增节点只同步新增数据，全量数据需要手工同步。
Priority：可选参数，policy的优先级
```

#### **设置示例**

在**虚拟机结点rabbitmq1**中完成操作：

```shell
# 所有队列exchangess 或者 queue都为镜像模式
[root@rabbitmq1 ~] rabbitmqctl set_policy ha-all "^" '{"ha-mode":"all"}'


# 对队列名称以“queue_”开头的所有队列进行镜像，并在集群的两个节点上完成进行，policy的设置命令为：
[root@rabbitmq1 ~] rabbitmqctl set_policy --priority 0 --apply-to queues mirror_queue "^queue_" '{"ha-mode":"exactly","ha-params":2,"ha-sync-mode":"automatic"}'
```