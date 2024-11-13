# 第一章 Kubernetes介绍

​	本章节主要介绍应用程序再服务器上部署方式演变以及kubernetes的概念、组件和工作原理

## 1.1应用部署方式演变

再部署应用程序的方式上，主要经历了三个时代：

- 传统部署：互联网早期，会直接将应用程序部署在物理机上

  - 优点：简单，不需要其他技术的参与
  - 缺点：不能为应用程序定义资源使用边界，很难合理地分配计算机资源，而且程序之间容器产生影响

- 虚拟化部署：可以再一台物理机上运行多个虚拟机，每个虚拟机都是独立的一个环境

  - 优点：程序环境不会相互影响，提供了一定程序的安全性
  - 缺点：增加了操作系统，浪费了部分资源

- 容器化部署：与虚拟化类似，但是共享了操作系统

  - 优点：

    >可以保证每个容器用自己的文件系统、CPU、内存、进程空间等
    >
    >运行应用程序所需的资源都被容器包装、并和底层基础架构解耦
    >
    >容器化的应用程序可以跨云服务商、跨Linux操作系统发行版进行解耦


![a](kubernetes_img/a.jpg)

容器化部署方式给带来的很多的遍历，但是也会出现一些问题，比如说：

- 一个容器故障停机了，怎么样让另一个容器立刻启动去替补停机的容器
- 当并发访问量大的时候，怎么样做到横向扩展容器数量

这些容器管理的问题统称为容器编排问题，为了解决这些容器编排问题，就产生了一些容器编排的软件：

- Swarm：Docker自己的容器编排工具
- Mesos：Apache的一个资源统一管控的工具，需要和Marathon结合使用
- Kubernetes：Google开源的容器编排工具

## 1.2Kubernetes简介

​	Kubernetes，是一个全新的基于容器技术的分布式架构领先方案，**本质是一组服务器集群**，它可以再集群的每个节点上运行特定的程序，来对节点中的容器进行管理。目的是实现资源管理的自动化

实现资源管理的自动化，主要提供如下功能

- 自我修复：一旦某个容器崩溃，能够再1秒钟左右循序启动新的容器

- 弹性伸缩：可以根据需要，自动对集群中正在运行的容器数量进行调整

- 服务发现：服务可以通过自动发现的形式找到他它所依赖的的服务

- 负载均衡：如果一个服务启动了多个容器，能够自动实现请求的负载均衡

- 版本回退：如果发现新版本的程序版本有问题，可以立即回退到原来的版本

- 存储编排：可以根据容器自身的需求自动创建存储卷

## 1.3 Kubernate组件

kubernetes集群主要由控制节点（master）、工作节点（node）构成，每个节点上都会安装不同的组件。

**master：集群的控制平面，负责集群的决策**

>ApiServer：资源操作的唯一入口，接受用户输入的命令，提供认证、授权、Api注册和发现等机制
>
>Scheduler：负责集群资源调度，按照约定的调度策略将Pod调度到相应的node节点
>
>ControllerManager：负责维护集群的状态，比如程序部署安排，故障检测，自动扩展，滚动更新等
>
>Etcd：负载存储集群中各种资源对象的信息



**node：集群的数据平面，负责为容器提供运行环境**

>Kubelet：负责维护容器的生命周期，即通过控制Docker，来创建，更新，销毁容器
>
>kuberProxy：负责提供集群内部的服务发现和负载均衡
>
>Docker：负责节点上容器的各种操作

<img src="kubernetes_img/b.jpg" style="zoom: 67%;" />

下面，部署一个Nginx服务来说明kubernetes系统各个组件调用关系

1. 首先要明确，一旦kubernetes环境启动后，master和node都会将自身的信息存储到etcd数据库中
2. 一个nginx服务的安装请求会首先被发送到master节点的apiServer组件
3. apiServer组件会调用scheduler组件来决定到底应该把这个服务安装到哪个node节点上，在此时，它会从etcd中读取各个node节点信息，然后按照一定的算法进行选择，并将结果告知apiServer
4. apiServer调用controler-manager去调度node节点安装nginx服务
5. kublet接收到指令后，会通知docker，然后由dokcer来启动一个nginx的pod，pod是kubernetes的最小操作单元，容器必须跑在pod中至此
6. 一个nginx服务就运行了，如果需要访问nginx，就需要通过kube-proxy来对pod产生访问的代理，这样，外界用户就可以访问集群中的nginx服务了

## 1.4 kubernetes概念

**Master**：集群控制节点，每个集群需要至少一个master节点负责集群的管控

**Node**：工作负载节点，由master分配容器到这些node工作节点，然后node节点上的docker负责容器的运行

**Pod**：kubernetes的最小控制单元，容器都是运行在pod中的，一个pod中可以有1个或者多个容器

**Controller**：控制器，通过它来实现对pod的管理，比如启动pod、停止pod、伸缩pod的数量等

**Service**：pod对外服务的统一入口，下面可以维护着同一类的多个pod

**Lable**：标签，用于对pod进行分类，同一类pod会拥有相同的标签

**NameSpace**：命名空间，用来隔离pod的运行环境

# 第二章 集群环境搭建

本章节主要介绍如何搭建Kubernetes的集群环境

## 2.1环境规划

### 2.1.1 集群类型

kubernetes集群大体上分为两类：一主多从和多主多从

- 一主多从：一台Master节点和多台Node节点，搭建简单，但是有单机故障风险，适合用于测试环境

- 多主多从：多台Master节点和多台Node节点，搭建麻烦，安全性高，适合用于生产环境

  <img src="kubernetes_img/c.jpg" style="zoom:67%;" />

### 2.1.2 安装方式

kubernetes有多种部署方式，目前流行的方式有kubeadm、minikube、二进制包

- minikube：一个用于快速搭建单节点kubernetes的工具
- kubeadm：一个用于快速搭建kubernetes集群的工具
- 二进制包：从官网下载每个组件的二进制包，依次安装，此方式对于理解kubernetes组件更加有效

>说明：现在需要安装kubernetes的集群环境，但是又不想过于麻烦，所以选择使用kubeadm方式

### 2.1.3 主机规划

| 作用   | IP地址        | 操作系统                 | 配置                  |
| ------ | ------------- | ------------------------ | --------------------- |
| Master | 192.168.5.101 | CentOS7.5 基础设施服务器 | 2颗CPU 2G内存 50G硬盘 |
| Node1  | 192.168.5.102 | CentOS7.5 基础设施服务器 | 2颗CPU 2G内存 50G硬盘 |
| Node2  | 192.168.5.103 | CentOS7.5 基础设施服务器 | 2颗CPU 2G内存 50G硬盘 |

## 2.2 环境搭建

​	本次环境搭建需要安装三台linux系统（一主二从），内置CentOS7.5系统，然后再每台linux中分别安装docker（18.06.3），kubeadm(1.17.4)、kubectl(1.17.4)程序

### 2.2.1 主机安装

操作系统环境：CPU（2C）	内存（2G）	硬盘（50G）

语言选择：中文简体

软件选择：基础设施服务器

分区选择：自动分区

网络配置：按照下面配置网络地址信息

>网络地址：192.168.5.101（每台主机都不一样 分别为 101、102、103）
>
>子网掩码：255.255.255.0
>
>默认网关：192.168.5.2
>
>DNS：233.5.5.5

### 2.2.2 环境初始化

1. 检查操作系统的版本

   >#此方式下安装kubernetes集群要求CentOS版本要在7.5或之上

2. 主机名解析

   为了方便后面集群节点间的直接调用,在这配置一下主机名解析,企业中推荐使用**内部DNS服务器**

   >#主机名成解析,编辑三台服务器的/etc/hosts文件,添加下面内容
   >
   >192.168.5.101 master
   >
   >192.168.5.102 node1
   >
   >192.168.5.103 node2

3. 时间同步

   kubernetes要求集群中的节点时间必须精确一致,这里直接使用chronyd服务从网络同步时间.企业中建议配置内部的时间**同步服务器**

   > #启动chronyd服务
   >
   > systemctl start chronyd
   >
   > #设置chronyd服务开机自启
   >
   > systemctl enable chronyd
   >
   > #chronyd 服务启动稍等几分钟，就可以使用date命令验证时间了
   >
   > date	

 4. 禁用iptables和firewalld服务

    kubernetes和docker在运行中会产生大量的iptables规则，为了不让系统规则跟它们混淆，直接关闭系统的规则

    >1. #关闭firewalld服务
    >
    >   systemctl stop firewalld
    >
    >   systemctl disable firewalld
    >
    >2. 关闭iptables服务
    >
    >   systemctl stop iptables
    >
    >   systemctl disable iptables

 5. 禁用selinux

    selinux是linux系统下的一个安全服务，如果不关闭它，它安装集群中会产生各种各样的奇葩问题

    >#编辑 /etc/selinux/config文件，修改SELINUX的值为disabled
    >
    >#注意修改完毕之后需要重启linux服务
    >
    >SELINUX=disabled

 6. 禁用**swap分区**

    swap分区指的是虚拟内存分区，它的作用是在物理内存使用之后，将磁盘空间虚拟成内存来使用，启用swap设备会对系统的性能产生非常负面的影响，因此kubernetes要求每个节点都要禁用swap设备，但是如果因为某些原因确实不能关闭swap分区，就需要在集群安装过程中明确的参数进行配置说明

    >#编辑分区配置文件/etc/fstab,注释swap分区一行
    >
    >#注意修改完毕之后需要重启linux服务
    >
    >UUID=455cc753-7a60-4c17-a424-7741728c44a1 /boot		xfs		defaults		0	0
    >
    >/dev/mapper/centos-home	/home										xfs		defaults		0	0
    >
    >#/dev/mapper/centos-swap	swap										swap	 defaults		0	0	

 7. 修改linux的内核参数

    >#修改linux的内核参数,添加网桥过滤和地址转发功能
    >
    >#编辑/etc/sysctl.d/kubernetes.conf文件,添加如下配置:
    >
    >net.bridge.bridge-nf-call-ip6tables = 1
    >
    >net.bridge.bridge-nf-call-iptables = 1
    >
    >net.ipv4.ip_forward = 1
    >
    > 
    >
    >#重新加载配置
    >
    >sysctl -p
    >
    >#加载网桥过滤模块
    >
    >modprobe br_netfilter
    >
    > 
    >
    >#查看网桥过滤模块是否加载成功
    >
    >lsmod | grep br_netfilter

 8. 配置ipvs功能

    在kunernetes中service有两种代理模型,一种是基于iptables的,一种是基于ipvs的,两者比较的话,ipvs的性能明显要更高一些,但是如果要使用它,需要手动载入ipvs模块

    >#1. 安装ipset和ipvsadm
    >
    >yum install ipset ipvsadmin -y
    >
    >#2. 添加需要加载的模块写入脚本文件
    >
    >cat <<EOF>> /etc/sysconfig/modules/ipvs.modules
    >
    >#!/bin/bash
    >
    >modprobe -- ip_vs
    >
    >modprobe -- ip_vs_rr
    >
    >modprobe -- ip_vs_sh
    >
    >modprobe -- nf_conntrack_ipv4
    >
    >EOF
    >
    >#3. 为脚本文件添加执行权限
    >
    >chmod +x /etc/sysconfig/modules/ipvs.modules
    >
    >#4. 执行脚本文件
    >
    >/bin/bash /etc/sysconfig/modules/ipvs.modules
    >
    >#5. 查看对应的模块是否加载成功
    >
    >lsmod | grep -e ip_vs -e nf_conntrack_ipv4

 9. 重启服务器

    上面步骤完成之后,需要重新启动linux系统

    >reboot

### 2.2.3 安装docker



```powershell
1. 切换镜像源

   wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -0 /etc/yum.repos.d/docker-ce.repo

2. 查看当前镜像源中支持的docker版本

   yum list docker-ce --showduplicates

3. 安装特定版本的docker-ce

   #必须指定--setopt=obsoletes=0,否则yum会自动安装更高版本

   yum install --setopt=obsoletes=0 docker-ce-18.06.3.ce-3.el7 -y 

4. 添加一个配置文件

   #Docker 在默认情况下使用的Cgroup Driver为cgroupfs,而kubernetes推荐使用systemd来代替cgroupfs

   mkdir /etc/docker

   cat <<EOF>> /etc/docker/daemon.json

   {

   ​	"exec-opts":["native.cgroupdriver=systemd"],

   ​	"registry-mirrors":["https://kn0t2bca.mirror.aliyuncs.com"]

   }

5. 启动docker

   systemctl  restart docker

   systemctl enable docker

6. 检查docker状态和版本

   docker version
```



### 2.2.4 安装kubernetes组件

```powershell
# 由于kubernetes的镜像源在国外,速度比较慢,这里切换成国内的镜像源
# 编辑/etc/yum.repos.d/kubernetes.repo,添加下面的配置
[kubernetes]
name=Kubernets
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enable=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
	   http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg

# 安装kubeadm,kubelet和kubectl
yum install --setopt=obsoletes=0 kubeadm-1.17.4.0 kubelet-1-17.4-0 kubectl-1.17.4-0 -y 

# 配置kubelet的cgroup
# 编辑/etc/sysconfig/kubelet,添加下面的配置
KUBELET_CGOUP_ARGS="--cgroup-driver=systemd"
KUBE-PROXY_MODE="ipvs"

# 设置kubelet开机自启
systemctl enable kubelet
```

### 2.2.5 准备镜像集群

```powershell
# 在安装kubernetes集群之前,必须要提前准备好集群需要的镜像,所需镜像可以通过下面命令查看
kubeadm config images list

# 下载镜像
# 此镜像在kubernetes的仓库中,由于网络原因,无法连接,下面提供了一种替代方案
images=(
	kube-apisever:v1.17.4
	kube-controller-manager:v1.17.4
	kube-scheduler:v1.17.4
	kube-proxy:v1.17.4
	pause:3.1
	etcd:3.4.3-0
	coredns:1.6.5
)
for imageName in ${images[@]} ; do
	docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
	docker tag registry.cn-hangzhou.aliyuncs.com/googe_containers/$imageName
	k8s.gcr.io/$imageName
	docker rmi registry.cn-hangzhou.aliyumcs.com/google_containers/$imageName
done
```

### 2.2.6集群初始化

下面开始对集群进行初始化,并将node节点加入到集群中

>下面的操作只需要master节点上执行即可

```powershell
# 创建集群
[root@master ~]# kubeadm init \
	--kubenetes-version=v1.17.4 \
	--pod-network-cidr=10.244.0.0/16 \
	--service-cidr=10.96.0.0/12 \
	--apiserver-advertise-address=192.168.5.101
	
# 创建必要文件
[root@master ~]# mkdir -p $HOME/.kube
[root@master ~]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@master ~]# sudo chown $(id -u):$(id -g) $HOME/.kube/config

注意事项:
	云服务器的公网ip默认没有绑定到网卡上,绑定一个就init成功了
	如果用云服务器的建议买三台放在一个子网里面,最好不要用没有绑定到网卡上的公网ip来配置

```

>下面的操作只需要在node节点上执行即可

```powershell
# 将node节点加入集群
[root@master ~]# kebeadm join 192.168.5.101:6443 \
	--token 8507uc.o0kniri8etnw2 \
	--discovery-token-ca-cert-hash \
	sha256:acc****************************************

# 查看集群状态 此时的集群状态为NotReady,这是因为还没有配置网络插件
[root@master ~]# kubectl get nodes
NAME	STATUS		ROLES	AGE		VERSION
maste	NotReady 	master	6m43s	v1.17.4
node1	NotReady	<node>	22s		v1.17.4
node2	NotReady	<node>	19s		v1.17.4
```

### 2.2.7	安装网络插件

kubernetes支持多种网络插件,比如flannel,calico,canal等等,任选一种使用即可,本次选择flannel

>下面操作依旧只在master节点执行即可,插件使用的是DaemonSet的控制器,它会在每个节点上都运行

```powershell
# 获取fannel的配置文件
[root@master ~]# wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

#修改文件中quay.io仓库为quay-mirror.qiniu.com

#使用配置文件启动fannel
[root@master ~]# kubectl apply -f kube-flannel.yml

#稍等片刻,再次查看集群节点的状态
[root@master ~]# kebectl get nodes
NAME	STATUS		ROLES	AGE		VERSION
maste	Ready 		master	6m43s	v1.17.4
node1	Ready		<node>	8m22s	v1.17.4
node2	Ready		<node>	8m19s	v1.17.4
```

至此,kubernetes的集群环境搭建完成

## 2.3 服务部署

接下来在kubernetes集群中部署一个nginx服务,测试下集群是否在正常工作

```powershell
# 部署nginx
[root@master ~]# kubectl create deployment nginx --image=nginx:1.14-alpine

# 暴露端口
[root@master ~]# kubectl expose deployment nginx --port=80 --type=NodePort

# 查看服务状态 [svc = service]
[root@master ~]# kubectl get pods
[root@master ~]# kubectl get service
[root@master ~]# kubectl get pods,svc
NAME						READY		STATUS		RESTARTS		AGE
pod/nginx-86c57db685-fdc2k	1/1			Running		0				18m

NAME				TYPE		CLUSTER-IP		EXTERNAL-IP		PORT(S)			AGE
service/kubernetes	ClusterIP	10.96.0.1		<none>			443/TCP			82m
service/nginx		NodePort	10.104.121.45	<none>			80:30073/TCP 	17m

# 最后再电脑上访问下部署的nginx服务
```

