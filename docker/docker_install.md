# Docker 安装

Docker 划分为 CE 和 EE. CE 即社区版(免费，支持周期三个月)，EE 即企业版，强调安全，付费使用。

Docker CE 每个月发布一个 Edge 版本，每三个月发布一个 Stable 版本, Docker EE 和 Stable 版本号保持一致，但每个版本提供一年维护。

> 官方网站上有各种环境下的[安装指南](https://docs.docker.com/install/).

## Ubuntu 安装 Docker CE

> 注意：切勿在没有配置 Docker APT 源的情况下直接使用 apt 命令安装 Docker。

### 系统要求

Docker CE 支持一下版本的 Ubuntu 操作系统：

- Artful 17.10 (Docker CE 17.11 Edge +)
- Xenial 16.04 (LTS)
- Trusty 14.04 (LTS)

Docker CE 可以安装在 64 位的 x86 平台或 ARM 平台上。Ubuntu 发行版本中，LTS(Long-Term-Support)长期支持版本，会获得 5 年的升级维护支持，这样的版本会更稳定，因此在生产环境中推荐使用 LTS 版本，当前最新的 LTS 版本为 Ubuntu 16.04。

### 卸载旧版本

就版本的 Docker 称为 docker 或者 docker-engine, 使用一下命令卸载旧版本：

```shell
$ sudo apt-get remove docker \
        docker-engine \
        docker.io
```

### 使用 APT 安装

由于 apt 源使用 HTTPS 以确保软件下载过程中不被篡改。因此，我们首先需要添加使用 HTTPS 传输的软件包以及 CA 证书。

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

然后，我们需要向 `source.list` 中添加 Docker 软件源

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

> 以上命令会添加稳定版本的 Docker CE APT 镜像源，如果需要最新或者测试版本的 Docker CE 请将 stable 改为 edge 或者 test。从 Docker 17.06 开始，edge test 版本的 APT 镜像源也会包含稳定版本的 Docker。

### 安装 Docker CE

更新 apt 软件包缓存，并安装 `docker-ce` ：

```shell
$ sudo apt-get update

$ sudo apt-get install docker-ce
```

### 使用脚本自动安装

在测试或者开发环境中 Docker 官方为了简化安装流程，提供了一套便捷的安装脚本，Ubuntu 系统上可以使用这套脚本安装：

```shell
$ curl -fsSL get.docker.com -o get-docker.sh

$ sudo sh get-docker.sh --mirror Aliyun
```

执行这个命令后，脚本就会自动的将一切准备工作做好，并且把 Docker CE 的 Edge 版本安装在系统中。

### 启动 Docker

```shell
$ sudo systemctl enable docker
$ sudo systemctl start docker
```

Ubuntu 14.04 请使用以下命令启动：

```shell
$ sudo service docker start
```

### 建立 docker 用户组

默认情况下，`docker` 命令会使用 Unix socket 与 Docker 引擎通讯。而只有 `root` 用户和 `docker` 组的用户才可以访问 Docker 引擎的 Unix socket。处于安全考虑，一般 Linux 系统上不会直接使用 `root` 用户。因此，更好的做法是将需要使用 `docker` 的用户加入 `docker` 用户组。

建立 `docker` 组：

```shell
$ sudo groupadd docker
```

将当期用户加入 `docker` 组：

```shell
$ sudo usermod -aG docker $USER
```

退出当期终端并重新登录，进行如下测试。

### 测试 Docker 是否安装正确

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

## CentOS 安装 Docker CE

> 注意：切勿在没有配置 Docker YUM 源的情况下直接使用 yum 命令安装 Docker.

### 系统要求

Docker CE 支持 64 位版本 CentOS 7，并且要求内核版本不低于 3.10。 CentOS 7 满足最低内核的要求，但由于内核版本比较低，部分功能（如 overlay2 存储层驱动）无法使用，并且部分功能可能不太稳定。

### 卸载旧版本

旧版本的 Docker 称为 `docker` 或者 `docker-engine`，使用以下命令卸载旧版本：

```shell
$ sudo yum remove docker \
        docker-client \
        docker-client-latest \
        docker-common \
        docker-latest \
        docker-latest-logratate \
        docker-logratate \
        docker-selinux \
        docker-engine-selinux \
        docker-engine
```

### 使用 yum 安装

执行一下命令安装依赖包：

```shell
$ sudo yum install -y yum-utils \
        device-mapper-persistent-data \
        lvm2
```

如果需要最新版本的 Docker CE 请使用一下命令：

```shell
$ sudo yum-config-manager --enable docker-ce-edge
```

如果需要测试版本的 Docker CE 请使用一下命令:

```shell
$ sudo yum-config-manager --enable docker-ce-test
```

### 安装 Docker CE

更新 yum 软件源缓存，并安装 `docker-ce`。

```shell
$ sudo yum makecache fast
$ sudo yum install docker-ce
```

### 启动 Docker CE

```shell
$ sudo systemctl enable docker
$ sudo systemctl start docker
```

### 建立 docker 用户组

默认情况下，`docker` 命令会使用 Unix socket 与 Docker 引擎通讯。而只有 `root` 用户和 `docker` 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统商不会直接使用 `root`用户。因此，更好地做法是将需要使用 `docker` 的用户加入 `docker` 用户组。组建 `docker` 组：

```shell
$ sudo groupadd docker
```

将当前用户加入 `docker` 组

```shell
$ sudo usermod -aG docker $USER
```

退出当前终端并重新登录，进行如下测试。

### 测试 Docker 是否安装正确

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

- 参考文档: https://yeasy.gitbooks.io/docker_practice/content/install/centos.html

## Windows 10 PC 安装 Docker CE

### 系统要求

Docker for Windows 支持 64 位版本的 Windows 10 Pro，且必须开启 Hyper-V。

### 安装

官方网站下载 Docker for Windows。

下载好之后双击 Docker for Windows Installer.exe 开始安装。

## 镜像加速器

国内从 Docker Hub 拉取镜像有时会遇到困难，此时可以配置镜像加速器。Docker 官方和国内很多云服务商都提供了国内加速器服务。例如：

- [Docker 官方提供的中国 registry mirror https://registry.docker-cn.com](https://docs.docker.com/registry/recipes/mirror/#use-case-the-china-registry-mirror)

### Ubuntu 14.04、Debian 7 Wheezy

对于使用 upstart 的系统而言，编辑 `/etc/default/docker` 文件，在其中的 DOCKER_OPTS 中配置加速器地址：

```shell
DOCKER_OPTS="--registry-mirror=https://registry.docker-cn.com"
```

重启服务：

```shell
$ sudo service docker restart
```

### Ubuntu 16.04+、Debian 8+、CentOS 7

对于使用 systemd 的系统，请在 `/etc/docker/daemon.json` 中写入如下内容(如果文件不存在的请新建该文件)

```json
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
```

> 注意：一定要保证该文件符合 json 规范，否则 Docker 将不能启动。

之后重新启动服务。

```shell
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

> 注意：如果您之前查看旧教程，修改了 docker.service 文件内容，请去掉您添加的内容（--registry-mirror=https://registry.docker-cn.com），这里不再赘述。

### Windows 10

对于使用 Windows 10 的系统，在系统右下角托盘 Docker 图标内右键菜单选择 `Settings`，打开配置窗口后左侧导航菜单选择 `Daemon`。在 `Registry mirrors` 一栏中填写加速器地址 `https://registry.docker-cn.com`，之后点击 `Apply` 保存后 `Docker` 就会重启并应用配置的镜像地址了。

### macOS

对于使用 macOS 的用户，在任务栏点击 Docker for mac 应用图标 -> Perferences... -> Daemon -> Registry mirrors。在列表中填写加速器地址 `https://registry.docker-cn.com`。修改完成之后，点击 `Apply & Restart` 按钮，Docker 就会重启并应用配置的镜像地址了。

### 检查加速器是否生效

配置加速器之后，如果拉取镜像仍然十分缓慢，请手动检查加速器配置是否生效，在命令行执行 `docker info`，如果从结果中看到了如下内容，说明配置成功。

```shell
$ docker info

...
Registry Mirrors:
 https://registry.docker-cn.com/
...
```
