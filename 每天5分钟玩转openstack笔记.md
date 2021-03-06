# 					Openstack

## 1.虚拟化

```
虚拟化是云计算的基础。简单的说，虚拟化使得在一台物理的服务器上可以跑多台虚拟机，虚拟机共享物理机的 CPU、内存、IO 硬件资源，但逻辑上虚拟机之间是相互隔离的。

物理机我们一般称为宿主机（Host），宿主机上面的虚拟机称为客户机（Guest）。

那么 Host 是如何将自己的硬件资源虚拟化，并提供给 Guest 使用的呢？
这个主要是通过一个叫做 Hypervisor 的程序实现的。

根据 Hypervisor 的实现方式和所处的位置，虚拟化又分为两种：
1型虚拟化和2型虚拟化
1型虚拟化

Hypervisor 直接安装在物理机上，多个虚拟机在 Hypervisor 上运行。Hypervisor 实现方式一般是一个特殊定制的 Linux 系统。Xen 和 VMWare 的 ESXi 都属于这个类型。

 2型虚拟化

物理机上首先安装常规的操作系统，比如 Redhat、Ubuntu 和 Windows。Hypervisor 作为 OS 上的一个程序模块运行，并对管理虚拟机进行管理。KVM、VirtualBox 和 VMWare Workstation 都属于这个类型。

 理论上讲：

1型虚拟化一般对硬件虚拟化功能进行了特别优化，性能上比2型要高；

2型虚拟化因为基于普通的操作系统，会比较灵活，比如支持虚拟机嵌套。嵌套意味着可以在KVM虚拟机中再运行KVM。 
```



## 2.KVM

```

```

## 3.云计算

### 3.1.IT系统架构

```
IT系统架构的发展到目前为止大致可以分为3个阶段：

    物理机架构
    这一阶段，应用部署和运行在物理机上。 比如企业要上一个ERP系统，如果规模不大，可以找3台物理机，分别部署Web服务器、应用服务器和数据库服务器。 如果规模大一点，各种服务器可以采用集群架构，但每个集群成员也还是直接部署在物理机上。 我见过的客户早期都是这种架构，一套应用一套服务器，通常系统的资源使用率都很低，达到20%的都是好的。

    虚拟化架构
    摩尔定律决定了物理服务器的计算能力越来越强，虚拟化技术的发展大大提高了物理服务器的资源使用率。
    这个阶段，物理机上运行若干虚拟机，应用系统直接部署到虚拟机上。
    虚拟化的好处还体现在减少了需要管理的物理机数量，同时节省了维护成本。

    云计算架构
    虚拟化提高了单台物理机的资源使用率，随着虚拟化技术的应用，IT环境中有越来越多的虚拟机，这时新的需求产生了：
    如何对IT环境中的虚拟机进行统一和高效的管理。
    有需求就有供给，云计算登上了历史舞台。

计算（CPU/内存）、存储和网络是 IT 系统的三类资源。
通过云计算平台，这三类资源变成了三个池子。
当需要虚机的时候，只需要向平台提供虚机的规格。
平台会快速从三个资源池分配相应的资源，部署出这样一个满足规格的虚机。
虚机的使用者不再需要关心虚机运行在哪里，存储空间从哪里来，IP是如何分配，这些云平台都搞定了。


OpenStack 针对的是 IT 基础设施，是 IaaS 这个层次的云操作系统
```

### 3.2浅谈云计算发展历程