# 第三章 资源管理

本章节主要介绍yaml语法和kubernetes的资源管理方式

## 3.1 资源管理介绍

再kubernetes中，所有的内容都抽象为资源，用户需要通过操作资源来管理kubernetes。

>​	kubernetes的本质就是一个集群系统，用户可以再集群中部署各种服务，所谓的部署服务，其实就是在kubernetes集群中运行一个个的容器，并将指定的程序跑在容器中。
>
>​	kubernetes的最小管理单元是pod而不是容器，所以只能将容器放在Pod中，而kubernetes一般也不会直接管理pod，而是通过Pod控制器来管理pod的。
>
>​	Pod可以提供服务之后，就要考虑如何访问Pod中服务了，kubernetes提供了Service资源实现这个功能。
>
>​	当然，如果Pod中程序的数据需要持久化，kebernetes还提供了各种存储系统。

![](kubernetes_img/d.jpg)

>学习kubernetes的核心，就是学习如何对集群上的Pod、Pod控制器、Service、存储等各种资源进行操作

## 3.2 YAML 语法介绍

​	YAML是一个类似XML、JSON的标记性语言。YAML强调以数据为中心，并不是以标识语言为重点。因而YAML本身的定义比较简单，号称“是一种人性化的数据格式语言”

```xml
<test>
    <sex>男</sex>
	<age>26</age>
</test>
```

```yaml
test:
  sex: 男
  age: 26
```

YAML的语法比较简单，主要有一下几个：

- 大小写敏感
- 使用缩进表示层级关系
- 缩进不允许使用tab，只允许空额
- 缩进的空格数不重要，只要相同层级的元素左对齐即可
- ‘#’表示注释

YAML支持以下几种数据类型

- 纯量：单个的，不可再分的值
- 对象：键值对的集合，又称为映射（mapping）/哈希（hash）/字典（dictionary）
- 数组：一组按次序排序的列，又称为序列（sequence）/列表（list）

```powershell
# 纯量，就是指的一个简单的值，字符串、布尔值、整数、浮点数、Null、时间、日期
# 布尔类型
c1: true（或者True）
# 整型
c2: 123
# 浮点型
c3: 3.14
# null类型
c4: ~ # 使用~表示null
# 日期类型
c5: 2023-11-14 # 日期必须使用ISO 8601格式，即yyyy-MM-dd
# 时间类型
c6: 2023-11-14T15:02:31+8:00 # 时间使用ISO 8601格式，时间和日期之间使用T连接，最后使用+代表时区
# 字符串类型
c7: hukai	# 简单写法，直接写值，如果字符串中间有特殊字符，必须使用双引号或单引号包裹
c8: line1
	line2	#字符串过多的情况下可以拆分为多行，每一行会被转成一个空格
```

```powershell
# 对象
# 形式一（推荐）
heima:
  age: 26
  sex: 男

# 形式二
heima: {age:26, adderss:Hunan}
```

```powershell
# 数组
# 形式一（推荐）
heima:
 - 顺义
 - 昌平

# 形式二
heima: [顺义,昌平]
```

>小提示：
>
>​	书写yaml切记  ：后面要加一个空格
>
>​	如果需要将多段yaml配置放在一个文件中，中间要使用---分割
>
>​	下面是一个yaml转json的网站，可以通过它验证yaml是否书写正确
>
>​	https://www.json2yaml.com/convert-yaml-to-json

## 3.3 资源管理方式

1. 命令式对象管理：直接使用命令去操作kubernetes资源

   > kubectl run nginx-pod --image=nginx:1.17.1 --port=80

2. 命令式对象配置：通过命令配置和配置文件去操作kubernetes资源

   > kubectl create/patch -f nginx-pod.yaml

3. 声明式对象配置：通过apply命令和配置文件去操作kubernetes资源

   > kubectl apply -f nginx-pod.yaml	适用于创建/更新

| 类型           | 操作对象 | 使用环境 | 优点           | 缺点                             |
| -------------- | -------- | -------- | -------------- | -------------------------------- |
| 命令式对象管理 | 对象     | 测试     | 简单           | 只能操作活动对象，无法审计、跟踪 |
| 命令式对象配置 | 文件     | 开发     | 可以审计、跟踪 | 项目大时，配置文件多，操作麻烦   |
| 声明式对象配置 | 目录     | 开发     | 支持目录操作   | 意外情况下难以调试               |

### 3.3.1 命令式对象管理

**kubectl命令**

​	kubectl是kubernetes集群的命令行工具，通过它能够对集群本身进行管理，并能够在集群上进行容器化应用的安装部署。kubectl命令的语法如下

> kubectl [command] [type] [name] [flags]

**comand**：指定要对资源执行的操作，例如create、get、delete

**type**：指定资源类型，比如deployment、pod、service

**name**：指定资源的名称，名称大小写敏感

**flags**：指定额外的可选参数

```powershell
# 查看所有pod
kubectl get pod

# 查看某个pod
kubectl get pod pod_name

# 查看某个pod，以yaml格式展示结果
kubectl get pod pod_name -o [yaml/json/wide]
```

**资源类型**

kubernetes中所有的内容都抽象为资源，可以通过下面的命令进行查看

> kubectl api-resources

经常使用的**资源**有下面这些：

| 资源分类      | 资源名称                | 缩写   | 资源作用        |
| ------------- | ----------------------- | ------ | --------------- |
| 集群级别资源  | nodes                   | no     | 集群组成部分    |
|               | namespaces              | ns     | 隔离pod         |
| pod资源       | pods                    | po     | 装载容器        |
| pod资源控制器 | replicationcontrollers  | rc     | 控制pod资源     |
|               | replicasets             | rs     | 控制pod资源     |
|               | deployments             | deploy | 控制pod资源     |
|               | daemonsets              | ds     | 控制pod资源     |
|               | jobs                    |        | 控制pod资源     |
|               | cronjobs                | cj     | 控制pod资源     |
|               | horizontalpodautocalers | hpa    | 控制pod资源     |
|               | statefulsets            | sts    | 控制pod资源     |
| 服务发现资源  | service                 | svc    | 统一pod对外接口 |
|               | ingress                 | ing    | 统一pod对外接口 |
| 存储资源      | volumeattachments       |        | 存储            |
|               | persistentvolumes       | pv     | 存储            |
|               | persistentcolumeclaims  | pvc    | 存储            |
| 配置资源      | configmaps              | cm     | 配置            |
|               | secrets                 |        | 配置            |

**操作**

kubernetes允许对资源进行多种操作，可以通过--help查看详细的操作命令

> kubeclt --help

经常使用的操作有下面这些

| 命令分类   | 命令         | 翻译     | 命令作用                     |
| ---------- | ------------ | -------- | ---------------------------- |
| 基本命令   | create       | 创建     | 创建一个资源                 |
|            | edit         | 编辑     | 编辑一个资源                 |
|            | get          | 获取     | 获取一个资源                 |
|            | patch        | 更新     | 更新一个资源                 |
|            | delete       | 删除     | 删除一个资源                 |
|            | explain      | 解释     | 展示资源文档                 |
| 运行和调式 | run          | 运行     | 在集群中运行一个指定的镜像   |
|            | expose       | 暴露     | 暴露资源为Service            |
|            | describe     | 描述     | 显示资源内部信息             |
|            | logs         | 日志     | 输出容器在pod中的日志        |
|            | attach       | 缠绕     | 进入运行中的容器             |
|            | exec         | 执行     | 执行容器中的一个命令         |
|            | cp           | 复制     | 在Pod内外复制文件            |
|            | rollout      | 首次展示 | 管理资源的发布               |
|            | scale        | 规模     | 扩（缩）容器Pod的数量        |
|            | autoscale    | 自动调整 | 自动调整Pod的数量            |
| 高级命令   | apply        | rc       | 通过文件对资源进行配置       |
|            | label        | 标签     | 更新资源上的标签             |
| 其他命令   | cluster-info | 集群信息 | 显示集群信息                 |
|            | version      | 版本     | 显示当前Server和Client的版本 |

下面以一个namespace的创建和删除简单演示下命令的作用：

```powershell
# 创建一个namespace
[root@master ~]# kubectl create namespace dev
namespace/dev created

# 获取namespace 除dev外，其它为集群自带
[root@master ~]# kubectl get ns
NAME			STATUS		AGE
default			Active		21h
dev				Active		21s
kube-node-lease	Active		21h
kube-public		Active		21h
kube-system		Active		21h

# 在此namespace下创建并运行一个nginx的Pod
[root@master ~]# kubectl run pod --image=nginx -n dev
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead. deployment.apps/pod created

# 查看新创建的pod
[root@master ~]# kubectl get pod -n dev
NAME					READY		STATUS		RESTARTS		AGE
pod-864f9875b9-pcw7x	1/1			Runing		0				21s

# 删除指定的pod
[root@master ~]# kubectl delete pod pod-864f9875b9-pcw7x
pod "pod-864f9875b9-pcw7x" deleted

# 删除指定的namespace
[root@master ~]# kubectl delete ns dev
namespace "dev" deleted
```

> 扩展：kubectl 可以在node节点上运行吗？

​	kubectl的运行是需要进行配置的，它的配置文件是$HOME/.kube，如果想要在node节点上运行此命令，需要将master上的.kube文件复制到node节点上，即在master节点上执行下面操作：

> scp -r HOME/.kube	node1:HOME/

### 3.3.2 命令式对象配置

命令式对象配置就是使用命令配合配置文件一起来操作kubernetes资源。

1）创建一个nginxpod.yaml，内容如下：

```yaml
apiVersion: v1
kind: Namespace
metadata: 
  name: dev
  
---

apiVersion: v1
kind: Pod
metadata: 
  name: nginxpod
  namespace: dev
spec:
  containers:
  - name: nginx-containers
    image: nginx:1.17.1
```

2）执行create命令，创建资源：

```shell
[root@master ~]# kubectl create -f nginxpod.yaml
namespace/dev created
pod/nginxpod created
```

此时发现创建了两个资源对象，分别是namespace和pod

3）执行get命令，查看资源：

```powershell
[root@master ~]# kubectl get -f nginxpod.yaml
NAME			STATUS		AGE
namespace/dev	Active		18s

NAME			STATUS		RESTARTS	AGE
pod/nginxpod	Running			0		17s
```

这样显示了两个资源对象的信息

4）执行delete命令，删除资源：

```powershell
[root@master ~]# kubectl delete -f nginxpod.yaml
namespace   "dev"  deleted
pod "nginxpod"  deleted
```

此时发现两个资源对象被删除了

> 总结：命令式对象配置的方式操作资源，可以简单的认为：命令 + yaml配置文件（里面是命令需要的各种参数）

### 3.3.3 声明式对象配置

声明式对象配置跟命令式对象配置很相识，但是它只有一个命令apply。

```powershell
# 首先执行一次kubectl apply -f yaml文件，发现创建了资源
[root@master ~]# kubectl apply -f nginxpod.yaml
namespace/dev created
pod/nginxpod created

# 再次执行一次kubectl apply -f nginxpod.yaml
[root@master ~]# kubectl apply -f nginxpod.yaml
namespace/dev unchanged
pod/nginxpod unchenged
```

> 总结：
>
> ​	其实声明式对象配置就是使用apply描述一个资源最终的状态（在yaml中定义状态）
>
> ​	使用apply操作资源
>
> ​			如果资源不存在，就创建，相当于kubectl create
>
> ​			如果资源已存在，就更新，相当于kubectl patch

> 扩展：kubectl 可以在node节点上运行吗？

​	kubectl的运行是需要进行配置的，它的配置文件是$HOME/.kube，如果想要在node节点运行此命令，需要将master上的.kube文件复制到node节点上，即在master节点上执行下面操作：

> scp -r HOME/.kube	node1:  HOME/

> 使用推荐：三种方式应该怎么用？
>
> 创建/更新资源		使用声明式对象配置 kubectl apply -f xxx.yaml
>
> 删除资源				 使用命令式对象配置 kubectl delete -f xxx.yaml
>
> 查询资源				 使用命令对象管理 kubectl get(describe)资源名称

# 第四章 实战入门

本章节介绍如何在kubernetes集群中部署一个nginx服务，并且能够对其进行访问。

## 4.1 Namespace

​	Namespace是kubernetes系统中的一种非常重要资源，他的主要作用是用来实现**多套环境的资源隔离**或者**多租户的资源隔离**

​	默认情况下，kubernetes集群中的所有pod都可以相互访问。但是在实际中，可能不想让两个pod之间相互的访问，那此时就可以将两个pod划分到不同的Namespace下。Kubernetes将集群内部的资源分配到不同的Namespace中，可以形成逻辑上的“组”，以方便不同组的资源隔离使用和管理。

​	可以使用kubernetes的授权机制，将不同的namespace交给不同的租户进行管理，这样就实现多租户资源隔离。此时还可以结合kubernetes的资源配额机制，限制不同租户可以占用的资源，如CPU使用量、内存使用量等，实现对租户可用资源的管理。

<img src="kubernetes_img/e.png" alt="e" style="zoom:67%;" />

kubernetes在集群启动之后，会默认创建几个namespace

```powershell
[root@master ~]# kubectl get namespace
NAME			STATUS		AGE
default			Active		21h		# 所有未指定Namespace的对象都会被分配在default命名空间
kube-node-lease	Active		21h		# 集群节点之间的心跳维护，v1.13开始引入
kube-public		Active		21h		# 此命名空间下的资源可以被所有人访问（包括未认证用户）
kube-system		Active		21h		# 所有由kubernetes系统创建的资源都处于这个命名空间
```

下面来看namespace资源的具体操作：

**查看**

```powershell
# 1 查看所有的ns 命令：kubectl get ns
[root@master ~]# kubectl get ns
NAME			STATUS			AGE
default			Active		21h		
kube-node-lease	Active		21h		
kube-public		Active		21h		
kube-system		Active		21h		

# 2 查看指定的ns	命令：kubectl get ns ns名称
[root@master ~]# kubectl get ns default
NAME			STATUS			AGE
default			Active			45h

# 3 指定输出格式 命令：kubectl get ns ns名称 -o 格式参数
# kubernetes支持的格式有很多种，比较常见的是wide【输出额外信息】、json、yaml
[root@master ~]# kubectl get ns default -o yaml
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: "2020-04-05T04:44:16Z"
  name: default
  resourceVersion: "151"
  selfLink: /api/v1/namespaces/default
  uid: 704614-5641-6504-640-40848
spec:
  finalizers:
  - kubernetes
status:
  phase: Active
  
# 4 查看ns详情 命令：kubectl describe ns ns名称
[root@master ~]# kubectl describe ns default
Name:			default
Labels:			<none>
Annotations: 	<none>
Status:			Active 	# Active命名空间正在使用中 Terminating正在删除命名空间

# ResourceQuota 针对namespace做的资源限制
# LimitRange 针对namespace中的每个组件做的资源限制
No resource quota
No LimitRange resource
```

**创建**

```powershell
# 创建namespace
[root@master ~]# kubectl create ns dev
namespace "dev" deleted
```

**删除**

```powershell
# 删除namespace
[root@master ~]# kubectl deleted ns dev
namespace "dev" deleted	
```

**配置方式**

首先准备一个yaml文件：ns-dev.yaml

```powershell
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

然后就可以执行对应的创建和删除命令了

​	创建：kubectl create -f ns-dev.yaml

​	删除：kubectl delete -f ns-dev.yaml

## 4.2 Pod

经过前面的学习已经知道，程序要运行，必须部署在容器中，而容器必须存在于Pod中。

Pod是kubernetes集群进行管理的最小单元，也是kubernetes中最为重要和基本的概念。

Pod可以认为是容器的封装，一个Pod中可以存在一个或者多个容器

![f](kubernetes_img/f.jpg)

kubernetes在集群启动之后，集群中的各个组件也都是以pod方式运行的。可以通过下面命令查看：

```powershell
[root@master ~]# kubectl get pod -n kube-system
NAMESPACE		NAME					REDAY	STATUS		RESTARTS	AGE
kube-system		coredns-69554-asdf  	1/1		Running		0			2d1h
kube-system		coredns-69554-aasf  	1/1		Running		0			2d1h
kube-system		etcd-master				1/1		Running		0			2d1h
kube-system		kube-apisever-master	1/1		Running		0			2d1h
...
```

**创建并运行**

kubernetes没有提供单独运行Pod的命令，都是通过Pod控制器来实现的

```powershell
# 命令格式：	 kubectl run (pod控制器名称)[参数]
# --image	指定pod的镜像
# --port	指定端口
# --namespace	指定namespace
[root@master ~]# kubectl run nginx --image=nginx:1.17.1 --port=80 --namespace dev
deployment.apps/nginx create
```

**查看pod信息**

```powershell
# 查看Pod基本信息
[root@master ~]# kubectl get pods -n dev
NAME			REDAY			STATUS			RESTARTS		AGE
nginx-asdf-asdf 1/1				Running			0				43s

[root@master ~]# kubectl get pods -n dev -o wide
...

