# Docker 安装

Docker 划分为CE和EE. CE即社区版(免费，支持周期三个月)，EE即企业版，强调安全，付费使用。

Docker CE每个月发布一个Edge版本，每三个月发布一个Stable版本, Docker EE 和Stable 版本号保持一致，但每个版本提供一年维护。

> 官方网站上有各种环境下的[安装指南](https://docs.docker.com/install/).

## Ubuntu 安装Docker CE 

> 注意：切勿在没有配置Docker APT源的情况下直接使用apt命令安装Docker。

### 系统要求 

Docker CE支持一下版本的Ubuntu 操作系统：

+ Artful 17.10 (Docker CE 17.11 Edge +)
+ Xenial 16.04 (LTS)
+ Trusty 14.04 (LTS)

Docker CE可以安装在64位的x86平台或ARM平台上。Ubuntu发行版本中，LTS(Long-Term-Support)长期支持版本，会获得5年的升级维护支持，这样的版本会更稳定，因此在生产环境中推荐使用LTS版本，当前最新的LTS版本为Ubuntu 16.04。

### 卸载旧版本

就版本的Docker称为docker或者docker-engine, 使用一下命令卸载旧版本：

```shell
$ sudo apt-get remove docker \
        docker-engine \
        docker.io
```

### 使用APT安装

由于apt源使用HTTPS以确保软件下载过程中不被篡改。因此，我们首先需要添加使用HTTPS传输的软件包以及CA证书。

```shell

$ sudo apt-get update

$ sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        software-properties-common
```

鉴于国内网络问题，强烈建议使用国内源，官方源请在注释中查看。

为了确认所下载软件包的合法性，需要添加软件源的 `GPG` 秘钥。

```shell
$ curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -

# 官方源
# $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

然后，我们需要向 `source.list` 中添加Docker软件源

```shell

$ sudo add-apt-repository \
    "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu \
    $(lsb_release -cs) \
    stable"

# 官方源
# $ sudo add-apt-repository \
#    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
#    $(lsb_release -cs) \
#    stable"
```

> 以上命令会添加稳定版本的Docker CE APT镜像源，如果需要最新或者测试版本的Docker CE请将stable改为edge或者test。从Docker 17.06开始，edge test 版本的APT镜像源也会包含稳定版本的Docker。

### 安装Docker CE 

更新apt软件包缓存，并安装 `docker-ce` ：

```shell
$ sudo apt-get update

$ sudo apt-get install docker-ce
```

### 使用脚本自动安装

在测试或者开发环境中Docker官方为了简化安装流程，提供了一套便捷的安装脚本，Ubuntu系统上可以使用这套脚本安装：

```shell
$ curl -fsSL get.docker.com -o get-docker.sh

$ sudo sh get-docker.sh --mirror Aliyun
```

执行这个命令后，脚本就会自动的将一切准备工作做好，并且把Docker CE的Edge版本安装在系统中。

### 启动Docker
```shell
$ sudo systemctl enable docker
$ sudo systemctl start docker
```

Ubuntu 14.04请使用以下命令启动：

```shell
$ sudo service docker start
```

### 建立docker 用户组

默认情况下，`docker` 命令会使用 Unix socket 与Docker引擎通讯。而只有 `root` 用户和 `docker` 组的用户才可以访问 Docker 引擎的Unix socket。处于安全考虑，一般Linux系统上不会直接使用 `root` 用户。因此，更好的做法是将需要使用 `docker` 的用户加入 `docker` 用户组。

建立 `docker` 组：

```shell
$ sudo groupadd docker
```

将当期用户加入 `docker` 组：

```shell
$ sudo usermod -aG docker $USER
```

退出当期终端并重新登录，进行如下测试。

### 测试Docker 是否安装正确

```shell

$ docker run hello-world 

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
ca4f61b1923c: Pull complete
Digest: sha256:be0cd392e45be79ffeffa6b05338b98ebb16c87b255f48e297ec7f98e123905c
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://cloud.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/

```

若能正常输出以上信息，则说明安装成功。
