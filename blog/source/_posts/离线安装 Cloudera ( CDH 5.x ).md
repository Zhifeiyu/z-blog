---
title: 离线安装 Cloudera ( CDH 5.x )
date: 2016-12-20 10:37:06
tags: [cdh,大数据,hadoop]
categories: [大数据]
---
## 准备
- 系统 centos 6.5
- jdk 1.8
- 三台主机节点
    节点角色说明

| ip                  | 主机名                 | 角色描述     |
| --------------|:------------------:| ------------:|
| 10.206.2.181 | hadoop-master | cm，agent  |
| 10.206.2.182 | hadoop-slave1  | agent          |
| 10.206.2.183 | hadoop-slave2  | agent          |

- 域名解析
配置/etc/hosts, 将以下代码追加到文件末尾即可
```
sudo vim /etc/hosts
```
```
10.206.2.181 hadoop-master 
10.206.2.182 hadoop-slave1
10.206.2.183 hadoop-slave2
```
- 关闭iptable 或配置 iptable策略
- 关闭SELinux
- 配置免密码ssh登录
- 安装jdk（在所有节点操作）
- 时间同步
- 准备包（用parcel 方式安装）
- 
