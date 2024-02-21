---
title: ESP8266-Linux环境搭建
date: 2021-09-17 22:25:22
tags:
	- esp8266
	- linux
	- 环境配置
categories:
    - 开发环境
cover: esp8266.jpg
---

## ESP8266_RTOS_SDK

```shell
sudo apt-get install gcc git wget make libncurses-dev flex bison gperf python python-serial
mkdir -p ~/Tool/esp
cd ~/Tool/esp
wget https://dl.espressif.com/dl/xtensa-lx106-elf-gcc8_4_0-esp-2020r3-linux-amd64.tar.gz # 64位机
wget https://dl.espressif.com/dl/xtensa-lx106-elf-gcc8_4_0-esp-2020r3-linux-i686.tar.gz # 32位机
tar -xzf xtensa-lx106-elf-gcc8_4_0-esp-2020r3-linux-amd64.tar.gz # 64位机
tar -xzf xtensa-lx106-elf-gcc8_4_0-esp-2020r3-linux-i686.tar.gz # 32位机
git clone --recursive https://github.com/espressif/ESP8266_RTOS_SDK.git
sudo vim  ~/.bashrc
```

在 vim 中补充以下命令

```shell
alias get_esp8266_rtos='
export PATH="$HOME/Tool/esp/xtensa-lx106-elf/bin:$PATH"
export IDF_PATH="$HOME/Tool/esp/ESP8266_RTOS_SDK"
'
```

```shell
source ~/.bashrc
get_esp8266_rtos
sudo apt install python-pip
python -m pip install --user -r $IDF_PATH/requirements.txt
```

此后，只需执行以下命令便能加载其环境了

```shell
source ~/.bashrc
get_esp8266_rtos
```

## ESP8266_NONOS_SDK

```shell
sudo apt-get install gcc git wget make libncurses-dev flex bison gperf python python-serial
mkdir -p ~/Tool/esp
cd ~/Tool/esp
wget https://dl.espressif.com/dl/xtensa-lx106-elf-gcc8_4_0-esp-2020r3-linux-amd64.tar.gz # 64位机
wget https://dl.espressif.com/dl/xtensa-lx106-elf-gcc8_4_0-esp-2020r3-linux-i686.tar.gz # 32位机
tar -xzf xtensa-lx106-elf-gcc8_4_0-esp-2020r3-linux-amd64.tar.gz # 64位机
tar -xzf xtensa-lx106-elf-gcc8_4_0-esp-2020r3-linux-i686.tar.gz # 32位机
git clone --recursive https://github.com/espressif/ESP8266_NONOS_SDK.git
sudo vim  ~/.bashrc
```
在 vim 中补充以下命令


```shell
alias get_esp8266_nonos='
export PATH="$HOME/Tool/esp/xtensa-lx106-elf/bin:$PATH"
export IDF_PATH="$HOME/Tool/esp/ESP8266_NONOS_SDK"
'
```

此后，只需执行以下命令便能加载其环境了

```shell
source ~/.bashrc
get_esp8266_nonos
```

