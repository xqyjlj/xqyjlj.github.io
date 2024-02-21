---
title: 修改WSL启动时的默认账户
date: 2022-09-24 09:23:09
tags:
	- WSL
	- 环境配置
categories:
    - 开发环境
cover: wsl-space-header.jpg
---

```shell
useradd --create-home --shell /bin/bash xqyjlj
passwd xqyjlj
usermod -aG sudo xqyjlj
vim /etc/wsl.conf
```

添加以下代码

```ini
[user]
default=xqyjlj
```

重启 wsl 即可
