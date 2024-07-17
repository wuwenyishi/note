服务器A是Linux系统，部署了Docker，Docker里安装了Mysql容器，网络配置是Host （此服务器配置了Hosts文件）；

服务器B是Windows系统，安装了Docker Desktop，在Docker Desktop 安转了Mysql容器，网络配置为Host；

在服务器B使用Navicat连接服务器A的Mysql能够连接成功，连接服务器B的Mysql连接失败；

将服务器B里的Mysql的网络配置改为 bridge ，Navicat连接成功；

在服务器B里部署Java服务容器，网络配置为Host，但Java服务容器需要连接服务器A和服务器B两个Mysql，启动容器时，能够连接服务器B的Mysql，不能连接服务器A的Mysql；

将服务器B里部署Java服务容器网络配置改为bridge，两个Mysql都能连接成功；（数据连接必须使用IP地址，不能使用localhost）；

查了资料得知以下：

* 在 Windows 上，`docker network host` 不起作用的原因是因为 Docker Desktop for Windows 使用了 Hyper-V 虚拟网络，而不是主机网络。因此，在 Windows 上使用 `docker network host` 命令将容器连接到虚拟网络，而不是主机网络，因此容器无法访问主机上的服务。





* 在 Windows 上，Docker 并不支持直接使用 `--network=host` 参数来设置容器的网络模式为主机模式。这是因为 Windows 和 Linux 在网络方面的架构有所不同，而 Docker 在 Windows 上不支持完全的主机网络模式。

  如果你尝试在 Windows 上运行 `docker network host` 命令，你会收到错误消息，指示网络主机不存在，因为 Docker 引擎无法识别或实现该命令。

  要在 Windows 上实现类似的功能，你可以考虑以下替代方法：

  1. **使用桥接网络或 Overlay 网络**：创建一个自定义的 Docker 网络，并将容器连接到该网络中。这样可以允许容器之间相互通信，也可以让容器与主机进行通信。

  2. **显式端口绑定**：使用 `-p` 或 `--publish` 参数，将容器的端口映射到主机上的端口。这样可以实现容器和主机之间的通信。

  3. **使用其他网络解决方案**：考虑使用其他网络解决方案，例如 Docker Compose 中的网络设置、容器编排工具等。

  综上所述，在 Windows 上，你需要使用其他方法来实现类似于主机网络模式的功能，而不能直接使用 `docker network host` 命令。





* Unfortunately *host* networking is [not available](https://docs.docker.com/network/network-tutorial-host/#prerequisites) for *Docker for Windows* and [neither is](https://docs.docker.com/network/network-tutorial-macvlan/#prerequisites) *macvlan* networking. If you are not stuck on *Hyper-V*, consider using [Docker Toolbox](https://docs.docker.com/toolbox/) on Windows.

  Quoted from: https://docs.docker.com/network/network-tutorial-host/#prerequisites

  > The host networking driver only works on Linux hosts, and is not supported on Docker for Mac, Docker for Windows, or Docker EE for Windows Server.

  [windows run docker with --network=host and access with 127.0.0.1 - Stack Overflow](https://stackoverflow.com/questions/48915458/windows-run-docker-with-network-host-and-access-with-127-0-0-1)





Docker 网络解析：

在 Docker 中，网络是一个重要的概念，用于管理容器之间的通信以及容器与外部世界的连接。Docker 提供了不同类型的网络模式，允许您根据需求配置容器的网络连接方式。以下是一些常见的 Docker 网络模式和解析：

1. **桥接网络（Bridge Network）**：
   - 默认情况下，Docker 使用桥接网络来连接容器。每个容器都有自己的 IP 地址，并且可以通过容器名称或 IP 地址相互通信。
   - 桥接网络允许容器与主机以及其他容器进行通信，但默认情况下不允许容器之间的外部访问。

2. **主机网络（Host Network）**：
   - 使用主机网络模式时，容器与主机共享网络命名空间。容器将直接使用主机的网络接口，因此可以访问主机上的所有网络服务。
   - 这种模式的优点是网络性能更好，因为容器与主机之间没有额外的网络地址转换。

3. **无网络（None Network）**：
   - 在无网络模式下，容器不会连接到任何网络。这意味着容器无法通过网络与外部通信，但仍可以在容器内部进行通信。

4. **覆盖网络（Overlay Network）**：
   - 覆盖网络允许跨多个 Docker 守护程序连接容器。这对于在多个主机上运行容器的分布式应用程序非常有用。
   - 使用覆盖网络，容器可以在不同主机上的不同容器之间建立通信。

5. **自定义网络（Custom Networks）**：
   - 您还可以创建自定义网络，以便更好地控制容器之间的通信。通过创建自定义网络，您可以定义子网、网关和其他网络设置。
   - 自定义网络可以帮助组织和管理容器，并允许容器在不同网络环境中运行，而不会受到外部网络的干扰。

通过选择适当的网络模式，您可以更好地管理容器之间的通信和与外部世界的连接。根据您的特定需求和应用程序架构，选择合适的网络模式对于确保容器正常运行和安全性非常重要。
