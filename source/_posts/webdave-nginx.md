---
title: Debian用Nginx搭建Webdav server
date: 2016-10-25 02:24:37
categories: Linux
tags: VPS
toc: true
mathjax:
---
### 需求
坑爹的115现在登陆还要用手机号验证，然而博主之前的手机号早就不用了，换手机号必须人工申诉，竟然要身份证信息，果断放弃了，还有半年的VIP也浪费了。然后呢，问题来了，学校的烂网连国内尤其慢，而且本来带宽就不高，之前用115配合kodi在线看，速度还可以接受，没了115用城通，慢的不行，某度更是垃圾，对于博主这种没事就找个电影看看的人实在没法忍，于是突然想到可以利用VPS进行中转，先把电影下到VPS上，然后在线看，就不卡了。怎么在线看呢？用webdav.

### 要求
1. Nginx full或者Nginx + nginx-extras
2. htpasswd，用来做用户名+密码验证

### 步骤
#### 安装nginx并配置
{% codeblock %}
aptitude install nginx nginx-extras
mkdir -p /var/www/webdav/files # Webdav根目录
{% endcodeblock %}

创建`/etc/nginx/conf.d/webdav.conf`，并添加
{% codeblock %}
server {
    listen       8080; # 端口
    server_name  webdav.Your_domain; #域名
    root /var/www/webdav/files; # 根目录
    client_body_temp_path /var/www/webdav/tmp;
    access_log  /var/log/nginx/webdav_access.log;
    error_log   /var/log/nginx/webdav_error.log;
    location / {
      auth_basic "Not currently available";
      auth_basic_user_file /etc/nginx/conf.d/.htpasswd; # htpasswd验证文件

      dav_methods PUT DELETE MKCOL COPY MOVE;
      dav_ext_methods PROPFIND OPTIONS;

      create_full_put_path on;
      dav_access user:rw group:r;

      autoindex on; # 可以在浏览器打开

      limit_except GET {
        allow 115.115.0.0/32; # 允许访问的IP段，可以填自己家的IP，这样就只有你家的IP可以访问，大大增加安全性。
        deny  all;
      }
   }
}
{% endcodeblock %}

创建`.htpasswd`，这是简单密码验证，因为限制了IP，所以安全性足够了。
{% codeblock %}
htpasswd -c /etc/nginx/conf.d/.htpasswd UserName #自己用户名，然后输入密码
{% endcodeblock %}
#### 配置防火墙
如果你的防火墙默认放过所有包，可以忽略此步骤。
{% codeblock %}
iptables -A INPUT -p tcp -m tcp --dport 8080 -m state --state NEW -j ACCEPT
{% endcodeblock %}

如果想重启后防火墙依然生效，请看[这里](https://www.thomas-krenn.com/en/wiki/Saving_Iptables_Firewall_Rules_Permanently)。
然后`service nginx restart`应该可以跑起来了。在浏览器输入`http://webdav.Your_domain:8080`如果要求登陆就说明成了，只需要将文件放在你设置的webdav根目录就可以访问，也可以用支持webdav的客户端访问。
