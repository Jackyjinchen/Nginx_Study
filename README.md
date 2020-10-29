# Nginx

高性能Web服务器和反向代理服务器

其他：Apache、Lighttpd、Tengine(阿里巴巴)......

## 正向代理

隐藏了真实的请求客户端，服务端不知道真实的客户端是谁，客户端请求的服务都由代理服务器代替来请求

![img](README.assets/v2-922c89b735165f9f47db025bd77941c2_1440w.png)

## 反向代理

反向代理隐藏了真实的服务端，当我们请求的时候，反向代理服务器会帮我们把请求转发到真实的服务器那里去。

![img](README.assets/v2-bf5255d84d73d10500cb9c717fac8b02_1440w.png)

## 裸机安装

安装依赖库

```shell
yum install gcc openssl openssl-devel pcre pcre-devel zlib zlib-devel -y
```

正式安装

```shell
#解压缩
tar -zxvf nginx-xxxx.tar.gz
cd nginx-xxxx
#执行configure文件
./configure --prefix=/usr/local/nginx
#编译构建
make
make install
```

## Nginx服务

### 目录结构

| 安装目录/usr/local/nginx                   |
| ------------------------------------------ |
| conf：配置文件--nginx.conf                 |
| html：50x.html错误地址，index.html欢迎页面 |
| logs：日志                                 |
| sbin：nginx启动命令                        |

### 启动

默认端口80，直接访问即可

```shell
#直接启动
/usr/local/nginx/sbin/nginx
#通过配置文件启动
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```

### 停止

```shell
ps -ef | grep nginx
# 优雅关闭，处理完请求后再关闭
Kill -QUIT 主pid
# 快速关闭，直接关闭所有请求
kill -TERM 主pid
```

### 重启

```shell
./nginx -s reload
```

### 配置检查

```shell
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf -t
# -t 若错误会报错
```



## Nginx配置文件

具体注解见nginx.conf



## Nginx应用

静态网站部署
负载均衡
静态代理
动静分离
虚拟主机

### 静态网站部署

```conf
# 路径/opt/www/ace/login.html
location /ace {
    root  /opt/www
    index login.html
}
```

### 负载均衡

**硬件负载均衡**：F5、深信服、Array等

**软件负载均衡**：Nginx、LVS、HAProxy

#### nginx负载均衡策略：

1. 默认轮询模式及权重模式

```conf
# 服务器端口地址 weight表示权重比
upstream backserver {
    server 192.168.0.1:8081 weight=3;
    server 192.168.0.1:8082 weight=1;
}
# 请求分发
location /myweb {
    proxy_pass  http://backserver
}
```

2. ip_hash

IP绑定，根据访问ip的hash值分配，每个访问客户会固定一个访问后端服务器，解决会话session丢失的问题。（动态ip无法解决）

```conf
upstream backserver {
    ip_hash;
    server 192.168.0.1:8081 weight=3;
    server 192.168.0.1:8082 weight=1;
}
```

3. 最少连接数

```conf
upstream backserver {
    least_conn;
    server 192.168.0.1:8081;
    server 192.168.0.1:8082;
}
```

#### 负载均衡的其他配置：

1. 备份服务：可用于服务更新

```conf
upstream backserver {
    server 192.168.0.1:8081;
    # 此服务器为备份服务器，当其他服务器均宕机后启用
    server 192.168.0.1:8082 backup;
}
```

2. 不参与负载均衡

```conf
upstream backserver {
    server 192.168.0.1:8081;
    # 此服务器down状态，不参与负载均衡
    server 192.168.0.1:8082 down;
}
```

### 静态代理

将静态资源从tomcat中抽离出来，nginx更擅长静态资源处理，性能效率更高。

1. 拦截扩展名

静态资源目录权限需要足够，否则会出现403错误，chmod 755

```conf
location ~ .*\.(js|css|htm|html|gif|jpg|jpeg|png|bmp|swf|ioc|rar|zip|txt|flv|mid|doc|ppt|pdf|xls|mp3|wma)${
    root /opt/static;
}
# ~正则匹配
# .表示任意字符
# *表示一个或多个字符
# \.是转义字符，是后面这个点的转义
# |表示或者
# $表示结尾
```

2. 拦截路径

```conf
location ~ .*/(css|js|img|images){
    root  /opt/static;
}
```

### 动静分离

```shell
#tomcat启动
./startup.sh | tail -f ../logs/catalina.out &
```

通过proxy进行负载均衡（针对丢失session问题，采用ip_hash方式），访问前端服务（静态资源通过nginx拦截访问），并调用后端服务。

<img src="README.assets/image-20201029111822388.png" alt="image-20201029111822388" style="zoom:33%;" />

### 虚拟主机

1. 基于端口的虚拟主机(较少采用)

```conf
server {
		#采用不同的端口
    listen 8080
    server_name www.myweb.com
    location /myweb {
        proxy_pass http://www.myweb.com
    }
}
```

2. 基于域名的虚拟主机

```conf
server {
    listen 80
    #采用相同的80端口 dns绑定不同的域名访问相同的服务
    server_name www.myweb.com
    location /myweb {
        proxy_pass http://www.myweb.com
    }
}
```



