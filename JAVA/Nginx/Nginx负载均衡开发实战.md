# Nginx

[TOC]



## 1 介绍

### 1.1 为什么需要Nginx

并发量小时

![image-20210128113726952](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210128113726952.png)

并发量大时

![image-20210128113801035](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210128113801035.png)

于是需要横向扩展，增加几台服务器，这时候几个项目启动在不同的服务器上，用户要访问，就需要增加一个代理服务器

![image-20210128113746244](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210128113746244.png)

我们希望这个代理服务器可以帮助我们接收用户的请求，然后将用户的请求按照规则帮我们转发到不同的服务器节点之上。这个过程用户是**无感知的，用户并不知道是哪个服务器返回的结果**，我们还希望他可以按照服务器的性能提供不同的权重选择。保证最佳体验！所以我们使用了Nginx

### 1.2 什么是Nginx

Nginx (engine x)  是一个`高性能`的HTTP和`反向代理`web服务器，同时也提供了IMAP/POP3/SMTP服务。Nginx是由伊戈尔·赛索耶夫为俄罗斯访问量第二的Rambler.ru站点（俄文：Рамблер）开发的，第一个公开版本0.1.0发布于2004年10月4日。2011年6月1日，nginx 1.0.4发布。

其特点是占有内存少，并发能力强，事实上nginx的并发能力在同类型的网页服务器中表现较好，中国大陆使用nginx网站用户有：百度、京东、新浪、网易、腾讯、淘宝等。在全球活跃的网站中有12.18%的使用比率，大约为2220万个网站。

Nginx 是一个安装非常的简单、配置文件非常简洁（还能够支持perl语法）、Bug非常少的服务。Nginx 启动特别容易，并且几乎可以做到7*24不间断运行，即使运行数个月也不需要重新启动。你还能够不间断服务的情况下进行软件版本的升级。

Nginx代码完全用C语言从头写成。官方数据测试表明能够支持高达 50,000 个并发连接数的响应。

### 1.3 常用命令

```
# 检查配置文件是否有误
nginx –t
# 重新加载配置文件
nginx –s reload
```

## 2 作用

### 2.1 代理

这里主要区分一下正向代理与反向代理

* 正向代理：在客户端进行代理，如VPN

	![image-20210128114340322](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210128114340322.png)

* 反向代理：在服务器端，用户无感知的

	![image-20210128114440907](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210128114440907.png)

### 2.2 负载均衡

Nginx提供的负载均衡策略有2种：

* 内置策略
	* 内置策略为轮询，加权轮询，Ip hash。
* 扩展策略。
	* 扩展策略，就天马行空，只有你想不到的没有他做不到的。

> 轮询
>
> ![image-20210128114955188](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210128114955188.png)
>
> 加权轮询
>
> ![image-20210128115017614](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210128115017614.png)
>
> iphash对客户端请求的ip进行hash操作，然后根据hash结果将同一个客户端ip的请求分发给同一台服务器进行处理，可以解决session不共享的问题
>
> （实际开发中用redis来解决session共享更好）
>
> ![image-20210128115051525](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210128115051525.png)

### 2.3 动静分离

在我们的软件开发中，有些请求是需要后台处理的，有些请求是不需要经过后台处理的（如：css、html、jpg、js等等文件），这些不需要经过后台处理的文件称为静态文件。

让动态网站里的动态网页根据一定规则把不变的资源和经常变的资源区分开来，动静资源做好了拆分以后，我们就可以根据静态资源的特点将其做缓存操作。提高资源响应的速度。

![image-20210128115116285](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210128115116285.png)

## 3 安装

> 这里介绍用docker安装Nginx

拉去最新镜像

```
$ docker pull nginx:latest
```

运行容器

```
$ docker run --name nginx-test -p 80:80 -d nginx
```

访问80端口即可看到Nginx页面

![image-20210128215127754](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210128215127754.png)

这时候已经部署成功了，但是Nginx有一些配置文件

