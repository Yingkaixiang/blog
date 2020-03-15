## 安装

### 自行编译安装

前往 nginx 开源项目[官网](http://nginx.org/en/download.html)下载源码，并自行编译安装。

安装时可能会报错缺少 pcre 这个库，可以自行去 pcre 官网(https://ftp.pcre.org/pub/pcre/)下载。

### 使用 brew

```bash
brew install nginx
brew services start nginx
brew services stop nginx

# 如果报错 unknown command services
# 可以执行以下命令进行升级
brew update
```

* 安装目录（虚拟根目录）：`/usr/local/Cellar/nginx/1.17.7`
* 安装目录下的 html 目录 symlink 到：`/usr/local/var/www`
* 配置文件目录：`/usr/local/etc/nginx`
* 日志文件目录：`/usr/local/var/log/nginx`

## 指令

### listen

```
location / {
    # 加上 ip 地址就可以限定只有
    # 本机进程访问这个端口
    listen 127.0.0.1:8080
}
```

### alias 和 root

同样访问 www.example.com/test/index.html

```
# location 的 path 直接和指定路径映射
# 即查到的就是 alias
location /test {
      alias html/test/;
}
```

```
# location 的 path 需要和上一级目录进行映射
# 即查找的是 root + path
location /test {
    root html/;
}
```

### gzip

```
gzip  on;
# 小于 1 字节的数据不进行压缩
gzip_min_length 1;
# 压缩级别
gzip_comp_level 2;
# 允许被压缩的文件类型
gzip_types text/html
```

### log_format

日志记录的格式

```bash
# main 为这个日志格式的自定义名称
log_format main '';
```

### access_log

指定日志记录的位置

```bash
# 使用日志格式为 main 
# 并记录在 logs/access.log 文件中
# 需要事先创建该目录
# logs/ --> /usr/local/Cellar/nginx/1.17.7/logs
access_log logs/access.log main
```

## 模块

### autoindex

用于展示当前访问的目录下的所有文件信息，前提是该目录下没有 index.html 文件

```
location / {
    autoindex on;
}
```

![](https://user-gold-cdn.xitu.io/2020/3/14/170d85ef5c1c8f25?w=725&h=217&f=png&s=19085)

## 内置变量

* `$limit_rate_after`
* `$limit_rate`

```
location / {
    # 带宽控制
    # 用户下载达到 500k 以后
    set $limit_rate_after 500k;
    # 限制每秒传输 1k 的数据
    set $limit_rate 1k;
}
```

* `$host`
* `$remote_addr`

## 反向代理以及缓存

```
http {
    # 设置缓存文件存放的目录
    # 地址为 /tmp/cahce
    # 设置共享内存大小
    # 缓存的 key 为 my_cache
    # 分配的大小为 10m
    proxy_cache_path /tmp/cache levels=1:2 keys_zone=my_cache:10m max_size=10g inactive=60m use_temp_path=off;
    
    # 反向代理的上游服务器列表
    upstream local {
        server 127.0.0.1:8080;
    }
    
    server {
        # 添加 ip 地址表示只有
        # 本地进程才能访问这个端口
        listen 127.0.0.1:8080;
        location / {
            alias html;
        }
    }
    
    server {
        listen 8081;
        location / {
            # 将客户端的 ip 发送给上游服务器
            # 如果不进行设置的话上游服务器拿到
            # 的其实是反向代理服务器的 ip
            proxy_set_header X-Real-IP $remote_addr;
            # 同上
            proxy_set_header Host $host;
            
            # 将刚刚开辟的共享内存
            # 配置到当前请求中
            proxy_cache my_cache;
            # 可以理解为 CDN 存文件使用的 key
            # 同样的 key 获取的缓存文件相同
            proxy_cache_key $host$uri$is_args$args;
            # 允许被缓存的 http status code
            # 缓存时间为1天
            proxy_cache_valid 200 304 302 1d;
            
            # 将 8081 端口的请求
            # 反向代理到上游的 local
            proxy_pass http://local;
        }
    }
}
```

## 可视化 access.log 工具

[GoAccess](https://goaccess.io/)

![截图](https://user-gold-cdn.xitu.io/2020/3/14/170d959ee4ec0bc9?w=730&h=435&f=png&s=13577)
