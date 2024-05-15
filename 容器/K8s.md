> 增加一个软件平台，意味着诞生一个新的故障节点

https://labs.play-with-k8s.com/p/cdln1kler0q0008dbos0#cdln1kle_cdln1tter0q0008dbot0

Pod

k8s使用POD管理容器，每个Pod可以包含一个或者多个紧密关联的容器

Pod是一组紧密关联的容器集合，共享IPC和Network namespace，是k8s调度的基本单位，Pod内的多个容器共享网络和文件系统，可以通过进程间通信和文件共享这种简单高效的方式组合完成服务。

Node

Node是Pod真正运行的主机，可以是物理机，也可以是虚拟机。为了管理Pod，每个Node节点上至少要运行container runtime（如docker或者rkt）、kubelet、kube-proxy服务。

NameSpace

Namespace 是对一组资源和对象的抽象集合，比如可以用来将系统内部的对象划分为不同的项目组或用户组。常见的 pods, services, replication controllers 和 deployments 等都是属于某一个 namespace 的（默认是 default），而 node, persistentVolumes 等则不属于任何 namespace。



# 基础介绍

## 应用部署方式的演变

- 传统部署：互联网早期，会直接将应用程序部署在物理机上
  - 优点：简单，不需要其他技术参与
  - 缺点：不能为应用程序定义资源使用的边界，很难合理地分配计算机资源，而且程序之间容易产生影响
- 虚拟化部署：可以在一台物理机上运行多个虚拟机，每个虚拟机都是一个独立的环境
  - 优点：程序环境不会互相产生影响，提供了一定程度的安全性
  - 缺点：增加了操作系统，浪费了部分资源
- 容器化部署：与虚拟化类似，但是可以共享操作系统
  - 优点：1、保证没个容器拥有自己的文件系统、CPU、内存、进程空间等；2、运行应用程序所需要的资源都被容器包装，并和底层基础架构解耦；3、容器化应用程序可以跨云服务商、跨Linux操作系统发行版本进行部署。
  - 缺点：维护成本

