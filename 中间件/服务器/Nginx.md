# Nginx基本概念

## 定义

Nginx（engine x）是一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务。Nginx是由伊戈尔·赛索耶夫为俄罗斯访问量第二的Rambler.ru站点开发的，因它的稳定性、丰富的功能集、简单的配置文件和低系统资源的消耗而闻名

Nginx是一款轻量级的web服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器，在BSD-like 协议下发行。其特点是在BSD-like 协议下发行。其特点是`占有内存小`，`并发能力强`，事实上Nginx的并发能力在同类型的网页服务器中表现较好

> Nginx专为性能优化而开发，性能是其最重要的考量，实现上非常注重效率，能经受高负载的考验，支持高达50000个并发连接数

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20211104165020.png" alt="image-20211104165016365" style="zoom:80%;" />

## 反向代理

### 正向代理

#### 定义

一般的访问流程是客户端直接访向目标服务器发送请求并获取内容，使用正向代理后，客户端改为向代理服务器发送请求，并指定目标服务器（原始服务器）然后有代理服务器和原始服务器通信，转交请求并获得内容，再返回给客户端。正向代理`隐藏了真实的客户端`，为客户端收发请求，使真实客户端对服务器不可见

> 举例：浏览器无法访问谷歌，这时候可以通过一个代理服务器来帮助你访问谷歌，这个服务器就叫正向代理	

### 反向代理

#### 定义

与一般流程相比，使用反向代理后，直接收到请求的服务器是代理服务器，然后将请求转发给内部网络上真正进行处理的服务器，得到的结果返回给客户端。反向代理隐藏了真实的服务器，为服务器收发请求，使真实的服务器对客户端不可见，一般在处理跨域请求的时候比较常用

> 举例：饭店吃饭，可以点川菜、粤菜、江浙菜，饭店也分别有三个菜系的厨师 👨‍🍳，但是你作为顾客不用管哪个厨师给你做的菜，只用点菜即可，小二将你菜单中的菜分配给不同的厨师来具体处理，那么这个小二就是反向代理服务器

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20211104165039.png" alt="image-20211104165036731" style="zoom:80%;" />

> 简单来说，一般给客户端做代理的都是正向代理，给服务器做代理的就是反向代理

### 使用反向代理的优点

> 反向代理服务器可以隐藏源服务器的存在和特征。它充当互联网云和web服务器之间的`中间层`。这对于安全方面来说是很好的，特别是当您使用web托管服务时

## 负载均衡

一般情况下，客户端发送多个请求到服务器，服务器处理请求，其中一部分可能要操作一些资源比如数据库、静态资源等，服务器处理完毕后，再将结果返给客户端

这种模式对于早起的系统来说，功能要求不复杂，且并发请求相对较少的情况下还能胜任，成本也低。随着信息数量不断增长，访问量和数据量飞速增长，以及系统业务复杂度持续增加，这种做法已无法满足要求，并发量特别大时，服务器容易崩

请求爆发式增长的情况下，单个机器性能再强劲也无法满足要求了，这个时候集群的概念产生了，单个服务器解决不了的问题，可以使用多个服务器，然后将请求分发到各个服务器上，将负载分发到不同的服务器上，这就是负载均衡，核心是「分摊压力」。Nginx 实现负载均衡，一般来说指的是将请求转发给服务器集群

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20211104165110.png" alt="image-20211104165106072" style="zoom:80%;" />

### 负载均衡算法怎么实现的

> 为了避免服务器崩溃，通过负载均衡的方式来分担服务器的压力，将多态服务器组成一个集群，当用户访问时，先访问到一个转发服务器，再由转发服务器将访问分发到压力刚小的服务器

### 负载均衡策略

#### 轮询

每个请求按时间顺序逐一分配到不同的后端服务器，如果后端某个服务器宕机，能自动剔除故障系统

```bash
upstream myserver {
	server 127.0.0.1:8080;
	server 127.0.0.1:8081;
}
```

#### 权重

- weight的值越大分配
- 到的访问概率越高，主要用于后端每台服务器性能不均衡的情况下。其次是为在主从的情况下设置不同的权值，达到合理有效的地利用主机资源。

