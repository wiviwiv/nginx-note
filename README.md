nginx-note nginx 服务器的应用实例笔记
---
[TOC]

### 1. 概述
#### 1.1 nginx 是什么
nginx 是一个单线程的 web 服务器, 反向代理服务器, 邮件服务器。

#### 1.2 nginx 的优点
nginx 非常快? 反向代理、 URL 重定向等功能不错, 你会在接下来的笔记中看到它的具体运用。

### 2. 安装
#### 2.1 Docker 安装
```bash
# pull image
docker pull nginx

# run container
# 这里由于我的外部 80 端口被占用, 使用外部的其他端口
docker run -p 9000:80 -p 5000:5000 -p 8000:8000 --name nginx -v $PWD/conf/www:/www -v $PWD/conf/conf/nginx.conf:/etc/nginx/nginx.conf -v $PWD/conf/logs:/wwwlogs -d nginx 
```

#### 2.2 启动
`$PWD` 是指当前执行目录, 为了启动 nginx 你需要在 `./conf/conf/nginx.conf` 进行配置:
```bash
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include mime.types;
    default_type  application/octet-stream;

    sendfile  on;
    keepalive_timeout  90;
    gzip  on;

    server {
	    listen  80;
	    server_name  example.com;
	    root /www;
	}
}
```
启动: 

```Bash
docker start nginx
```

### 3. 静态资源服务器
#### 3.1 纯静态服务器
如下配置启动一个服务器
```bash
http {
	# 
    include mime.types;
    
    # 默认响应为下载流
    default_type  application/octet-stream;
    
    server {
            # 监听端口
            listen  80;

            # 域名, 多个域名空格隔开
            server_name example.com *.wivwiv.com;

            # 位置与首页
            location / {
                root /www;
                index index.html;
            }

            # 404 页面
            # error_page 404 /www/404.html;
            error_page 404 https://github.com/wiviwiv/nginx-note;
            
            # 500 页面
            error_page 500 502 503 504 /50X.html
            location /50X.html {
				root /www/500.html;
				# rewrite https://github.com/wiviwiv/nginx-note permanent;
            }
    }
}
```
此时 `./www/index.html` 的内容将在 `http://localhost:9000` 展示。