![image-20200505183738289](https://gitee.com/zhang-songyao/blog-images/raw/master/202211141757205.png)

容器化部署存在的问题：

- 一个容器故障停机，怎么让另外的容器立刻启动去替补停机的容器
- 当并发访问量变大的时候，怎么做到横向扩展容器数量

> 为了解决容器编排问题，产生了一些容器编排的软件

- MESOS APACHE 分布式资源管理框架
- Docker Swarm 
- Kubernetes Google 前身：borg 特点：轻量级；消耗资源小；开源；弹性伸缩；负载均衡（IPVS）

## 简介

kubernetes本质是**一组服务器集群**，它可以在集群的每个节点上运行特定的程序，来对节点中的容器进行管理。他的目的就是实现资源管理的自动化，主要提供了如下功能

- 自我修复：一旦某一个容器崩溃，能够在1秒中左右迅速启动新的容器
- 弹性伸缩：可以根据需要，自动对集群中正在运行的容器数量进行调整
- 服务发现：服务可以通过自动发现的形式找到它所依赖的服务
- 负载均衡：如果一个服务起动了多个容器，能够自动实现请求的负载均衡
- 版本回退：如果发现新发布的程序版本有问题，可以立即回退到原来的版本
- 存储编排：可以根据容器自身的需求自动创建存储卷

## 组件

一个kubernetes集群主要由**控制节点（master）**、**工作节点（node）**构成，每个节点上都会安装不同的组件。

### master

> 集群的控制平面，负责集群的决策

- ApiServer：资源操作的唯一入口，接受用户的输入命令，提供认证、授权、API注册和发现机制
- Scheduler：负责集群资源调度，按照预定的调度策略将Pod调度到相应的node节点上
- ControllerManager：负责维护集群的状态，比如程序部署安排、故障检测、自动扩展、滚动更新等
- [Etcd](https://www.redhat.com/zh/topics/containers/what-is-etcd)：负责存储集群的各种资源信息

![image-20200406184656917](https://gitee.com/zhang-songyao/blog-images/raw/master/202211141757077.png)

### node

> 集群的数据平面，负责为容器提供运行环境

- Kubelet：负责维护容器的生命周期，即通过控制docker，来创建、更新、销毁容器
- KubeProxy：负责提供集群内部的服务发现和负载均衡
- Docker：负责节点上容器的各种操作

### eg

> 以部署一个nginx服务说明kubernetes系统各个组件调用关系

1. 一旦kubernetes环境启动之后，master和node的信息都会将自身的信息存储在etcd数据库中
2. 一个nginx服务的安装请求会先被发送到master的ApiServer组件
3. ApiServer组件会调用Scheduler组件来决定到底把这个服务安装到那个node节点上（此时，会从etcd中读取到各个node的节点信息，然后按照一定的算法进行选择，并告知ApiServer）
4. ApiServer调用Controller-manager去调度node节点安装nginx
5. kubelet接收到命令后，会通知docker，然后由docker来启动一个nginx的Pod（Pod是kubernetes的最小操作单元，容器必须跑在Pod中）
6. 到此，一个nginx服务运行成功，如果需要访问nginx，就需要通过kube-proxy来对Pod产生访问代理，这样，外界就可以访问到集群中的nginx服务

## 概念

- Master：集群的控制节点，每个集群至少有一个master节点负责集群的管控
- Node：工作负载节点，由master分配容器到这些node工作节点，node节点上的docker负责运行容器
- Pod：kubernetes的最小控制单元，容器都是运行在Pod中的，一个Pod可以有一个或者多个容器
- Controller：控制器，通过它来实现对Pod的管理，比如：启动Pod、停止Pod、伸缩Pod等（一类，有多种控制器，有不同的应用场景）
- Service：Pod对外服务的统一入口，下面可以维护多个Pod
- Label：标签，用于对Pod进行分类，同一类Pod会有相同的Label（如app:tom、app:tomcat）
- NameSpace：命名空间，用来隔离Pod的运行环境（逻辑隔离）

![image-20221113221500608](https://gitee.com/zhang-songyao/blog-images/raw/master/202211132215197.png)

# 集群环境搭建

## 环境规划

### 集群类型

大体分为两类：一主多从和多住多从

- 一主多从：一台Master节点和多台Node节点，搭建简单，但是有单机故障，适合测试环境
- 多主多从：多台Master节点和多台Node节点，搭建复杂，安全性高，适合用于生产环境

![image-20200404094800622](https://gitee.com/zhang-songyao/blog-images/raw/master/202211141758204.png)

### 安装方式

kubernetes有多种安装方式，目前主流的方式有kubeadm、minikube、二进制包

- minikube：用于快读搭建单节点的kubernetes的工具
- kubeadm：快速搭建kubernetes集群的工具
- 二进制包：从官网下载每个组件的二进制包，依次安装，此方式对于理解kubernetes组件更加有效，但是组件间的验证鉴权，不好搞

## Docker搭建

### 自定义bridge网路

```bash
docker network create --driver bridge --subnet 192.168.109.0/24 --gateway 192.168.109.1 k8s_cluster_net
```

拉取镜像并在创建的网络上运行centos 8

```bash
# 不这样启动 systemctl命令无法使用
docker run -itd -P --name centos_master1 --privileged=true --net k8s_cluster_net centos:7.9.2009 /usr/sbin/init

docker run -tid --name test  ubuntu:18.04 /sbin/init

docker exec -it  centos_master /bin/bash

# ip 设置成功
ip address
164: eth0@if165: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:c0:a8:6d:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.109.2/24 brd 192.168.109.255 scope global eth0
```

这里偷懒，不需要去hosts里面配置域名解析，当然，公司中建议用自己的DNS域名解析服务器

[centos:8 yum update报错](http://www.manongjc.com/detail/28-iexcmzlmfbmhemk.html)

### 环境检查

#### 检查系统版本、关闭防火墙

```bash
# 检查centos版本，要求版本再7.5以上
cat /etc/redhat-release
# 添加域名解析 /etc/hosts文件，添加下面内容
ip master
ip node1
ip node2
# 时间同步
yum update -y
yum install chrony -y
systemctl start chronyd
报错:Failed to get D-Bus connection: No such file or directory
原因:在centos7的docker容器里面不能用service启动服务。报这个错的原因是dbus-daemon没能启动。systemctl并不是不能使用。将CMD或者entrypoint设置为/usr/sbin/init即可。docker容器会自动将dbus等服务启动起来。

Docker的设计理念是在容器里面不运行后台服务，容器本身就是宿主机上的一个独立的主进程，也可以间接的理解为就是容器里运行服务的应用进程。一个容器的生命周期是围绕这个主进程存在的，所以正确的使用容器方法是将里面的服务运行在前台。再说到systemd，这个套件已经成为主流Linux发行版（比如CentOS7、Ubuntu14+）默认的服务管理，取代了传统的SystemV风格服务管理。systemd维护系统服务程序，它需要特权去会访问Linux内核。而容器并不是一个完整的操作系统，只有一个文件系统，而且默认启动只是普通用户这样的权限访问Linux内核，也就是没有特权，所以自然就用不了！因此，请遵守容器设计原则，一个容器里运行一个前台服务！

# 容器没有这个两个命令，不需要关闭
# 1 关闭firewalld服务
[root@master ~]# systemctl stop firewalld
[root@master ~]# systemctl disable firewalld
# 2 关闭iptables服务
[root@master ~]# systemctl stop iptables
[root@master ~]# systemctl disable iptables
```

#### 禁用selinux

selinux是linux系统下的一个安全服务，如果不关闭它，在安装集群中会产生各种各样的奇葩问题

```bash
# 编辑 /etc/selinux/config 文件，修改SELINUX的值为disable
# 注意修改完毕之后需要重启linux服务
SELINUX=disabled
```

#### 禁用swap分区

swap分区指的是虚拟内存分区，它的作用是物理内存使用完，之后将磁盘空间虚拟成内存来使用，启用swap设备会对系统的性能产生非常负面的影响，因此kubernetes要求每个节点都要禁用swap设备，但是如果因为某些原因确实不能关闭swap分区，就需要在集群安装过程中通过明确的参数进行配置说明

```bash
# 编辑分区配置文件/etc/fstab，注释掉swap分区一行
# 注意修改完毕之后需要重启linux服务
vim /etc/fstab
注释掉 /dev/mapper/centos-swap swap
# /dev/mapper/centos-swap swap
```

#### 修改linux的内核参数

```bash
# 修改linux的内核采纳数，添加网桥过滤和地址转发功能
# 编辑/etc/sysctl.d/kubernetes.conf文件，添加如下配置：
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1

# 重新加载配置
[root@master ~]# sysctl -p
# 安装br_netfilter该模块
yum install bridge-utils  -y
# 加载网桥过滤模块
[root@master ~]# modprobe br_netfilter
# 查看网桥过滤模块是否加载成功
[root@master ~]# lsmod | grep br_netfilter
```

#### 配置ipvs功能	

在Kubernetes中Service有两种带来模型，一种是基于iptables的，一种是基于ipvs的两者比较的话，ipvs的性能明显要高一些，但是如果要使用它，需要手动载入ipvs模块

```shell
# 1.安装ipset和ipvsadm
[root@master ~]# yum install ipset ipvsadm -y
# 2.添加需要加载的模块写入脚本文件
[root@master ~]# cat <<EOF> /etc/sysconfig/modules/ipvs.modules
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF
# 3.为脚本添加执行权限
[root@master ~]# chmod +x /etc/sysconfig/modules/ipvs.modules
# 4.执行脚本文件
[root@master ~]# /bin/bash /etc/sysconfig/modules/ipvs.modules
# 5.查看对应的模块是否加载成功
[root@master ~]# lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```

### 环境搭建

本次环境需要安装三台centos服务器（一主二从），然后再每台服务器中分别安装docker(18.06.3)、kubeadm(1.17.4)、kubelet(1.17.1)、kubectl(1.17.4)程序



#### docker安装

```shell
# 1、切换镜像源
[root@master ~]# wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo

# 2、查看当前镜像源中支持的docker版本
[root@master ~]# yum list docker-ce --showduplicates

# 3、安装特定版本的docker-ce
# 必须制定--setopt=obsoletes=0，否则yum会自动安装更高版本
[root@master ~]# yum install --setopt=obsoletes=0 docker-ce-18.06.3.ce-3.el7 -y

# 4、添加一个配置文件
#Docker 在默认情况下使用Vgroup Driver为cgroupfs，而Kubernetes推荐使用systemd来替代cgroupfs
[root@master ~]# mkdir /etc/docker
[root@master ~]# cat <<EOF> /etc/docker/daemon.json
{
	"exec-opts": ["native.cgroupdriver=systemd"],
	"registry-mirrors": ["https://kn0t2bca.mirror.aliyuncs.com"]
}
EOF

# 5、启动dokcer
[root@master ~]# systemctl restart docker
[root@master ~]# systemctl enable docker
```





