```bash
upstream myserver {    
	server 127.0.0.1:8080 weight=8;    
	server 127.0.0.1:8081 weight=2;
}
```

- 权重越高，在被访问的概率越大，如上例，分别是20%，80%。

#### ip_hash(IP绑定)

每个请求按访问IP的哈希结果分配，使来自`同一个IP的访客固定访问一台后端服务器`，并且可以有效解决动态网页存在的session共享问题

```bash
upstream myserver {    
	ip_hash
	server 127.0.0.1:8080;    
	server 127.0.0.1:8081;
}
```

#### fair(第三方插件)

- 必须安装upstream_fair模块
- 对比 weight、ip_hash更加智能的负载均衡算法，fair算法可以根据页面大小和加载时间长短智能地进行负载均衡，`响应时间短的优先分配`
- 哪个服务器的响应速度快，就将请求分配到那个服务器上

```bash
upstream myserver {
	server 127.0.0.1:8080;
	server 127.0.0.1:8081;
	fair
}
```

#### url_hash(第三方插件)

- 必须安装Nginx的hash软件包
- 按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，可以进一步提高后端缓存服务器的效率

## 动静分离

为了加快网站的解析度，可以吧`动态页面和静态页面由不同的服务器来解析`，加快解析速度，降低原来单个服务器的压力

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20211105132452.png" alt="image-20211105132448574" style="zoom:80%;" />

一般来说，都需要将动态资源和静态资源分开，由于Nginx的高并发和静态资源缓存等特性，经常将静态资源部署在Nginx上，如果请求的是静态资源，直接到静态资源目录获取资源，如果是动态资源的请求，则利用反向代理的原理，把请求转发给对应后台应用去处理，从而实现动静分离

> 使用前后端分离后，可以很大程度提升静态资源的访问速度，即使动态服务不可用，静态资源的访问也不会受到影响。

## Nginx的优缺点

### 优点

1. 占用内存小，可以实现高并发连接，处理响应快
2. 可实现http服务器、虚拟主机、方向代理、负载均衡
3. Nginx配置简单
4. 可以不暴露真正的服务器IP地址

### 缺点

动态处理差：Nginx处理静态文件好，耗费内存少，但是处理动态页面很慢，现在一般前端用nginx作为反向代理抗住压力

## 应用场景

1. http服务器。Nginx是一个http服务可以独立提供http服务。可以做网页静态服务器
2. 虚拟主机。可以实现在一台服务器虚拟出多个网站，例如个人网站使用的虚拟机
3. 反向代理，负载均衡。当网站的访问量达到一定程度后，单台服务器不能满足用户的请求时，需要用多台服务器集群可以使用nginx做反向代理。并且多台服务器可以平均分担负载，不会应为某台服务器负载高宕机而某台服务器闲置的情况
4. nginx 中也可以配置安全管理、比如可以使用Nginx搭建API接口网关,对每个接口服务进行拦截
   

# Nginx安装、常用命令及配置文件

