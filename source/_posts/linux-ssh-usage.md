---
title: linux中ssh的使用
date: 2018-11-23 21:23:21
tags:
---

实现linux服务器的免密码登录

<!-- more -->

A是客户端，B是要登录的服务端。

A的操作

1. 用`ssh-keygen`生成A的秘钥对。
2. 用`ssh-copy-id [B的ip]`命令把A的公钥发送到B。
3. A可以直接用`ssh 用户名@[B的ip]`登录