# 查看Pod的详细信息
[root@master ~]# kubectl describe pod nginx-asdf-asdf -n dev
Name:		nginx-asdf-asdf
Namespace:	dev
Priority:	0
Node:		node1/192.168.5.102
Start Time: Web, 08 Apr 2020 09:29:24 + 0800
Labels:		pod-template-hash=5ff7956ff6
			run=nginx
Annotations: <none>
...
```

**访问Pod**

```powershell
# 获取nodeIp
[root@master ~]# kubectl get pods -n dev -o wide
NAME 				REDAY		STATUS		RESTART		AGE		IP				NODE ...
nginx-asdf-asdf		1/1			Running		0			190s	192.168.6.100	node1 ...

# 访问POD
[root@master ~]# curl http://192.168.5.101:80
...nginx 欢迎页面 ...
```

**删除指定pod**

```powershell
# 删除指定pod
[root@master ~]# kubectl delete pod nginx-asdf-asdf -n dev
pod "nginx-asdf-asdf" deleted

# 此时，显示删除Pod成功但是在查询，发现又新产生了一个
[root@master ~]# kubectl get pods -n dev -o wide
NAME 				REDAY		STATUS		RESTART		AGE		IP				NODE ...
nginx-asdf-qwer		1/1			Running		0			190s	192.168.6.100	node1 ...

# 这是因为当前Pod是由Pod控制器创建的，控制器会监控Pod状况，一旦发现Pod死亡，会立即重建
# 此时要想删除Pod，必须删除Pod控制器

# 先来查询一下当前namespace下的pod控制器
[root@master ~]# kubectl get deploy -n dev
NAME		REDAY		UP-TO-DATE		AVAILABLE		AGE
nignx		1/1			1				1				9m7s

# 接下来，删除此PodPod控制器
[root@master ~]# kubectl delete deploy nginx -n dev
deployment.apps "nginx" deleted

# 在查询Pod，发现Pod被删除了
[root@master ~]# kubectl get pods -n dev
No resource found in dev namespace.
```

**配置操作**

创建一个pod-nginx.yaml，内容如下：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: dev
spec:
  containers:
  - image: nginx:1.17.1
  	imagePullPolicy: IfNotPresent
  	name: pod
  	ports: 
  	- name: nginx-port
  	  containerPort: 80
  	  protocol: TCO
```

然后就可以执行对应的创建和删除命令了：

​	创建：kubectl create -f pod-nginx.yaml

​	删除：kubectl delete -f pod-nginx.yaml

## 4.3 Lable

Lable是kubernetes系统中的一个重要概念。它的作用就是在资源上添加标识，用来对它们进行区分和选择。

Lable的特点：

- 一个Label会以key/value键值对的形式附加到各种对象上，如Node、Pod、Service等等
- 一个资源对象可以定义任意数量的Lable，同一个Lable也可以被添加到任意数量的资源对象上去
- Lable通常在资源对象定义时确定，当然也可以在对象创建后动态添加和删除

可以通过Label实现资源的多维分组，以便灵活、方便地进行资源分配、调度、配置、部署等等工作。

> 一些常用的Label实例如下：
>
> - 版本标签："version":"release"，"version":"stable"......
> - 环境标签："environment":"dev"，"environment":"test"，"environment":"pro"......
> - 架构标签："tier":"frontend"，"tier":"backend"......

标签定义完毕之后，还要考虑到变迁的选择，这就要使用到Label Selector，即：	

​	Label用于给某个资源对象定义标识

​	Label Selector用于查询和筛选拥有某些标签的资源对象

当前有两种Label Selector：

- 基于等式的Label Selector

  name = slave：选择所有包含Label中key="name"且value="slave"的对象

  env != production：选择所有包括Label中的key="env"且value不等于"production"的对象

- 基于集合的Label Selector、

  name in(master,slave)：选择所有包含Label中的key="name"且value="master"或"slave"的对象

  name not in(frontend)：选择所有包含Label中的key="name"且value不等于"master的对象

标签的选择条件可以使用多个，此时将多个Label Selector进行组合，使用逗号","进行分割即可。例如：

​		name=slave，env != production

​		name not in (frontend), env!=production

**命令方式**

```powershell
# 为pod资源打标签
[root@master ~]# kubectl label pod nginx-pod version=1.0 -v dev
pod/nginx-pod labeled

# 为pod资源更新标签 【--overwrite】
[root@master ~]# kubectl label pod nginx-pod version=2.0 -v dev --overwrite
pod/nginx-pod labeled

# 查看标签
[root@master ~]# kubectl get pod nginx-pod -n dev --show-labels
NAME		REDAY		RESTARTS		AGE		LABELS
nignx-pod	1/1			Running			10m		version=2.0

# 筛选标签
[root@master ~]# kubectl get pod -n dev -l version=2.0 --show-lables
NAME		REDAY		RESTARTS		AGE		LABELS
nignx-pod	1/1			Running			10m		version=2.0

[root@master ~]# kubectl get pod -n dev -l version!=2.0 --show-lables
NAME		REDAY		RESTARTS		AGE		LABELS
No resource found in dev namespace.

# 删除标签
[root@master ~]# kubectl label pod nignx-pod version- -n dev
pod/nginx-pod labeled
```

**配置方式**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: dev
  labels:
  	version: "3.0"
  	env: "test"
spec:
  containers:
  - image: nginx:1.17.1
  	imagePullPolicy: IfNotPresent
  	name: pod
  	ports: 
  	- name: nginx-port
  	  containerPort: 80
  	  protocol: TCO
```

然后就可以执行对用的更新命令了：kubectl apply -f pod-nginx.yaml

## 4.4 Deployment

​	在kubernetes中，Pod是最小的控制单元，但是kubernetes很少直接控制Pod，一般都是通过Pod控制器来完成的，Pod控制器用于Pod的管理，确保Pod资源符合预期的状态，当pod的资源出现故障时，会尝试进行重启或中间Pod。

​	在kubernetes中Pod控制器的种类很多，本章节介绍一种：Deployment。

![g](kubernetes_img/g.jpg)

```powershell
# 命令行格式：kubectl run deployment名称 [参数]
# --image	指定pod的镜像
# --port	指定端口
# --replicas	指定创建pod数量
# --namespace	指定namespace
# kubectl run nginx --image=nginx:1.17.1	--port=80  --replicas=3 --namespace dev
[root@master ~]# kubectl run nginx --image=nginx:1.17.1	--port=80  --replicas=3 -n dev
deployment.apps/nginx created

# 查看创建的Pod
[root@master ~]# kubectl get pods -n dev
NAME				REDAY	STATUS		RESTARTS	AGE
nginx-5ffjjklj-jkj	1/1		Runing		0			19s
nginx-5ffjjklj-fas	1/1		Runing		0			19s
nginx-5ffjjklj-asd	1/1		Runing		0			19s

# 查看deployment的信息
[root@master ~]# kubectl get deploy -n dev
NAME	REDAY	UP-TODATE	AVAILABLE	AGE
nginx	3/3		3			3			2m42s

# 查看deployment和pods
[root@master ~]# kubectl get deployment,pods -n dev
NAME	REDAY	UP-TODATE	AVAILABLE	AGE
nginx	3/3		3			3			2m42s

NAME				REDAY	STATUS		RESTARTS	AGE
nginx-5ffjjklj-jkj	1/1		Runing		0			19s
nginx-5ffjjklj-fas	1/1		Runing		0			19s
nginx-5ffjjklj-asd	1/1		Runing		0			19s


# UP-TO-DATE：成功升级的副本数量
# AVAILABLE：可用副本数量
[root@master ~]# kubectl get deploy -n dev -o wide
NAME	REDAY	UP-TO-DATE	AVAILABLE	AGE		CONTANINERS	IMAGES			SELECTOR
nginx	3/3		3			3			2m51s	nginx		nginx:1.17.1	run=nginx	

# 查看deployment的详细信息
[root@master ~]# kubectl describe deploy nginx -n dev
NAME:					nginx
NAMESPACE:				dev
CREATIONTIMESTAMP		Web, 08 Apr	2020 11:14:14 +0800
Lables:					run=nginx
...

# 删除deploy
[root@master ~]# kubectl delete deploy nginx -n dev
deployment.apps "nginx" deleted
```

**配置操作**

创建一个deploy-nginx.yaml，内容如下

```yaml
apiVesion: apps/v1
kind: Deployment
metadata: 
  name: nginx
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLables:
      run: nginx
  template:
    metadata:
    lables:
      run: nginx
    spec:
      containers:
      - image: nginx:1.17.1
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP	
```

然后就可以执行对应的创建和删除命令了：

​	创建：kubectl create -f deploy-nginx.yaml

​	删除：kubectl delete -f deploy-nginx.yaml

## 4.5 Service

通过上节课的学习，已经能够利用Deployment来创建一组Pod来提供高可用性的服务

虽然每个Pod都会分配一个单独的Pod IP，然后却存在如下两个问题：

- Pod IP会随着Pod的重建产生变化
- Pod IP仅仅是集群内可见的虚拟IP，外部无法访问

这样对于访问这个服务带来了难度。因此，kubernetes设计了Service来解决这个问题。

Service可以看作是一组同类的Pod**对外的访问接口**。借助Service，应用可以方便地实现服务发现和负载均衡。

![h](kubernetes_img/h.jpg)

操作一：创建集群内部可访问的Service

```powershell
# 暴露Service
[root@master ~]# kubectl expose deploy nginx --name=svc-nginx1 --type=ClusterIP --port=80 --target-port=80 -n dev
service/svc-nginx1 exposed

# 查看Service
# kubectl get service -n dev
# kubectl get svc -n dev
[root@master ~]# kubectl get svc svc-nginx1 -n dev -o wide
NAME		TYPE		CLUSTER-IP		EXTERNAL-IP		PORT(S)		AGE		SELECTOR
svc-nginx1	ClusterIP	10.109.179.231	<none>			80/TCP		3m52s	run=nginx

# 这里产生了一个CLUSTER-IP,这就是service的IP，在Service的生命周期中，这个地址是不会变化的
# 可以通过这个IP访问当前Service对应的POD
[root@master ~]# curl 10.109.179.231:80
...
<h1>Welcom to nginx!</h1>
...
```

**操作二：创建集群外部可以访问的Service**

```powershell
# 上面创建的Service的type类型为ClusterIP，这个ip地址只有集群内部可访问
# 如果需要创建外部也可以访问的Service，需要修改type为NodePort
[root@master ~]# kubectl expose deploy nginx --name=svc-nginx2 --type=NodePort --port=80 --target-port=80 -n dev
service/svc-nginx2 exposed

# 此时查看，会发现出现了NodePort类型的Service，而且有一对Port(80:31928/TCp)
[root@master ~]# kubectl get svc svc-nginx2 -n dev -o wide
NAME		TYPE		CLUSTER-IP		EXTERNAL-IP		PORT(S)		 AGE		SELECTOR
svc-nginx1	NodePort	10.109.94.0		<none>			80:31928/TCp 9S			run=nginx

# 接下来就可以通过集群外的主机访问 节点IP:31928访问服务了
# 例如在电脑主机上通过浏览器访问下面的地址
http://192.168.5.101:31928
```

**删除Service**

```powershell
[root@master ~]# kubectl delete svc svc-nginx-1 -n dev
service "svc-nginx-1" deleted
```

**配置方式**

创建一个svc-nginx.yaml

```yaml
apiVersion: v1
kind: Service
netadata:
  name: svc-nginx
  namespace: dev
spec:
  clusterIP: 10.109.179.231
  ports:
  - port: 80
  	protocol: TCP
  	targetPort: 80
 selector:
   run: nginx
 type: ClusterIP
```

然后就可以执行对应的创建和删除命令了：

​	创建：kubectl create -f svc-nginx.yaml

​	删除：kubectl delete -f svc-nginx.yaml

> 小结：
>
> ​	至此，已经掌握了Namespace、Pod、Deployment、Service资源的基本操作，有了这些操作，就可以在kubernetes集群中实现一个服务的简单部署和访问了，但是如果想要更好的使用kubernetes，就需要深入学习这几种资源的细节和原理。

# 第五章 Pod详解

本章节将详细介绍Pod资源的各种配置（yaml）和原理。

## 5.1 Pod介绍

### 5.1.1 Pod结构

![i](kubernetes_img/i.jpg)

每个Pod中都可以包含一个或者多个容器，这些容器可以分为两类：

- 用户程序所在的容器，数量可多可少

- Pause容器，这是每个Pod都会有的一个根容器，他的作用有两个：	

  - 可以以它为依据，评估整个Pod的健康状态

  - 可以在根容器上设置Ip地址，其他容器都此Ip（Pod Ip），以实现Pod内部的网络通信

    - > 这里是Pod内部的通讯，Pod的之间的通讯采用虚拟二层的网络技术来实现的，我们当前环境的是Flannel

### 5.1.2 Pod定义

下面是Pod的资源清单

```yaml
apiVersion: v1		# 必选，版本号，例如：v1
kind: Pod			# 必选，资源类型，例如：Pod
metadata:			# 必选，元数据
  name: string		# 必选，Pod名称
  namespace: string	# Pod所属的命名空间，默认为"default"
  lables:			# 自定义标签列表
  	- name: string
spec:				# 必选，Pod中容器的详细定义
  containers: 		# 必选，Pod中容器列表
  - name: string	# 必选，容器名称
    image: string	# 必选，容器的镜像名称
    imagePullPolicy: [Always|Never|IfNotPresent]	# 获取镜像的策略
    command: [string]	# 容器的启动命令列表，如果不指定，使用打包时使用的启动命令
    args: [string]		# 容器的启动命令参数列表
    workingDir: string	# 容器的工作目录
    volumnMounts:		# 挂载到容器内部的存储卷配置
     - name: string		# 引用Pod定义的共享存储卷的名称，需用volumes[]部分定义的卷名
       mountPath: string	# 存储卷在容器内mount的绝对路径，应少于512字符
       readOnly: boolean	# 是否为只读模式
     ports: # 需要暴露的端口号列表
      - name: string		# 端口的名称
        containerPort: int	# 容器需要监听的端口号
        hostPort: int		# 容器所在主机需要监听的端口号，默认与Container相同
        protocol: string	# 端口协议，支持TCP和UDP，默认TCP
      env: 	# 容器运行前需设置的环境变量列表
       - name: string		# 环境变量名称
         value: string 		# 环境变量的值
       resources: 		# 资源限制和请求的设置
         limits: 		# 资源限制的设置
           cpu: string		# CPU的限制，单位为core数，将用于docker run --cpu-shares参数
           memory: string	# 内存限制，单位可以为Mib/Gib，将用于docker run --memory参数
         requests:		# 资源请求的设置
          cpu: string		# Cpu请求，容器启动的初始可用数量
          memory: string	# 内存请求，容器启动的初始可用数量
        lifecycle:		# 生命周期钩子
          postStart:	# 容器启动后立即执行此钩子，如果执行失败，会根据重启策略进行重启
          preStop:		# 容器终止前执行此钩子，无论结果如何，容器都会停止
        livenessProbe:  # 对Pod内各容器健康检查的设置，当前探测无响应几次后将自动重启该容器
          exec:			# 设置Pod容器内检查方式为exec方式
            command: [string]  # exec方式需要制定的命令或脚本
          httpGet:				# 对Pod每个容器健康检查方法设置为HttpGet，需要指定Path、port
            path: string 
            port: number
            host: string
            scheme: string
            HttpHeaders: 
             - name: string
               value: string
          tcpSocket:		# 对Pod某个容器健康检查方式设置为tcpSocket方式
            port: number
          initialDelaySeconds: 0	# 容器启动完成后首次探测的时间，单位为秒
          timeoutSeconds: 0			# 对容器健康检查探测等待响应的超时时间，单位秒，默认1秒
          periodSeconds: 0			# 对容器监控检查的定期探测时间设置，单位秒，默认10秒一次
          successThreshold: 0		# 连续探测成功多少次才被认定为成功。默认是1
          failureThreshold: 0		# 连续探测失败多少次才被认定为成功。默认是3。最小值是1
          securityContext:
            privileged: false
  restartPolicy: [Always | Never | OnFailure]    # Pod的重启策略
  nodeName: <string> 		# 设置NodeName表示将该Pod调度到指定到名称的node节点上
  imageSelector: object		# 设置imageSelector表示将该Pod调度到包含这个label的node上
  imagePullSecrets:			# Pull镜像时使用的secret名称，以key: securetkey格式指定
  - name: string			
  hostNetwork: false		# 是否使用主机网络模式，默认为false，如果设置为true，表示使用宿主机网络
  volumes:					# 在该Pod上定义共享存储卷列表
  - name: string			# 共享存储卷名称（volumes类型有很多种）
  	emptyDir: {}			# 类型为emptyDir的存储卷，与Pod同生命周期的一个临时目录。为空值
  	hostPath: string		# 类型为hostPath的存储卷,表示挂载Pod所在的宿主机的目录	
  	  path: string				# Pod所在宿主机的目录，将被用于同期中mount的目录
  	secret:					# 类型为secret的存储卷，挂载集群与定义的secret对象到容器内部
  	  secretname: string	
  	  items:
  	  - key: string
  	    path: string
  	configMap:				# 类型为configMap的存储卷，挂载预定义的configMap对象到容器内部
  	  name: string
  	  items:
  	  - key: string
  	    path: string
