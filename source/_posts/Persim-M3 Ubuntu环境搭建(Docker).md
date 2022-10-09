---
title: Persim-M3 Ubuntu环境搭建(Docker)
date: 2022-09-10 10:43:02
tags:
	- persim
	- linux
	- ubuntu
	- 柿饼
	- 搭建环境
	- docker
categories:
    - 开发环境
cover: preview.jpg
---

## 环境搭建

首先进行换源和docker安装，深圳这边华为云比较快，其他地区可以用阿里云

```shell
# sudo sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
# sudo sed -i 's/security.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
sudo sed -i 's/archive.ubuntu.com/repo.huaweicloud.com/g' /etc/apt/sources.list
sudo sed -i 's/security.ubuntu.com/repo.huaweicloud.com/g' /etc/apt/sources.list
sudo apt update
sudo apt upgrade
sudo curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
sudo docker pull ubuntu:20.04
sudo docker run --name persim-m3 -itd ubuntu:20.04
sudo docker exec -it persim-m3 /bin/bash
```

在 docker 的 shell 中，执行以下命令

```bash
# sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
# sed -i 's/security.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
sed -i 's/archive.ubuntu.com/repo.huaweicloud.com/g' /etc/apt/sources.list
sed -i 's/security.ubuntu.com/repo.huaweicloud.com/g' /etc/apt/sources.list
apt-get update
apt-get install sudo apt-utils vim net-tools inetutils-ping wget curl tree lbzip2 bzip2 git xz-utils python3 python2 python3-pip scons telnet
apt-get install -qq libncurses5-dev lib32z1 > /dev/null
apt-get install u-boot-tools zlib1g-dev dosfstools mtools 
curl -s https://armkeil.blob.core.windows.net/developer//sitecore/shell/-/media/Files/downloads/gnu-rm/5_4-2016q3/gcc-arm-none-eabi-5_4-2016q3-20160926-linux,-d-,tar.bz2 | tar xjf - -C /opt
curl -s https://armkeil.blob.core.windows.net/developer/Files/downloads/gnu-rm/10-2020q4/gcc-arm-none-eabi-10-2020-q4-major-x86_64-linux.tar.bz2 | tar xjf - -C /opt
vim /root/.bashrc
```

在 vim 中补充以下命令

```bash
alias get_persim_m3='
    export RTT_ROOT=`pwd`/../rt-thread
    export RTT_CC=gcc
    export RTT_EXEC_PATH=/opt/gcc-arm-none-eabi-5_4-2016q3/bin
    $RTT_EXEC_PATH/arm-none-eabi-gcc --version
'
```

```shell
vim /root/.bash_profile
```

在 vim 中补充以下命令

```bash
if test -f .bashrc ; then
	source .bashrc 
fi
```

以后只需在文件的 scons 路径下执行 get_persim_m3 命令，即可获取环境变量，随后便可实现编译

配合 DockerFile 食用更佳，构建命令： `docker build -f ./dockerfile -t persim-m3:0.0 .`

```dockerfile
FROM ubuntu:20.04

RUN set -x

# 换源
# RUN sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
# RUN sed -i 's/security.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
RUN sed -i 's/archive.ubuntu.com/repo.huaweicloud.com/g' /etc/apt/sources.list
RUN sed -i 's/security.ubuntu.com/repo.huaweicloud.com/g' /etc/apt/sources.list

RUN apt-get update
RUN apt-get upgrade -y
RUN apt-get install -y sudo apt-utils vim net-tools inetutils-ping wget curl tree lbzip2 bzip2 git xz-utils python3 python2 python3-pip scons telnet
RUN DEBIAN_FRONTEND="noninteractive" apt -y install tzdata
RUN TZ=Asia/Shanghai && \
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime

RUN apt-get install -qq libncurses5-dev lib32z1 > /dev/null
RUN apt-get install -y u-boot-tools zlib1g-dev dosfstools mtools 
RUN curl -s https://armkeil.blob.core.windows.net/developer//sitecore/shell/-/media/Files/downloads/gnu-rm/5_4-2016q3/gcc-arm-none-eabi-5_4-2016q3-20160926-linux,-d-,tar.bz2 | tar xjf - -C /opt
RUN curl -s https://armkeil.blob.core.windows.net/developer/Files/downloads/gnu-rm/10-2020q4/gcc-arm-none-eabi-10-2020-q4-major-x86_64-linux.tar.bz2 | tar xjf - -C /opt

RUN printf " alias get_persim_m3='\n" >> /root/.bashrc
RUN printf "     export RTT_ROOT=`pwd`/../rt-thread\n" >> /root/.bashrc
RUN printf "     export RTT_CC=gcc\n" >> /root/.bashrc
RUN printf "     export RTT_EXEC_PATH=/opt/gcc-arm-none-eabi-5_4-2016q3/bin\n" >> /root/.bashrc
RUN printf "     \$RTT_EXEC_PATH/arm-none-eabi-gcc --version\n" >> /root/.bashrc
RUN printf "'\n" >> /root/.bashrc

RUN printf "if test -f .bashrc ; then\n" >> /root/.bash_profile
RUN printf "    source .bashrc\n" >> /root/.bash_profile
RUN printf "fi\n" >> /root/.bash_profile

RUN apt clean
RUN rm -rf /var/lib/apt/lists/*
RUN mkdir /var/lib/apt/lists/partial

RUN cat /etc/issue
RUN python3 -V
RUN scons --version

```