```
https://blog.csdn.net/p5deyt322jacs/article/details/80745723
1. 物理设备不灵活
首先第一个阶段就是物理机，或者说物理设备时期。这个时期相当于客户需要一台电脑，我们就买一台放在数据中心里。物理设备当然是越来越牛，例如服务器，内存动不动就是百G内存，例如网络设备，一个端口的带宽就能有几十G甚至上百G，例如存储，在数据中心至少是PB级别的(一个P是1024个T，一个T是1024个G)。
然而物理设备不能做到很好的灵活性。首先它不能够达到想什么时候要就什么时候要、比如买台服务器，哪怕买个电脑，都有采购的时间。突然用户告诉某个云厂商，说想要开台电脑，如果使用物理服务器，当时去采购啊就很难，如果说供应商啊关系一般，可能采购一个月，供应商关系好的话也需要一个星期。用户等了一个星期后，这时候电脑才到位，用户还要登录上去开始慢慢部署自己的应用，时间灵活性非常差。第二是空间灵活性也不行，例如上述的用户，要一个很小很小的电脑，现在哪还有这么小型号的电脑啊。不能为了满足用户只要一个G的内存是80G硬盘的，就去买一个这么小的机器。但是如果买一个大的呢，因为电脑大，就向用户多收钱，用户说他只用这么小的一点，如果让用户多付钱就很冤。

2 虚拟化灵活多了
有人就想办法了。第一个办法就是虚拟化。用户不是只要一个很小的电脑么？数据中心的物理设备都很强大，我可以从物理的CPU，内存，硬盘中虚拟出一小块来给客户，同时也可以虚拟出一小块来给其他客户，每个客户都只能看到自己虚的那一小块，其实每个客户用的是整个大的设备上其中的一小块。虚拟化的技术能使得不同的客户的电脑看起来是隔离的，我看着好像这块盘就是我的，你看这呢这块盘就是你的，实际情况可能我这个10G和您这个10G是落在同样一个很大很大的这个存储上的。
而且如果事先物理设备都准备好，虚拟化软件虚拟出一个电脑是非常快的，基本上几分钟就能解决。所以在任何一个云上要创建一台电脑，一点几分钟就出来了，就是这个道理。

3 虚拟化的半自动和云计算的全自动
虚拟化软件似乎解决了灵活性问题，其实不全对。因为虚拟化软件一般创建一台虚拟的电脑，是需要人工指定这台虚拟电脑放在哪台物理机上的，可能还需要比较复杂的人工配置，所以使用Vmware的虚拟化软件，需要考一个很牛的证书，能拿到这个证书的人，薪资是相当的高，也可见复杂程度。所以仅仅凭虚拟化软件所能管理的物理机的集群规模都不是特别的大，一般在十几台，几十台，最多百台这么一个规模。这一方面会影响时间灵活性，虽然虚拟出一台电脑的时间很短，但是随着集群规模的扩大，人工配置的过程越来越复杂，越来越耗时。另一方面也影响空间灵活性，当用户数量多的时候，这点集群规模，还远达不到想要多少要多少的程度，很可能这点资源很快就用完了，还得去采购。所以随着集群的规模越来越大，基本都是千台起步，动辄上万台，甚至几十上百万台，如果去查一下BAT，包括网易，包括谷歌，亚马逊，服务器数目都大的吓人。这么多机器要靠人去选一个位置放这台虚拟化的电脑并做相应的配置，几乎是不可能的事情，还是需要机器去做这个事情。
人们发明了各种各样的算法来做这个事情，算法的名字叫做调度(Scheduler)。通俗一点的说，就是有一个调度中心，几千台机器都在一个池子里面，无论用户需要多少CPU，内存，硬盘的虚拟电脑，调度中心会自动在大池子里面找一个能够满足用户需求的地方，把虚拟电脑启动起来做好配置，用户就直接能用了。这个阶段，我们称为池化，或者云化，到了这个阶段，才可以称为云计算，在这之前都只能叫虚拟化。
```



## 4.搭建openstack环境

```
使用docker镜像创建一个ubuntu14.04
先从dockerhub上拉取镜像：dockr pull ansible/ubuntu14.04-ansible
运行 docker run -t -i ansible/ubuntu14.04-ansible /bin/bash （它的ip会默认的从172.17.0.2开始分配,就不使用书上推荐的192.168.104.x的网段了）
为了能让宿主机能够访问docker容器，环境说明：虚拟机为Centos7(桥接模式) ip:192.168.1.199，docker中容器的 网关：172.17.42.1，容器ip为：172.17.0.2 想要在宿主机（windows）访问它
只需要在主机上的cmd执行：ROUTE -p add 172.17.0.0 mask 255.255.0.0 192.168.1.199
即可

由于没有找到liberty分支，所以使用了rocky分支
git clone https://github.com/openstack-dev/devstack -b stable/rocky


```