[Nginx下载地址](http://nginx.org/en/download.html)

## Nginx安装

> yum无法安装，缺少rpm包

```bash
rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```

然后使用yum安装

```bash
yum list | grep nginx
```

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20211105132940.png" alt="image-20211105132933116" style="zoom:80%;" />

```bash
# 安装
yum install nginx

# 查看安装版本
nginx - v
```

### 查看安装文件夹

```bash
rpm -ql nginx
```

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20211105134010.png" alt="image-20211105134006818" style="zoom:80%;" />

> /etc/nginx/conf.d/文件夹，是我们进行子配置的配置项存放处，/etc/nginx/nginx.conf主配置文件会默认把这个文件夹中所有子配置项都引入
>
> /usr/share/nginx/html/文件夹，通常静态文件都放在这个文件夹，也可以根据你自己的习惯放

### 通过ip地址和端口号访问nginx服务器

> 操作防火墙，开启80端口

```bash
systemctl start firewalld  # 开启防火墙
systemctl stop firewalld   # 关闭防火墙
systemctl status firewalld # 查看防火墙开启状态，显示running则是正在运行
firewall-cmd --reload      # 重启防火墙，永久打开端口需要reload一下
 
# 添加开启端口，--permanent表示永久打开，不加是临时打开重启之后失效
firewall-cmd --permanent --zone=public --add-port=8888/tcp
 
# 查看防火墙，添加的端口也可以看到
firewall-cmd --list-all
```

<img src="C:/Users/dell/AppData/Roaming/Typora/typora-user-images/image-20211107122112464.png" alt="image-20211107122112464" style="zoom:80%;" />

> 启动nginx

```bash
systemctl start nginx
```

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20211107103824.png" alt="image-20211107103820056" style="zoom:80%;" />

## Nginx常用命令

Nginx 的命令在控制台中输入 `nginx -h` 就可以看到完整的命令

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20211107105725.png" alt="image-20211107105721907" style="zoom:80%;" />

```bash
nginx -s reload  # 向主进程发送信号，重新加载配置文件，热重启
nginx -s reopen	 # 重启 Nginx
nginx -s stop    # 快速关闭
nginx -s quit    # 等待工作进程处理完成后关闭
nginx -T         # 查看当前 Nginx 最终的配置
nginx -t -c <配置路径>    # 检查配置是否有问题，如果已经在配置目录，则不需要-c
```

systemctl是linux系统应用管理工具systemd的主命令，主要用于管理系统，我们也可以用它来对Nginx进行管理

```bash
systemctl start nginx    # 启动 Nginx
systemctl stop nginx     # 停止 Nginx
systemctl restart nginx  # 重启 Nginx
systemctl reload nginx   # 重新加载 Nginx，用于修改配置后
systemctl enable nginx   # 设置开机启动 Nginx
systemctl disable nginx  # 关闭开机启动 Nginx
systemctl status nginx   # 查看 Nginx 运行状态
```

## Nginx配置文件

> Nginx的配置文件位置/etc/nginx/nginx.conf

结构图

```bash
main        # 全局配置，对全局生效
├── events  # 配置影响 Nginx 服务器或与用户的网络连接
├── http    # 配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置
│   ├── upstream # 配置后端服务器具体地址，负载均衡配置不可或缺的部分
│   ├── server   # 配置虚拟主机的相关参数，一个 http 块中可以有多个 server 块
│   ├── server
│   │   ├── location  # server 块可以包含多个 location 块，location 指令用于匹配 uri
│   │   ├── location
│   │   └── ...
│   └── ...
└── ...
```

典型配置

```bash
user  nginx;                        # 运行用户，默认即是nginx，可以不进行设置
worker_processes  1;                # Nginx 进程数，一般设置为和 CPU 核数一样
error_log  /var/log/nginx/error.log warn;   # Nginx 的错误日志存放目录
pid        /var/run/nginx.pid;      # Nginx 服务启动时的 pid 存放位置
 
events {
    use epoll;     # 使用epoll的I/O模型(如果你不知道Nginx该使用哪种轮询方法，会自动选择一个最适合你操作系统的)
    worker_connections 1024;   # 每个进程允许最大并发数
}
 
http {   # 配置使用最频繁的部分，代理、缓存、日志定义等绝大多数功能和第三方模块的配置都在这里设置
    # 设置日志模式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
 
    access_log  /var/log/nginx/access.log  main;   # Nginx访问日志存放位置
 
    sendfile            on;   # 开启高效传输模式
    tcp_nopush          on;   # 减少网络报文段的数量
    tcp_nodelay         on;
    keepalive_timeout   65;   # 保持连接的时间，也叫超时时间，单位秒
    types_hash_max_size 2048;
 
    include             /etc/nginx/mime.types;      # 文件扩展名与类型映射表
    default_type        application/octet-stream;   # 默认文件类型
 
    include /etc/nginx/conf.d/*.conf;   # 加载子配置项
    
    server {
    	listen       80;       # 配置监听的端口
    	server_name  localhost;    # 配置的域名
    	
    	location / {
    		root   /usr/share/nginx/html;  # 网站根目录
    		index  index.html index.htm;   # 默认首页文件
    		deny 172.168.22.11;   # 禁止访问的ip地址，可以为all
    		allow 172.168.33.44；# 允许访问的ip地址，可以为all
    	}
    	
    	error_page 500 502 503 504 /50x.html;  # 默认50x对应的访问页面
    	error_page 400 404 error.html;   # 同上
    }
}
```

一个 Nginx 配置文件的结构就像 nginx.conf 显示的那样，配置文件的语法规则：

- 配置文件由指令与指令块构成；
- 每条指令以 ; 分号结尾，指令与参数间以空格符号分隔；
- 指令块以 {} 大括号将多条指令组织在一起；
- include 语句允许组合多个配置文件以提升可维护性；
- 使用 # 符号添加注释，提高可读性；
- 使用 $ 符号使用变量；
- 部分指令的参数支持正则表达式；

### Nginx配置实例

#### 反向代理配置实例

##### 实例1

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20211107192357.png" alt="image-20211107192355705" style="zoom:80%;" />

linux中安装tomcat服务器，并且开启防火墙8080端口，访问测试

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20211107192303.png" alt="image-20211107192257738" style="zoom:80%;" />

本机配置域名

> C:\Windows\System32\drivers\etc\hosts

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20211107193100.png" alt="image-20211107193055150" style="zoom:80%;" />

nginx中配置代理对象

```bash
server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
			proxy_pass http://127.0.0.1:8080;
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
	
```

##### 实例2

<img src="C:/Users/dell/AppData/Roaming/Typora/typora-user-images/image-20211108143225174.png" alt="image-20211108143225174" style="zoom:80%;" />

nginx配置文件

```bash
		server {
        listen       18081 default_server;
        # listen       [::]:80 default_server;
        server_name  _反向代理;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

		# 正则表达式
        location ~ /edu/ {	
                        proxy_pass http://127.0.0.1:8080;
        }

        location ~ /vod/ {
                        proxy_pass http://127.0.0.1:8081;
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
```

#### 负载均衡配置实例

两台tomcat8080和8081webapps下面都包含/edu/a.html通过nginx监听80端口负载均衡到8080和8081端口

```bash
	upstream myserver {
        server 127.0.0.1:8080;
        server 127.0.0.1:8081;
    }
    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
                        proxy_pass http://myserver;
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```

#### 动静分离实例

两种方式

- 纯粹把静态文件独立成单独的域名，放在独立的服务器上，也是主流推崇的方案
- 把动态和静态文件混合一起发布，铜鼓Nginx配置分开

通过 location 指定不同的后缀名实现不同的请求转发。通过 expires 参数设置，可以使浏览器缓存过期时间，减少与服务器之前的请求和流量。具体 expires 定义：是给一个资源设定一个过期时间，也就是说无需去服务端验证，直接通过浏览器自身确认是否过期即可，所以不会产生额外的流量。此种方法非常适合不经常变动的资源。（如果经常更新的文件，不建议使用 expires 来缓存），我这里设置 3d，表示在这 3 天之内访问这个URL，发送一个请求，比对服务器该文件最后更新时间没有变化。则不会从服务器抓取，返回状态码 304，如果有修改，则直接从服务器重新下载，返回状态码 200

```bash
location /www/ {
	root /data/;
}
                
location /images/ {
	root /data/;
	autoindex on; # 列出文件夹中的内容
} 
```

#### 高可用配置实例

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20211108174509.png" alt="image-20211108174501415" style="zoom:80%;" />

步骤：

1. 远程阿里云服务器部署Nginx1服务器和两台tomcat服务器tomcat18080和tomcat8081

2. 本地linuxe虚拟机配置Nginx2服务器

3. 通过负载均衡实例将两台服务器80端口负载均衡接在两天tomcat上

4. 在两台服务器上安装keepalived

5. 安装keepalived

   ```bash
   yum  install keepalived -y
   ```

6. 更改/etc/keepalived/下的配置文件keepalived.conf，并在vrrp_script中定义一个外围检测机制，并在vrrp_instanceVI_1中定义track_script来追踪脚本的执行过程，实现节点转移

   ```bash
   global_defs{
      notification_email {
           acassen@firewall.loc
      }
      notification_email_from Alexandre@firewall.loc
      smtp_server 127.0.0.1
      smtp_connect_timeout 30 // 上面都是邮件配置
      router_id LVS_DEVEL     // 当前服务器名字，用hostname命令来查看
   }
   vrrp_script chk_maintainace { // 检测机制的脚本名称为chk_maintainace
       // script "[[ -e/etc/keepalived/down ]] && exit 1 || exit 0" // 可以是脚本路径或脚本命令
       script "/usr/local/src/nginx_check.sh"    // 比如这样的脚本路径
       interval 2  // 每隔2秒检测一次
       weight 2  // 当脚本执行成立，那么把当前服务器优先级改为-20
   }
   vrrp_instanceVI_1 {   // 每一个vrrp_instance就是定义一个虚拟路由器
       state MASTER      // 主机为MASTER，备用机为BACKUP
       interface ens33    // 网卡名字，可以从ifconfig中查找
       virtual_router_id 51 // 虚拟路由的id号，一般小于255，主备机id需要一样
       priority 100      // 优先级，master的优先级比backup的大
       advert_int 1      // 默认心跳间隔
       authentication {  // 认证机制
           auth_type PASS
           auth_pass 1111   // 密码
       }
       virtual_ipaddress {  // 虚拟地址vip
          192.168.19.135
       }
   }
   ```

   在/usr/local/src下的检测脚本 `nginx_check.sh`

   ```bash
   #!/bin/bash
   A=`ps -C nginx --no-header | wc -l`
   if [ $A -eq 0 ];then
       /usr/sbin/nginx # 尝试重新启动nginx
       sleep 2         # 睡眠2秒
       if [ `ps -C nginx --no-header | wc -l` -eq 0 ];then
           killall keepalived # 启动失败，将keepalived服务杀死。将vip漂移到其它备份节点
       fi
   fi
   ```

7. 启动两台linux上的nginx和keepalived

8. 测试，关掉主nginx服务器测试

# Nginx执行原理

## Reactor模型

> 与java底层通信框架Netty在原理上有很多相似之处

Nginx对高并发IO的处理使用了Reactor事件驱动模型。Reactor模型的接班组件包括事件收集器、事件发送器、事件处理器3个基本单元，其核心思想是将所有要处理的I/O事件注册到一个I/O多路复用器上，同时主线程/进程阻塞在多路复用器上，一旦有I/O事件到来或者准备就绪（文件描述符或Socket可读、写），多路复用器返回并将事件先注册到响应的I/O事件分发到对应的处理器中

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20211109120809.png" alt="image-20211109120805171" style="zoom:80%;" />

- 事件收集器：负责手机Worker进程的各种I/O请求
- 事件发送器：负责将I/O事件发送到事件处理器
- 事件处理器：负责各种事件的响应工作

事件收集器将各个连接通道的IO事件放入一个待处理时间列，通过事件发送器发送给对应的事件进行处理。而事件收集器之所以可以同时管理上百万个连接通过的是事件，是基于操作系统提供的“多路IO复用”技术，常见的包括select、epoll两种模型

## Nginx的两类进程

一般来说，Nginx在启动后会以daemon方式在后台运行，其后台进程有两类：一类称为master进程（相当于管理进程），另一类称为Worker进程（工作进程）

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20211109121433.png" alt="image-20211109121427172" style="zoom:80%;" />

- Master管理进程主要负责调度Worker工作进程，比如加载配置、启动工作进程、接受外界信号、向各Worker工作进程发送信号、监控Worker进程的运行状态等
- Master负责创建监听套接口，交由Worker进程进行连接监听
- Worker进程主要用来处理网络事件，当一个Worker进程接受到一条连接通道之后，就开始读取请求、解析请求、处理请求，处理完成产生数据后，返回给客户端，最后断开连接通道
- 各个Worker进程之间是对等且独立的，他们同等`竞争`来自客户端的请求，一个请求只可能在一个Worker进程中处理（典型的Reactor模型中Worker进程的只能）
- 如果启动了多个Worker进程，那么每个Worker子进程独自尝试接受已连接的Socket监听通道，accept操作默认上锁，优先使用操作系统的共享内存原子锁，如果操作系统不支持，使用文件上锁



<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20211109122418.png" alt="image-20211109122414042" style="zoom:80%;" />

Nginx启动两种方式：

- 单进程启动（调试）：此时系统中仅有一个进程，该进程既充当Master管理进程角色，也充当Worker工作进程角色
- 多进程启动（生产）：此时系统有且仅有一个Master进程，至少有一个Worker工作进程（工作进程数量和机器CPU核数配置不一样）

# 限流原理和实战

> 通信领域，限流技术被用来控制网络接口收发通信数据的速率，实现通信时的优化性能、较少的延迟和提高带宽

举例：假设某接口能抗住的QPS为10000，这时有20000个请求进来，经过限流模块，会先放10000个请求，其余的请求会阻塞一段时间。不简单粗暴地返回404，让客户端重试，同时又能起到流量削峰的作用

接口限流算法：计数器限流、漏桶算法、令牌桶算法

# Nginx面试题

## 什么是Nginx

Nginx是一个轻量级、高性能的反向代理Web服务器，他实现非常高效的反向代理、负载均衡，他可以处理2-3万并发连接数，官方检测支持5万并发

## 为什么要用Nginx

- 跨平台、配置简单、方向代理、高并发连接，内存消耗小：开启10个Nginx才占150M内存，Nginx处理静态文件好，耗费内存少
- Nginx含有内置的健康检查功能：如果有一个服务器宕机，会做一个健康检查，再发送的请求就不会发送到宕机的服务器了。重新将请求提交到其他节点
- 其他特性：
  - 节省带宽：支持GZIP压缩，可以添加浏览器本地缓存
  - 稳定性高：宕机概率非常小
  - 接受用户请求是`异步`的

## 为什么Nginx性能高

因为他的事件处理机制：异步非阻塞事件处理机制：运用epoll模型，提供一个队列，排队解决

## Nginx怎么处理请求

Nginx接受一个请求后，首先有listen和server_name指令匹配server模块，再匹配server模块里的location，location就是实际地址

## 正向代理和反向代理

- 正向代理：就是一个人发送一个请求直接就到达了目标服务器
- 反向代理：请求统一被Nginx接受，Nginx反向代理服务器接受到之后，按照一定的规则分发给了后端的业务处理服务器进行处理

## 如何用Nginx解决前端跨域问题

### 什么是跨域？

> 在浏览器上当前访问的网站向另一个网站发送请求获取数据的过程就是**跨域请求**

浏览器从一个域名的网页去请求另一个域名的资源时，域名、端口、协议任一不同，都是跨域

### 如何解决跨域问题？

#### jsonp跨域

JSONP（JSON with Padding，填充式JSON），应用JSON的一种新方法

与JSON的区别：

- JSON返回的是一串数据，JSONP返回的是脚本代码（包含一个函数调用）
- JSONP只支持get请求、不支持post请求

> 类似往页面添加一个script标签，通过src属性去触发对指定地址的请求，故只能是get请求

#### Nginx反向代理

使用Nginx转发请求，把跨域的接口写成本域的接口，然后将这些接口转发到真正请求的地址

www.baidu.com/index.html需要调用www.sina.com/server.php，可以写成一个接口www.baidu.com/server.php，由这个接口在后端调用www.sina.com/server.php并拿到返回值，然后返回给index.html

#### PHP端修改header

- header(‘Access-Control-Allow-Origin:*’);//允许所有来源访问
- header(‘Access-Control-Allow-Method:POST,GET’);//允许访问的方式



参考文献：

[Nginx 从入门到实践(超级详细)](https://blog.csdn.net/Janson_Lin/article/details/105954705?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163651740816780262594616%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=163651740816780262594616&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-2-105954705.pc_search_result_cache&utm_term=Nginx&spm=1018.2226.3001.4187)

[Nginx面试题（总结最全面的面试题！！！）](https://blog.csdn.net/weixin_43122090/article/details/105461971?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163651751316780274129840%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=163651751316780274129840&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-1-105461971.pc_search_result_cache&utm_term=Nginx%E9%9D%A2%E8%AF%95%E9%A2%98&spm=1018.2226.3001.4187)

SpringCloud、Nginx高并发核心编程