...
```

```powershell
# 小提示
# 在这里，可通过一个命令来查看每种资源的可配置项
# kubectl explain 资源类型		查看某种资源可以配置的一级属性
# kebectl explain 资源类型.属性	查看属性的子属性
[root@master ~]# kubectl explain pod
KIND:			Pod
VERSION:		v1
FIELDS:			
	apiVersion		<string>
	kind			<string>
	metadata		<object>
	spec			<object>
	status			<object>

[root@master ~]# kubectl explain pod.metadata
KIND:			Pod
VERSION:		v1
RESOURCE:		metadata	<Object>
FIELDS:			
	annotations:		<map[string]string>
	clustrName			<string>
	creationTimestamp	<string>
	deletionGracePeriodSeconds	<integer>
	deletionTimestamp	<string>
	finalizers			<[]string>
	generateName		<string>
	generation			<integer>
	labels				<map[string]string>
	managedFields		<[]Object>
	name				<string>
	namespace			<string>
	ownerReferences		<[]Object>
	resourceVersion		<string>
	selfLink			<string>
	uid					<string>
```

在kubernetes中基本所有资源的一级属性都是一样的，主要包含5部分：

- apiVersion	<string>	版本，由kubernetes内部定义，版本号必须可以用kubectl api-versions查询到
- kind <string>  	            类型，由kubernetes内部定义，版本号必须可以用kubectl api-resources查询到
- metadata  <Object>       元数据，主要是资源标识和说明，常用的有name、namespace、labels等
- spec <Object>                描述，这是配置中最重要的一部分，里面是对各种资源配置的详细描述
- status <Object>             状态信息，里面的内容不需要定义，由kubernetes自动生成

在上面的属性中，sepc是接下来研究的重点，继续看下它的常见属性：

- containers	<[]Object>	容器列表，用于定义容器的详细信息
- nodeName 	<string>  根据nodeName的值将pod调度到指定的Node节点上
- nodeSelector  <map[]>    根据NodeSelector中定义的信息选择将该Pod掉地到包含这些label的Node上
- hostNetwork  <boolean> 是否使用主机网络模式，默认为false，如果设置为true，表示使用宿主机网络
- valumes  	    <[]Object>  存储卷，用于定义Pod上面挂载的存储信息
- restartPolicy  <string>   重启策略，表示Pod在遇到故障的时候的处理策略

## 5.2  Pod配置

本小节主要来研究pod.spec.containers属性，这也是pod配置中最为关键的一项配置。

```
[root@master ~]# kubectl explain pod.spec.containers
KIND:       Pod
VERSION:    v1
RESOURCE:   containers <Object>     # 数组，代表可以有多个容器
FIELDS: 
    name    <string>        # 容器名称
    image   <string>        # 容器需要的镜像地址
    imagePullPolicy <string>    # 镜像拉去策略
    command <[]string>      # 容器的启动命令列表，如不指定，使用打包时使用的启动命令
    args    <[]string>      # 容器的启动命令需要的参数列表
    env     <[]Object>      # 容器环境变量的配置
    ports                   # 容器需要暴露的端口号列表
    resources   <[]Object>  # 资源限制和资源请求的设置
```

### 5.2.1 基本配置

创建pod-base.yaml，内容如下

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-base
  namespace: dev
  labels:
    user: heima
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
  - name: busybox
    image: busybox:1.30
```

上面定义了一个比较简单的Pod的配置，里面有两个容器：

- nginx：用1.17.1版本的nginx镜像创建，（nginx是一个轻量级web容器）
- busybox：用1.30版本的busybox镜像创建，（busybox是一个小巧的linux命令集合）

```
# 创建Pod
[root@master ~]# kubectl apply -f pod-base.yaml
pod/pod-base created

# 查看Pod状况
# READY 1/2：表示当前Pod中有2个容器，其中1个准备就绪，1个未就绪
# RESTART：重启次数，因为有1个容器故障了，Pod一直在重启视图恢复它
[root@master ~]# kubectl get pod -n dev
NAME            REDAY       STATUS      RESTARTS    AGE
pod-base        1/2         Running     4           95s

# 可以通过describe查看内部情况
# 此时已经运行起来了一个基本的Pod，虽然它暂时有问题
[root@master ~]# kubectl describe pod pod-base -n dev
```



### 5.2.2 镜像拉取

创建pod-imagepullpolocy.yaml文件，内容如下：

```
apiVersion: v1
kind: pod
metadata:
  name: pod-imagepullpolicy
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx1.17.1
    imagePullPolicy: Always # 用于设置镜像拉取策略
  - name: busybox
    image: busybox:1.30
```

imagePullPolicy，用于设置镜像拉取策略，kubernetes支持配置三种策略：

- Always：总是从远程仓库拉取镜像（一直用远程的）
- IfNotPresent：本地有则使用本地镜像，本地没有则从远程仓库拉取镜像
- Never：只使用本地镜像，从不去远程仓库拉取，本地没有就报错

> 默认值说明：
>
> ​	如果镜像tag为具体版本号，默认策略是：IfNotPresent
>
> ​	如果镜像tag为：laster（最终版本），默认策略是：Always

### 5.2.3 启动命令

​	在前面的案例中，一直有一个问题没有解决，就是busybox容器一直没有成功运行，那么到底是什么原因导致这个容器的故障呢？

​	原来busybox并不是一个程序，而是类似于一个工具类的集合，kubernetes集群启动管理后，它会自动关闭。解决方法就是让其一直在运行，这就用到了command配置。

创建pod-command.yaml文件，内容如下：

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-command
  namespace: dev
spec:
  contaners:
  - name: nginx
    image: nginx:1.17.1
  - name: busybox
    image: busybox:1.30
    command: ["/bin/bash", "-c", "touch /tmp/hello.txt;while true;do /bin/echo $(date +%T) >> /tmp/hello.txt; sleep 3; done;"]
```

command，用于在pod中的容器初始化完毕之后运行的命令。

> 稍微解释下上面命令的意思：
>
> ​	"/bin/sh","-c"，使用sh执行命令
>
> ​	touch /tmp/hello.txt；创建一个/tmp/hello.txt文件
>
> ​	while true;do /bin/echo $(date +%T) >> /tmp/hello.txt; sleep 3; done;" 每隔3秒向文件中写入当前时间

```
# 创建 Pod
[root@master ~]# kubectl create -f pod-command.yaml
pod/pod-command created

# 查看Pod状态
# 此时发现两个pod都正常运行了
[root@master ~]# kubectl get pods pod-command -n dev
NAME            REDAY       STATUS      RESTARTS    AGE
pod-command     2/2         Running     0           95s

# 进入pod中的busybox容器，查看文件内容
# 补充一个命令：kubectl exec pod名称 -n 命名空间 -it -c 容器名称 /bin/bash 在容器内部执行的命令
# 使用这个命令就可以进入某个容器的内部，然后进行相关操作了
# 比如，可以查看txt文件的内容
[root@master ~]# kubectl exex pod-command -n dev -it -c busybox /bin/bash
/ # tail -f /tmp/hello.txt
13:35:35
13:35:38
13:35:41
特别说明
    通过上面发现command已经可以完成启动命令和传递参数的功能，为什么这里还要提供一个args选项，用于传递参数呢？这其实和docker有关系，kubernetes中的command、args两项其实是实现覆盖Dockerfile中ENTRYPOINT的功能。
    1 如果command和args都没有写，那么用Dockerfile的配置
    2 如果command写了，args没写，那么Dockerfile默认的配置会被忽略，执行输入的command
    3 如果command没写，args写了，那么Dockerfile中配置的ENTRYPOINT的命令会被执行，使用当前args的参数
    4 如果都写了，那么Dockerfile的配置被忽略，执行command并追加args参数
```

### 5.2.4 环境变量

创建pod-env.yaml，内容如下：

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-env
  namespace: dev
spec:
  containers:
  - name: busybox
    image: busybox:1.30
    command: ["/bin/bash", "-c", "while true;do /bin/echo $(date +%T);sleep 60; done;"]
    env: # 设置环境变量列表
    - name: "username"
      value: "admin"
    - name: "password"
      value: "123456"
```

env，环境变量，用于在pod中的容器设置环境变量。

```
# 创建Pod
[root@master ~]# kubectl create -f pod-env.yaml
pod/pod-env created

# 进入容器，输出环境变量
[root@master ~]# kubectl exec pod-env -n dev -c busybox -it /bin/bash
/ # echo $username
admin
/ # echo $password
123456
```

这种方式不是很推荐，推荐将这些配置单独存储在配置文件中，这种方式将在后面介绍。

### 5.2.5 端口设置

本小节来介绍容器的端口暴露，也就是containers的ports选项

首先看写ports支持的子选项

```
[root@master ~]# kubectl explain pod.spec.containers.ports
KIND:       Pod
VERSION:    v1
RESOURCE:   ports <Object>
FIELDS: 
    name            <string>        # 容器名称
    containerPort   <integer>       # 容器要监听的端口（0<x<65536）
    hostPort        <integer>       # 容器要在主机上公开的端口，如果设置，主机上只能运行容器的一个副本（一般省略）
    hostIP          <string>        # 要将外部端口绑定主机IP（一般省略）
    protocol        <string>        # 端口协议。必须是UDP、TCP或SCTP。默认为"TCP"
```

接下来，创建pod-ports.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-ports
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
    ports: # 设置容器暴露的端口列表
    - name: nginx-port
      containerPort: 80
      protocol: TCP
# 创建
[root@master ~]# kubectl create -f pod-ports.yaml
pod/pod-ports created

# 查看Pod
[root@master ~]# kubectl get pod pod-ports -n dev -o yaml
...
spec:
  containers:
  - image: nginx1.17.1
    imagePullPolicy: IfNotPresent
    name: nginx
    ports:
    - containerPort: 80
      name: nginx-port
      protocol: TCP
...
```

访问容器中的程序需要使用的是`podIp:containerPort`

### 5.2.6 资源配额

​	容器中的成需要运行，肯定是要占用一定资源的，比如cpu和内存等，如果不对某个容器的资源做限制，那么它就可能吃掉大量资源，导致其它容器无法运行。针对这种情况，kubernetes提供了对内存和cpu的资源进行配额的机制，这种机制主要通过resources选项实现，他有两个子选项：

- limits：用于限制运行时容器的最大占用资源，当容器申请内存超过limits时会被终止，并进行重启
- request：用于设置容器需要的最小资源，如果资源不够，容器将无法启动

可以通过上面两个选项设置资源的上下限。

接下来，编写一个测试案例，创建pod-resources.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-resources
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx1.17.1
    resources:  # 资源配额
      limits:   # 资源配置（上限）
        cpu: "2"    # CPU限制，单位是core数
        memory: "10Gi"  # 内存限制
     requests:      # 请求限制（下限）
        cpu: "1"    # CPU限制，单位是core数
        memory: "10Mi"  # 内存限制
```

在这对cpu和memory的单位做一个说明：

- cpu：core数，可以为整数或小数
- memory：内存太小，可以使用Gi、Mi、G、M等形式

```
# 运行
[root@master ~]# kubectl create -f pod-resources.yaml
pod/pod-resources created

# 查看发现pod运行正常
[root@master ~]# kubectl get pod pod-resources -n dev
NAME            REDAY       STATUS      RESTARTS    AGE
pod-resources   1/1         Running     0           95s

# 停止Pod
[root@master ~]# kubectl delete -f pod-resources.yaml
pod/"pod-resources" deleted
```

## 5.3 生命周期

我们一般将pod对象从创建至终得这段时间范围称为Pod的生命周期，它主要包含下面的过程：

- pod创建过程
- 运行初始化容器（init container）过程
- 运行主容器（main container）过程
  - 容器启动后钩子（post start）、容器终止前钩子（pre stop）
  - 容器的存活性探测（liveness probe）、就绪性探测（readiness probe）
- pod终止过程

![5-3-01](kubernetes_img/5-3-01.jpg)

在整个生命周期中，Pod会出现5种状态（相位），分别如下：

- 挂起（Pending）apiServer已经创建了pod资源对象，但它尚未被调度完成或者仍处于下载镜像的过程中
- 运行中（Running）Pod已经被调度至某节点，并且所有容器都已经被kubelet创建完成
- 成功（Succeeded）pod中的所有容器都已经成功终止并且不会被重启
- 失败（Failed）所有容器都已经终止，但至少有一个容器终止失败，即容器返回了非0值的退出状态
- 未知（Unknown）apiSever无法正常获取到pod对象的状态信息，通常由网络通信失败所导致

### 5.3.1 创建和终止

**pod的创建过程**

1. 用户通过kubectl或其他api客户端提交需要创建的pod信息给apiServer
2. apiServer开始生成pod对象的信息，并将信息存入etcd，然后返回确认信息至客户端
3. apiServer开始反应etcd中的pod对象的变化，其它组件使用watch机制来跟踪检查apiServer上的变动
4. schduler发现有新的pod对象要创建，开始为pod分配主机，并将结果信息更新至apiServer
5. node节点上的kubelet发现有pod调度过来，尝试调用docker启动器，并将结果返回至apiServer
6. apiSever将接收到的pod状态信息存入etcd中

![5-3-1.01](kubernetes_img/5-3-1.01.jpg)

**Pod的终止过程**

1. 用户向apiServer发送删除pod对象的命令
2. apiServer中的pod对象信息会随着时间的推移而更新，在宽限期内（默认30s），pod被视为dead
3. 将pod标记为terminating状态
4. kubelet在监控到pod对象转为terminating状态的同时启动pod关闭过程
5. 端点控制器监控到pod对象的关闭行为时，将其从所有匹配到此端点的service资源的端点列表中移除
6. 如果当前pod对象定义了preStop钩子处理器，则在其标记为terminating后即会以同步的方式启动执行
7. pod对象中的容器进程收到停止信号
8. 宽限期结束后，若pod中还存在仍在运行的进程，那么pod对象会收到立即终止的信号
   1. kubelet请求apiServer将此pod资源的宽限期设置为0，从而完成删除操作，此时pod对于用户已不可见

### 5.3.2 初始化容器

初始化容器是在pod的主容器启动之前要运行的容器，主要是做一些前置工作，他们具有两种典型特征：

1. 初始化容器必须运行完成直至结束，若某初始化容器运行失败，那么kubernetes需要重启它知道成功完成
2. 初始化容器必须按照定义的顺序执行，并且仅当前一个成功之后，后面的一个才能运行

初始化容器有很多的应用场景，下面列出的是最为常见的几个：

- 提供主容器镜像中不具备的工具程序或自定义代码
- 初始化容器要先于应用容器串行启动并运行完成，因此可用于延后应用容器的启动直至其依赖的条件得到满足

接下来做一个案例，模拟下面这个需求：

​	假设要以主容器来运行nginx，但是要求在运行nginx之前要能够连接上mysql和redis所在的服务器

​	为了简化测试，实现规定好mysql (192.168.5.201)和redis（192.168.5.202）服务器的地址

创建pod-initcontainer.yaml，内容如下：

```
apiSever: v1
kind: Pod
metadata:
  name: pod-initcontainer
  namespace: dev
spec:
  containers:
  - name: main-container
    image: nginx:1.17.1
    ports:
    - name: nginx-port
      containerPort: 80
 initContainers:
 - name: test-mysql
   image: busybox:1.30
   command: ['sh', '-c', 'until ping 192.168.5.201 -c 1; do echo waiting for mysql ...; sleep 2; done;']
 - name: test-redis
   image: busybox:1.30
   command: ['sh', '-c', 'until ping 192.168.5.202 -c 1; do echo waiting for redis...;sleep 2; done;']
# 创建pod
[root@master ~]# kubectl create -f pod-initcontainer.yaml
pod/pod-initcontainer created

# 查看pod状态
# 发现pod卡在启动第一个初始化容器过程中，后面的容器不会运行
[root@master ~]# kubectl descibe pod pod-initcontainer -n dev
...
Events:
Type        Reason      Age     From        Message
...

# 动态查看pod
[root@master ~]# kubectl get pods pod-initcontainer -n dev -w
NAME                READY       STATUS      RESTARTS        AGE
pod-initcontainer   0/1         Init:0/2    0               15s
pod-initcontainer   0/1         Init:1/2    0               52s
pod-initcontainer   0/1         Init:1/2    0               53s
pod-initcontainer   0/1         PodInitiallizing:0/2    0               8s9
pod-initcontainer   0/1         Running     0               s

# 接下来新开一个shell，为当前服务器新增两个ip，观察pod的变化
[root@master ~]# ifconfig ens33:1 192.168.5.201 netmask 255.255.255.0 up
[root@master ~]# ifconfig ens33:2 192.168.5.202 netmask 255.255.255.0 up

```

### 5.3.3 钩子函数

钩子函数能够感知自身生命周期中的事件，并在相应的时刻到来时运行用户指定的程序代码。

kubernetes在主容器的启动之后和停止之前提供了两个钩子函数：

- post start：容器创建之后执行，如果失败了会重启容器
- pre stop：容器终止之前执行，执行完成之后容器将成功终止，在其完成之前会阻塞删除容器的操作

钩子处理器支持使用下面三种方式定义动作：

- Exec命令：在容器内执行一次命令

  ```
  ......
    lifecycle:
      postStart:
        exec:
          command:
          - cat
          - /tmp/healthy
  ......
  ```

