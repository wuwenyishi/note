什么是 Nacos

Nacos 是阿里巴巴开源的一款优秀的中间件，在分布式微服务场景下用的非常多。

**Nacos** 英文全称 **Naming and Configuration Service**，其中 Na 是 naming 的缩写，注册中心；co为 configuration 的缩写， 配置中心；

不管是配置中心还是注册中心本质都是围着**服务（微服务）**转的，用官方的话来说：**服务在 Nacos 里是一等公民**。

Nacos 分为服务端和客户端，这一点不要含糊。

**Nacos Server**

Nacos Server 使用 Java 语言编写，提供了服务注册发现和服务配置功能；对外提供了 SDK 接入以及HTTP RESTful 开放接口，SDK 接入和 RESTful 接口的功能是一致的。

**Nacos Client**

Nacos提供了官方的 SDK，遗憾的是只有 Java 版本，官方的 SDK 可以很方便与一些主流的框架进行集成，比如：Spring Cloud、Dubbo 等。

Nacos Client 主要的作用是订阅获取服务实例信息以及配置信息。

**数据模型**

在 Nacos中最重要的就是服务，为了方便管理还引入了**数据模型**这个概念，数据模型主要分为**命名空间**、**集群**、**服务**。

![图片](https://xuemingde.com/pages/image/2022/03/31/141103.jpeg)

数据模型主要作用是给服务分类，方便维护管理。如果觉得抽象，简单举个例子：假如你叫张三，生活在武汉，国籍是中国，在其他城市或者省也有张三这个人。

# Nacos 注册中心原理

![图片](https://xuemingde.com/pages/image/2022/03/31/141114.jpeg)

服务注册成功后，为了向 Nacos Server 报告自己的健康状态，客户端每5秒向 Nacos server发送一次心跳，心跳带上了服务名，服务ip，服务端口等信息。当然 Nacos server 也会向 client 主动发起健康检查，支持tcp/http检查。

如果15秒内无心跳且健康检查失败则会将该实例标记为不健康的状态，如果30秒内健康检查再次失败则会直接剔除实例。

服务消费者订阅成功后，如果服务提供者的实例不健康或者被剔除掉了，Nacos Server 会发送变更通知。

# 小试牛刀：Nacos 服务注册发现功能体验

对理论知识稍加理解后，我们可以动动小手，把 Nacos真正用起来吧。

## 搭建运行 Nacos Server

**（1）下载 Nacos Server 安装包，启动 Nacos Server**

进入 Github Nacos 下载主页：

> 主页链接：
>
> https://*github*.com/alibaba/nacos/releases

目前最新的稳定版本是 2.0.4，直接下载 zip 包或者 tar.gz包即可，windows 和 Linux 均可运行。

![图片](https://xuemingde.com/pages/image/2022/03/31/141124.png)

解压完毕，如果是 Linux 系统，执行以下命令，以单实例的方式运行：

```
sh startup.sh -m standalone
```

如果是 windows 系统，执行以下命令：

```
startup.cmd -m standalone
```

**（2）使用 Docker 启动 Nacos Server**

如果你电脑上已经装了 Docker，建议你直接使用 Docker 来运行。

先拉取最新镜像：

```
docker pull nacos/nacos-server
```

拉取成功后启动实例：

```shell
docker run --name nacos-demo -e MODE=standalone -p 8848:8848 -p 9848:9848 -d nacos/nacos-server
```

这里有坑需要注意下，8848 是 Nacos Server 的主端口需要暴露出来，如果你安装的是 Nacos 2.0 版本还需要将 9848 端口暴露出来，这里我含泪调试了一下午才知道的，哭晕……

为什么需要 9848 端口呢？因为 Nacos 2.0 版本之后默认将这个端口作为 grpc 的通信端口，官方提供的 Client SDK使用 grpc 来与 Nacos Server 进行通信，包括服务实例注册、心跳检查等功能。

> 这里需要说明下：grpc 的端口 = 主端口 + 偏移量1000，假如你的主端口是 8848，加上偏移量就是 9848

## 熟悉 Nacos Server 控制台界面

Nacos Server 运行成功后我们可以打开后台管理界面，查看其运行状态和管理信息。

> 本地访问地址：http://127.0.0.1:8848/nacos

第一次打开默认进入后台登录界面，默认用户名和密码都是：**nacos**

![图片](https://xuemingde.com/pages/image/2022/03/31/141137.png)

登录成功后可以看到左侧的菜单栏，主要功能有：**配置管理**、**服务管理**、**权限管理**、**命名空间**、**集群管理**。

![图片](https://xuemingde.com/pages/image/2022/03/31/141149.png)

**（1）服务管理**

Nacos 的主要功能分为两块：**配置管理**和**服务管理**，这次主要展开讲解服务管理。

![图片](https://xuemingde.com/pages/image/2022/03/31/141200.png)

展开菜单后，有**服务列表**和**订阅者列表**两块，服务列表会显示所有注册到 Nacos Server 的服务，包括实例数、实例健康状态等信息。

**订阅者列表**会显示某个服务下有哪些客户端订阅了，以及包括客户端的版本信息等。

**（2）权限控制**

权限控制主要的作用是维护管理后台系统的用户角色和权限，一般的系统都有这个功能，这里不再赘述了。

![图片](https://xuemingde.com/pages/image/2022/03/31/141212.png)

**（3）命名空间**

命名空间比较好理解，比如同样一个服务ServiceA可能会在研发环境、集成测试环境、生产环境各自部署一套，那如何区分它们呢？命名空间可以起这个作用，在下图中，我新建了好几个命名空间，用于将服务的注册订阅信息在逻辑上隔离开来。

![图片](https://xuemingde.com/pages/image/2022/03/31/141223.png)

**（4）集群管理**

Nacos Server可以是集群部署的也可以是单机部署，在实际生产环境中为了防止单点故障我们肯定不可能部署一个节点，为了方便测试演示，我在本地只启动了一个节点，下图中可以看到这个节点的ip、状态等信息。

![图片](https://xuemingde.com/pages/image/2022/03/31/141236.png)

## 学习使用 Nacos Client SDK

目前官网只推出了 Java 版的 SDK，其他语言版本暂时靠社区自行贡献。

我们拿 Java 版本 SDK 为例进行说明。

新建一个 Java Maven 工程，引入nacos-client 依赖：

```xml
<dependency>
  <groupId>com.alibaba.nacos</groupId>
  <artifactId>nacos-client</artifactId>
  <version>2.0.3</version>
</dependency>
```

上面讲过 Nacos 主要分为两块大的功能：**配置中心**、**注册中心**，为了方便使用，Nacos-client 提供了三个工厂类：`NacosFactory`、`ConfigFactory`、`NamingFactory`，使用这些工厂类很容易生成对应实例对象。

![图片](https://xuemingde.com/pages/image/2022/03/31/141256.jpeg)

NacosFactory 包含了 ConfigFactory 和 NamingFactory的所有功能，如果你想创建一个注册中心功能的实例，你可以使用：

```
NacosFactory.createNamingService()` 或者使用 `NamingFactory.createNamingFactory()
```

## 使用 Nacos Client 注册实例

Nacos Client SDK 代码非常简单，基本一看就能学会怎么用，下面直接贴代码，可以运行的那种代码。

```java
import com.alibaba.nacos.api.exception.NacosException;
import com.alibaba.nacos.api.naming.NamingFactory;
import com.alibaba.nacos.api.naming.NamingService;

/**
 * 模拟服务提供者注册服务实例
 */
public class App {
    public static void main(String[] args) throws NacosException {
        // 使用工厂类创建注册中心对象，构造参数为 Nacos Server 的 ip 地址，连接 Nacos 服务器
        NamingService naming = NamingFactory.createNamingService("127.0.0.1:8848");
        // 打印 Nacos Server 的运行状态
        System.out.println("server status: " + naming.getServerStatus());
        // 模拟注册当前服务实例，传入参数：服务名、ip 地址、端口
        naming.registerInstance("com.leixiaoshuai.rpc.provider", "11.11.11.11", 8888);

        // 模拟当前进程不退出
        while (true) {
        }
    }
}
```

看完 demo 之后有没有同学比较好奇，Nacos Client 是通过什么协议与 Nacos Server通信的？简单看下源码就可以得到答案：Google grpc

这里需要特别说明一下：在 Nacos 1.x 时代是使用 HTTP RESTful 接口与Nacos Server交互的，后面 2.x 时候为了提升效率改成了 grpc。

## 使用 Nacos Client 消费实例

假如服务 A 需要调用服务 B 的接口，首先得知道服务 B 实例的 ip 和端口，在这个场景下服务 A 就是服务消费者了，通过简单的代码很容易获取到服务实例信息，为了方便感知服务实例变化，Nacos 还提供了事件通知能力。

```java
import com.alibaba.nacos.api.exception.NacosException;
import com.alibaba.nacos.api.naming.NamingFactory;
import com.alibaba.nacos.api.naming.NamingService;
import com.alibaba.nacos.api.naming.listener.Event;
import com.alibaba.nacos.api.naming.listener.EventListener;
import com.alibaba.nacos.api.naming.listener.NamingEvent;

/**
 * 模拟服务消费者订阅服务实例
 */
public class App {
    public static void main(String[] args) throws NacosException {
        // 使用工厂类创建注册中心对象，构造参数为 Nacos Server 的 ip 地址，连接 Nacos 服务器
        NamingService naming = NamingFactory.createNamingService("127.0.0.1:8848");
        // 打印 Nacos Server 的运行状态
        System.out.printf("server status: %s", naming.getServerStatus());
        // 获取指定服务所有的实例列表
        System.out.println(naming.getAllInstances("com.leixiaoshuai.rpc.provider"));
        // 订阅指定服务，并注册回调接口
        naming.subscribe("com.leixiaoshuai.rpc.provider", new EventListener() {
            @Override
            public void onEvent(Event event) {
                // 服务实例有变动就自动收到通知
                System.out.println("~~event start");
                System.out.println(((NamingEvent) event).getServiceName());
                System.out.println(((NamingEvent) event).getInstances());
                System.out.println("~~event end");
            }
        });

        // 模拟当前进程不退出
        while (true) {}
    }
}
```

# 小结

Nacos 是阿里巴巴开源的一款中间件，常用于分布式微服务场景，主要功能包括两大块：服务注册发现、服务配置。

Nacos 分为 server 和 client，可以在官网下载安装包在本地运行，运行成功后通过后台管理界面对 Nacos Server 进行管理和维护；

Client 端的接入方式有 SDK 和 HTTP RESTful 两种方式，功能都是一样的。

> 原文连接：[折腾一个周末，撸Nacos可真不容易](https://mp.weixin.qq.com/s/2cRfCsXe5UoK5ZoiX9xliA)