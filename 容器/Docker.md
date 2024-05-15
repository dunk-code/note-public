> 应用放在`/usr/local/bin`可以全局使用

[Docker理论与实践系列章节一：Docker安装部署](https://blog.csdn.net/yy339452689/article/details/106489978?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162330838416780255262824%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=162330838416780255262824&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-1-106489978.nonecase&utm_term=docker&spm=1018.2226.3001.4450)

[Docker](https://blog.csdn.net/thinkwon/category_9380833.html)

[Docker命令](https://www.kuangstudy.com/bbs/1377188745530343425)

[官网](https://docs.docker.com/engine/reference/commandline/images/)

[菜鸟](https://www.runoob.com/docker/docker-image-usage.html)

[视频](https://www.bilibili.com/video/BV1og4y1q7M4?spm_id_from=333.999.0.0)

# 虚拟机和容器

- 虚拟机和容器都属于虚拟化技术

- 虚拟机是在一套硬件上，虚拟出一个完整的操作系统，在该系统上再运行所需的应用程序

- 容器不是虚拟出一个完整的操作系统，而是对进程的隔离

> 容器是在操作系统层面上实现虚拟化，直接复用本地主机的操作系统，而虚拟机则是在硬件层面实现

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210610002121.png" alt="image-20210610002118655" style="zoom:80%;" />

## 虚拟机和容器的区别

> Docker有着比虚拟机更少的抽象层

|  **特性**  |       **虚拟机**        |         **容器**         |
| :--------: | :---------------------: | :----------------------: |
|    量级    |         重量级          |          轻量级          |
|    性能    |        接近原生         |         弱于原生         |
|  启动时间  |      以分钟为单位       |       以毫秒为单位       |
|  硬盘使用  |        一般为 GB        |        一般为 MB         |
|  占用资源  | 占用更多的内存和CPU资源 | 占用较少的内存和CPU资源  |
|   隔离性   |  完全隔离，因此更安全   | 进程级隔离，可能不太安全 |
| 系统支持量 |       一般几十个        |      支持上千个容器      |

# 认识Docker

## 什么是Docker

- Docker是开源应用容器引擎，轻量级容器技术，开发者可以打包自己的应用到容器里面，然后迁移到其他机器的docker应用中，可以实现快速部署，如果出现故障是，可以通过镜像，快速恢复服务
- Docker诞生于2013年初，最初是dotCloud公司内部的一个业余项目。它基于Google公司推出的Go语言实现。项目后来加入了Linux基金会，遵从了 Apache 2.0 协议，项目代码在 [GitHub](https://github.com/docker/docker) 上进行维护
- Docker 自开源后受到广泛的关注和讨论，以至于 dotCloud 公司后来都改名为 Docker Inc。RedHat 已经在其 RHEL6.5 中集中支持 Docker；Google 也在其 PaaS 产品中广泛应用
- Docker 项目的目标是实现轻量级的操作系统虚拟化解决方案。 Docker 的基础是 Linux 容器（LXC）等技术

> LXC（Linux Container）：容器是一种内核虚拟化技术，可以提供轻量级的虚拟化，以便隔离进程和资源

- 在 LXC 的基础上 Docker 进行了进一步的封装，让用户不需要去关心容器的管理，使得操作更为简便。用户操作 Docker 的容器就像操作一个快速轻量级的虚拟机一样简单

## Docker的优点

> 作为一种新兴的虚拟化技术，Docker 跟传统的虚拟机相比具有众多的优势

- 一致的运行环境：Docker 的镜像提供了除内核外完整的运行时环境，确保了应用运行环境的一致性，从而不会再出现“这段代码在我机器上没问题啊”这类问题。
- 更快速的启动时间：可以做到秒级、甚至毫秒级的启动时间。大大的节约了开发、测试、部署的时间。
- 隔离性：避免公用的服务器，资源会容易受到其他用户的影响。
- 弹性伸缩，快速扩展：善于处理集中爆发的服务器使用压力。
- 更轻松的迁移：Docker 容器几乎可以在任意的平台上运行，包括物理机、虚拟机、公有云、私有云、个人电脑、服务器等。 这种兼容性可以让用户把一个应用程序从一个平台直接迁移到另外一个。
- 持续交付和部署：使用Docker可以通过定制应用镜像来实现持续集成、持续交付、部署。

## Docker的应用场景

- Web应用的自动打包，自动测试和持续集成、快速部署
- 弹性的云服务：因为Docker容器可以随开随关，很适合动态扩容和缩容
- 提供一致性的环境：同步开发环境和生产环境

## Docker核心

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210610160418.jpg" alt="1" style="zoom:80%;" />

三个基本概念即生命周期

- 镜像（Image）：镜像好比一个模板，可以通过这个模板来创建容器服务，一个镜像可以创建多个容器
- 容器（Container）：独立运行一个或者一组应用，通过镜像创建
- 仓库（repository）：存放镜像的地方
  - 公有仓库
  - 私有创库

三个关键动作

- Build（构建镜像）：镜像就像是集装箱包括文件及运行环境等资源
- Ship（运输镜像）：主机和仓库间运输，这里的仓库就像超级码头一样
- Run（运行镜像）：正在运行的镜像就是一个容器，容器就是运行程序的地方

### Docker镜像

- 一个只读模板，可以用来创建容器，一个镜像可以创建多个容器
- Docker 提供了一个很简单的机制来创建和更新现有的镜像，甚至可以直接从其他人那里获取做好的镜像直接使用

### Docker容器

- 容器是从镜像创建的运行实例，也就是镜像启动后的一个实例称为容器，是独立运行的一个或一组应用
- Docker利用容器来运行应用，他可以被启动、开始、停止、删除，每个容器都是相互隔离的、保证安全的平台（沙箱）
- 可以把容器看做是一个简易版的 Linux（包括 root 用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序

> 注：镜像是只读的，容器在启动的时候创建一层可写层作为最上层。

### Docker仓库

- 仓库是集中放镜像文件的场所，有时候会把仓库和仓库注册服务器（Registry）混为一谈，并不严格区分。实际上，仓库注册服务器上往往放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签（tag）
- 仓库分为公开仓库（Public）和私有仓库（Private）两种形式
- 最大的公开仓库是 [Docker Hub](https://hub.docker.com/)，存放了数量庞大的镜像供用户下载。 国内的公开仓库包括 [Docker Pool](https://promotion.aliyun.com/ntms/act/kubernetes.html) 等，可以提供大陆用户更稳定快速的访问
- 当然，用户也可以在本地网络内创建一个私有仓库
- 当用户创建了自己的镜像之后就可以使用 `push` 命令将它上传到公有或者私有仓库，这样下次在另外一台机器上使用这个镜像时候，只需要从仓库上 `pull` 下来就可以了

> 注：Docker 仓库的概念跟 [Git](http://git-scm.com/) 类似，注册服务器可以理解为 GitHub 这样的托管服务

## 扩展问题

### Docker为什么快？

> 这里主要说一下Docker为什么比VMware快

- 首先Docker的抽象层比VMware虚拟机少，Docker不需要Hypervisor实现硬件资源的虚拟化，Docker需要的硬件都是实际物理机上的硬件资源。所以磁盘占用、CPU、内存利用率上Docker有很大的优势
- Docker利用的宿主机的内核，不需要Guest OS，当新建一个容器时，Docker不需要像虚拟机一样重新加载一个操作系统内核，这样就减少了加载操作系统内核这样的费时费资源过程。因为Docker使用宿主机的操作系统，新建一个Docker容器只需要几秒

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210627101557.png" alt="在这里插入图片描述" style="zoom:80%;" />



## 安装Docker

[官网教程](https://docs.docker.com/engine/install/centos/)

卸载旧版本

```bash
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

在新主机上首次安装 Docker Engine 之前，您需要设置 Docker 存储库。之后，您可以从存储库安装和更新 Docker

```bash
sudo yum install -y yum-utils
 
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

添加镜像

```shell
# 国外镜像
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# 阿里云镜像
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```



安装Docker引擎

```bash
sudo yum install docker-ce docker-ce-cli containerd.io
```

启动docker

```bash
sudo systemctl start docker
```

测试是否安装成功

```bash
sudo docker run hello-world
```

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210610172406.png" alt="image-20210610172402532" style="zoom:67%;" />

卸载docker

卸载 Docker Engine、CLI 和 Containerd 包

```bash
sudo yum remove docker-ce docker-ce-cli containerd.io
```

主机上的映像、容器、卷或自定义配置文件不会自动删除。删除所有镜像、容器和卷

> /var/lib/docker：默认工作路径

```bash
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```

.配置镜像加速器

针对Docker客户端版本大于 1.10.0 的用户

您可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://clcohfyf.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## Docker常用命令

> [命令文档](https://docs.docker.com/engine/reference/run/)

### 帮助命令

```shell
docker version				#docker版本信息
docker info					#docker详细信息，镜像和容器的数量
docker [命令] --help			#查看具体docker命令的解释
```

### 镜像命令

>docker images

```shell
docker images    #获取已下载镜像列表	

Options:
  -a, --all             #列出本地所有镜像（包含中间映像层）
      --digests         #显示镜像摘要信息			
  -f, --filter filter   #过滤
      --format string   Pretty-print images using a Go template
      --no-trunc        Don't truncate output
  -q, --quiet           #只显示镜像 ID
  --no-trunc			#显示完整的镜像信息
```

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210620115932.png" alt="image-20210620115928987" style="zoom:80%;" />

- `RESPOSITORY`：镜像名；
- `TAG`：镜像版本，`latest` 代表最新版；
- `IMAGE_ID` ：镜像唯一 ID；
- `CREATED` ：镜像的创建时间；
- `SIZE`：镜像的大小。

>docker search

```shell
docker search    #搜索镜像

-f, --filter filter：根据提供的条件过滤输出
#比如
docker search -f stars=3000 mysql	#列出星数不小于3000的mysql镜像；
--no-trunc：显示镜像完整描述信息；
--limit int：最大搜索结果数（默认 25）
```

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210620120451.png" alt="image-20210620120446005" style="zoom:80%;" />

- `NAME`：镜像名称
- `DESCRIPTION`：镜像描述
- `STARS`：星数（点赞）
- `OFFICAL`：是否是官方镜像
- `AUTOMATED`：是否是自动构建的

> docker  pull

```shell
docker pull [IMAGE_NAME]:[TAG]      #下载镜像
#IMAGE_NAME：镜像名，TAG ：标签，镜像版本，可选的，默认为 latest

docker pull mysql
Using default tag: latest
latest: Pulling from library/mysql
# 分层下载   结合联合文件系统
75646c2fb410: Pull complete   
878f3d947b10: Pull complete 
1a2dd2f75b04: Pull complete 
8faaceef2b94: Pull complete 
b77c8c445ec2: Pull complete 
074029aeaa5f: Pull complete 
5a1122545c6c: Pull complete 
6c95ccd00139: Pull complete 
60a719448fdb: Pull complete 
f31898a387a3: Pull complete 
bcf402a978dc: Pull complete 
cf0bc7da512e: Pull complete 
Digest: sha256:c35eb76bbccfd0138c8c68ccb9b4cffe42c488a27f64ddc31a2b5f65aa93fce6   #签名
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest   真实地址
```

> docker rmi

```shell
docker rmi       #删除镜像
docker rmi -f 镜像ID #删除指定镜像
#删除多个镜像
docker rmi -f ID ID ID
#删除所有镜像
docker rmi -f $(docker images -aq)
```

### 容器命令

> docker run [参数]  image

**docker run ：**根据镜像创建一个容器并运行一个命令，操作的对象是 **镜像**；

**docker exec ：**在运行的容器中执行命令，操作的对象是 **容器**。

```shell
#运行容器
docker run [参数]  image

-- name			#为容器起一个名称
-i				#以交互方式运行容器，通常与 -t 搭配使用
-t				#为容器重新分配一个伪输入终端，通常与 -i 搭配使用
-d				#执行完这句命令后，控制台将不会阻塞，可以继续输入命令操作，不会阻塞，也就是启动守护式容器
-it  			#会进入启动容器的命令控制台，也就是启动交互式容器	退出使用exit
-P				#随机端口映射
-p				#指定端口映射  8080:8080和主机端口映射
image-name		#要运行的镜像名称

Ctrl + p + q	#不会退出镜像
```

> docker ps

```bash
docker ps [参数]		#列出所有运行的容器

-a				#查看所有容器，包括已停止运行的
-q				#静默模式，只显示容器编号
-l				#显示最近创建的容器
-n 3			#显示最近创建的 num（此处为 3）个容器
--no-trunc		#不截断输出，显示完整信息
```

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210620122040.png" alt="image-20210620122035895" style="zoom:80%;" />

- `CONTAINER ID`：启动时生成的容器 ID；
- `IMAGE`：该容器使用的镜像；
- `COMMAND`：容器启动时执行的命令；
- `CREATED`：容器创建时间；
- `STATUS`：当前容器状态；
- `PORTS`：当前容器所使用的端口号；
- `NAMES`：启动时给容器设置的名称。

> docker stop 

```shell
docker stop container-name/container-id			#停止容器
```

> docker kill

```shell
docker kill container-name/container-id			#强制停止容器（类似强制关机）
```

> docker start 

```shell
docker start container-name/container-id		#启动容器
```

> docker rm

```shell
docker rm  $(docker ps -a -q)					# 删除所有容器
docker rm id									# 删除某个容器
```

#### run 和 start的区别

##### docker run

docker run只有在第一次运行时使用，将镜像放到容器中，以后再次启动这个容器的时候，只需要使用命令docker start就可以

docker run相当于执行了两步操作：将镜像（Image）放到容器（Container）中，这一步过程叫做docker create，然后将容器启动，使之变成运行时容器（docker start）

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210620195439.png" alt="在这里插入图片描述" style="zoom:80%;" />

##### docker start

docker start的作用是：重新启动已经存在的容器。也就是说，如果使用这个命令，我们必须先要知道这个容器的ID、或者这个容器的名字，我们可以使用docker ps命令找到这个容器的信息

> docker restart

```shell
docker restart container-name/container-id		#重启容器
```

### 其他命令

> docker run -d存在的问题

启动后发现立刻停止掉了

docker 容器使用后台运行 ,就必须有一个前台进程 docker发现没用应用会立刻停止

通过Ctrl + p + q退出容器不会停止容器

> docker logs container-id/container-name

```bash
docker logs 	#查看容器日志

参数含义
-t				#加入时间戳
-f				#跟随最新的日志打印
-n				#显示最后多少条

 docker logs -tf --tail 10 		#容器id  显示指定行数的日志
 docker logs -tf  				#容器id  显示所有日志
```

> docker top

```bash
docker top 				#查看容器中运行的进程信息
```

> docker inspect

```bash
docker inspect 			#查看镜像元数据
```

> docker exec -it containerId 	

```bash
docker exec -it cId		#进入正在运行的容器*(进入容器后，开启一个新的中断)

docker attach 			#容器id /bin/bash (进入正在运行的终端（当前中断）)
```

>将容器中的文件复制到主机路径下

```shell
docker cp 容器id:容器内路径   目的的主机路径
```

> 将主机路径文件复制到容器中  使用挂载	

### 常用命令小结

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210626163007.png" alt="image-20210626163004385" style="zoom:80%;" />

| **命令** |                           **英文**                           |                           **中文**                           |
| :------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|  attach  |                Attach to a running container                 |            当前 shell 下 attach 连接指定运行镜像             |
|  build   |               Build an image from a Dockerfile               |                   通过 Dockerfile 定制镜像                   |
|  commit  |         Create a new image from a container changes          |                    提交当前容器为新的镜像                    |
|    cp    | Copy files/folders from the containers filesystem to the host path |            从容器中拷贝指定文件或者目录到宿主机中            |
|  create  |                    Create a new container                    |            创建一个新的容器，同 run，但不启动容器            |
|   diff   |         Inspect changes on a container’s filesystem          |                     查看 docker 容器变化                     |
|  events  |             Get real time events from the server             |                从 docker 服务获取容器实时事件                |
|   exec   |            Run a command in an existing container            |                   在已存在的容器上运行命令                   |
|  export  |     Stream the contents of a container as a tar archive      |      导出容器的内容流作为一个 tar 归档文件[对应 import]      |
| history  |                 Show the history of an image                 |                     展示一个镜像形成历史                     |
|  images  |                         List images                          |                       列出系统当前镜像                       |
|  import  | Create a new filesystem image from the contents of a tarball |    从 tar 包中的内容创建一个新的文件系统映像[对应 export]    |
|   info   |               Display system-wide information                |                       显示系统相关信息                       |
| inspect  |         Return low-level information on a container          |                       查看容器详细信息                       |
|   kill   |                   Kill a running container                   |                    kill 指定 docker 容器                     |
|   load   |               Load an image from a tar archive               |            从一个 tar 包中加载一个镜像[对应 save]            |
|  login   |       Register or Login to the docker registry server        |               注册或者登陆一个 docker 源服务器               |
|  logout  |            Log out from a Docker registry server             |                 从当前 Docker registry 退出                  |
|   logs   |                Fetch the logs of a container                 |                     输出当前容器日志信息                     |
|   port   | Lookup the public-facing port which is NAT-ed to PRIVATE_PORT |               查看映射端口对应的容器内部源端口               |
|  pause   |            Pause all processes within a container            |                           暂停容器                           |
|    ps    |                       List containers                        |                         列出容器列表                         |
|   pull   | Pull an image or a repository from the docker registry server |         从 docker 镜像源服务器拉取指定镜像或者库镜像         |
|   push   | Push an image or a repository to the docker registry server  |           推送指定镜像或者库镜像至 docker 源服务器           |
| restart  |                 Restart a running container                  |                        重启运行的容器                        |
|    rm    |                Remove one or more containers                 |                     移除一个或者多个容器                     |
|   rmi    |                  Remove one or more images                   | 移除一个或多个镜像[无容器使用该镜像才可删除，否则需删除相关容器才可继续或 -f 强制删除] |
|   run    |               Run a command in a new container               |                创建一个新的容器并运行一个命令                |
|   save   |                Save an image to a tar archive                |             保存一个镜像为一个 tar 包[对应 load]             |
|  search  |            Search for an image on the Docker Hub             |                   在 docker hub 中搜索镜像                   |
|  start   |                  Start a stopped containers                  |                           启动容器                           |
|   stop   |                  Stop a running containers                   |                           停止容器                           |
|   tag    |                Tag an image into a repository                |                       给源中镜像打标签                       |
|   top    |         Lookup the running processes of a container          |                   查看容器中运行的进程信息                   |
| unpause  |                  Unpause a paused container                  |                         取消暂停容器                         |
| version  |             Show the docker version information              |                      查看 docker 版本号                      |
|   wait   |   Block until a container stops, then print its exit code    |                  截取容器停止时的退出状态值                  |
|          |                                                              |                                                              |

# 练习

## 部署Nginx

```shell
# 查找镜像
docker search
# 拉取镜像
docker pull nginx:1.14.1
# 后台运行镜像
docker run -d -- name nginx01 -p 1122:80 nginx:1.14.1
# 向本地端口1122发送请求
curl localhost:1122
# 查找nginx文件
whereis nginx
```

> 每次改动nginx的配置文件，都需要进入容器内部，十分麻烦，可以在容器外部提供一个映射路径，达到在容器内部修改文件——数据卷

## 部署Tomcat

```shell
# 官方的使用
docker run -it --rm tomcat:9.0.30
# 用完之后直接删掉，一般用来测试

# 运行
docker run -d -p 3344:8080 --name tomcat01 tomcat:9.0.30

# 进入镜像
docker exec -it tomcat01 /bin/bash

# 复制页面
# 因为当前的tomcat是被压缩版的，webapps下没有页面
cp -r webapps.dist/ webapps

curl localhost:3344
```

> 每次部署项目，都需要进入容器十分麻烦，可以在容器外部提供一个映射路径webapps

## 部署ES + Kibana

```shell
docker pull elasticsearch:7.13.1
# es暴露的端口很多
# es十分耗内存
# es的数据一般需要放置到安全目录！挂载
# --net somenetwork 网路配置

# 运行镜像
docker run -d --name elasticsearch --net somenetwork -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.13.1

# 查看当前内存使用情况
docker stats

# -e环境配置修改
docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx512m" elasticsearch:7.13.1
```

## portainer

Docker图形化界面管理工具，提供一个后台面板供我们操作	 

```shell
# 安装
docker run -d -p 8088:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
```

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220105170147.png" alt="image-20220105170136560" style="zoom:80%;" />

# docker镜像详解

## 什么是镜像

镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软件的所需要的所有内容，包括代码，运行时（一个程序在运行或者在被执行的依赖）、库，环境变量和配置文件

## Docker镜像加载原理

Docker实际上由一层一层的文件系统组成，这种层级的文件系统是UnionFS联合系统

>UnionFS (联合文件系统)

 Union文件系统( UnionFS)是一种分层、轻量级并且高性能的文件系统,它支持对文件系统的修改作为一次提交来层层的叠加,同时可以将不同目录挂载到同-个虚拟文件系统下(unite several directories into a single virtual filesystem)。Union 文件系统是Docker镜像的基础。镜像可以通过分层来进行继承,基于基础镜像(没有父镜像) , 可以制作各种具体的应用镜像

特性: 一次同时加载多个文件系统,但从外面看起来,只能看到一个文件系统,联合加载会把各层文件系统叠加起来,这样最终的文件系统会包含所有底层的文件和目录

 bootfs(boot file system)主要包含bootloader和kernel, bootloader主要是引导加载kernel, Linux刚启动时会加载bootfs文件系统,在Docker镜像的最底层是bootfs.这-层与我们典型的Linux/Unix系统是一样的,包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了,此时内存的使用权E Qbp fs转交给内核,此时系统也会卸载bootfs.

 rootfs (root file system) , 在bootfs之上。包含的就是典型Linux系统中的/dev, /proc, /bin, /etc等标准目录和文件。rootfs就是各种不同的操作系统发行版,比如Ubuntu , Centos等等。

![image-20220105171553562](https://gitee.com/zhang-songyao/blog-images/raw/master/20220105171556.png)

分层的好处：

最大的好处在于资源共享，如果有多个镜像都使用了相同的Base镜像，name宿主机只需要保存一份base镜像,在加载的时候，都使用这一个镜像，节省了空间，带宽

> docker镜像都是只读的，从镜像在启动了后，就会将一层新的可写层加载到镜像的顶部.
>
> 这一层被称为容器层，在这个之下的都被称为镜像层
>
> 同时我们可以将改动后的操作打包为一个新的镜像.

```shell
# 提交自己的镜像
docker commit -m="提交的描述信息" -a"作者" 容器id 目标镜像名:[TAG]
docker commit contain_id image_name:tag
```

> 修改tomcat中webapps的文件，并提交新的镜像

```shell
docker commit -m 'add webapps app' -a 'dunkcode' bb62a4670f61 tomcat01:1.0
```

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220105174332.png" alt="image-20220105174047411" style="zoom:80%;" />



# 容器数据卷

> Docker将数据和环境打包成一个镜像，如果将数据存放在容器中，那么容器删除，数据就会丢失

容器数据卷技术：Docker容器中产生的数据，同步到本地。实质是目录的挂载，将容器内的目录，挂载在Linux上

## 使用数据卷（volume）

```shell
# -v => -volume
docker run -d -it --name centos -v /home/ceshi:/home centos:latest
# 查看当前容器信息
docker inspect f82c3399ac
```

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220105181211.png" alt="image-20220105181206328" style="zoom:80%;" />

尝试在本机/home/ceshi路径添加文件，发现在容器的/home路径下会产生相同的文件

> 以后修改配置，只用在本地修改即可，不用进入容器修改	

## mysql同步数据

```shell
# 映射配置文件
-v /home/mysql/conf:/etc/mysql/conf.d
# 映射数据
-v /home/mysql/data:/var/lib/mysql

docker run -d --name mysql -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7
```

> 创建数据库会保存到本地目录下

## 具名挂载和匿名挂载

> 匿名和具名是指挂载名是否指定

- 具名挂载：-v 卷名 /容器内路径
- 匿名挂载：-v /容器内路径
- -v /宿主机路径: 容器内路径

```shell
#匿名挂载
docker run -d -P --name nginx01 -v /ext/nginx nginx
DRIVER              VOLUME NAME
local               d50fc7dd550f516f0399bc44a7a6bb36f5c8a027d150d20186d4bcb1bf97a770
#具名挂载
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx nginx
DRIVER              VOLUME NAME
local               juming-nginx



# 查看所有挂载信息
docker volume ls

docker volume inspect 卷名

# 所有匿名挂载信息路径
/var/lib/docker/volumes
```

> 拓展

```shell
ro readonly #只读 对于docker只能读取,写入只能从宿主机开始
rw readwrite#可写

docker run -d -P --name nginx02 -v juming-nginx:etc/nginx:ro nginx
docker run -d -P --name nginx02 -v juming-nginx:etc/nginx:rw nginx # 默认情况
```

## Dockerfile

> Dockerfile就是用来构建Docker镜像的构建文件

在Dockerfile文件中使用VOLUME命令可以添加一个或者多个数据卷

**举例**

Dockerfile文件如下

匿名挂载了volume01和volume02两个目录

```shell
FROM centos

VOLUME ["volume01","volume02"]

CMD echo "----end----"
CMD /bin/bash
```

构建镜像

```shell
docker build -f dockfile01 -t centos .

Sending build context to Docker daemon  2.048kB
Step 1/4 : FROM centos
 ---> 300e315adb2f
Step 2/4 : VOLUME ["volume01","volume02"]
 ---> Running in 986b3976eddf
Removing intermediate container 986b3976eddf
 ---> e5309738ebb3
Step 3/4 : CMD echo "----end----"
 ---> Running in 9d5de5eef151
Removing intermediate container 9d5de5eef151
 ---> 2d33c712bb81
Step 4/4 : CMD /bin/bash
 ---> Running in 9e04dfe71453
Removing intermediate container 9e04dfe71453
 ---> 64b62f1ff36f
Successfully built 64b62f1ff36f
Successfully tagged my/centos:1.0
```

查看镜像

```shell
docker images

REPOSITORY            TAG       IMAGE ID       CREATED          SIZE
my/centos             1.0       64b62f1ff36f   27 seconds ago   209MB
```

运行镜像

```shell
docker run -d -it --name mycentos my/centos:1.0
```

查看挂载

```shell
docker inspect cfb3382ce9c8
```

![image-20220105213854750](C:/Users/dell/AppData/Roaming/Typora/typora-user-images/image-20220105213854750.png)

## 数据卷容器

多个mysql同步数据

> 通过参数`--volums-from`，设置容器1和容器2建立数据卷挂载关系

```shell
docker run -d -p 6603:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7

docker run -d -p 6604:3306 -e MYSQL_ROOT_PASSWORD=123456 --name mysql02 --volumes-from mysql01 mysql:5.7
```

实现了两个mysql数据库的同步

> 容器之间配置信息的传递,==数据卷容器的生命周期持续到没有容器使用位置==

# Dockerfile

> Dockerfile是用来构建Docker镜像文件，命令参数脚本

构建步骤：

-  编写一个Dockerfile文件
- docker build 构建一个镜像
- docker run 运行镜像
- docker push 发布镜像（DockerHub 或者 阿里云镜像仓库

CentOS的Dockerfile

[centos](https://github.com/CentOS/sig-cloud-instance-images/blob/b2d195220e1c5b181427c3172829c23ab9cd27eb/docker/Dockerfile)

```shell
FROM scratch
ADD centos-7-x86_64-docker.tar.xz /

LABEL \
    org.label-schema.schema-version="1.0" \
    org.label-schema.name="CentOS Base Image" \
    org.label-schema.vendor="CentOS" \
    org.label-schema.license="GPLv2" \
    org.label-schema.build-date="20201113" \
    org.opencontainers.image.title="CentOS Base Image" \
    org.opencontainers.image.vendor="CentOS" \
    org.opencontainers.image.licenses="GPL-2.0-only" \
    org.opencontainers.image.created="2020-11-13 00:00:00+00:00"

CMD ["/bin/bash"]
```

## 命令

```shell
FROM                   	# 基础镜像，一切从这里开始构建    CentOS
MAINTAINER       		# 镜像是谁写的，姓名+邮箱
RUN                     # 镜像构建的时候需要运行的命令
ADD                     # 步骤，tomcat镜像，tomcat压缩包，添加内容
WORKDIR            		# 镜像的工作目录
VOLUME              	# 挂载的目录
EXPOSE                	# 保留端口配置
CMD                     # 指定这个容器启动的时候要运行的命令，只有最后一个会生效，可被替代
ENTRYPOINT       		# 指定这个容器启动的时候要运行的命令，可以追加命令
ONBULID             	# 当构建一个被继承 DockerFile 这个时候就会运行 ONBULID 指令，触发指令
COPY                    # 类似ADD，将我们文件拷贝到镜像中
ENV                     # 构建的时候设置环境变量
```



<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220105231936.png" alt="image-20220105231932901" style="zoom:80%;" />



<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220105232148.png" alt="image-20220105232145029" style="zoom:80%;" />

## 练习

### 编写一个centos

```shell
# 1、编写Dockerfile文件
[root@localhost dockerfile]# cat mydockerfile-centos 

FROM centos

MAINTAINER dunkcode

ENV MYPATH /usr/local

WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "----end----"
CMD /bin/bash

# 2、通过文件构建镜像
# 命令 docker build -f dockerfile文件路径 -t 镜像名:[tag]
docker build -f mydockerfile-centos -t mycentos:0.1 .
Successfully built 1bd5d6b9029c
Successfully tagged mycentos:0.1
# 3、测试运行
```

> docker history 查看镜像构建的过程

> CMD 和 ENTRYPOINT 区别

```shell
CMD                 # 指定这个容器启动的时候要运行的命令，只有最后一个会生效，可被替代

FROM centos
CMD ["ls","-a"]

ENTRYPOINT    # 指定这个容器启动的时候要运行的命令，可以追加命令

FROM centos
ENTRYPOINT ["ls","-a"]
```

### 编写一个Tomcat

> 准备Tomcat压缩包和JDK压缩包

> 编写Dockerfile文件

```shell
FROM centos
MAINTAINER candy<2019704946@qq.com>

COPY readme.txt /usr/local/readme.txt

ADD jdk-8u11-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-9.0.22.tar.gz /usr/local/

# RUN yum -y install vim
ENV MYPATH /usr/local
WORKDIR $MYPATH

ENV JAVA_HOME /usr/local/jdk1.8.0_11
ENV CLASSPATH $JAVA_HOME/lib/rt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.22
ENV CATALINA_BASH /usr/local/apache-tomcat-9.0.22
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin

EXPOSE 8080

CMD /usr/local/apache-tomcat-9.0.22/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.22/logs/catalina.out
```

> 构建镜像

```shell
# 官方命名==Dockerfile==，build会自动寻找这个文件，就不需要-f指定了
docker build -t diytomcat:1.0 .
```

> 启动镜像

```shell
# 启动镜像
# 挂载了两个文件
docker run -d -p 9090:8080 --name candytomcat 
-v /home/tomcat/test:/usr/local/apache-tomcat-9.0.22/webapps/test 
-v /home/tomcat/tomcatlogs/:/usr/local/apache-tomcat-9.0.22/logs diytomcat:1.0
# 由于做了挂载，直接在本地路径下编写项目即可，可以直接同步到容器中

# 进入镜像
docker exec -it 3c4ca4f0c5c70bbb087565 /bin/bash
```

> 发布镜像

DockerHub

```shell
# 登录DockerHub
docker login -u dunkcode -p ***
# 提交镜像，必须携带版本号
docker tag 镜像id dunkcode/tomcat:1.0
docker push dunkcode/tomcat:1.0
# 退出登录
docker logout
```

阿里云镜像

创建实例

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220106222107.png" alt="image-20220106222059128" style="zoom:80%;" />

创建命名空间

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220106222356.png" alt="image-20220106222350402" style="zoom:80%;" />

创建镜像仓库	

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220106222329.png" alt="image-20220106222323399" style="zoom:80%;" />

```shell
# 登录阿里云
docker login --username=******* registry.cn-beijing.aliyuncs.com

# 从Registry中拉取镜像
docker pull registry.cn-beijing.aliyuncs.com/dunkcode/dunkcode-test:[镜像版本号]

# 将镜像推送到Registry
docker login --username=******* registry.cn-beijing.aliyuncs.com
docker tag [ImageId] registry.cn-beijing.aliyuncs.com/dunkcode/dunkcode-test:[镜像版本号]
docker push registry.cn-beijing.aliyuncs.com/dunkcode/dunkcode-test:[镜像版本号]
```

## 小结

> Docker全流程	

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220106222759.png" alt="img" style="zoom:80%;" />



# Docker网络

## docker0

> 查看网卡

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220106224120.png" alt="image-20220106224115528" style="zoom:80%;" />

> 运行tomcat并查看tomcat的网络

```shell
docker run -d -p 8080:8080 --name tomcat01 tomcat:9.0.30
docker exec -it tomcat01 ip addr
```

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220106232133.png" alt="image-20220106232130043" style="zoom:80%;" />

==没启动一个docker容器，docker就会给容器分配一个ip，只要安装了docker，就会有一个docker0==

> 容器中的网卡和linux中的网卡一一对应 

==evth-pair== 就是一对的虚拟设备接口，它们是成对出现的，一端连着协议，一端彼此相连

因为这个特性，evth-pair充当一个桥梁，连接各种虚拟网络设备

OpenStac，Docker容器之间的连接，OVS的连接，都是使用 evth-pair 技术	

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220106233251.png" alt="image-20220106233250950" style="zoom:80%;" />

> 再次启动一个tomcat，ping两个容器

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220106234457.png" alt="image-20220106234454748" style="zoom:80%;" />

**结论**

所有容器共用一个路由器——docker0，所有容器不指定网络的情况下，都是docker0路由的，docker会给所有的容器分配一个默认的访问ip

Docker中的所有网络接口都是虚拟的，虚拟的转发效率高

## --link

> --link可以让两个容器相连

```shell
# 再次启动一个tomcat03
docker run -d -P --name tomcat03 --link tomcat02 tomcat:9.0.30

# 通过容器名ping
docker exec -it tomcat03 ping tomcat02
# 单向的，tomcat02 ping不通tomcat03
```

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220107154246.png" alt="image-20220107154243615" style="zoom:80%;" />

查看tomcat03中`/etc/hosts`文件发现，将tomcat02写入进去了

```shell
root@2a97a26a83fc:/etc# cat hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.3	tomcat02 9618717f0b04
172.17.0.4	2a97a26a83fc
```

> 发现docker0存在的问题：无发使容器之间通过容器名互连

## 自定义网络

`docker network ls`：查看docker网络

### 网络模式

- bridge：桥接 docker (默认)
- none：不配置网络
- host ：和宿主机共享网络
- container：容器网络连通！（用的少！局限很大）

### 自定义网络

```shell
Create a network

Options:
      --attachable           Enable manual container attachment
      --aux-address map      Auxiliary IPv4 or IPv6 addresses used by Network driver (default map[])
      --config-from string   The network from which to copy the configuration
      --config-only          Create a configuration only network
  -d, --driver string        Driver to manage the Network (default "bridge")
      --gateway strings      IPv4 or IPv6 Gateway for the master subnet
      --ingress              Create swarm routing-mesh network
      --internal             Restrict external access to the network
      --ip-range strings     Allocate container ip from a sub-range
      --ipam-driver string   IP Address Management Driver (default "default")
      --ipam-opt map         Set IPAM driver specific options (default map[])
      --ipv6                 Enable IPv6 networking
      --label list           Set metadata on a network
  -o, --opt map              Set driver specific options (default map[])
      --scope string         Control the network's scope
      --subnet strings       Subnet in CIDR format that represents a network segment

# 自定义网络
docker network create --driver bridge --subnet 192.170.0.0/16 --gateway 192.170.0.1 mynet

# 查看自定义网络
docker (network) inspect id

# 运行tomcat到自己的网络上
docker run -d -P --name tomcat-mynet-01 --net mynet tomcat:9.0.30	

```

> 自定义网络的网关

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220107182550.png" alt="image-20220107182544271" style="zoom:80%;" />

==自定义的网络维护了容器之间的关系，可以通过名称互相连接==

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220107200733.png" alt="image-20220107200730169" style="zoom:80%;" />

### 不同网络下的容器相连

```shell
docker network connect mynet tomcat01
```

查看mynet信息

```shell
docker inspect 网络id
```

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220107201451.png" alt="image-20220107201447610" style="zoom:80%;" />

进入tomcat01中查看ip

==一个容器，两个ip==

```shell
docker exec -it tomcat01 ip addr
```

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220107201558.png" alt="image-20220107201555268" style="zoom:80%;" />

发现tomcat01被加入到`192.170.0.0/16`的网段下

现在，tomcat01和mynet网络下的容器可以互相通过名称	ping通了

## 部署Redis集群

> 定义自己的网络

```shell
docker network create redis --subnet 172.38.0.0/16
```

> 通过shell命令，生成6个配置文件

```shell
for port in $(seq 1 6); \ 
do \
mkdir -p /mydata/redis/node-${port}/conf
touch /mydata/redis/node-${port}/conf/redis.conf
cat << EOF >/mydata/redis/node-${port}/conf/redis.conf
port 6379
bind 0.0.0.0
cluster-enabled yes
cLuster-config-file nodes.conf
cLuster-node-timeout 5000
cluster-announce-ip 172.38.0.1${port}
cLuster-announce-port 6379
cLuster-announce-bus-port 16379
appendonLy yes
EOF
done
```

> 通过shell命令，运行Redis容器

```shell
for port in $(seq 1 6); \
do \
docker run -p 637${port}:6379 -p 1637${port}:16379 --name redis-${port} \
-v /mydata/redis/node-${port}/data:/data \
-v /mydata/redis/node-${port}/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.1${port} redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf; \
done
```

>进入容器，创建集群

```shell
docker exec -it redis-1 sh

# 创建集群
redis-cli --cluster create 172.38.0.11:6379 172.38.0.12:6379 172.38.0.13:6379 172.38.0.14:6379 172.38.0.15:6379 172.38.0.16:6379 --cluster-replicas 1
```

> 查看集群信息

```shell
# 集群模式启动
redis-cli -c

# 查看集群信息
127.0.0.1:6379> cluster info
```

> 停掉redis-3查看是否可以查到数据

```shell
127.0.0.1:6379> set a b
-> Redirected to slot [15495] located at 172.38.0.13:6379
OK

docker stop redis-3

# 继续查找a	
127.0.0.1:6379> get a
-> Redirected to slot [15495] located at 172.38.0.14:6379
"b"
```

# java项目部署Docker

> java项目完成后，打包jar，编写Dockerfile

```shell
FROM java:8

COPY *.jar /app.jar

CMD ["--serve.port=8082"]

EXPOSE 8082

ENTRYPOINT ["java", "-jar", "/app.jar"]
```

> 将Dockerfile和jar包全部移到linux上

```shell
docker build -t project .
```

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220107211518.png" alt="image-20220107211514389" style="zoom:80%;" />

> 运行镜像

```shell
docker run -d -p 8082:8082 --name gq_sys project
```

# Docker Compose

## 概念

Docker Compose项目是Docker官方的开源项目，负责实现==对Docker容器集群的快速编排==

[项目地址](https://github.com/docker/compose/releases)

[官方文档](https://docs.docker.com/compose/)

==定义和运行多个容器的docker应用工具==。使用compose，通过yaml文件配置你自己的服务，然后通过一个命令，使用配置文件创建和运行所有的服务

## 组成

> Docker-compose将所管理的容器分为三层，分别是工程（project）、服务（service）以及容器

`Compose` 的默认管理对象是项目，通过子命令对项目中的一组容器进行便捷地生命周期管理

Docker-compose运行目录下的所有文件（docker-compose.yml，extends文件或环境变量文件等）组成一个工程，若无特殊指定工程名即为当前目录名。一个工程当中可包含多个服务，每个服务中定义了容器运行的镜像，参数，依赖。一个服务当中可包括多个容器实例

使用一个Dockerfile模板文件，可以让用户很方便的定义一个单独的应用容器。在工作中，经常会碰到需要多个容器相互配合来完成某项任务的情况。例如：要部署一个Web项目，除了Web服务容器，往往还需要再加上后端的数据库服务容器，甚至还包括负载均衡容器等

`Compose` 恰好满足了这样的需求。它允许用户通过一个单独的 `docker-compose.yml` 模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目（project）

- 服务 (service)：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。每个服务都有自己的名字、使用的镜像、挂载的数据卷、所属的网络、依赖哪些其他服务等等，即以容器为粒度，用户需要Compose所完成的任务


- 项目 (project)：由一组关联的应用容器组成的一个完整业务单元，在 docker-compose.yml 文件中定义。即是Compose的一个配置文件可以解析为一个项目，Compose通过分析指定配置文件，得出配置文件所需完成的所有容器管理与部署操作

Docker-Compose的工程配置文件默认为docker-compose.yml，可通过环境变量COMPOSE_FILE或-f参数自定义配置文件，其定义了多个有依赖关系的服务及每个服务运行的容器

## 安装

> 下载

```shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 国内镜像
curl -L https://get.daocloud.io/docker/compose/releases/download/1.29.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```

> 授权

```shell
sudo chmod +x /usr/local/bin/docker-compose
或者
sudo chmod 777 /usr/local/bin/docker-compose
```

## 测试

### Step 1: Setup

>Create a directory for the project

```shell
mkdir composetest
cd composetest
```

> Create a file called `app.py` in your project directory and paste this in

````shell
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
````

> Create another file called `requirements.txt` in your project directory and paste this in

```
flask
redis
```

### Step 2: Create a Dockerfile

>In your project directory, create a file named `Dockerfile` and paste the following

```shell
# syntax=docker/dockerfile:1
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
```

### Step 3: Define services in a Compose file

> Create a file called `docker-compose.yml` in your project directory and paste the following

```yaml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
```

> 添加挂载

```yaml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
    environment:
      FLASK_ENV: development
  redis:
    image: "redis:alpine"
```

- volumes：将主机上的项目目录挂载到`/code`容器内，允许即时修改代码，无需重建映像
- environment：设置`FLASK_ENV`，开发模式下，重新加载更改代码

### Step 4: Build and run your app with Compose

```shell
docker-compose up

# 后台运行程序
docker-compose up -d 

# 查看当前运行内容
docker-compose ps

# 停止、删除容器、网络
docker-compose down
```

> 查看镜像

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220108170154.png" alt="image-20220108170150344" style="zoom:80%;" />

> 查看网络信息

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220108170414.png" alt="image-20220108170411375" style="zoom:80%;" />

网关信息

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220108171015.png" alt="image-20220108171012140" style="zoom:80%;" />

> 使用命令访问redis的客户端

```shell
docker exec -it redis redis-cli
```

## docker-compose.yml

Docker Compose 模板文件我们需要关注的顶级配置有 `version`、`services`、`networks`、`volumes` 几个部分

- version：描述 Compose 文件的版本信息，当前最新版本为 3.8，对应的 Docker 版本为 19.03.0+；
- services：定义服务，可以多个，每个服务中定义了创建容器时所需的镜像、参数、依赖等；
- networkds：定义网络，可以多个，根据 DNS server 让相同网络中的容器可以直接通过容器名称进行通信；
- volumes：数据卷，用于实现目录挂载。

```yaml
#三层
version: "3.9" #版本
services: #服务
  web：
   images：
   bulid：
   ports:
   network：
   ...
  redis:
   ...
  mysql：
   ...
#第三层 其他配置 网络、数据卷、全局配置
networks:
  frontend:
  backend:
volumes:
  db-data:
configs：
```

## 搭建WordPress个人博客

> 创建目录

```shell
cd my_wordpress/
```

> 创建docker-compose.yml

```yaml
version: "3.9"
    
services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    
  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    volumes:
      - wordpress_data:/var/www/html
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
volumes:
  db_data: {}
  wordpress_data: {}
```

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220108173152.png" alt="image-20220108173149166" style="zoom:80%;" />

## 微服务项目部署docker

> java配置文件

```properties
server.port=8080
# 服务器docker中是使用容器名访问的
spring.redis.host=redis
```

> Dockerfile文件

```shell
FROM java:8

# 将当前的目录所有的jar文件全部复制到app.jar中
COPY *.jar /app.jar

CMD ["--serve.port=8080"]

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "/app.jar"]
```

> docker-compose.yml

```yaml
version: "3.8"
services:
  dunkcodeapp:
    build: .
    image: dunkcodeapp
    depends_on:
    - redis
    ports:
      - "8080:8080"
  redis:
    image: "redis:alpine"
```

> 重新构建

```shell
docker-compose up --build
```

# Docker Swarm

## 简介

Docker Swarm是Docker官方提供的一款集群管理工具，其主要作用是把若干台Docker主机抽象为一个整体，并且通过一个入口统一管理这些Docker主机上的各种Docker资源。Swarm和Kubernetes比较类似，但是更加轻，具有的功能也较kubernetes更少一些

[文档](https://docs.docker.com/engine/swarm/how-swarm-mode-works/nodes/)

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220108193347.png" alt="image-20220108193344623" style="zoom:80%;" />

- swarm：集群的管理和编号，Docker可以初始化一个swarm集群，其他节点可以加入（管理器、工作器）
- Node：就是Docker的一个节点，多个节点就组成了一个网络集群
- Service：任务，可以在管理节点和工作节点运行
- Task：容器内命令，细节命令



# 参考

参考博客：

[Docker 从入门到实践系列一 - 什么是Docker](https://blog.csdn.net/ThinkWon/article/details/107477065?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162475954316780269879931%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=162475954316780269879931&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-3-107477065.nonecase&utm_term=docker&spm=1018.2226.3001.4450)

[Docker 从入门到实践系列四 - Docker 容器编排利器 Docker Compose](https://blog.csdn.net/ThinkWon/article/details/119511551?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164162582416780265418407%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164162582416780265418407&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-2-119511551.nonecase&utm_term=docker&spm=1018.2226.3001.4450)	

[官网文档](https://docs.docker.com/engine/reference/commandline/images/)

参考书籍：

《Docker从入门到实践》

视频地址：

[Docker最新超详细版教程通俗易懂](https://www.bilibili.com/video/BV1og4y1q7M4?spm_id_from=333.999.0.0)

[Docker进阶篇超详细版教程通俗易懂](https://www.bilibili.com/video/BV1kv411q7Qc?p=1)

