---
title: Persim-M7 Ubuntu环境搭建(Docker)
date: 2022-09-11 10:34:29
tags:
	- Persim
	- Liunx
	- 柿饼
	- 搭建环境
categories:
    - 开发环境
cover: preview.jpg
---

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

以后只需在文件的 scons 路径下执行 get_persim_m7 命令，即可获取环境变量，随后便可实现编译

配合 DockerFile 食用更佳

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
apt-get install sudo apt-utils vim net-tools inetutils-ping wget curl tree lbzip2 bzip2 git xz-utils python3 python2 python3-pip scons telnet openssl libssl-dev
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

