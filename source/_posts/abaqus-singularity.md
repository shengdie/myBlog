---
title: Manjaro借助singularity安裝abaqus 2018
date: 2018-06-09 19:02:20
categories:
tags:
toc:
mathjax:
---

### 前言

最近博主在学习[Abaqus](https://www.3ds.com/zh/products-services/simulia/products/abaqus/)， 但是发现Abaqus只支持Windows，Suse，和Redhat，但是楼主习惯使用Manjaro，所以就想在Manjaro上面安装。在网上搜了半天，发现关于Linux版的安装教程实在是少的可怜，一搜基本都是在讲Windows平台，要不就是几百年的老版本怎么在ubuntu安装，关于最新版（2018），一点都没有，不过各种搜索之后，最终还是发现了一篇[神文](http://learningpatterns.me/posts-output/2018-01-30-abaqus-singularity/)，讲的是在arch上安装2017版，简直就是正中下怀，因为Manjaro就是arch based，区别不大。于是研究大半天，终于玩转2018版，特此记录，希望对大家有帮助。

### 安装前提

* Linux（理论上所有发行版都可以）
* root access
* Abaqus License服务器
* Abaqus安装盘（iso）
* 30G硬盘空间

### Singularity

首先，因为Abaqus不支持Manjaro，所以直接安装是不行的，理论可以修改原版的script来安装，之前很多老版本在ubuntu安装确实也是这样干的，但是这明显不是一个好的解决方式，不同的平台装起来不一样，所以就想到用Container，比如最出名的Docker，本质上是把软件和软件的依赖打包在一起，达到在任何linux发行版都能跑的效果（就是windows的安装包）。

说实话，之前没用过任何Container解决方式，也就听说过Docker，至于[Singularity](https://www.sylabs.io/)，是在神文里发现的，看了一下官方的说明


>Singularity is a container solution created by necessity for scientific and application driven workloads.

>Singularity also allows you to leverage the resources of whatever host you are on. This includes HPC interconnects, resource managers, file systems, GPUs and/or accelerators


也就是说Singularity本来就是为了高性能计算软件设计的，支持MPI，GPU加速等，又是正中下怀有没有，所以不需要犹豫，用起来。

安装很简单，尤其arch系更不需要担心，直接aur就行了，其他看[官方说明](https://www.sylabs.io/guides/2.6/user-guide/quick_start.html#installation)

### 创建Centos镜像，测试安装

我们要创建一个centos7的镜像，然后在里面安装Abaqus，安装之前一定要确认license server在运行，这里只列出简单的过程，更具体的信息，请到Singularity官网查找。

```
#创建镜像，直接从docker pull过来
sudo singularity build --sandbox centos docker://centos:7
#进入镜像，some/dir是解压后abaqus安装盘
sudo shell --bind some/dir:/iso-unpacked centos
#镜像内操作
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
yum update -y
yum install -y ksh csh which redhat-lsb-core perl
export NOLICENSECHECK=true
cd /iso-unpacked/
ksh ./1/StartTUI.sh
#然后跟着指示安装，安装完成莫退出镜像
```

当然这只是个测试，你退出来，所有的镜像修改都会消失，所以在退出之前，先找到所有文件名符合`*UserIntentions*`的文件，然后copy出镜像外。官方推荐直接从Recipe build镜像，这样镜像更纯净，不会有shell进去的垃圾，而且具有可复制性，更新也更方便。

*如果你不想再操作，或者你自信一次成功，不需测试，你可以在进入镜像时加入`--writable`来直接修改sandbox，安装成功就可以直接封装了。（极不推荐）*

### Singularity recipe
你在主机上得到`*UserIntentions*`文件后，都放在（新创建）一个`assets`文件夹，同时你要去[virtualgl](https://virtualgl.org/Downloads/YUM)下载`VirtualGL.repo`，也放在`assets`里面，然后就可以创建[recipe](https://www.sylabs.io/guides/2.6/user-guide/container_recipes.html)文件（`abaqus_centos.def`），下面是个例子

```
BootStrap: docker
From: centos:7

%setup
    mkdir -p ${SINGULARITY_ROOTFS}/iso-unpacked/
    bsdtar xf abaqus2018.iso -C ${SINGULARITY_ROOTFS}/iso-unpacked/
    mkdir -p ${SINGULARITY_ROOTFS}/cfgs/
    cp assets/VirtualGL.repo ${SINGULARITY_ROOTFS}/etc/yum.repos.d
    cp assets/UserIntentions_DOC.xml ${SINGULARITY_ROOTFS}/cfgs/
    cp assets/simulationservice_UserIntentions_CODE.xml ${SINGULARITY_ROOTFS}/cfgs/
    cp assets/cae_UserIntentions_CODE.xml ${SINGULARITY_ROOTFS}/cfgs/
    cp assets/UserIntentions_CAA.xml ${SINGULARITY_ROOTFS}/cfgs/

%post
    cd /iso-unpacked/
    rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
    yum update -y
    yum install -y ksh csh which redhat-lsb-core perl VirtualGL libjpeg-turbo
    yum clean all && rm -rf /var/cache/yum
    export NOLICENSECHECK=true
    ksh ./1/SIMULIA_Documentation/AllOS/1/StartTUI.sh --silent /cfgs/UserIntentions_DOC.xml
    ksh ./2/SIMULIA_AbaqusServices/Linux64/1/StartTUI.sh --silent /cfgs/simulationservice_UserIntentions_CODE.xml
    ksh ./2/SIMULIA_AbaqusServices_CAA_API/Linux64/1/StartTUI.sh --silent /cfgs/UserIntentions_CAA.xml
    ksh ./3/SIMULIA_Abaqus_CAE/Linux64/1/StartTUI.sh --silent /cfgs/cae_UserIntentions_CODE.xml
    rm -rf /iso-unpacked
    rm -rf /cfgs

%environment
    export PATH=/var/DassaultSystemes/SIMULIA/Commands:$PATH

%runscript
    vglrun abaqus cae

%labels
Maintainer qingtian
Version v0.1
```

注意`*UserIntentions*`文件改成你自己命名的，`export PATH=...`修改成你自定义abaqus commands安装目录。这样的话还有一个缺点，就是帮助文档打不开，因为找不到浏览器，这是singularity的一大缺陷，我已经提了issue，希望以后GUI软件可以直接调用host的浏览器，目前的解决方法是在镜像里再装一个firefox，只需要在上面加上firefox即可。

### 创建最终镜像

确认一下，目录结构

```
--Folder
   |--assets/
       |--*UserIntentions*
       |--VirtualGL.repo
   |--abaqus2018.iso
   |--abaqus_centos.def
```

然后在`Folder`里运行，

```
mkdir -p -m 0700 $HOME/.cache/singularity 
sudo SINGULARITY_TMPDIR=$HOME/.cache/singularity singularity build abaqus_2018_centos7.simg ./abaqus_centos.def
```

等一会就好了。你会得到`abaqus_2018_centos7.simg`，大约`3G`，你就可以运行了，

```
singularity run --nv abaqus_2018_centos7.simg
```

CAE出现，你就成功了。