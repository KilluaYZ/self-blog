---
title: 怎么在本地Docker上配置MPI计算集群
date: 2022-10-29 14:48:42
tags: 并行
---

最近我在上并行与分布式计算这门课，学着MPI，有不少实验需要在华为云上购买多台ECS才能做，在云上做实验有不少麻烦，最主要的就是我每次实验都要重配环境，虽然连续几个实验都与MPI有关，但是每个实验之间间隔较长，华为云ECS就算关机，甚至释放实例，保留云硬盘都会扣不少钱，~~让我薅不到学校免费券的羊毛~~，碰巧又有另一门课——程序设计安全的一个实验中用到了docker，实验内容大概就是在同一个网络下创建两个ubuntu的容器——server和client，用client黑server。这时我便想，能不能用docker的容器来模拟ECS，在本地搭建一个虚拟的计算集群呢？这样一来，我只用配置一次MPI环境，~~而且可以把学校发的券都用去买自己的服务器~~，废话不多说，直接开始！



## 下载安装docker

这一部分直接跳过，网上一搜一大片，写的都比我好。

最后能够在PowerShell里运行成功即可

![img1-create-container](http://43.138.62.72/data/blog-data/course/parallel/mpi-docker/img1-create-container.png)



## 创建网络和容器

### 创建网络

我们要创建一个网络将各个容器连接在一起。使用以下命令创建网络

```bash
docker network create mpi-test
```

创建完之后再用命令

```bash
docker network list
```

就可以看到创建的网络

![img2-create-container](http://43.138.62.72/data/blog-data/course/parallel/mpi-docker/img2-create-container.png)

### 创建容器

方便起见，我使用家喻户晓的ubuntu。首先将ubuntu镜像pull下来

![img3-create-container](http://43.138.62.72/data/blog-data/course/parallel/mpi-docker/img3-create-container.png)

```bash
docker pull ubuntu
```

再运行

```bash
docker images
```

![img4-create-container](http://43.138.62.72/data/blog-data/course/parallel/mpi-docker/img4-create-container.png)

可以看到，ubuntu的镜像已经被成功Pull到本地了

接下来，运行以下命令创建四个容器

```bash
docker create --name docker-hw-0001 --hostname docker-hw-0001 --network mpi-test -it ubuntu
docker create --name docker-hw-0002 --hostname docker-hw-0002 --network mpi-test -it ubuntu
docker create --name docker-hw-0003 --hostname docker-hw-0003 --network mpi-test -it ubuntu
docker create --name docker-hw-0004 --hostname docker-hw-0004 --network mpi-test -it ubuntu
```

然后到Docker Desktop里把对应的容器开启就好开启就好~

![img5-create-container](http://43.138.62.72/data/blog-data/course/parallel/mpi-docker/img5-create-container.png)

![img6-create-container](http://43.138.62.72/data/blog-data/course/parallel/mpi-docker/img6-create-container.png)

可以看到，我们的四台容器已经开始运行啦。可以选择从Docker Desktop里进入容器![img7-mpi-docker](http://43.138.62.72/data/blog-data/course/parallel/mpi-docker/img7-mpi-docker.png) 

也可以选择从powershell里进，还可以选择从vscode里进入，这里我就用vscode进入容器。在vscode里安装Dev Containers就可以用vscode访问容器了

![img8-mpi-vsc](http://43.138.62.72/data/blog-data/course/parallel/mpi-docker/img8-mpi-vsc.png)

![img9-mpi-vsc](http://43.138.62.72/data/blog-data/course/parallel/mpi-docker/img9-mpi-vsc.png)



## 配置MPI

配置MPI这部分主要参考了华为的实验操作手册，方便起见，全程都在root下配置。为了能够让容器之间互相通达，首先要配置一下sshd。以下配置，若无特殊说明，需要在四个容器中都配置一次

### 配置sshd

```bash
apt update
apt install -y openssh vim
```

安装完后

```bash
vim /etc/ssh/sshd_config
```

重点修改

```bash
PermitRootLogin yes
PubkeyAuthentication yes
AuthorizedKeysFile      .ssh/authorized_keys .ssh/authorized_keys2
AllowTcpForwarding yes
```

这些选项在sshd_config原文件中都有，只需要取消对应的注释，稍作修改就行

然后运行以下命令，重启ssh

```bash
service ssh restart
```



### 配置免密连接

输入以下命令，修改hosts

```bash
vim /etc/hosts
```

![img10-mpi-vsc](http://43.138.62.72/data/blog-data/course/parallel/mpi-docker/img10-mpi-vsc.png)



输入以下命令，生成密钥

```bash
ssh-keygen -t rsa -b 4096
```

将公钥添加到别的机器中，注意要一条一条输入

```bash
ssh-copy-id root@docker-hw-0001 
ssh-copy-id root@docker-hw-0002 
ssh-copy-id root@docker-hw-0003 
ssh-copy-id root@docker-hw-0004 
```



### 编译安装mpi

现在我们的系统环境里还缺少很多编译需要的东西，所以先要安装一下

```bash
apt install build-essential gfortran
```

有人可能会问，为什么用了gfortran，而不是实验说明上的gcc-gfortran？因为实验说明里用的系统是openeuler，包管理软件是yum，而ubuntu的apt库里没有，所以就安装了gfortran，但是这会导致一个问题，请看下文分解。

mpi安装包在实验程序包中都有，如果找不到了，也可以点击[这里](http://43.138.62.72/data/blog-data/course/parallel/mpi-docker/docker-mpi-src.zip)下载。用vscode的好处之一就是，给容器传文件非常简单方便，只需要动一动鼠标就好！在/root下建立目录mpi

![img11-mpi-vsc](http://43.138.62.72/data/blog-data/course/parallel/mpi-docker/img11-mpi-vsc.png)

将安装包mpich-3.3.2.tar.gz拖进去，进入到mpi目录里，并用以下命令解压

```bash
tar -zxvf mpich-3.3.2.tar.gz
```

![img12-mpi-vsc](http://43.138.62.72/data/blog-data/course/parallel/mpi-docker/img12-mpi-vsc.png)

现在可以进入到mpich-3.3.2中，进行configure、make、make install一通操作啦。

```bash
cd mpich-3.3.2
./configure
make && make install
```

不过要注意，在configure中可能会报错，报错内容是

```bash
checking whether gfortran allows mismatched arguments... no
configure: error: The Fortran compiler gfortran will not compile files that call
the same routine with arguments of different types.
```

这提示我们，gfotran和实验手册上的gcc-gfortran还是有些区别的，在网上冲浪一通后，用以下方式解决

```bash
./configure FFLAGS=-fallow-argument-mismatch FCFLAGS=-fallow-argument-mismatch
make && make install
```

这次安装成功！



## 测试环境

在四台机器中都导入实验说明给的文件，找不到了可以点击[这里](http://43.138.62.72/data/blog-data/course/parallel/mpi-docker/docker-mpi-src.zip)下载。我将其放在/root/hello下

![img13-test-vsc](http://43.138.62.72/data/blog-data/course/parallel/mpi-docker/img13-test-vsc.png)

修改config文件如下

![img14-test-vsc](http://43.138.62.72/data/blog-data/course/parallel/mpi-docker/img14-test-vsc.png)

并执行以下命令

```bash
make
```

最后，我选用docker-hw-0001来运行mpi_hello_world

```bash
mpiexec -n 8 -f /root/hello/config  /root/hello/mpi_hello_world
```

运行结果如下

![img15-test-vsc](http://43.138.62.72/data/blog-data/course/parallel/mpi-docker/img15-test-vsc.png)

可见四台容器虚拟出的小小计算集群，成功在本地跑起来啦！！！（完结撒花）~~现在可以省了不少钱，薅了一大波羊毛~~



但是这还没完，还有最后一点坑

- 重启容器可能会导致/etc/hosts文件被重置，所以每次开启后要检查，如果被重置了需要修改

- 重启容器后需要在每个容器里运行service ssh restart，才能互相通信



## 后记

这是我在做学校里作业的过程中遇到的问题解决方法和经验总结，其中有很多黑话不是同班同学可能听不懂，鉴于这篇文章最多也就班里看看，我就不做更多修改了！如有问题，欢迎联系！邮箱2959243019@qq.com

