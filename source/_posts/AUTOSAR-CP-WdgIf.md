---
title: AUTOSAR CP Watchdog Interface模块详解
date: 2024-02-23 15:59:49
tags:
  - AUTOSAR CP
  - BSW
  - Wdg
categories:
  - AUTOSAR
  - AUTOSAR Wdg
cover: AUTOSAR_03.png
---

> 本文章所参考的 AUTOSAR 标准为 4.4.0 版本

# 简介

`Watchdog Interface`（简称` Watchdog If`）模块作为底层 `Watchdog Driver` 的抽象层，旨在实现底层硬件与上层软件之间的解耦。它的核心功能是提供一个统一的接口，使得上层 `Watchdog Manager` 能够与底层看门狗驱动进行交互，而无需关心底层驱动的具体实现细节。

`Watchdog If` 模块并不负责底层看门狗驱动的特性设置，如窗口控制、时间周期等。这些特性的配置和管理仍然由底层的 `Watchdog Driver` 负责。`Watchdog If` 仅作为中介，确保上层 `Manager` 与底层 `Driver` 之间的通信顺畅。

当多个看门狗驱动被上层 `Watchdog Manager` 管理时，`Watchdog If` 模块通过 `DeviceIndex` 来区分和识别不同的驱动。如果在交互过程中出现错误，`Watchdog If` 模块将负责及时上报，确保系统能够作出相应的响应和处理。

总之，`Watchdog If` 模块在 `AUTOSAR` 架构的 `Watchdog` 协议栈中扮演着关键角色，它确保了底层硬件与上层软件之间的顺畅通信，提高了系统的可维护性和可扩展性。

# API 列表

| 函数名                        | 函数功能                                                     |
| ----------------------------- | ------------------------------------------------------------ |
| **WdgIf_SetMode**             | 设置看门狗模式，模式可分为：<br />- `WDGIF_OFF_MODE`<br />- `WDGIF_SLOW_MODE`<br />- `WDGIF_FAST_MODE` |
| **WdgIf_SetTriggerCondition** | 设置看门狗触发计数器的超时时间值                             |
| **WdgIf_GetVersionInfo**      | 返回 `WdgIf` 模块的版本信息                                    |

# 软件实现

待更新

# 参考

- https://www.zhihu.com/tardis/zm/art/651467544?source_id=1005