- TCPSocket：在当前容器尝试访问指定的socket

  ```
  ......
      lifecycle:
        postStart:
          tcpScoket:
            port: 8080
  ......
  ```

- HTTPGet：在当前容器中向某url发起Http请求

  ```
  ......
      lifecycle:
        postStart:
          httpGet:
            path: / # URI地址
            port: 80 # 端口号
            host: 192.168.5.101 # 主机地址
            scheme: HTTP # 支持的协议，http或者https
  ......
  ```

  接下来，以exec方式为例，演示下钩子函数的使用

  创建pod-hook-exec.yaml文件，内容如下：

  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: pod-hook-exec
    namespace: dev
  spec:
   containers:
    - name: main-container
      image: nginx:1.17.1
      ports: 
      - name: nginx-port
        containerPort: 80
      lifecycle:
        postStart:
          exec: # 在容器启动的时候执行一个命令，修改掉nginx的默认首页内容
            conmmand: ['/bin/bash', '-c', "echo postStart... > /usr/share/nginx/html/index.html"]
        preStop:
          exec: # 在容器停止之前停止nginx服务
            command: ['/usr/sbin/nginx', '-s', 'quit']
  ```

  ```
  # 创建pod
  [root@master ~]# kubectl create -f pod-hook-exec.yaml
  pod/pod-hook-exec created
  
  # 查看pod
  [root@master ~]# kubectl get pods pod-hool-exec -n dev -o wide
  NAME            REDAY       STATUS      RESTARTS        AGE     IP              NODE
  pod-hook-exec   1/1         Running     0               29s     10.244.2.48     node2
  
  # 访问pod
  [root@master ~]# curl 10.244.2.48
  postStart...
  ```

### 5.3.4 容器探测

容器探测用于检测容器中的应用实例是否正常工作，是保障业务可用一种传统机制。如果经过然探测，实例的状态不符合预期，那么kubernetes就会把该问题实例"摘除"，不承担业务流量。kubernetes提供了两种探针来实现容器探测，分别是：

- liveness preobes：存活性探针，用于检测应用实例当前是否处于正常运行状态，如果不是，k8s会重启容器
- reading probes：就绪性探针，用于检测应用实例当前是否可以接受请求，如果不能，k8s不会转发流量

> livenessProbe决定是否重启容器，readingProbe决定是否将请求转发给容器
>
> 注：需要测试readingProbe和livenessProbe写法一致

上面两种探针目前均支持三种探测方式

- Exec命令：在容器内执行一次命令，如果命令执行的退出码为0，则认为程序正常，否则不正常

  ```
  ......
    livenessProbe:
      exec:
       command:
       - cat
       - /tmp/healthy
  ......
  ```

  

- TCPSocket：将会尝试访问一个用户容器的端口，如果能够建立这条连接，则认为程序正常，否则不正常

  ```
  ......
      livenessProbe:
        tcpScoket:
          port: 8080
  ......
  ```

  

- HTTPGet：调用容器内Web应用的URL，如果返回的状态码在200和399之间，则认为程序正常，否则不正常

  ```
  ......
      livenessProbe:
        httpGet:
          path: / # URI地址
          port: 80 # 端口号
          host: 192.168.5.101 # 主机地址
          scheme: HTTP # 支持的协议，http或者https
  ......
  ```

  下面以liveness probes为例，做几个演示：

  **方式一：Exec**

  创建pod-liveness-exec.yaml

  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: pod-liveness-exec
    namespace: dev
  spec:
    containers:
    - name: nginx
      image: nginx:1.17.1
      ports:
      - name: nginx-port
        containerPort: 80
      livenessProbe:
        exec:
         command: ['/bin/cat', '/tmp/hello.txt'] # 执行一个查看文件的命令
  ```

  创建Pod，观察效果

  ```powershell
  # 创建
  [root@master ~]# kubectl create -f pod-liveness-exec.yaml
  pod/pod-liveness-exec created
  
  # 查看pod详情
  [root@master ~]# kubectl get pods pod-liveness-exec -n dev
  ...
  
  # 观察上面的信息就会发现nginx容器启动之后就进行了健康检查
  # 检查失败之后，容器被kill掉，然后尝试进行重启（这是重启策略的作用，后面详解）
  # 稍等一会后，在观察pod信息，就可以看到RESTARTS不再是0，而是一直增长
  [root@master ~]# kubectl get pods pod-liveness-exec -n dev
  NAME                REDAY       STATUS              RESATRTS        AGE
  pod-liveness-exec   0/1         CrashLoopBackOff    2               3m19s
  
  # 当然接下来，可以修改成一个存在的文件，比如新建文件/tmp/hello.txt，再试，就正常了。或使用命令['/bin/ls', '/tmp/']
  ```

  **方式二：TCLSocket**

创建pod-liveness-tcpsocket.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-liveness-tcpsocket
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
  	port:
  	- name: nginx-port
  	  containerPort: 80
  	livenessProbe:
  	  tcpSocket:
  	    port: 8080 # 尝试访问8080端口
```

创建Pod，观察效果

```powershell
# 创建Pod
[root@master ~]# kubectl create -f pod-liveness-tcpsocket.yaml
pod/pod-liveness-tcpsocket  created

# 查看Pod详情
[root@master ~]# kubectl describe pods pod-liveness-tcpsocket -n dev
......
	connect: connnect refused
......

# 观察上面的信息，发现尝试访问8080端口，但是失败了
# 稍等一会之后，在观察pod的信息，就可以看到RESARTS不再是0，而是一直增长
[root@master ~]# kubectl describe pods pod-liveness-tcpsocket -n dev
NAME                	REDAY       STATUS              RESATRTS        AGE
pod-liveness-tcpsocket   0/1        CrashLoopBackOff    2               3m19s

# 当然接下来，可以修改成一个可以访问，比如80，再尝试，结果就正常了......
```

**方式三：HTTPGet**

创建pod-liveness-httpget.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-liveness-httpget
  namespace: dev
spec:
  containers:
  - name: nginx
  	image: nginx:1.17.1
 	ports:
 	- name: nginx-port
 	  containerPort: 80
 	livenessProbe:
 	  httpGet: 			# 其实就是访问scheme://host:port/path http://127.0.0.1:80/hello
 	    scheme: HTTP	# 支持的协议，http或者https
 	    port: 80		# 端口号
 	    path: /hello    # URI地址
```

创建pod，观察效果

```powershell
 # 创建Pod
[root@master ~]# kubectl create -f pod-liveness-httpget.yaml
pod/pod-liveness-httpget  created

# 查看Pod详情
[root@master ~]# kubectl describe pods pod-liveness-httpget -n dev
......
	Livenss probe faild: HTTP probe	failed with statuscode: 404
......

# 观察上面的信息，发现访问路径，但是未找到，出现404错误
# 稍等一会之后，在观察pod的信息，就可以看到RESARTS不再是0，而是一直增长
[root@master ~]# kubectl describe pods pod-liveness-httpget -n dev
NAME                	REDAY       STATUS              RESATRTS        AGE
pod-liveness-httpget    1/1         CrashLoopBackOff     2               3m19s

# 当然接下来，可以修改成一个可以访问的路径path，比如/，再尝试，结果就正常了......
```

​	至此，已经使用livenessProbe演示了三种探测方式，但是查看livenessProbe的子属性，会发现除了这三种方式，还有一些其他的配置，在这里一并解释下：

```powershell
[root@master ~]# kubectl expain pod.spec.containers.livenessProbe
FIELDS:
  exec:			<Object>
  tcpSocket:	<Object>
  httpGet:		<Object>
  initialDelaySeconds: 	<integer>	# 容器启动完成后首次探测的时间，单位为秒
  timeoutSeconds: 		<integer>	# 对容器健康检查探测等待响应的超时时间，单位秒，默认1秒
  periodSeconds: 		<integer>	# 对容器监控检查的定期探测时间设置，单位秒，默认10秒一次
  successThreshold:		<integer>	# 连续探测成功多少次才被认定为成功。默认是1
  failureThreshold: 	<integer>	# 连续探测失败多少次才被认定为成功。默认是3。最小值是1
```

下面稍微配置两个，演示下效果即可：

```powershell
[root@master ~]# more pod-liveness-httpget.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-liveness-httpget
  namespace: dev
spec:
  containers:
  - name: nginx
  	image: nginx:1.17.1
 	ports:
 	- name: nginx-port
 	  containerPort: 80
 	livenessProbe:
 	  httpGet: 			# 其实就是访问scheme://host:port/path http://127.0.0.1:80/hello
 	    scheme: HTTP	# 支持的协议，http或者https
 	    port: 80		# 端口号
 	    path: /hello    # URI地址
      initailDelaySeconds: 30 # 容器启动后30s开始探测
      timeoutSeconds: 5 #探测超时时间为5s
```

### 5.3.5 重启策略

​	在上一节中，一旦容器探测出现了问题，kubernetes就会对容器所在的Pod进行重启，其实这是由pod的重启策略决定的，pod的重启策略有3中，分别如下：

- Always：容器失效时，自动重启该容器，这也是默认值。
- OnFailure：容器终止运行且退出码不为0时重启
- Never：不论状态为何，都不重启该容器

​        重启策略适用于pod对象中的所有容器，首次需要重启的容器，将在其需要时立即进行重启，随后再次需要重启的操作将由kubelet延迟一段时间后进行，且反复的重启操作的延长以此为10s，20s，40s，80s，160s和300s，300秒是最大延迟时长。

创建pod-restartpolicy.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-liveness-httpget
  namespace: dev
spec:
  containers:
  - name: nginx
  	image: nginx:1.17.1
 	ports:
 	- name: nginx-port
 	  containerPort: 80
 	livenessProbe:
 	  httpGet: 			# 其实就是访问scheme://host:port/path http://127.0.0.1:80/hello
 	    scheme: HTTP	# 支持的协议，http或者https
 	    port: 80		# 端口号
 	    path: /hello    # URI地址
  restartPolicy: Never	# 设置重启策略为 Never
```

运行Pod测试

```powershell
[root@master ~]# kubectl create -f pod-restartpolicy.yaml
pod/pod-restartpolicy created

# 查看pod详情，发现nginx容器失败
[root@master ~]# kubectl describe pods pod-restartpolicy -n dev
......
	Stopping container nginx
......
```

## 5.4 Pod调度

​	在默认情况下，一个Pod在哪各Node节点上运行，是由Scheduler组件采用相应的算法计算出来的，这个过程是不受人工控制的。但是实际使用中，这并不满足的需求，因为很多情况下，我们想控制某些Pod到达某些节点上，那么应该怎么做呢？这就要求了解kubenetes对Pod的调度规则，kubernetes提供了四大类调度方式：

- 自动调度：运行在哪个节点上完全由Scheduler经过一系列的算法计算得出
- 定向调度：NodeName、NodeSelector
- 亲和度调度：NodeAffinity、PodAffinity、PodAntiffinity
- 污点（容忍）调度：Tains、Toleration

### 5.4.1 定向调度

​	定向调度，指的是利用在pod上声明nodeName或者nodeSelector，以此将Pod调度到期望的node节点上。注意，这里的调度是强制的，这就意味着即使要调度的目标Node不存在，也会向上面进行调度，只不过pod运行失败而已。

**1）NodeName**

​	NodeName用于强制约束将Pod调度到指定的Name的Node节点上。这种方式，其实是直接跳过Scheduler的调度逻辑，直接写入PodList列表。

接下来，试验一下：创建一个pod-nodename.yaml文件

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nodename.yaml
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
   nodeName: node1 		# 指定调度到node1节点上
```

```powershell
# 创建Pod
[root@master ~]# kubectl create -f pod-nodename.yaml
pod/pod-nodename created

# 查看pod调度到Node属性，确实是调度到了node1节点上
[root@master ~]# kubectl get pods pod-nodename -n dev -o wide
NAME           REDAY       STATUS     RESATRTS        AGE		IP				NODE		...
pod-nodename   1/1         Running    0               3m19s		10.244.1.87		node1 		...

# 接下来，删除pod，修改nodeName的值为node3(并没有node3节点)
[root@master ~]# kubectl delete -f pod-nodename.yaml
pod/"pod-nodename" deleted
[root@master ~]# vim pod-nodename.yaml
[root@master ~]# kubectl create -f pod-nodename.yaml
pod/pod-nodename created

# 在次查看，发现已经向Node3节点调度，但是由于不存在node3节点，所以pod无法正常运行
[root@master ~]# kubectl get pods pod-nodename -n dev -o wide
NAME           REDAY       STATUS     RESATRTS        AGE		IP				NODE		...
pod-nodename   0/1         Pending    0               6s		<none>   		node3 		...

```

**2）NodeSelector**

​	NodeSelector用于将Pod调度到添加了指定标签的node节点上，它是通过kubernetes的label-selector机制实现的，也就是说，在pod创建之前，会由scheduler使用MatchNodeSelector调度进行label匹配，找出目标node，然后将pod调度到目标节点，该匹配规则是强制约束。

接下来，试验一下：

1）首先分别为node节点添加标签

```powershell
[root@master ~]# kubectl label nodes node1 nodeenv=pro
node/node1 labeled
[root@master ~]# kubectl label nodes node2 nodeenv=pro
node/node2 labeled

```

2）创建一个pod_nodeselector.yaml文件，并使用它创建Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nodeselector
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
  nodeSelector:
    nodeenv: pro # 指定调度到具有nodeenv=pro标签的节点上
```

```powershell
# 创建Pod
[root@master ~]# kubectl create -f pod-nodeselector.yaml
pod/pod-nodeselector created

# 查看pod调度到Node属性，确实是调度到了node1节点上
[root@master ~]# kubectl get pods pod-nodeselector -n dev -o wide
NAME           		REDAY       STATUS     RESATRTS        AGE			IP				NODE		...
pod-nodeselector    1/1         Running    0               19s			10.244.1.87		node1 		...

# 接下来，删除pod，修改nodeSelector的值为nodeenv:xxx（不存在打有此标签的节点）
[root@master ~]# kubectl delete -f pod-nodeselector.yaml
pod/"pod-nodeselector" deleted
[root@master ~]# vim pod-nodeselector.yaml
[root@master ~]# kubectl create -f pod-nodeselector.yaml
pod/pod-nodeselector created

# 在次查看，发现pod无法正常运行，Node的值为none
[root@master ~]# kubectl get pods pod-nodename -n dev -o wide
NAME           REDAY       STATUS     RESATRTS        AGE		IP				NODE		...
pod-nodename   0/1         Pending    0               3m20s		<none>   		<none> 		...

# 查看详情，发现node selector匹配失败的提示
[root@master ~]# kubectl describe pods pod-nodeselector -n dev
......
	didn't match node selector.
......
```

### 5.4.2 亲和性调度

​	上一节，介绍了两种定向调度的方式，使用起来非常方便，但是也有一定的问题，那就是如果没有满足的条件的Node，那么Pod将不会被运行，即使在集群中还有可用的Node列表也不行，这就限制了它的使用场景。

​	基于上面的问题，kubernetes还提供了一种亲和性调度（NodeAffinity）。它在NodeSelector的基础之上的进行了扩展，可以通过配置的形式，实现优先选择满足条件的Node进行调度，如果没有，也可以调度到不满足条件的节点上，使调度更加灵活。

Affinity主要分为三类：

- nodeAffinity（node亲和性）：以node为目标，解决pod可以调度到哪些node的问题
- podAffinity（pod亲和性）：以pod为目标，解决pod可以和哪些已存在的pod部署在同一个拓扑域中的问题
- podAntiAnffinity（pod反亲和性）：以pod为目标，解决pod不能和哪些已存在的pod部署在同一个拓扑域中的问题

> 关于亲和性（反亲和性）使用场景说明：
>
> **亲和性**：如果两个应用频繁交互，那就有必要利用亲和性让两个应用尽可能的靠近，这样可以减少因网络通信而带来的性能损耗
>
> **反亲和性**：当应用的采用多副本部署时，有必要采用反亲和性让各个应用实例打散分布在各个node上，这样可以提高服务的高可用性

**1）NodeAffinity**

首先来看一下*NodeAffinity*的可配置项：

```markdown
pod.spec.affinity.nodeAffinity
	requireDuringSchedulingIgnoreDuringExecution 	# Node节点必须满足指定的所有规划才可以，相当于硬限制
	  nodeSelectorTerms		# 节点选择列表
	    matchFields				# 按节点字段列出的节点选择器要求列表
	    matchExpressions		# 按节点标签列出的节点选择器要求列表（推荐）
	      key						# 键
	      values					# 值
	      operator					# 关系符 支持 In、NotIn、Exists、DoesNotExist、Gt、Lt
	preferredDuringSchedulingIgnoreDuringExecution		#优先调度到满足指定的规则的Node，相当于软限制（倾向）
	  preference		# 一个节点选择器项，与相应的权重相关联
	    matchFields			# 按节点字段列出的节点选择器要求列表
	    matchExpressions	# 按节点标签列出的节点选择器要求列表（推荐）
	      key						# 键
	      values					# 值
	      operator					# 关系符 支持 In、NotIn、Exists、DoesNotExist、Gt、Lt
	  weight			# 倾向权重，在范围1-100
