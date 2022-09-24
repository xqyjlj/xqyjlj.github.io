---
title: Window SSH 远程开发环境搭建
date: 2022-09-11 11:21:40
tags:
	- persim
	- linux
	- ubuntu
	- 柿饼
	- 搭建环境
    - 远程开发
categories:
    - 开发环境
cover: 幽灵公主.jpg
---
## Ubuntu 设置
```shell
sudo apt-get install openssh-server
sudo service ssh start
sudo ps -e |grep ssh
sudo ifconfig
sudo apt-get install samba samba-common
sudo smbpasswd -a xqyjlj #此处用户名必须存在
#等待设置密码

sudo vim /etc/samba/smb.conf
#在配置文件中找到usershare allow guests = yes，在后面一行添加 usershare owner only = false
sudo chmod 777 /home/ -R
```

添加共享文件夹，为了方便，我这里直接共享 home

   ![image-20220911133530992](https://raw.githubusercontent.com/xqyjlj/xqyjlj.github.io/img/image-20220911133530992.png)

## Windows 设置

1. 打开映射网络驱动器

   ![image-20220911133651408](https://raw.githubusercontent.com/xqyjlj/xqyjlj.github.io/img/image-20220911133651408.png)

2. 输入 Ubuntu 的 IP

   ![image-20220911133753309](https://raw.githubusercontent.com/xqyjlj/xqyjlj.github.io/img/image-20220911133753309.png)

3. 映射完成后的效果

   ![image-20220911133814999](https://raw.githubusercontent.com/xqyjlj/xqyjlj.github.io/img/image-20220911133814999.png)

## SSH推荐

### VSCode

1. 首先安装 Remote - SSH 插件

   ![image-20220911134920019](https://raw.githubusercontent.com/xqyjlj/xqyjlj.github.io/img/image-20220911134920019.png)

2. 点击左下角的远程窗口，选择 Connect to Host

   ![image-20220911135134685](https://raw.githubusercontent.com/xqyjlj/xqyjlj.github.io/img/image-20220911135134685.png)

3. 按照提示完成操作即可享受远程开发

4. 效果图

   ![image-20220911135546681](https://raw.githubusercontent.com/xqyjlj/xqyjlj.github.io/img/image-20220911135546681.png)

### Jetbrains

使用此远程开发，按照提示一路操作即可

![image-20220911135727217](https://raw.githubusercontent.com/xqyjlj/xqyjlj.github.io/img/image-20220911135727217.png)