## VSCode配置文件

launch.json

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "arm-debug-bootloader",
            "type": "cppdbg",
            "request": "launch",
            "miDebuggerPath": "/opt/gcc-arm-none-eabi-10-2020-q4-major/bin/arm-none-eabi-gdb",
            "program": "${workspaceRoot}/software/bootloader/rtthread.elf",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "text": "target remote localhost:3333"
                },
                {
                    "text": "restore ${workspaceRoot}/software/bootloader/rtthread.elf"
                },
                {
                    "text": "set $pc=0x81B00000"
                },
            ],
            "launchCompleteCommand": "None",
            "cwd": "${workspaceFolder}"
        },
        {
            "name": "arm-debug-app",
            "type": "cppdbg",
            "request": "launch",
            "miDebuggerPath": "/opt/gcc-arm-none-eabi-5_4-2016q3/bin/arm-none-eabi-gdb",
            "program": "${workspaceRoot}/software/firmware/rtthread.elf",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "text": "target remote localhost:3333"
                },
                {
                    "text": "restore ${workspaceRoot}/software/firmware/rtthread.elf"
                },
                {
                    "text": "set $pc=0x80000000"
                },
            ],
            "launchCompleteCommand": "None",
            "cwd": "${workspaceFolder}"
        }
    ]
}
```

tasks.json

```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "build_bootloader",
            "type": "shell",
            "command": [
                "export RTT_ROOT=`pwd`/software/rt-thread;",
                "export RTT_CC=gcc;",
                "export RTT_EXEC_PATH=/opt/gcc-arm-none-eabi-10-2020-q4-major/bin;",
                "cd ${workspaceFolder}/software/bootloader;",
                "scons -j8;"
            ],
            "problemMatcher": []
        },
        {
            "label": "clean_bootloader",
            "type": "shell",
            "command": [
                "export RTT_ROOT=`pwd`/software/rt-thread;",
                "export RTT_CC=gcc;",
                "export RTT_EXEC_PATH=/gcc-arm-none-eabi-10-2020-q4-major/bin;",
                "cd ${workspaceFolder}/software/bootloader;",
                "scons -c;"
            ],
            "problemMatcher": []
        },
        {
            "label": "build_app",
            "type": "shell",
            "command": [
                "export RTT_ROOT=`pwd`/software/rt-thread;",
                "export RTT_CC=gcc;",
                "export RTT_EXEC_PATH=/opt/gcc-arm-none-eabi-5_4-2016q3/bin;",
                "cd ${workspaceFolder}/software/firmware;",
                "scons -j8;"
            ],
            "problemMatcher": []
        },
        {
            "label": "clean_app",
            "type": "shell",
            "command": [
                "export RTT_ROOT=`pwd`/software/rt-thread;",
                "export RTT_CC=gcc;",
                "export RTT_EXEC_PATH=/opt/gcc-arm-none-eabi-5_4-2016q3/bin;",
                "cd ${workspaceFolder}/software/firmware;",
                "scons -c;"
            ],
            "problemMatcher": []
        }
    ]
}
```

使用 VSCode 一键编译

在SDK的根目录打开 VSCode，按下 F1，选择 Run Tasks，选择 build_app，即可编译。

![image-20220917192322748](https://raw.githubusercontent.com/xqyjlj/xqyjlj.github.io/img/image-20220917192322748.png)

![image-20220917192349139](https://raw.githubusercontent.com/xqyjlj/xqyjlj.github.io/img/image-20220917192349139.png)