```

```markdown
关系符的使用说明：
- matchExpressions				
  - key：nodeenv				# 匹配存在标签的key为nodeenv的节点
    operator: Exists		# 匹配标签的key为nodeenv,且value是"xxx"和"yyy"的节点
  - key: nodeenv
    operator: In			
    values: ["xxx", "yyy"]
  - key: nodeenv
    operator: Gt			# 匹配标签的key为nodeenv,且value大于"xxx"
    values: "xxx"
```

接下来首先演示一下 **requiredDuringSchedulingIgnoredDuringExecution**

创建pod-nodeaffinity-required.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nodeaffinity-required
  namespace: dev
spec:
  containers:
  - name: nginx
  	image: nginx:1.17.1
  affinity:			# 亲和性设置
    nodeAffinity:	# 设置node亲和性
      requiredDuringSchedulingIgnoredDuringExecution:	# 硬限制
        nodeSelectorTerms:
        - matchExpressions:		# 匹配env的值在["xxx", "yyy"]中的标签
          - key: nodeenv
            operator: In
            values: ["xxx", "yyy"]  
```

```powershell
# 创建pod
[root@master ~]# kubectl create -f pod-nodeaffinity-required.yaml
pod/pod-nodeaffinity-required

# 查看pod状态(运行失败)
[root@master ~]# kubectl get pods pod-nodeaffinity-required -n dev-o wide
NAME						REDAY		STATUS		RESTARTS		AGE		IP		NODE		...
pod-nodeaffinity-required	0/1			Pending		0				16s		<none>	<none>		...

# 查看pod详情 
# 发现调度失败，提示node选择失败
[root@maser ~]# kubectl describe pod pod-nodeaffinity-required -n dev
......
	Warning FailedScheduling <unknow> default-schduler 0/3 nodes are avaliable: 3 node(s) did't matcha node selector
......

# 接下来，停止pod
[root@maser ~]# kubectl delete -f pod-podaffinity-required.yaml
pod "pod-podaffinity-required" deleted	

# 修改文件，将values：["xxx", "yyy"] -----> ["pro", "yyy"]
[root@maser ~]# vim pod-podaffinity-required.yaml

# 再次启动
[root@master ~]# kubectl create -f pod-nodeaffinity-required.yaml
pod/pod-nodeaffinity-required

# 此时查看，发现调度成功，已经将pod调度到了node1上
[root@master ~]# kubectl get pods pod-nodeaffinity-required -n dev -p wide
NAME						REDAY		STATUS		RESTARTS		AGE		IP			NODE		...
pod-nodeaffinity-required	1/1			Running		0				11s		10.244.1.89	node1		...

```

接下来演示一下：**preferredDuringSchedulingIgnoreDuringExecution**

创建pod-nodeaffinity-preferred.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nodeaffinity-preferred
  namespace: dev
spec:
  containers:
  - name: nginx
  	image: nginx:1.17.1
  affinity:			# 亲和性设置
    nodeAffinity:	# 设置node亲和性
      preferredDuringSchedulingIgnoreDuringExecution:	# 软限制
      - weight: 1
      	preference:
          matchExpressions:		# 匹配env的值在["xxx", "yyy"]中的标签（当前环境没有）
          - key: nodeenv
            operator: In
            values: ["xxx", "yyy"]  
```

```powershell
# 创建pod
[root@master ~]# kubectl create -f pod-nodeaffinity-preferred.yaml
pod/pod-nodeaffinity-preferred

# 查看pod(运行失败)
# 修改yaml的nodeenv为环境中已存在的
# 运行成功
[root@maser ~]# kubectl get pod pod-nodeaffinity-preferred -n dev
NAME						REDAY		STATUS		RESTARTS		AGE		
pod-nodeaffinity-preferred	1/1			Running		0				11s		
```

```markdown
NodeAffinity规则设置的注意事项:
   1）如果同时定义了nodeSelector和nodeAffinity，那么必须两个条件都得到满足，Pod才能运行在指定的Node上
   2）如果NodeAffinity指定了多个nodeSelectorTerms，那么只需要其中一个能够匹配成功即可
   3）如果一个nodeSelectorTerms中有多个matchExpressions，则一个节点必须满足所有的才能匹配成功
   4）如果一个pod所在的Node，在Pod运行期间其标签发生了改变，不在符合该Pod的节点亲和性需求，则系统将忽略此变化
```

**2）PodAffinity**

PodAffinity主要实现一个运行的Pod为参照物，让新创建的Pod跟参照pod在一个拓扑域中的功能。

首先来看一下PodAffinity的可配置项：

```markdown
pod.spec.affinity.podAffinity
  requiredDuringSchedulingIgnoreDuringExecution		# Node节点必须满足指定的所有规划才可以，相当于硬限制
    namespaces		指定参照pod的namespace
	topologyKey		指定调度作用域
	labelSelector	标签选择器
	  matchExpressions	按节点标签列出的节点选择器要求列表（推荐）
	    key					键
	    values				值
	    operator			关系符 支持 In、NotIn、Exists、DoesNotExists
	  matchLabels		指多个matchExpressions映射的内容
  preferredDuringSchedulingIgnoredDuringExecution	优先调度到满足指定的规则的Node，相当于软限制
    podAffinityTerm	选项
      namespace
      topologyKey
      labelSelectr
        matchExpressions
          key	
          values
          operator
        matchLabels
    wright	倾向权重，在范围1-100
```

```
topologyKey用于指定调度时作用域，例如：
  如果指定为kubernetes.io/hostname，那就是以Node节点为区分范围
  如果指定为beta.kubernetes.io/os，则以Node节点的操作系统类型来区分
```

接下来，演示下requiredDuringSchedulingIgnoredDuringExecution

**2-1）首先创建一个参照Pod，pod-podaffinity-target.yaml:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-podaffinity-target
  namespace: dev
  labels:
    podenv: pro		# 设置标签
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
  nodeName: node1		# 将目标pod明确指定到node1上
```

```powershell
# 启动目标pod
[root@master ~]# kubectl create -f pod-podaffinity-target.yaml
pod/pod-podaffinity-target created

# 查看pod状况
[root@master ~]# kubectl get pods pod-podaffinity-target -n dev
NAME						REDAY		STATUS		RESTARTS		AGE
pod-podaffinity-target		1/1			Running		0				4s
```

**2-2）创建pod-podaffinity-required.yaml，内容如下：**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-podaffinity-required
  namespace: dev
spec:
  containers:
  - name: nginx
  	image: nginx:1.17.1
 affinity:		# 亲和性设置
   podAffinity:		# 设置pod亲和性
     requiredDuringSchedulingIgnoreDuringExecution：		# 硬限制
     - labelSelector:
         matchExpressions:		# 匹配env的值在["xxx", "yyy"]中的标签
         - key: podenv
           operator: In
           values: ["xxx", "yyy"]
       topologyKey: kubernetes.io/hostname
     
```

​	上面配置表达的意思是：新Pod必须要与拥有标签nodeenv=xxx或nodeenv=yyy的Pod在同一个Node上，显然现在没有这样的pod，接下来，运行测试一下。

```powershell
# 创建pod
[root@master ~]# kubectl create -f pod-podaffinity-required.yaml
pod/pod-podaffinity-required

# 查看pod状态(运行失败)
[root@master ~]# kubectl get pods pod-podaffinity-required -n dev-o wide
NAME						REDAY		STATUS		RESTARTS		AGE		IP		NODE		...
pod-podaffinity-required	0/1			Pending		0				16s		<none>	<none>		...

# 查看pod详情 
# 发现调度失败，提示node选择失败
[root@maser ~]# kubectl describe pod pod-podaffinity-required -n dev
......
	Warning FailedScheduling <unknow> default-schduler 0/3 nodes are avaliable: 2 node(s) did't matcha pod affinity rules，1 node(s) had taints the pod didn't tolerate
......

# 接下来，停止pod
[root@maser ~]# kubectl delete -f pod-podaffinity-required.yaml
pod "pod-podaffinity-required" deleted	

# 修改文件，将values：["xxx", "yyy"] -----> ["pro", "yyy"]
[root@maser ~]# vim pod-podaffinity-required.yaml

# 再次启动
[root@master ~]# kubectl create -f pod-podaffinity-required.yaml
pod/pod-podaffinity-required

# 此时查看，发现调度成功，已经将pod调度到了node1上
[root@master ~]# kubectl get pods pod-podaffinity-required -n dev -p wide
NAME						REDAY		STATUS		RESTARTS		AGE		IP			NODE		...
pod-podaffinity-required	1/1			Running		0				11s		10.244.1.89	node1		...

```

关于PodAffinity的preferredDuringSchdulingIgnoredDuringExecution，这里不在演示。

**3）PodAntAffinity**

PodAntAffinity主要实现以一个运行的pod为参照，让新建的Pod跟参照pod不在一个拓扑域中的功能

它的配置方式和选项跟PodAffinity是一样的，这里不在做详细解释，直接做一个测试案例。

3-1）继续使用上个案例中目标pod

创建pod-podandaffinity-required.yaml，内容如下：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-podaffinity-required
  namespace: dev
spec:
  containers:
  - name: nginx
  	image: nginx:1.17.1
 affinity:		# 亲和性设置
   podAntAffinity:		# 设置pod反亲和性
     requiredDuringSchedulingIgnoreDuringExecution：		# 硬限制
     - labelSelector:
         matchExpressions:		# 匹配env的值在["xxx", "yyy"]中的标签
         - key: podenv
           operator: In
           values: ["xxx", "yyy"]
       topologyKey: kubernetes.io/hostname
```

上面表达的意思是，新pod必须要与拥有标签nodeenv=pro的pod不在同一Node上，运行测试。

```powershell
# 创建pod
[root@master ~]# kubectl create -f pod-podandaffinity-required.yaml
pod/pod-podandaffinity-required

# 查看pod
# 发现调度到node2上
[root@master ~]# kubectl get pods pod-podandaffinity-required -n dev-o wide
NAME						REDAY		STATUS		RESTARTS		AGE		IP			NODE		...
pod-podaffinity-required	1/1			Pending		0				16s		10.244.1.96	node2		...
```

### 5.4.3 污点和容忍

**污点**

​	前面的调度方式，都是站在pod的角度上，通过在pod上添加属性，来确定pod是否会调度到耨写Node上，其实kubernetes也允许站在Node的角度上，通过Node上添加污点（Taints）属性，来决定是否允许pod调度过来。	

​	Node被设置上污点之后就和Pod之间存在一种相斥的关系，进而拒绝Pod调度进来，甚至可以将已存在的Pod驱逐出去。

污点的格式为：key=value:effect，key和value是污点的标签，effect描述污点的作用，支持如下三个选项：

- PreferNoSchedule：kubernetes将尽量避免把Pod调度到具有该污点的Node上，除非没有其他节点可调度
- NoSchedule：kubernetes将不会把Pod调度到具有该污点的Node上，但不会影响当前Node上已存在的Pod
- NoExecute：kubernetes将不会把Pod调度到具有该污点的Node上，同时也会将当前Node上已存在的Pod驱离

![5-4-3-01](C:\language\document\kubernetes\kubernetes_img\5-4-3-01.jpg)

使用kubectl设置和去除污点的命令示例如下：

```powershell
# 设置污点
[root@master ~]# kubectl taint nodes node1 key=value:effect

# 去除污点
[root@master ~]# kubectl taint nodes node1 key:effect-

# 去除所有污点
[root@master ~]# kubectl taint nodes1 key-
```

接下来，演示下污点的效果：

1. 准备节点node1（为了演示效果更加明显，暂时停止node2节点）
2. 为node1节点，设置一个污点：tag=heima:PreferNoSchedule；然后创建pod1（pod1可以）
3. 修改node1节点，设置一个污点：tag=heima:NoSchedule；然后创建pod2（pod1可以，pod2失败）
4. 修改node1节点，设置一个污点：tag=heima:NoExecute；然后创建pod3（3个pod都失败）

```powershell
# 为node1设置污点（PerferNoSchedule）
[root@master ~]# kubectl taint nodes node1 tag=heima:PreferNoSchedule

# 创建pod1
[root@master ~]# kubectl run taint1 --image=nginx:1.17.1 -n dev
[root@master ~]# kubectl get pods -n dev -o wide
NAME				REDAY	STATUS	RESTARTS	AGE		IP			NODE
taint1-7affsadf-as  1/1		Running	0			2m24s	10.244.1.59	node1

# 为node1设置污点（取消PreferNoSchedule，设置NoSchedule）
[root@master ~]# kubectl taint nodes node1 tag:PreferNoSchedule-
[root@master ~]# kubectl taint nodes node1 tag=heima:NoSchedule

# 创建node2
[root@master ~]# kubectl run taint2 --image=nginx:1.17.1 -n dev
[root@master ~]# kubectl get pods taint2 -n dev -o wide
NAME				REDAY	STATUS	RESTARTS	AGE		IP			NODE
taint1-7affsadf-as  1/1		Running	0			2m24s	10.244.1.59	node1
taint1-7affasff-ds  0/1		Pending	0			24s		<none>		<none>

# 创建pod3
[root@master ~]# kubectl run taint3 --image=nginx:1.17.1 -n dev
[root@master ~]# kubectl get pods -n dev -o wide
NAME				REDAY	STATUS	RESTARTS	AGE		IP			NODE
taint1-7affsadf-as  0/1		Pending	0			24s		<none>		<none>
taint1-7affasff-ds  0/1		Pending	0			22s		<none>		<none>
taint1-7afsdfff-ss  0/1		Pending	0			21s		<none>		<none>
```

```markdown
小提示：
	使用kubeadm搭建的集群，默认就会给master节点添加一个污点标记，所以pod就不会调度到master节点上。
```

容忍（Toleration）

​	上面介绍了污点的作用，可以在node上添加污点用于拒绝pod调度上来，但是如果就是想将一个pod调度到一个有污点的node上去，这时候应该怎么做？这就要使用到容忍。

![5-4-3-02](C:\language\document\kubernetes\kubernetes_img\5-4-3-02.jpg)

> 污点就是拒绝，容忍就是忽略，Node通过污点拒绝pod调度上去，pod通过容忍忽略拒绝

下面先通过一个案例看下效果：

1. 上一小节，已经在node1节点上打上了NoExecute的污点，此时pod是调度不上去的
2. 本小节，可以通过给pod添加容忍，然后将其调度上去

创建pod-toleration.yaml，内容如下：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-toleration
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
  tolerations:			# 添加容忍
  - key: "tag"				# 要容忍的污点的key
    operator: "Equal"		# 操作符
    value: "heima"			# 容忍的污点的value
    effect: "NoExecute"		# 添加容忍的规则，这里必须和标记的污点的规则相同
```

```powershell
# 添加容忍之前的pod
[root@master ~]# kubectl get pods -n dev -o wide
NAME				REDAY	STATUS	RESTARTS	AGE		IP			NODE
taint1-7affsadf-as  0/1		Pending	0			24s		<none>		<none>
```

下面看一下容忍的详细配置：

```powershell
[root@master ~]# kubectl explian pod.spec.tolerations
......
FIELDS:
  key					# 对应的要容忍的污点的键，空意味着匹配所有的键
  value					# 对应的要容忍的污点的值
  operator				# key-value的运算符，支持Equal和Exists（默认）
  effect				# 对应污点的effect，空意味着匹配所有影响
  tolerationSeconds		# 容忍时间，当effect为NoExecute时生效，表示pod在Node上的停时间
```

# 第六章 Pod控制器详解

本章节主要介绍各种pod控制器的详细使用

## 6.1 Pod控制器介绍

在kubernetes中，按照pod的创建方式可以将其分为两类：

- **自主式pod**：kubernetes直接创建出来的pod，这种pod删除后就没有了，也不会重建
- **控制器创建的pod**：通过控制器创建的pod，这种pod删除了之后还会自动创建

> 什么是Pod控制器
>
> ​	Pod控制器就是管理pod的中间层，使用了pod控制器之后，我们只需要告诉pod控制器，想要多少个什么样的pod就可以了，它就会创建出满足条件的pod，并确保每一个pod处于用户期望的状态，如果pod在运行中出现故障，控制器会基于指定的策略重启或重建pod。

在kubernetes中，有很多类型的pod控制器，每种都有自己的适合的场景，常见的有下面这些：

- ReplicationController：比较原始的pod控制器，已经被废弃，由ReplicaSet代替
- ReplicaSet：保证指定数量的pod运行，并支持pod数量变更，镜像版本变更
- **Deployment**：通过控制ReplicaSet来控制pod，并支持滚动升级、版本回退**（常用）**
- Horizontal Pod Autoscaler：可以根据集群负载自动调整Pod的数量，实现削峰填谷
- DaemonSet：在集群中的指定Node上都运行一个副本，一般用于守护进程类的任务
- Job：它创建出来的pod只要完成任务就立即退出，用于执行一次性任务
- Cronjob：它创建的pod会周期性的执行，用于执行周期性任务
- StatefulSet：管理有状态应用

## 6.2 ReplicaSet（RS）

​	ReplicatinSet的主要作用是维持一组pod服务的运行，说白了，就是保证一定数量的pod能够在集群中正常运行，它会坚持监听这些pod的运行状态，一旦pod发生故障，就会重启pod或重建。同时它还支持对pod数量的**扩缩容**和**版本镜像的升级**

![6-2-01](C:\language\document\kubernetes\kubernetes_img\6-2-01.jpg)

ReplicaSet的资源清单文件：

```yaml
apiVersion: apps/v1		# 版本号
kind: ReplicaSet		# 类型
metadata:				# 元数据
  name:						# rs名称
  namespace:				# 所属命名空间
  labels:					# 标签
    controller: rs
