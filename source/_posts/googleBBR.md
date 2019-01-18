---
title: BBR阻塞算法
date: 2017-05-08 21:46:45
categories: Linux
tags: VPS
toc: true
mathjax:
---

最近浏览网页，无意间看到了这个东西，大家都说这个东西是黑科技，可以大幅度提高带宽利用率，博主也忍不住试了一下。博主提供的方法只适合你能自己修改内核的情况，OpenVZ架构的VPS是不行的，不过OpenVZ也可以通过其他方法开启，有需要可以搜一下。

### 原理

Google的[github](https://github.com/google/bbr)上面给出了原理说明，不过是英文的，而且还注明这不是Google官方搞得，不过却在Google的github上。。。[知乎](https://www.zhihu.com/question/53559433)有一个别人写的中文版。当然博主是懒得看了。

### 安装

#### 升级内核

该算法已经整合到linux V4.9+的内核源码了，所以第一步就是把自己系统的内核升级到V4.9以上，升级内核有多种方法，你可以自己从源码编译，也可以下载编译好的包来安装，不过最方便莫过于从源安装，博主以Debian 8为例：

首先田间backports源，看[官方说明](https://backports.debian.org/Instructions/)。
然后安装新版内核（32bit系统包名称可能有区别）：
{% codeblock %}
apt-get install linux-image-amd64 -t jessie-backports
{% endcodeblock %}

#### 开启BBR

执行：
{% codeblock %}
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
{% endcodeblock %}

之后重启，执行`ss --tcp -i`，看看有没有bbr关键词，有的话表示成功开启了（至少有一个建立的网络连接才能看到）。



