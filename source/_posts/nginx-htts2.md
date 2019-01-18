---
title: Nginx启用HTTPS和HTTP/2
date: 2016-10-27 21:45:43
categories: 网站
tags: VPS
toc: true
mathjax:
---
### 为什么要用HTTPS
HTTP直接用明文传输，所以非常不安全，在传输过程中数据很容易被窥探甚至串改，比如国内某些网络运营商就经常靠这种方式推送广告，特别烦人。为了防止这个问题，便有了SSL，原理简单的说就是，服务器和客户端之间传输数据前先进行加密，到了对方那再进行解密，那么在传输的过程中，数据都是加密，就能防止别人的窥测了。至于具体的加密过程可以看[这儿](http://cjting.me/web2.0/2016-09-05-从零开始搭建一个HTTPS网站.html)。
### CA的证书
如果详细了解了加密的过程，就知道想用HTTPS就要一个CA（Certificate Authority, 权威机构）的证书，这个证书一般都是要购买的，而且一般都很贵，虽然也有便宜的，但像博主这种博客几乎没人看，域名都是免费的，又何必花钱买个证书，所以就用免费的吧。俗话说便宜没好货，一般免费的CA证书也有各种问题，在这里推荐使用[Let's Encrypt](https://letsencrypt.org/)，来自Internet Security Research Group，算是个公益组织，其目的就是让大家都用上免费的HTTPS，先赞一个。
### 具体步骤
首先说一下，博主的VPS是Debian 8，如果是其他版本的Linux要注意。
#### 安装客户端
Let's Encrypt之前的官方客户端就叫`letsencrypt`，但是最近改名了，叫`certbot`. 要在Debian 8安装先要加backports源。具体看[官方教程](https://backports.debian.org/Instructions/)。
{% codeblock %}
aptitude install certbot -t jessie-backports
{% endcodeblock %}

注意一下，在这自动安装`crontab`，并在`/etc/cron.d/certbot`里加入了一天`renew`两次的命令。因为生成的证书有效期只有三个月，所以要经常续签，官方便做了一个定期任务。
#### 生成证书
{% codeblock %}
certbot certonly --webroot -w /var/www/example -d example.com -d xxx.example.com
{% endcodeblock %}
期间会让你输入email做紧急联系方式。

其中`/var/www/example`是你网站的根目录，`example.com`和`xxx.example.com`是你的域名，注意Let's Encrypt不支持Wildcard，也就是泛解析`*.example.com`，所以每一个次域名你都要加进去。
如果你有两个独立的server，比如`example.com`和`www.example.com`是你的主站，`...main_webroot/`是你主站的根目录，另外又有`xxx.example.com`（也可以是顶级域名）是你的其他站，比如博客或者什么的，其根目录是`...xxx_webroot/`，那么你一样可以只用一个证书，
{% codeblock %}
certbot certonly --webroot -w ...main_webroot/ -d example.com -d www.example.com  -w ...xxx_webroot/ -d xxx.example.com
{% endcodeblock %}

注意，之所以要指定根目录，是因为`certbot`要在根目录下建一个隐藏的`.well-known/`，用来检测域名确实是你所有，所以要注意Nginx的设置不能把隐藏文件作为特殊文件处理。
不出意外，这个时候你能在`/etc/letsencrypt/live/example.com/`下找到证书了。
#### 配置Nginx
修改你原来的nginx配置文件，
{% codeblock %}
server {
    listen	   80;
    server_name example.com www.example.com;
    return 301 https://$server_name$request_uri; # 重定向到https
}
server {
    listen         443 ssl;
    root /var/www/example; # 网站根目录
    ssl_prefer_server_ciphers on;
    ssl_ciphers EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout  1d;
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    
    # 其余的配置
    ...
}
{% endcodeblock %}
#### 开启HTTP/2
HTTP/2有很多优势，可以看[这儿](https://www.qianduan.net/a-simple-performance-comparison-of-https-spdy-and-http2/)，所以可以开启了，不过要注意，Nginx只有1.9.5之上的版本支持HTTP/2。Debian 8下开以安装backports版的Nginx。
开启很简单，只需要在上面的配置`443 ssl;`后面添加`http2`变成`443 ssl http2;`即可。
然后`systemctl restart nginx`重启服务。

#### 配置防火墙
如果你防火墙默认放过所有，跳过此步骤。
{% codeblock %}
iptables -A INPUT -p tcp -m tcp --dport 443 -m state --state NEW -j ACCEPT
{% endcodeblock %}
然后用你自己的方法save一下保证重启不失效。
#### 配置crontab
前面说了，证书三个月到期，所以要定期刷新，官方已经给你建好了任务文件，只需要指向它就可以了。
{% codeblock %}
crontab /etc/cron.d/certbot
{% endcodeblock %}

至此你再打开你的网站，如果前面显示绿锁就OK了，然后你可以到<https://tools.keycdn.com/http2-test>测试你的网站有没有成功开启HTTP/2。

### 后续处理
然后记得，去[Google Search Console](https://www.google.com/webmasters/tools/)添加HTTPS，去[百度站长](http://zhanzhang.baidu.com)设置HTTPS，如果你用了LeanCloud计数，还要去[LeanCloud](https://leancloud.cn/)修改域名绑定。

### 参考链接
1. <https://certbot.eff.org/#debianjessie-nginx>
2. <https://blog.ishell.me/a/Lets-Encrypt-and-HTTP2.html>