spec:					# 详情描述
  replicas: 3				# 副本数量
  selector:					# 选择器，通过它指定该控制器管理哪些pod
    matchLabels:				# Labels匹配规则
      app: nginx-pod
    matchExpressions:			# Expressions匹配规则
    - {key: app, operator: In, values: [nginx-pod]}
  template:					# 模板，当副本数量不足时，会根据下面的模板创建pod副本
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
      - name: nginx
      	image: nginx:1.17.1
      	ports:
      	- containerPort: 80
```

在这里面，需要新了解的配置项就是spec下面几个选项：

- replicas：指定副本数量，其实就是当前rs创建出来的pod数量，默认为1
- selector：选择器，它的作用是建立pod控制器和pod之间的关联关系，采用的Label Selector机制在pod模板上定义label，在控制器上定义选择器，就可以表明当前控制器能管理哪些pod了
- template：模板，就是当前控制器创建pod所用的模板，里面其实就是前一章学过的pod的定义

**创建ReplicaSet**

创建pc-replicaset.yaml文件，内容如下：

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: pc-replicaset
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.1
```

```powershell
# 创建rs
[root@master ~]# kubectl create -f pc-replicaset.yaml

# 查看rs
# DESIRED：期望副本数量
# CURRENT：当前副本数量
# REDAY：已经准备好提供服务的副本数量
[root@master ~]# kubectl get rs pc-replicaset -n dev -o wide
NAME			DESIRED		CURRENT		REDAY	AGE		CONTAINTERS		IMAGES		 SELECTOR
pc-replicaset	3			3			3		22s		nginx			nginx:1.17.1 app=nginx-pod

# 查看当前控制器创建出来的pod
# 这里发现控制器创建出来的pod的名称是在控制器名称后面拼接了-xxxxx随机码
[root@master ~]# kubectl get pods -n dev
NAME							REDAY		STATUS		RESTARTS		AGE
pc-replicaset-asdfv 1/1			Running		0			0				54s
pc-replicaset-sfasf 1/1			Running		0			0				54s
pc-replicaset-wrwqd 1/1			Running		0			0				54s
```

**扩缩容**

```powershell
# 编辑rs的副本数量，修改spec:replicaset: 6即可
# 改命令以vim的形式展开，可直接编辑,:wq保存
[root@master ~]# kubectl edit rs pc-replicaset -n dev
replicaset.apps/pc-replicaset edited

# 查看pod
[root@master ~]# kubectl get pod pods -n dev
NAME							REDAY		STATUS		RESTARTS		AGE
pc-replicaset-asdfv 1/1			Running		0			0				114m
pc-replicaset-sfasf 1/1			Running		0			0				54s
pc-replicaset-wrwqd 1/1			Running		0			0				54s
pc-replicaset-rerwd 1/1			Running		0			0				114m
pc-replicaset-gdgeh 1/1			Running		0			0				54s
pc-replicaset-lsrgf 1/1			Running		0			0				114m

# 当然也可以直接使用命令实现
# 使用sacle命令实现扩缩容，后面--replicas=n直接指定目标数量即可
[root@master ~]# kubectl scale rs pc-replicaset --replicas=2 -n dev
replicaset.apps/pc-replicaset scaled

# 命令执行完毕，立即查看，发现已经有4个开始准备退出了
[root@master ~]# kubectl get pods -n dev
NAME							REDAY			STATUS		RESTARTS		AGE
pc-replicaset-asdfv 1/1			Terminating		0			0				118m
pc-replicaset-sfasf 1/1			Terminating		0			0				118m
pc-replicaset-wrwqd 1/1			Terminating		0			0				118m
pc-replicaset-rerwd 1/1			Running			0			0				4m17s
pc-replicaset-gdgeh 1/1			Terminating		0			0				118m
pc-replicaset-lsrgf 1/1			Running			0			0				4m17s

# 稍等片刻，就只剩下2个了
[root@master ~]# kubectl get pods -n dev
NAME							REDAY			STATUS		RESTARTS		AGE
pc-replicaset-rerwd 1/1			Running			0			0				4m17s
pc-replicaset-lsrgf 1/1			Running			0			0				4m17s
```

**镜像升级**

```powershell
# 编辑rs的容器镜像 --image: nginx:1.17.2
# 改命令以vim的形式展开，可直接编辑,:wq保存
[root@master ~]# kubectl edit rs pc-replicaset -n dev
replicaset.apps/pc-replicaset edited

# 查看pod
[root@master ~]# kubectl get rs -n dev -o wide
NAME			DESIRED		CURRENT		REDAY	AGE		CONTAINTERS		IMAGES		 SELECTOR
pc-replicaset	2			2			2		145m	nginx			nginx:1.17.2 app=nginx-pod

# 同样的道理，也可以使用命令完成这个工作
# kubectl set image rs rs名称 容器=镜像版本 -n namespace
[root@master ~]# kubectl set image rs pc-replicaset nginx=nginx:1.17.1 -n dev
replicaset.apps/pc-replicaset image updated

# 再次查看，发现镜像版本已经变更了
[root@master ~]# kubectl get rs -n dev -o wide
NAME			DESIRED		CURRENT		REDAY	AGE		CONTAINTERS		IMAGES		 SELECTOR
pc-replicaset	2			2			2		145m	nginx			nginx:1.17.1 app=nginx-pod

```

**删除ReplicaSet**

```powershell
# 使用kubectl delete命令会删除此RS以及它管理的pod
# 在kubernetes删除RS前，会将RS的replicasclear调整为0，等待所有的pod删除后，在执行RS对象的删除
[root@master ~]# kubectl delete rs pc-replicaset -n dev
replicaset.apps "pc-replicaset" deleted
[root@master ~]# kubectl get pod pods -n dev
No resource found in dev namespace

# 如果希望仅仅删除RS对象（保留pod），可以使用kubectl delete命令时添加--casecade=false选项（不推荐）。pod由控制器创建，控制器删除不在保留pod
[root@master ~]# kubectl delete rs pc-replicaset -n dev --casecade=false
replicaset.apps "pc-replicaset" deleted

[root@master ~]# kubectl get pod pods -n dev
NAME							REDAY			STATUS		RESTARTS		AGE
pc-replicaset-asdfv 1/1			Running			0			0				75s
pc-replicaset-sfasf 1/1			Running			0			0				75s

# 也可以直接使用yaml直接删除（推荐）
[root@master ~]# kubectl delete -f pc-replicaset.yaml
replicaset.apps "pc-replicaset" deleted
```

## 6.3 Deployment

​	为了更好解决服务编排的问题，kubernetes在V1.2版本开始，引入了Deployment控制器，值得一提的是，

这种控制器并不直接管理pod，而是提供管理ReplicaSet来间接管理Pod，即：Deployment管理ReplicaSet,

ReplicaSet管理Pod。所以Deployment比ReplicaSet功能更加强大。

![6-3-01](C:\language\document\kubernetes\kubernetes_img\6-3-01.jpg)

```yaml
apiVersion: apps/v1		# 版本号
kind: ReplicaSet		# 类型
metadata:				# 元数据
  name:						# rs名称
  namespace:				# 所属命名空间
  labels:					# 标签
    controller: rs
spec:					# 详情描述
  replicas: 3				# 副本数量
  revisionHistoryLimit: 3	# 保留历史版本，默认是10
  paused: false				# 暂停部署，默认是false
  progressDeadlineSeconds: 600	# 部署超时时间（s），默认是600
  strategy:					# 策略
    type: RollingUpdate			# 滚动更新策略
    rollingUpdate:				# 滚动更新
      maxSurege: 30%				# 最大额外可以存在的副本数，可以为百分比，也可以为整数
      maxUnavailable: 30%			# 最大不可用状态的Pod的最大值，可以为百分比，也可以为整数
  selector:					# 选择器，通过它指定该控制器管理哪些pod
    matchLabels:				# Labels匹配规则
      app: nginx-pod
    matchExpressions:			# Expressions匹配规则
    - {key: app, operator: In, values: [nginx-pod]}
  template:					# 模板，当副本数量不足时，会根据下面的模板创建pod副本
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
      - name: nginx
      	image: nginx:1.17.1
      	ports:
      	- containerPort: 80
```

**创建Deployment**

创建pc-deployment.yaml，内容如下：

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: pc-replicaset
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.1
```

```powershell
# 创建deployment
# --record=true 记录整个deployment创建过程
[root@master ~]# kubectl create -f pc-deployment -n dev
deployment.apps/pc-deployment created

# 查看deployment
# UP-TO-DATE 最新版本的pod的数量
# AVAILABLE  当前可用的pod的数量

[root@master ~]# kubectl get deployment pc-deployment -n dev
NAME			REDAY	UP-TO-DATE		AVAILABLE	AGE
pc-deployment	3/3		3				3			15s

# 查看rs
# DESIRED：期望副本数量
# CURRENT：当前副本数量
# REDAY：已经准备好提供服务的副本数量
# 发现rs的名称是在原来deployment的名字后面添加了一个10位数的随机串
[root@master ~]# kubectl get rs -n dev
NAME						DESIRED		CURRENT		REDAY	AGE
pc-deployment-asdfcsdfwq	3			3			3		22s		

# 查看pod
[root@master ~]# kubectl get pods -n dev
NAME						REDAY		STATUS		RESTARTS		AGE
pc-deployment-asdfcsdfwq	1/1			Running		0				107s
pc-deployment-bswecsdfwq	1/1			Running		0				107s
pc-deployment-fsdscergwq	1/1			Running		0				107s

```

**扩缩容**

```powershell
# 变更副本数量为5个（命令行形式）
[root@master ~]# kubectl scale deploy pc-deployment --replicas=5 -n dev
deployment.apps/pc-deployment scaled

# 查看deployment
[root@master ~]# kubectl get deploy pc-deployment -n dev
NAME			REDAY	UP-TO-DATE		AVAILABLE	AGE
pc-deployment	5/5		35				5 			15s

# 查看pod
[root@master ~]# kubectl get pods -n dev
NAME						REDAY		STATUS		RESTARTS		AGE
pc-deployment-asdfcsdfwq	1/1			Running		0				4m107s
pc-deployment-bswecsdfwq	1/1			Running		0				107s
pc-deployment-fsdscergwq	1/1			Running		0				107s
pc-deployment-asdfwefgsw	1/1			Running		0				4m107s
pc-deployment-grhrthtrr		1/1			Running		0				4m107s

# 编辑deployment的副本数量，修改spec:replicas:4即可
[root@master ~]# kubectl edit deploy pc-deployment -n dev
deployment.apps/pc-deployment edited


[root@master ~]# kubectl get pods -n dev
NAME						REDAY		STATUS		RESTARTS		AGE
pc-deployment-asdfcsdfwq	1/1			Running		0				5m107s
pc-deployment-bswecsdfwq	1/1			Running		0				2m38s
pc-deployment-fsdscergwq	1/1			Running		0				2m38s
pc-deployment-asdfwefgsw	1/1			Running		0				5m107s
```

**镜像更新**

Deployment支持两种镜像更新的策略：**重建策略**和**滚动更新（默认）**，可以通过strategy选项进项配置。

```yaml
strategy: 		# 指定新的Pod替换旧的Pod的策略，支持两个属性：
  type: 			# 指定策略类型，支持两种策略
    Recreate: 			# 在创建出新的pod之前会先杀掉所有已存在的pod
    RollingUndata:		# 滚动更新，就是杀死一部分，就启动一部分，在更新过程中，存在两个版本pod
  rollingUpdate: 	# 当type为RollingUndata时生效，用于为RollingUndata设置参数，存在两个版本pod
    maxUnavailable:		# 用来指定在升级过程中不可用pod的最大数量，默认为25%
    maxSurege:			# 用来指定在升级过程中可以超过期望的pod的最大数量，默认为25%
```

1）重建更新

​	1-1）编辑pc-deployment.yaml，在spec节点下添加更新策略

```yaml
spec:
  strategy: 	# 策略
  	type: Recreate	# 重建更新策略
```

​	1-2）创建deploy进行验证

```powershell
# 变更镜像
[root@master ~]# kubectl set image deployment pc-deployment nginx=nginx1.17.2 -n dev
deployment.apps/pc-deployment image updated

# 观察升级过程
[root@master ~]# kubectl get pods -n dev -w
NAME							REDAY   STATUS			RESTARTS	AGE
pc-deployment-asdfasdfvw-jhgf	1/1		Running			0			78s
pc-deployment-asdfasdfvw-wsfa	1/1		Running			0			78s
pc-deployment-asdfasdfvw-gger	1/1		Running			0			78s

pc-deployment-asdfasdfvw-jhgf	1/1		Terminating		0			97s
pc-deployment-asdfasdfvw-wsfa	1/1		Terminating		0			97s
pc-deployment-asdfasdfvw-gger	1/1		Terminating		0			97s


pc-deployment-rthswqvgrt-asdfe	1/1		Pending			0			0s
pc-deployment-yyafwefvsw-yregb	1/1		Pending			0			0s
pc-deployment-utwasdfasd-wefvt	1/1		Pending			0			0s

pc-deployment-rthswqvgrt-asdfe	1/1		ContainerCreating			0			0s
pc-deployment-yyafwefvsw-yregb	1/1		ContainerCreating			0			0s
pc-deployment-utwasdfasd-wefvt	1/1		ContainerCreating			0			0s

pc-deployment-rthswqvgrt-asdfe	1/1		Running			0			78s
pc-deployment-yyafwefvsw-yregb	1/1		Running			0			78s
pc-deployment-utwasdfasd-wefvt	1/1		Running			0			78s
```

2）滚动更新

​	2-1）编辑pc-deployment.yaml，在spec节点下添加更新策略

```yaml
spec:
  strategy: 				
  	type: RollingUpdate			
  rollingUpdate: 			
    maxUnavailable:	20%		
    maxSurege: 20%	
```

​	2-2）创建deploy进行验证

```powershell
# 变更镜像
[root@master ~]# kubectl set image deployment pc-deployment nginx=nginx1.17.2 -n dev
deployment.apps/pc-deployment image updated

# 观察升级过程
[root@master ~]# kubectl get pods -n dev -w
......
# 书写过程，pod会新启一个，停止一个
```

滚动更新的过程

<img src="C:\language\document\kubernetes\kubernetes_img\6-3-02.jpg" alt="6-3-02" style="zoom:67%;" />

镜像更新中rs的变化：

```powershell
# 查看rs，发现原来的rs依旧存在，只是pod数量变为了0，而后又新产生了一个rs，pod数量为4
# 其实这就是deployment能够进行版本回退的奥妙所在，后面会详细解释
[root@master ~]# kubectl get rs -n dev
NAME						DESIRED		CURRENT		READY		AGE
pc-deployment-5d89bdasfd	0			0			0			11m
pc-deployment-65asdfasdf	0			0			0			11m
pc-deployment-werasdafas	3			3			3			3m1s
```

**版本回退**

deployment支持版本升级过程中的暂停，继续功能以及版本回退等诸多功能，下面具体来看：

kubectl rollout：版本升级相关功能：支持下面的选项：

- status		显示当前升级状态
- history      显示升级历史记录
- pause       状态版本升级过程
- resume     继续已暂停的版本升级过程
- restart      重启版本升级过程
- undo         回滚到上一级版本（可以使用--to-reversion回滚到指定版本）

```powershell
# 查看当前升级版本的状态
[root@master ~]# kubectl rollout status deploy pc-deployment -n dev
deployment "pc-deployment" successfully	rolled out

# 查看升级历史记录
[root@master ~]# kubectl rollout history deploy pc-deployment -n dev
deployment.apps/pc-deployment 
REVERSION		CHANGE-CAUSE
1				kubectl create --filename=pc-deployment.yaml --record=true
2				kubectl create --filename=pc-deployment.yaml --record=true
3				kubectl create --filename=pc-deployment.yaml --record=true
# 可以发现有三次版本记录，说明完成过两次升级

# 版本回滚
# 这里直接使用--to-reversion=1回滚到了1版本，如果省略这个选项，就是回退到上个版本，就是2版本
[root@master ~]# kubectl rollout undo deployment pc-deployment --to-reversion=1 -n dev
deployment.apps/pc-deployment rolled back

# 查看发现，通过nginx镜像版本可以发现到了第一版
[root@master ~]# kubectl get deploy -n dev -o wide
NAME				REDAY		UP-TO-DATE		AVAILABLE	AGE		CONTAINERS		IMAGES
pc-deployment		4/4			4				74m			nginx	nginx			nginx:1.17.1

