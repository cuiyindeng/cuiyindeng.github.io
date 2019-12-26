---
title: 源码方式安装redis
date: 2019-12-26 15:23:36
tags:
---

### 在Ubuntu上用源码的方式安装redis

<!-- more -->

1. 环境准备
```bash
$ sudo apt update
$ sudo apt upgrade
$ sudo apt install build-essential
```

2. 下载源码包编译
```bash
$ sudo wget http://download.redis.io/releases/redis-5.0.7.tar.gz
$ sudo tar xzf redis-5.0.7.tar.gz
$ sudo cd redis-5.0.7
$ sudo make MALLOC=libc ##解决错误：zmalloc.h:50:10: fatal error: jemalloc/jemalloc.h: No such file or directory
```
3. 拷贝命令到指定目录，用于使用redis_init_script脚本安装并启动redis服务
```bash
$ sudo cp src/redis-server /usr/local/bin/
$ sudo cp src/redis-cli /usr/local/bin/
$ sudo mkdir /etc/redis
$ sudo mkdir /var/redis
$ sudo mkdir /var/redis/6379
$ sudo cp utils/redis_init_script /etc/init.d/redis_6379
$ sudo cp redis.conf /etc/redis/6379.conf
```

4. 修改配置文件，将daemonize项改成yes，dir项改为/var/redis/6379，logfile项改为/var/redis/6379/log/redis.log
```bash
$ sudo vi /etc/redis/6379.conf
```

5. 设置redis系统服务和启动redis服务
```bash
$ sudo update-rc.d redis_6379 defaults
$ sudo /etc/init.d/redis_6379 start ##或者service redis_6379 start
```
6. 登录redis服务
```bash
$ redis-cli -h 10.12.9.18 -p 7000 -c -a 12345
##ip：10.12.4.45 集群中的一个点
##-c 以集群方式登陆。cluster
##-a 密码。authority
```


