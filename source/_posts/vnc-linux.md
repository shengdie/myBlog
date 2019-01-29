---
title: Linux设置VNC服务器
date: 2019-01-29 13:38:10
categories:
tags:
toc:
mathjax:
---

最近配了一个主要作为服务器的电脑，但也时常用到图形软件，所以只有ssh不够用，Windows的远程桌面（RDP）不用自己设，默认开启的，效率也不错，但是Linux想搞一个远程桌面就不容易了，首先尝试了[xrdp](https://wiki.archlinux.org/index.php/xrdp)，因为rdp效率比较高一些，但是不能用GPU加速，需要OpenGL的软件都运行不了，虽然找到了一些tweak，但是设置麻烦，表现也不好，所以又将目标转向[VNC](https://zh.wikipedia.org/wiki/VNC)，但一般的vnc也是要配合[VirtualGL](https://www.virtualgl.org/)，一般情况还行，但是我用singularity包已经用了一层，所以再用一层就不知道能不能用了，最后发现了[x11vnc](http://www.karlrunge.com/x11vnc/)，它的原理就是将现有物理显示器的图像给传过来，所以本地能干的事，远程也都能干，可惜原作者已经停止开发，虽然community还再维护，但已经不会有太大的更新了，另外就是需要接一个物理显示器才行。

### Server设置
首先安装，不同Linux发行版不一样，关键词`x11vnc`自己装就行了。然后要实现远程登录，就得通过启动器，常见的有`gdm, lightdm, sddm`等， 区别就是auth文件不一样，这里给出`lightdm`和`sddm`的例子，为了开机启动，我们要写一个service，一般放在`/etc/systemd/system/`，注意调整。
先创建密码

```
sudo x11vnc -storepasswd [你的密码] /etc/x11vnc.pass
```

serivce文件，

{% codeblock x11vnc.service %}

[Unit]
Description="x11vnc"
After=multi-user.target

[Service]
ExecStart=/usr/bin/x11vnc -xkb -noxrecord -display :0 -auth /var/run/lightdm/root/:0 -rfbauth /etc/x11vnc.pass
ExecStop=/usr/bin/killall x11vnc

[Install]
WantedBy=multi-user.target
{% endcodeblock %}

`lightdm`用的`light-locker`有一个bug，每次锁屏会开一个新的display，这样的话，你再连就是黑屏，目前我是用`xscreensaver`代替`light-locker`。

对于不同的启动器只需要修改`-auth xxx`，目前我遇到的只有`sddm (KDE5)`比较特殊，每次启动，auth文件都是随机命名的，好在路径固定，我们只需要调用`sh`就行了

```
ExecStart=/bin/sh -c "/usr/bin/x11vnc -xkb -noxrecord -display :0 -auth /var/run/sddm/* -rfbauth /etc/x11vnc.pass"
```

然后enable并启动这个服务，
{% codeblock lang:bash %}
sudo systemctl daemon-reload
sudo systemctl enable x11vnc
sudo systemctl start x11vnc
{% endcodeblock %}

没有意外就应该可以成功启动了，客户端推荐[TightVNC](https://www.tightvnc.com/)。

### Reference
- https://askubuntu.com/questions/453109/add-fake-display-when-no-monitor-is-plugged-in



