---
title: Persim-M7 Ubuntu环境搭建(Docker)
date: 2022-09-11 10:34:29
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
sudo docker run --name persim-m7 -itd ubuntu:20.04
sudo docker exec -it persim-m7 /bin/bash
```

在 docker 的 shell 中，执行以下命令

```bash
# sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
# sed -i 's/security.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
sed -i 's/archive.ubuntu.com/repo.huaweicloud.com/g' /etc/apt/sources.list
sed -i 's/security.ubuntu.com/repo.huaweicloud.com/g' /etc/apt/sources.list
apt-get update
apt-get install sudo apt-utils vim net-tools inetutils-ping wget curl tree lbzip2 bzip2 git xz-utils python3 python2 python3-pip scons telnet openssl libssl-dev
apt-get install -qq libncurses5-dev lib32z1 > /dev/null
apt-get install u-boot-tools zlib1g-dev dosfstools mtools 
curl -s https://occ-oss-prod.oss-cn-hangzhou.aliyuncs.com/resource//1652757104469/Xuantie-900-gcc-elf-newlib-x86_64-V2.2.6-20220516.tar.gz | tar xzf - -C /opt
vim /root/.bashrc
```

在 vim 中补充以下命令

```bash
alias get_persim_m7='
    export RTT_CC=gcc
    export RTT_EXEC_PATH=/opt/Xuantie-900-gcc-elf-newlib-x86_64-V2.2.6/bin
    export RTT_CC_PREFIX=riscv64-unknown-elf-
    $RTT_EXEC_PATH/riscv64-unknown-elf-gcc --version
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

以后只需执行 get_persim_m7 命令，即可获取环境变量，随后便可实现编译

配合 DockerFile 食用更佳，构建命令： `docker build -f ./dockerfile -t persim-m7:0.0 .`

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
RUN apt-get install -y sudo apt-utils vim net-tools inetutils-ping wget curl tree lbzip2 bzip2 git xz-utils python3 python2 python3-pip scons telnet openssl libssl-dev
RUN DEBIAN_FRONTEND="noninteractive" apt -y install tzdata
RUN TZ=Asia/Shanghai && \
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime

RUN apt-get install -qq libncurses5-dev lib32z1 > /dev/null
RUN apt-get install -y u-boot-tools zlib1g-dev dosfstools mtools 
RUN curl -s https://occ-oss-prod.oss-cn-hangzhou.aliyuncs.com/resource//1652757104469/Xuantie-900-gcc-elf-newlib-x86_64-V2.2.6-20220516.tar.gz | tar xzf - -C /opt

RUN printf " alias get_persim_m7='\n" >> /root/.bashrc
RUN printf "     export RTT_CC=gcc\n" >> /root/.bashrc
RUN printf "     export RTT_EXEC_PATH=/opt/Xuantie-900-gcc-elf-newlib-x86_64-V2.2.6/bin\n" >> /root/.bashrc
RUN printf "     export RTT_CC_PREFIX=riscv64-unknown-elf-\n" >> /root/.bashrc
RUN printf "     \$RTT_EXEC_PATH/riscv64-unknown-elf-gcc --version\n" >> /root/.bashrc
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
            "name": "riscv-debug-bootloader",
            "type": "cppdbg",
            "request": "launch",
            "miDebuggerPath": "/opt/Xuantie-900-gcc-elf-newlib-x86_64-V2.2.6/bin/riscv64-unknown-elf-gdb",
            "program": "${workspaceRoot}/software/bootloader/f133/rtthread.elf",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "text": "target remote localhost:1025"
                },
                {
                    "text": "restore ${workspaceRoot}/software/bootloader/f133/rtthread.elf"
                },
                {
                    "text": "set $pc=0x41000000"
                },
            ],
            "launchCompleteCommand": "None",
            "cwd": "${workspaceFolder}"
        },
        {
            "name": "riscv-debug-app",
            "type": "cppdbg",
            "request": "launch",
            "miDebuggerPath": "/opt/Xuantie-900-gcc-elf-newlib-x86_64-V2.2.6/bin/riscv64-unknown-elf-gdb",
            "program": "${workspaceRoot}/software/firmware/f133/rtthread.elf",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "text": "target remote localhost:1025"
                },
                {
                    "text": "restore ${workspaceRoot}/software/firmware/f133/rtthread.elf"
                },
                {
                    "text": "set $pc=0x40000000"
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
                "export RTT_CC=gcc;",
                "export RTT_EXEC_PATH=/opt/Xuantie-900-gcc-elf-newlib-x86_64-V2.2.6/bin;",
                "export RTT_CC_PREFIX=riscv64-unknown-elf-;",
                "cd ${workspaceFolder}/software/bootloader/f133;",
                "scons -j8;"
            ],
            "problemMatcher": []
        },
        {
            "label": "clean_bootloader",
            "type": "shell",
            "command": [
                "export RTT_CC=gcc;",
                "export RTT_EXEC_PATH=/opt/Xuantie-900-gcc-elf-newlib-x86_64-V2.2.6/bin;",
                "export RTT_CC_PREFIX=riscv64-unknown-elf-;",
                "cd ${workspaceFolder}/software/bootloader/f133;",
                "scons -c;"
            ],
            "problemMatcher": []
        },
        {
            "label": "build_app",
            "type": "shell",
            "command": [
                "export RTT_CC=gcc;",
                "export RTT_EXEC_PATH=/opt/Xuantie-900-gcc-elf-newlib-x86_64-V2.2.6/bin;",
                "export RTT_CC_PREFIX=riscv64-unknown-elf-;",
                "cd ${workspaceFolder}/software/firmware/f133;",
                "scons -j8;"
            ],
            "problemMatcher": []
        },
        {
            "label": "clean_app",
            "type": "shell",
            "command": [
                "export RTT_CC=gcc;",
                "export RTT_EXEC_PATH=/opt/Xuantie-900-gcc-elf-newlib-x86_64-V2.2.6/bin;",
                "export RTT_CC_PREFIX=riscv64-unknown-elf-;",
                "cd ${workspaceFolder}/software/firmware/f133;",
                "scons -c;"
            ],
            "problemMatcher": []
        }
    ]
}
```

使用 VSCode 一键编译

在SDK的根目录打开 VSCode，按下 F1，选择 Run Tasks，选择 build_app，即可编译。

![image-20220917192215890](https://raw.githubusercontent.com/xqyjlj/xqyjlj.github.io/img/image-20220917192215890.png)

![image-20220917192248446](https://raw.githubusercontent.com/xqyjlj/xqyjlj.github.io/img/image-20220917192248446.png)