我们可以进入容器中查看

```
$ docker exec -it nginx-test
```

发现配置文件在/etc/nginx/nginx.conf下

我们将其映射到本地

```
mkdir /opt/nginx-test/conf
mkdir /opt/nginx-test/logs
mkdir /opt/nginx-test/html

docker cp 5786502:/usr/share/nginx/html /opt/nginx-test/html
docker cp 5786502:/usr/share/nginx/html /opt/nginx-test/
docker cp 5786502:/etc/nginx/conf.d/default.conf /opt/nginx-test/conf.d/default.conf
```

删除容器，重新启动

```
docker rm nginx-test -f
docker run --name nginx-test -p 80:80 -v /opt/nginx-test/conf/nginx.conf:/etc/nginx/nginx.conf -v /opt/nginx-test/logs:/var/log/nginx -v /opt/nginx-test/html:/usr/share/nginx/html -d nginx
```

##  4 实战

### 4.1 配置文件结构

```
...              #全局块

events {         #events块
   ...
}

http      #http块
{
    ...   #http全局块
    server        #server块
    { 
        ...       #server全局块
        location [PATTERN]   #location块
        {
            ...
        }
        location [PATTERN] 
        {
            ...
        }
    }
    server
    {
      ...
    }
    ...     #http全局块
}
```

1、**全局块**：配置影响nginx全局的指令。一般有运行nginx服务器的用户组，nginx进程pid存放路径，日志存放路径，配置文件引入，允许生成worker process数等。

2、**events块**：配置影响nginx服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。

3、**http块**：可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type定义，日志自定义，是否使用sendfile传输文件，连接超时时间，单连接请求数等。

4、**server块**：配置虚拟主机的相关参数，一个http中可以有多个server。

* upstream：负载均衡

	```
	upstream lb{
	    server 127.0.0.1:8080 weight=1;
	    server 127.0.0.1:8081 weight=1;
	}
	```

* server：代理

	```
	server{
		lisen 80;
		server_name localhost;
		location / {
			root html;
			index index.html;
			proxy_pass http://lb;
		}
	}
	```

5、**location块**：配置请求的路由，以及各种页面的处理情况

### 4.2 部署web应用

写一个最简单的后端应用

```java
@RestController
@RequestMapping("/test")
public class TestController {
    @PostMapping("/test")
    public void test(){
        System.out.println("test");
    }
}
```

把应用启动起来

```
nohup java -jar com.nginxtest-1.0-SNAPSHOT.jar --server.port=8081 > test1.log &
nohup java -jar com.nginxtest-1.0-SNAPSHOT.jar --server.port=8082 > test2.log &
```

查看，确保已经启动了

![image-20210128221359793](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210128221359793.png)

修改conf文件

```
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;
    #gzip  on;

    include /etc/nginx/conf.d/*.conf;

    upstream mysvr {
        server 172.17.59.248:8081 weight=1;
        server 172.17.59.248:8082 weight=2;
    }

    server{
        listen 80;
        server_name 39.97.124.144;  #自己的域名获取当前nginx服务器的IP;

        location / {
                add_header X-Content-Type-Options nosniff;
                proxy_set_header X-scheme $scheme;
          # 作用是我们可以获取到客户端的真实ip
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
                proxy_set_header X-Nginx-Proxy true;
                proxy_hide_header X-Powered-By;
                proxy_hide_header Vary;
         # 重点是这里，将代理转发给上方 upstream 中配置>的两台服务器去处理，这里的 http:// 后的值必须和 upstream 后面的值一致
                proxy_pass http://mysvr;
        }
    }
}
```

注意，修改完以后需要`docker restatr nginx-test`重启容器

之后，向`http://39.97.124.144:80/test/test`发送请求，观察两个web应用，实现了负载均衡

![image-20210128231826059](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210128231826059.png)

## 参考

https://www.kuangstudy.com/bbs/1353634800149213186