## 一、概述

### 1、Instruction

Nginx 是一款高性能的 HTTP 服务器、反向代理服务器以及电子邮件（IMAP/POP3）代理服务器。

正向代理：就是当客户访问服务器资源时，用户知道自己想要请求的服务器是哪个，由用户发起请求，代理服务器去访问指定的网页，再由代理服务器将结果返回给用户。

反向代理：反向代理正好相反，用户请求某一资源时，用户并不清楚自己访问的是哪台服务器，只是将请求交给了反向代理服务器，这一切都由反向代理服务器去解决，最后用户只拿到自己想要的结果。

### 2、Install

[官网下载](http://nginx.org/en/download.html)

````shell
# 安装依赖
yum -y install gcc
yum install -y pcre pcre-devel
yum install -y zlib zlib-devel
yum install -y openssl openssl-devel

# 下载
curl -O http://nginx.org/download/nginx-1.20.1.tar.gz
# 解压
tar -zxvf nginx-1.20.1.tar.gz
cd nginx-1.20.1

# 配置
# 会生成 Makefile 文件，否则无法编译
# --prefix 指定安装目录，默认 /usr/local/nginx
./configure --prefix=/usr/local/nginx

# 编译 & 安装
make && make install

# 配置环境变量
vi /etc/profile

```
# Nginx
export PATH=$PATH:/usr/local/nginx/sbin
```

# 启动 Nginx
nginx

# 强制关闭（立即停止）
nginx -s stop

# 请求结束后停止
nginx -s quit

# 重启
nginx -s reload
````

> 访问 `127.0.0.1:80` ：

![Nginx_Welcome](Images/Nginx_Welcome.png)

注意：开启防火墙端口（否则无法访问）

```shell
# 开启防火墙端口
firewall-cmd --zone=public --add-port=80/tcp --permanent

# 查看防火墙端口列表
firewall-cmd --list-ports

# 重新加载配置
firewall-cmd --reload
```



### 3、添加到系统服务

> vi /etc/init.d/nginx

```shell
#!/bin/bash
# nginx Startup script for the Nginx HTTP Server
# it is v.0.0.2 version.
# chkconfig: - 85 15
# description: Nginx is a high-performance web and proxy server.
#              It has a lot of features, but it's not for everyone.
# processname: nginx
# pidfile: /usr/local/nginx/logs/nginx.pid
# config: /usr/local/nginx/conf/nginx.conf
nginxd=/usr/local/nginx/sbin/nginx
nginx_config=/usr/local/nginx/conf/nginx.conf
nginx_pid=/usr/local/nginx/logs/nginx.pid
RETVAL=0
prog="nginx"
# Source function library.
. /etc/rc.d/init.d/functions
# Source networking configuration.
. /etc/sysconfig/network
# Check that networking is up.
[ "${NETWORKING}" = "no" ] && exit 0
[ -x $nginxd ] || exit 0
# Start nginx daemons functions.
start() {
if [ -e $nginx_pid ];then
   echo "nginx already running...."
   exit 1
fi
   echo -n $"Starting $prog: "
   daemon $nginxd -c ${nginx_config}
   RETVAL=$?
   echo
   [ $RETVAL = 0 ] && touch /var/lock/subsys/nginx
   return $RETVAL
}
# Stop nginx daemons functions.
stop() {
        echo -n $"Stopping $prog: "
        killproc $nginxd
        RETVAL=$?
        echo
        [ $RETVAL = 0 ] && rm -f /var/lock/subsys/nginx /usr/local/nginx/logs/nginx.pid
}
# reload nginx service functions.
reload() {
    echo -n $"Reloading $prog: "
    #kill -HUP `cat ${nginx_pid}`
    killproc $nginxd -HUP
    RETVAL=$?
    echo
}
# See how we were called.
case "$1" in
start)
        start
        ;;
stop)
        stop
        ;;
reload)
        reload
        ;;
restart)
        stop
        start
        ;;
status)
        status $prog
        RETVAL=$?
        ;;
*)
        echo $"Usage: $prog {start|stop|restart|reload|status|help}"
        exit 1
esac
exit $RETVAL
```

设置权限

```
chmod 755 /etc/init.d/nginx
```

加入开启自启 

```
vi /etc/rc.local

# 在末尾增加一行
/usr/local/nginx/sbin/nginx
```

nginx 服务开机自启

```
chkconfig nginx on
```

管理 nginx 服务

```
systemctl status nginx
systemctl start nginx
systemctl stop nginx
systemctl reload nginx
systemctl restart nginx
```

### 4、负载均衡服务器

````shell
# 配置负载均衡
vi /usr/local/nginx/conf/nginx.conf

```
http {
    upstream name {  # 连接池，存放提供 web 服务的服务器地址
        server 192.168.56.102 weight=5;	# 一台web服务器地址，权重 5/6
        server 192.168.56.103 weight=1;	# 一台web服务器地址，权重 1/6
    }

    server {
        listen       80;
        server_name  localhost;

        localtion / {
            proxy_pass http://name;                       # 指定代理连接池
            proxy_set_header Host $host;                  # 转发请求头信息
            proxy_set_header X-Forward-For $remote_addr;  # 转发请求IP地址
        }
        
        location = /50x.html {
            root   html;
        }
    }
}
```

# 重启 Nginx
nginx -s reload
````
