---
title: SSH免密登录
date: 2022-10-14 00:10:00
tags:
	- 环境配置
	- ssh
categories:
    - 开发环境
cover: ssh-big.png
---

## 生成公钥

如果你本地有公钥，则跳过此步。

```bash
# 输入命令，一直按回车即可
ssh-keygen  #生成公钥
```

## 上传公钥

```bash
ssh-copy-id -i C:\Users\username\.ssh\id_rsa.pub username@userip
```

## 可能遇到的问题

### 多密钥

config 文件添加 IdentityFile

```ini
Host userip
  HostName userip
  User username
  IdentityFile C:/Users/username/.ssh/id_rsa
  ServerAliveInterval 60
  Port 22
```

### 无法免密

首先开 debug 模式，在服务端执行

```bash
sudo /usr/sbin/sshd -d -p 2222
```

在客户端执行

```bash
ssh -vvv username@userip -p 2222 -i C:/Users/username/.ssh/id_rsa
```

观察日志，看是否有以下日志

- Authentication refused: bad ownership or modes for directory

- Could not open authorized keys : Permission denied

### 原因

sshd 为了安全，对属主的目录和文件权限有所要求。如果权限不对，则 ssh 的免密码登陆不生效。

- 用户目录权限为 755 或者 700，就是不能是 77x

- .ssh 目录权限一般为 755 或者 700

- rsa_id.pub 及 authorized_keys 权限一般为644

- rsa_id 权限必须为 600

### 解决方法

使用`chmod`命令进行权限设置。

如若依然存在`Could not open authorized keys : Permission denied`，在用户路径下执行`sudo restorecon -FRv ~/.ssh`重置 ssh 路径的安全上下文。
