---
title: Centos代理设置
date: 2016-11-28 08:50:02
tags: [centos, 代理, dokcer]
categories: [linux]
---
## 系统级代理
vi /etc/profile
添加下面内容
```
http_proxy = http://username:password@yourproxy:8080/
ftp_proxy = http://username:password@yourproxy:8080/
export http_proxy
export ftp_proxy
```
## yum 代理
vi /etc/yum.conf
添加下面内容
```
proxy = http://username:password@yourproxy:8080/
```
## wget代理
vi /etc/wgetrc 
添加下面内容
```
http_proxy=http://username:password@proxy_ip:port/
ftp_proxy=http://username:password@proxy_ip:port/
```
## docker代理
1. 创建目录
    ```
    mkdir /etc/systemd/system/docker.service.d
   ```
2. 创建文件
    ```
    touch /etc/systemd/system/docker.service.d/http-proxy.conf
    ```
3. 配置http-proxy.conf文件
    ```
    [Service]
    Environment="HTTP_PROXY=http://proxy.ip.com:80"
    ```
4. daemon重新reload 并重启docker
    ```
    systemctl daemon-reload
    systemctl restart docker
    ```
5. 检查变量是否加载
    ```
    systemctl show docker --property Environment
    ```


