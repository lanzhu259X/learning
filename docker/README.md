# Docker 介绍

> 出处：[Docker — 从入门到实践](https://yeasy.gitbooks.io/docker_practice/content/) 

Docker 是个划时代的开源项目，他彻底释放了计算虚拟化的威力，极大提高了应用的维护效率，降低了云计算应用开发的成本！使用 Docker, 可以让应用的部署、测试和分发都变得前所未有的高效和轻松！

无论是应用开发者、运维人员、还是其他信息技术从业人员，都有必要认知和掌握 Docker, 节约有限的生命。

> 软件开发最大的麻烦事之一，就是环境配置。用户计算机的环境都不相同，你怎么知道自家的软件，能在那些机器商跑起来？
>
> 用户必须保证两件事：操作系统的设置，各种库和组件的安装。只有他们都正确，软件才能运行。举例来说，安装一个 Python 应用，计算机必须有 Python 引擎，还必须有各种依赖，可能还要配置环境变量。
>
> 环境配置如此麻烦，换一台机器，就要重来一次，旷日费时。很多人想到，能不能从根本商解决问题，软件可以带环境安装？也就是说，安装的时候，把原始环境一模一样的复制过来。

Docker 使用 Googleg 公司推出的 Go 语音进行开发实现，基于 Linux 内核的 cgroup, namespace，以及 AUFS 类的 Union FS 等技术，堆进程进行封装隔离，属于 **操作系统层面的虚拟化技术**。由于隔离的进程独立于宿主和其他的隔离的进程，因此也称其为容器。最初实现是基于 LXC，从 0.7 版本以后开始去除 LXC，转而使用自行开发的 libcontainer，从 1.11 开始，则进一步演进为使用 runC 和 containerd。

Docker 在容器的基础商，进行了进一步的封装，从文件系统。网络互联到进程隔离等等，极大的简化了容器的创建和维护。使得 Docker 技术虚拟机技术更为轻便、快捷。

下面的图片比较了 Docker 和传统虚拟化方式的不同之处。传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上在运行所需应用进程；而容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核，而且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便。

传统虚拟化：

![image](./image/readme-1.png)

Docker: 

![image](./image/readme-2.png)

- **虚拟机**

  虚拟机(virtual machine)就是带环境安装的一种解决方案。他可以在一种操作系统里面运行另一种操作系统，比如在 windows 系统里运行 linux 系统。应用程序堆此毫无感知，因此虚拟机看上去跟真实系统一模一样，而对于底层系统来说，虚拟机就是一个普通文件，不需要了就删除掉，对其他部分毫无影响。虽然用户可以通过虚拟机还原软件的原始环境。但是，这几个方案有几个缺点：

  1.  资源占用多

      虚拟机会独占一部分内存和硬盘空间。他运行的时候，其他程序就不能使用这些资源了，哪怕虚拟机里面的应用程序，真正使用的内存只有 1MB，虚拟机依然需要几百 MB 的内存才能运行。

  2.  冗余步骤多

      虚拟机是完整的操作系统，一些系统级别的操作步骤，往往无法跳过，比如用户登录。

  3.  启动慢

      启动操作系统需要多久，启动虚拟机就需要多久。可能要等几分钟，应用程序才能真正运行。

- **Linux 容器**

  由于虚拟机存在这些缺点，Linux 发展出了另一种虚拟化技术：linux 容器(Linux containers 缩写 LXC)。

  **LXC 不是模拟一个完整的操作系统，而是堆进程进行隔离。** 或者说，在正常进程的外面套了一个保护壳，对于容器里面的进程来说，它接触到的各种资源都是虚拟的，从而实现与底层系统的隔离。

  由于容器是进程级别的，相比虚拟机有很多优势：

  1.  启动快

      容器里面的应用，直接就是底层系统的一个进程，而不是虚拟机内部的进程。所以，启动容器相当于启动本机的一个进程，而不是启动一个操作系统，速度就快很多。

  2.  资源占用少

      容器只占用需要的资源，不占用那些没有用到的资源；虚拟机由于是完整的操作系统，不可避免要占用所有资源。另外，多个容器可以共享资源，虚拟机都是独享资源。

  3.  体积小

      容器只要包含用到的组件即可，而虚拟机是整个操作系统的打包，所以容器文件比虚拟机文件要小很多。

- **Docker**

  **Docker 属于 Linux 容器的一种封装，提供简单易用的容器使用接口。** 它是目前最流行的 linux 容器解决方案。

  Docker 将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生产一个虚拟容器。程序在这个虚拟容器里运行，就好像在真实的物理机商运行一样。有了 Docker，就不用担心环境问题。

  总体来说，Docker 的接口相当简单，用户可以方便地创建和使用容器，把自己的应用放入容器。容器还可以进行版本管理、复制、分享、修改，就像管理普通的代码一样。

- **Docker 用途**

  Docker 的主要用途，目前有三大类：

  1.  **提供一次性的环境**。比如，本地测试他人软件、持续集成的时候提供单元测试和构建的环境。
  2.  **提供弹性的云服务**。因为 Docker 容器可以随开随关，很适合动态扩容和缩容。
  3.  **组建微服务架构**。通过多个容器，一台机器可以跑多个服务，因此在本机就可以模拟出微服务架构。

对比传统虚拟机总结：

| 特性       | 容器               | 虚拟机     |
| ---------- | ------------------ | ---------- |
| 启动       | 秒级               | 分钟级     |
| 硬盘使用   | 一般为 MB          | 一般为 GB  |
| 性能       | 接近原生           | 弱于       |
| 系统支持量 | 单机支持上千个容器 | 一般几十个 |
