---
title: CentOS 6 搭建Hexo
date: 2016-10-09 02:29:59
categories: 网站
tags:
 - Hexo
 - VPS
toc: true
---
### 为什么用Hexo
其实博主也只是新手，最近刚买了[BandwagonHost](http://bandwagonhost.com "BandwagonHost")便宜VPS，只搭个VPN实在是太浪费，于是便想着搭个博客，在网上搜了下，便发现了[Hexo](https://hexo.io "Hexo")，主要看重<font color=#00BFFF>Hexo可以配合git实现本地更新Blog</font>，这样就不用在总是在VPS上操作了，而且方便备份和迁移，因为你所有源码都在本地，换服务器只需要备份几个配置文件而已。另外Hexo是中国人开发的，也算支持一下国产。

### 博主VPS配置和建站前提
- CPU：1 core
- RAM：512 MB
- Bandwidth：1000 GB
- OS：CentOS 6 x86-64

你要申请一个域名备用。如果你只是练手，或者不要求博客流量，你可以去[Freenom](http://www.freenom.com/ "Freenom")申请免费域名，就像博主的`.ml`域名。如果你希望更多的人看到你的博客，推荐你去[Godaddy](https://www.godaddy.com/ "Godaddy")买一个`.com .org`等常见的顶级域名，因为免费域名垃圾站太多，所以搜索引擎一般不收录或给的权重很低。

### 思路
- 在VPS安装Nginx和Git，搭建Git服务器。
- 在本地主机安装Hexo，Git，Node.js。
- 本地利用Hexo生成静态文件，推送至VPS的Git仓库，由VPS的Nginx解析。

### 本机操作
#### 本机安装软件
以MacOS为例，到[Node.js](https://nodejs.org "Nodejs")官网下载安装Node.js，4.6稳定版便可，Git推荐使用[Homebrew](http://brew.sh "Homebrew")安装，具体去官方看吧。Hexo默认安装到`/usr/local`，而npm global包默认安装到`/usr/local/lib`，这在最新的Mac Sierra会出现问题，因为用户没有了`/usr/local`的权限，所以最好改变global包的安装位置，
{% codeblock %}
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
{% endcodeblock %}

再安装Hexo，
{% codeblock %}
npm install -g hexo-cli
{% endcodeblock %}

然后在`~/.bash_profile`加入
{% codeblock %}
export PATH=~/.npm-global/bin:$PATH
{% endcodeblock %}

然后`source ~/.bash_profile`，接着初始化博客
{% codeblock %}
mkdir myBlog && cd myBlog
hexo init
npm install
npm install hexo-deployer-git --save
hexo g
hexo s #启动本地服务器用于测试
{% endcodeblock %}

这时在浏览器输入<http://localhost:4000>，应该可以看到博客了。
#### 本机配置Git和SSH
{% codeblock %}
git config --global user.email "Your Email"
git config --global user.name " Your Username"
{% endcodeblock %}
其中“Your Username“是待会要在VPS加的用户。然后生成ssh key，
{% codeblock %}
ssh-keygen -t rsa -C "You Emial or other information"
{% endcodeblock %}
可以一路回车，最后生成`id_rsa`和`id_rsa.pub`文件，其中`id_rsa.pub`里是公钥，待会要传给VPS. 然后如果你VPS的默认端口如果不是`22`那就需要配置ssh端口，新建`~/.ssh/config`文件，填入以下代码：
{% codeblock %}
Host VPS的IP
User 你之前设的Username
Port SSH端口
IdentityFile ~/.ssh/id_rsa
{% endcodeblock %}
如果在VPS上有多用户可以生成多个ssh keys，并在config加入，具体方法参考[这儿](http://riny.net/2014/git-ssh-key/ "ssh-key").
### VPS操作
#### VPS安装Git和Nginx
{% codeblock %}
yum install epel-release # centos源里没有nginx，所以加上EPEL的repo
yum install git-core nginx -y
{% endcodeblock %}
#### VPS新建用户
{% codeblock %}
adduser Your Username #之前你设置的
chmod 740 /etc/sudoers
nano /etc/sudoers
{% endcodeblock %}
nano中找到
{% codeblock %}
## Allow root to run any commands anywhere
root    ALL=(ALL)     ALL
{% endcodeblock %}
在下面添加
{% codeblock %}
Your username	All=(ALL)	ALL
{% endcodeblock %}
恢复权限
{% codeblock %}
chmod 440 /etc/sudoers
{% endcodeblock %}
#### 创建网站目录并设置所有权
{% codeblock %}
mkdir -p /var/www/hexo
chown Your username:Your username -R /var/www/hexo
{% endcodeblock %}
#### VPS创建Git仓库，配置SSH
{% codeblock %}
su Your username
cd ~/
mkdir .ssh
nano .ssh/authorized_keys #在此文件黏贴本机生成的id_rsa.pub的里的内容
chmod -R 755 .ssh/  # 很重要！确认只有用户拥有写权限
mkdir hexo.git && cd hexo.git
git init --bare
{% endcodeblock %}
#### 配置Git hooks
{% codeblock %}
nano ~/hexo.git/hooks/post-receive
{% endcodeblock %}
输入以下内容
{% codeblock %}
#!/bin/bash
GIT_REPO=/home/Your username/hexo.git #git仓库
TMP_GIT_CLONE=/tmp/hexo
PUBLIC_WWW=/var/www/hexo #网站目录
rm -rf ${TMP_GIT_CLONE}
git clone $GIT_REPO $TMP_GIT_CLONE
rm -rf ${PUBLIC_WWW}/*
cp -rf ${TMP_GIT_CLONE}/* ${PUBLIC_WWW}
{% endcodeblock %}
赋予执行权限
{% codeblock %}
chmod +x post-receive
{% endcodeblock %}
#### 配置Nginx
创建`/etc/nginx/conf.d/hexo.conf`并输入
{% codeblock %}
server {
    listen         80 ;
    root /var/www/hexo; #你的网站目录，本例子在/var/www/hexo
    server_name example.com www.example.com; #这里输入你的域名或IP地址
    access_log  /var/log/nginx/hexo_access.log;
    error_log   /var/log/nginx/hexo_error.log;
    location ~* ^.+\.(ico|gif|jpg|jpeg|png)$ {
            root /var/www/hexo;
            access_log   off;
            expires      1d;
    }
    location ~* ^.+\.(css|js|txt|xml|swf|wav)$ {
        root /var/www/hexo;
        access_log   off;
        expires      10m;
    }
    location / {
        root /var/www/hexo; #你的网站目录地址
        if (-f $request_filename) {
            rewrite ^/(.*)$  /$1 break;
        }
    }
}
{% endcodeblock %}
重启`nginx`
{% codeblock %}
service nginx restart
{% endcodeblock %}
### 本机剩余工作
修改hexo根目录下的配置文件`_config.yml`，修改`deploy`选项，
{% codeblock %}
deploy:
  type: git
  message: update
  repo:
    s1: Your username@VPS的ip地址或域名:git仓库地址,master
    #如
    s1: xxx@123.456.789.111:hexo.git,master
{% endcodeblock %}
至此，搭建基本完成，只需执行
{% codeblock %}
hexo g
hexo d
{% endcodeblock %}
便可推送到VPS. 如果报错，请检查以上步骤，至于博客怎么写，可以参考官网，以后我也会更新一些东西。

---
<i> Hint: 在本机上，因为经常要进博客根目录（`myBlog`），为了方便，可以在`~/.bash_profile`里加入`alias blg="cd ~/myBlog"`，这样以后直接打`blg`就可以进入博客根目录了。</i>

---

### 参考链接
1. <http://tiktoking.github.io/2016/01/26/hexo/>
2. <http://www.hansoncoder.com/2016/03/02/VPS-building-Hexo/>