# 查看rs，发现第一个rs中有4个pod运行，后面两个版本的rs中pod为运行
# 其实deployment之所以可以实现版本的回滚，就是通过记录下历史rs来实现的
# 一旦想回滚到哪个版本，只需要将当前版本pod数量改为0，然后将回滚版本的pod提升为目标数量就可以了
[root@master ~]# kubectl get rs -n dev
NAME						DESIRED		CURRENT		READY		AGE
pc-deployment-5d89bdasfd	4			4			4			11m
pc-deployment-65asdfasdf	0			0			0			11m
pc-deployment-werasdafas	0			0			0			3m1s
```

**金丝雀发布**

​	Deployment支持更新过程中的控制，如"暂停（pause）"或"继续（resume）"更新操作。

​	比如有一批新的pod资源创建完成后立即暂停更新过程，此时，仅存在一部分新版本的应用，主题部分还是旧的版本，然后，在筛选一小部分的用户请求路由到新版本的pod应用，继续观察能否稳定地按期望的方式运行。确定没问题之后在继续完成余下的pod的资源滚动更新，否则立即回滚更新操作。这就是所谓的金丝雀发布。

```powershell
# 更新deployment的版本，并配置暂停deployment
[root@master ~]# kubectl set image deploy pc-deployment nginx=nginx:1.17.1 -n dev && kubectl rollout pause deployment pc-deployment -n dev
deployment.apps/pc-deployment image updated
deployment.apps/pc-deployment puase

# 观察更新状态
[root@master ~]# kubectl rollout status deploy pc-deployment -n dev
Waiting for deployment "pc-deployment" rollout to finish: 2 out of 4 new replicas have been updated ...

# 监控更新的过程，可以看到已经更新了一个资源，但是并未按照预期的状态去删除一个旧的资源，就是因为使用了pause暂停命令
[root@master ~]# kubectl get rs -n dev -o wide
NAME					DESIRED	CURRENT	READY		AGE		CONTAINERS		IMAGES
pc-deployment-asdffsdsf	3		3		3			19m		nginx			nginx:1.17.1
pc-deployment-659asdfas 0		0		0			19m		nginx			nginx:1.17.2
pc-deployment-266asfdas	2		2		2			19m		nginx			nginx:1.17.4

[root@master ~]# kubectl get pods -n dev
NAME								READY		STATUS		RESTARTS		AGE
pc-deployment-asdffsdsf-asdfe		1/1			Running		0				7m34s	
pc-deployment-asdffsdsf-gdger	 	1/1			Running		0				7m32s		
pc-deployment-asdffsdsf-egasd		1/1			Running		0				7m25s
pc-deployment-266asfdas-hergv		1/1			Running		0				3m25s	
pc-deployment-266asfdas-tyter		1/1			Running		0				3m25s	

# 确保更新的pod没问题了，继续更新
[root@master ~]# kubectl get rs -n dev -o wide
NAME					DESIRED	CURRENT	READY		AGE		CONTAINERS		IMAGES
pc-deployment-asdffsdsf	0		0		0			19m		nginx			nginx:1.17.1
pc-deployment-659asdfas 0		0		0			19m		nginx			nginx:1.17.2
pc-deployment-266asfdas	4		4		4			19m		nginx			nginx:1.17.4

[root@master ~]# kubectl get pods -n dev
NAME								READY		STATUS		RESTARTS		AGE
pc-deployment-266asfdas-asdfe		1/1			Running		0				7m34s	
pc-deployment-266asfdas-gdger	 	1/1			Running		0				7m32s		
pc-deployment-266asfdas-egasd		1/1			Running		0				7m25s
pc-deployment-266asfdas-hergv		1/1			Running		0				3m25s	
pc-deployment-266asfdas-tyter		1/1			Running		0				3m25s	
```

**删除Deployment**

```powershell
# 删除deployment，其下的rs和pod也将被删除
[root@master ~]# kubectl delete -f pc-deployment.yaml
deployment.apps/"pc-deployment" deleted
```

## 6.4 Horizontal Pod Autoscaler

​	在前面的课程中，我们可以通过手工执行**kubectl scale**命令执行pod扩容，但是这显然不符合kubernetes在定位目标--自动化、智能化。kubernetes期望可以通过检测pod的使用情况，实现pod数量的自动调整，于是就产生了HPA这种控制器。

​	HPA可以获取每个pod利用率，然后和HPA中定义的指标进行对比，同时计算出需要伸缩的具体值，最后实现pod的数量的调整。其实HPA与之前的Deployment一样，也属于一种Kubernetes资源对象，它通过追踪分析目标pod的负载变化情况，来确定是否需要针对性地调整目标pod的副本数。

![6-4-01](C:\language\document\kubernetes\kubernetes_img\6-4-01.jpg)

**1）安装metrics-server**

metrics-server可以用来采集集群中的资源使用情况

```powershell
# 安装git
[root@master ~]# yum install git -y

# 获取metrics-server，注意使用的版本
[root@master ~]# git clone -b v0.3.6 https://github.com/kubernetes-inclubator/metrics-server

# 修改deployment，注意修改的是镜像和初始化参数
[root@master ~]# cd /root/metrics-server/deploy/1.8+/
[root@master 1.8+]# vim metrics-sever-deployment.yaml

# 按照图中添加下面选项
hostNetWork: true
registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server-amd64:v0.3.6
- --kubelet-insecure-tls
- --kubelet-preferred-address-types=InternalIP,Hostname.InternalDNS,ExternalDNS,ExternalIP
```

![6-4-02](C:\language\document\kubernetes\kubernetes_img\6-4-02.jpg)

```powershell
# 安装metrics-server
[root@master 1.8+]# kubectl apply -f ./

# 查看pod运行情况
[root@master 1.8+]# kubectl get pod -n kube-system
NAME								READY		STATUS		RESTARTS	AGE
metrics-server-adfasdfsdf-asdfd		1/1			Running		0			90s

# 使用kubectl top node 查看资源使用情况
[root@master 1.8+]# kubectl top node
NAME			CPU(cores)		CPU%		MEMORY(bytes)		MEMORY%
master			98m				4%			1067Mi				62%
node1			27m				1%		    727Mi				42%
node2			34m				1%			800Mi				46%

[root@master 1.8+]# kubectl top pod -n kube-system
NAME						CPU(cores)		MEMORY%	
coredns-asdfasdfss-sagwe	3m				9Mi			
master-joiasjfiaos-ojion	3m				8Mi		  
etcd-master					14m			    145Mi
...
# 至此，metrics-server安装完成
```

**2）准备deployment和service**

为了操作简单，直接使用命令

```powershell
# 创建deployment
[root@master 1.8+]# kubectl run nginx --image=nginx:1.17.1 --requests=cpu=100m -n dev

# 创建service
[root@master 1.8+]# kubectl expose deployment nginx --type=NodePort --port=80 -n dev

# 查看
[root@master 1.8+]# kubectl get deployment,pod,svc -n dev
NAME					READY		UP-TO-DATE		AVAILABLE		AGE
deployment.apps/nginx	1/1			1				1				47s

NAME						READAY		STATUS			RESTARTS		AGE
pod/nginx-fsfasdfass-kojo	1/1			Running			0				47s

NAME			TYPE		CLUSTER-IP		EXTERNAL-IP		PORT(S)			AGE
service			NodePort	10.109.27.248	<none>			80:31136/TCP 	35s	
```

**3）部署HPA**

创建pc-hpa.yaml

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: pc-hpa
  namespace: dev
spec:
  minReplicas: 1		# 最小pod数量
  maxReplicas: 10		# 最大pod数量
  targetCPUUtilizationPercentage: 3		# CPU使用率指标
  scaleTargetRef:						# 指定要控制的nginx信息
  	apiVersion: apps/v2
  	kind: Deployment
  	name: nginx
```

```powershell
# 创建hpa
[root@master 1.8+]# kubectl create -f pc-hpa.yaml
horizontalpodautoscaler.autoscaling/pc-hpa created

# 查看hpa
[root@master 1.8+]# kubectl get hpa -n dev
NAME			REFERENCE		TARGETS			MINPODS		MAXPODS		REPLICAS		AGE
pc-hpa			Deployment		0%/3%			1			10			1				62s
```

**4）测试**

使用压测工具对service地址192.168.5.101:31136进行压测，然后通过控制台查看hpa和pod的变化

## 6.5 DaemonSet

​	DaemonSet类型的控制器可以保证集群中的每一台（或指定）节点上都运行一个副本，一般适用于日志收集、节点监控等场景。也就是说，如果一个pod提供的功能是节点级别的（每个节点都需要且只需要一个），那么这类pod就适合使用DaemonSet类型的控制器创建。

![6-5-01](C:\language\document\kubernetes\kubernetes_img\6-5-01.jpg)

DaemonSet控制器的特点：

- 每当向集群添加一个节点时，指定的pod副本也将添加到该节点上
- 当节点从集群中移除时，pod也就被垃圾回收了



下面先来看下DaemonSet的资源清单文件

```yaml
apiVersion: apps/v1		# 版本号
kind: Deployment		# 类型
metadata:				# 元数据
  name:						# rs名称
  namespace:				# 所属命名空间
  labels:					# 标签
    controller: daemonset
spec:					# 详情描述
  revisionHistoryLimit: 3	# 保留历史版本，默认是10
  updateStrategy:			# 更新策略
#  strategy:					# 策略
    type: RollingUpdate			# 滚动更新策略
    rollingUpdate:				# 滚动更新
#      maxSurege: 30%				# 最大额外可以存在的副本数，可以为百分比，也可以为整数
      maxUnavailable: 30%			# 最大不可用状态的Pod的最大值，可以为百分比，也可以为整数
  selector:					# 选择器，通过它指定该控制器管理哪些pod
    matchLabels:				# Labels匹配规则
      app: nginx-pod
    matchExpressions:			# Expressions匹配规则
    - {key: app, operator: In, values: [nginx-pod]}
  template:					# 模板，当副本数量不足时，会根据下面的模板创建pod副本
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
      - name: nginx
      	image: nginx:1.17.1
      	ports:
      	- containerPort: 80
```

创建pc-daemonset.yaml，内容如下：

```yaml
apiversion: apps/v1
kind: DaemonSet
metadata:
  name: pc-daemonset
  namespace: dev
spec:
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
      - name: nginx
      	image: nginx:1.17.1
```

```powershell
# 创建daemonset
[root@master 1.8+]# kubectl create -f pc-daemonset.yaml
daemonset.apps/pc-daemonset created

# 查看daemonset
[root@master 1.8+]# kubectl get daemonset -n dev
NAME			DESIRED		CURRENT		UP-TO-DATE		VAILABLE	AGE		CONTAINERS	IMAGES
pc-daemonset	2			2			2				2			24S		nginx		nginx:1.17.1

# 查看pod，发现在每个Node上都运行一个pod	
[root@master 1.8+]# kubectl get pods -n dev -o wide
NAME				READY	STATUS		RESTARTS		AGE		IP				NODE
pc-daemonset-afdwe	1/1		Running		0				37s		10.244.1.43		node1
pc-daemonset-kokpo	1/1		Running		0				37s		10.244.1.74		node2

# 删除daemonset
[root@master 1.8+]# kubectl delete -f pc-daemonset.yaml
daemonset.apps "pc-daemonset" deleted
```

## 6.6 Job	

Job，主要用于负责批量处理（一次要处理指定数量任务）短暂的一次性（每个任务仅运行一次就结束）任务，仅执行一次，并保证处理的一个或者多个pod成功结束。它的特点如下：

- 当Job创建的Pod执行成功并正常结束时，Job将记录成功结束的Pod数量
- 当成功结束的pod达到指定的数量时，job将完成执行
- 删除Job对象时，将清理掉由Job创建的Pod

Job的资源清单文件：

![6-6-01](C:\language\document\kubernetes\kubernetes_img\6-6-01.jpg)

```yaml
apiVersion: batch/v1		# 版本号
kind: Job				# 类型
metadata:				# 元数据
  name:						# rs名称
  namespace:				# 所属命名空间
  labels:					# 标签
    controller: job
spec:					# 详情描述
  completions: 3	# 指定job需要与成功运行pods的次数。默认值：1
  parallelism: 1	#  指定job在任一时刻应该并发运行pods的数量。默认值：1
  activeDeadlineSeconds: 30	# 指定job可运行的时间期限，超过时间还未结束，系统将会尝试进行终止。
  backoffLimit: 6			# 指定job失败后进行重试的次数。默认是6
  manualSelector: true		# 是否可以使用selector选择器选择pod，默认是false
  selector:					# 选择器，通过它指定该控制器管理哪些pod
    matchLabels:				# Labels匹配规则
      app: nginx-pod
    matchExpressions:			# Expressions匹配规则
    - {key: app, operator: In, values: [nginx-pod]}
  template:					# 模板，当副本数量不足时，会根据下面的模板创建pod副本
    metadata:
      labels:
        app: nginx-pod
    spec:
      restartPolicy: Never	# 重启策略只能设置为Never或者OnFailure
      containers:
      - name: counter
      	image: busybox:1.30
      	command: ["bin/sh", "-c", "for i in 9 8 7 6 5 4 3 2 1; do echo $i;sleep 2; done"]
```

```markdown
关于重启策略设置的说明：
	如果指定为OnFailure，则job会在pod出现故障时重启容器，而不是创建pod,failed次数不变
	如果指定为Never，则job会在pod出现故障时创建新的pod，并且故障pod不会消失，也不会重启，failed次数加1
	如果指定为Always，就意味着一直重启，意味着job任务会重复去执行，当然不对，所以不能设置为Always
```

创建pc-job.yaml，内容如下：

```yaml
apiversion: apps/v1
kind: job
metadata:
  name: pc-job
  namespace: dev
spec:
  manualSelector: true
  selector:
    matchLabels:
      app: counter-pod
  template:
    metadata:
      labels:
        app: counter-pod
    spec:
      restartPolicy: Never
      containers:
      - name: counter
      	image: bosybox:1.30
      	command: ["bin/sh", "-c", "for i in 9 8 7 6 5 4 3 2 1; do echo $i;sleep 2; done"]
```

```powershell
# 创建daemonjobset
[root@master 1.8+]# kubectl create -f pc-job.yaml
job.batch/pc-job created

# 查看job
[root@master 1.8+]# kubectl get job -n dev -o wide -w
NAME		COMPLETIONS		DURATION		AGE		CONTAINERS		IMAGES		    SLECTOR	
pc-job		0/1				21s				21s		counter			busybox:1.30	app=counter-pod
pc-job		1/1				21s				21s		counter			busybox:1.30	app=counter-pod

# 查看pod，发现在每个Node上都运行一个pod	
[root@master 1.8+]# kubectl get pods -n dev -o wide
NAME				READY	STATUS		RESTARTS		AGE		IP				NODE
pc-daemonset-afdwe	1/1		Running		0				37s		10.244.1.43		node1
pc-daemonset-kokpo	1/1		Running		0				37s		10.244.1.74		node2

# 通过观察pod状态可以看到，pod运行完毕任务后，就会变成Complete状态
[root@master ~]# kubectl get job -n dev -w
NAME			READY 		STATUS		RESTARTS		AGE
pc-job-wrmsg	1/1			Running		0				29s
pc-job-wrmsg	0/1			Complete	0				29s

# 接下来，调整下pod运行的总数量和并行数量，即：在spec下设置下面两个选项
# completions: 6 # 指定job需要成功运行的次数为6
# parallelism: 3 # 指定job并发运行pods的数量为3
# 然后重新运行job，观察效果，此时会发现，job会每次运行3个pod，总共执行了6个pod
[root@master ~]# kubectl get pods -n dev -w
NAME			READY		STATUS			RESTARTS		AGE
pc-job-464fs	1/1			Running			0				5s
pc-job-jojos	1/1			Running			0				5s
pc-job-jijwo	1/1			Running			0				5s
pc-job-464fs	0/1			Completed		0				11s
pc-job-v7rhr	0/1			Pending			0				0s
pc-job-v7rhr	0/1			Pending			0				0s
pc-job-v7rhr	0/1			ContainerCreating			0				0s
......查看实测效果


# 删除job
[root@master ~]# kubectl delete -f pc-job.yaml
job.batch "pc-job" deleted
```

## 6.7 CronJob	

​	CronJob控制器以Job控制器资源为其管控对象，并借助它管理pod资源对象，Job控制器定义的作业任务在其控制器资源创建之后便会立即执行，但CronJob可以以类似与Linux操作系统的周期性任务作业计划的方式控制其运行时间点及重复运行的方式。也就是说，CronJob可以在特定的时间点（反复的）去运行job任务。

![6-7-01](C:\language\document\kubernetes\kubernetes_img\6-7-01.jpg)

CronJob的资源清单文件：

```yaml
apiVersion: batch/v1beta1		# 版本号
kind: CronJob				# 类型
metadata:				# 元数据
  name:						# rs名称
  namespace:				# 所属命名空间
  labels:					# 标签
    controller: cronjob
spec:					# 详情描述

  jobTemplate:					# 模板，当副本数量不足时，会根据下面的模板创建pod副本
    metadata:
      labels:
        app: nginx-pod
    spec:
      completions: 1
      parallelism: 1
      activeDeadlineSeconds: 30
      backoffLimit: 6
      manualSelector: true
      selector:					# 选择器，通过它指定该控制器管理哪些pod
        matchLabels:				# Labels匹配规则
          app: conter-pod
        matchExpressions: # 规则	
        - {key: app, operator: In, values: [counter-pod]}
      containers:
      - name: counter
      	image: busybox:1.30
      	command: ["bin/sh", "-c", "for i in 9 8 7 6 5 4 3 2 1; do echo $i;sleep 2; done"]
```

