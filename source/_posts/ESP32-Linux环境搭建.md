---
title: ESP32-Linux环境搭建
date: 2021-09-17 22:46:08
tags:
	- ESP32
	- Liunx
	- ESP
	- 搭建环境
categories:
    - 开发环境
cover: 43278.png
---

```shell
sudo apt-get install gcc git wget make flex bison gperf python python-pip python-setuptools python-serial python-cryptography python-future python-pyparsing python-pyelftools libffi-dev libssl-dev
mkdir -p ~/Tool/esp
cd ~/Tool/esp
wget https://dl.espressif.com/dl/xtensa-esp32-elf-gcc8_4_0-esp-2021r1-linux-amd64.tar.gz #64位
wget https://dl.espressif.com/dl/xtensa-esp32-elf-gcc8_4_0-esp-2021r1-linux-i686.tar.gz #32位
tar -xzf xtensa-esp32-elf-gcc8_4_0-esp-2021r1-linux-amd64.tar.gz #64位
tar -xzf xtensa-esp32-elf-gcc8_4_0-esp-2021r1-linux-i686.tar.gz #32位
git clone --recursive https://github.com/espressif/esp-idf.git
sudo vim  ~/.bashrc
```
在 vim 中补充以下命令
```shell
alias get_esp32='
export PATH="$HOME/Tool/esp/xtensa-esp32-elf/bin:$PATH"
export IDF_PATH="$HOME/Tool/esp/esp-idf"
'
```

```shell
source ~/.bashrc
get_esp32
sudo apt install python-pip
python -m pip install --user -r $IDF_PATH/requirements.txt
```

此后，只需执行以下命令便能加载其环境了

```shell
source ~/.bashrc
get_esp32
```

