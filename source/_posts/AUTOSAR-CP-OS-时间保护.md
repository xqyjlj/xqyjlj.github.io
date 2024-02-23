---
title: AUTOSAR-CP-OS-时间保护
date: 2024-02-22 11:44:26
tags:
  - AUTOSAR-CP
  - OS
categories:
  - AUTOSAR
cover: image.png
---

# 前言

> 本文章所参考的 AUTOSAR 标准为 4.4.0 版本

我们已知道 `AUTOSAR OS` 来源于 `OSEK OS`，随着汽车电子信息安全，功能安全等需求的不断提出，传统的 `OSEK OS` 已无法满足当前的需求，因此 `AUTOSAR` 组织在 `OSEK OS` 的基础上为不同的用户提供四类不同功能安全的 OS 可裁剪类型，分别为 `SC1`-`SC4`，具体含义如下：

- **SC1:** `OSEK OS` + `Schedule Table`；
- **SC2:** `OSEK OS` + `Schedule Table` + `Timing Protection`;
- **SC3:** `OSEK OS` + `Schedule Table` + `Memory Protection`;
- **SC4:** `OSEK OS` + `Schedule Table` + `Timing Protection` + `Memory Protection`；

如下图所示，较为清晰了描述了这四种不同可裁剪类型的区别与联系。

![image](AUTOSAR-CP-OS-%E6%97%B6%E9%97%B4%E4%BF%9D%E6%8A%A4/image.png)

# AUTOSAR OS 时间保护

## 概念

从 `AUTOSAR OS` 四种可裁剪类型可以看出，时间保护(`Timing Protection`)是一项非常重要的功能保护机制。`AUTOSAR OS` 作为一实时操作系统，那么就需要在预定的时间内完成特定的任务，但有时由于某些原因导致超时错误，`OS` 必须采用有效的方式来预防超时任务的发生，而这类措施则可以称为时间保护。

一种较为常见的时间保护就是 `Deadline Monitoring`。即当 OS 检测到某一任务的运行时间超过其截止时间时，则会调用相应的 `Hook` 函数向系统报错，但是 `AUTOSAR OS` 并不是通过监控截止时间方式来实现时间保护的，因为针对截止时间的保护并不能准确识别出当前错误的原因。具体解释如下：

- 现象：任务 1 的运行时间超过其截止时间时，任务 A 本身可能并没有出错；
- 过程：在执行任务 1 之前的任务 2 频繁的抢占或者过长的阻塞资源的访问；
- 原因：正由于任务 2 的上述行为，进而导致任务 1 执行超时，从而直观的认为任务 1 发生错误便停止任务 1，反而让真正的罪魁祸首任务 2 继续逍遥法外，这就不合情理，也起不到对 OS 中各任务的时间保护。

这个问题最好用例子来说明。假设存在一个操作系统中存在下列任务 A，B，C，并明确各自任务的优先级，执行时间，Deadline 如下表中所示：

![image-20240222143603586](AUTOSAR-CP-OS-%E6%97%B6%E9%97%B4%E4%BF%9D%E6%8A%A4/image-20240222143603586.png)

假设所有任务都准备好在时间 0 运行，那么将会出现下面的执行跟踪，并且所有任务都将满足各自的截止日期。期望运行的任务执行时序如下图所示。具体的任务执行时序如下所述：

- 由于任务 A 优先级最高，因此任务 A 先执行；
- 1 个 Tick 之后任务 B 开始执行，3 个 Tick 之后任务 C 执行；
- 当任务 C 执行 1 个 Tick 后被任务 A 打断，任务 A 执行完毕后，任务 C 继续运行；
- 到第 10 个 Tick 任务 C 执行完毕，周而复始，整个过程并未出现超时现象，并且仍有一个 Tick 的空闲状态；

![image-20240222143826727](AUTOSAR-CP-OS-%E6%97%B6%E9%97%B4%E4%BF%9D%E6%8A%A4/image-20240222143826727.png)

现在考虑任务 A 和 B 行为不正确的情况。下图显示了一种异常状态

- 任务 A 的第二个周期与任务 B 的第一个周期都出现运行时间过长的现象，但并没有超过其截止时间。
- 任务 B 在第二个周期提前进入运行状态，但也未超过其截止时间；
- 任务 C 按照正确的方式运行，但由于任务 A 与任务 B 的出错导致任务 C 运行超时，则发生超时错误；

![image-20240222144202158](AUTOSAR-CP-OS-%E6%97%B6%E9%97%B4%E4%BF%9D%E6%8A%A4/image-20240222144202158.png)

因此，从上述例子分析可以得出:在像 `AUTOSAR OS` 这样的固定优先级抢占式操作系统中，任务或 ISR 是否满足截止日期取决于以下三大基本要素：

1. **系统中任务/ISR 的执行时间**

2. **任务/ISR 因低优先级任务/ISR 锁定共享资源或禁用中断而遭受的阻塞时间**

3. **系统中任务/ISR 的到达间隔率**

针对上述三大基本要素，`AUTOSAR OS` 采取了下列三种时间保护机制：

- **对于 1**：`AUTOSAR OS` 通过使用执行时间保护来保证执行时间的静态配置上限(称为执行预算)，此保护措施只对以下进行保护：

  - 任务执行时间（指任务在得到 `cpu` 执行权到**主动放弃 cpu** 期间的时间）。

  - 第二类中断（指中断从开始到结束的时间）。

- **对于 2**：`AUTOSAR OS` 通过使用锁定时间保护来保证静态配置的上限(称为锁定预算)，此保护措施只对以下进行保护：

  - 共享资源由任务/第 2 类 ISR 持有的时间（指`GetResource`到`ReleaseResource`的时间）。

  - 操作系统中断被任务/2 类 ISR 挂起（指`OSInterrupts`从关闭到打开的时间）。

  - 任务/2 类 ISR 暂停/禁用所有中断（指`AllInterrupts`从关闭到打开的时间）。

- **对于 3**：`AUTOSAR OS` 通过使用到达间隔时间保护来保证以下时间之间的静态配置下限(称为时间帧)，此保护措施只对以下进行保护：
- 由于以下原因，任务被允许转换到 `READY` 状态（指任务连续两次获得 `CPU` 执行权的间隔时间）:

  - 激活(从 `SUSPENDED` 状态到 `READY` 状态的转换)
    - 释放(从 `WAITING` 状态到 `READY` 状态的转换)

- 第 2 类 ISR 到达（此第 2 类 ISR 连续两次执行的间隔时间）。

下图显示了执行时间保护和到达间隔时间保护如何与 `AUTOSAR OS` 的任务状态转换模型进行交互。

![image-20240222151032475](AUTOSAR-CP-OS-%E6%97%B6%E9%97%B4%E4%BF%9D%E6%8A%A4/image-20240222151032475.png)

特别的，需要注意的是 `AUTOSAR OS` 时间保护存在一些基本特性：

- 时间保护仅仅用于任务或者二类中断，对于一类中断不起作用；
- 在 OS 未开启之前，时间保护将不起作用；
- 在 trusted OS-Applications 的情况下，所有时间信息都必须正确，否则系统可能会在运行时失败。对于 non-trusted OS Application，可以使用时间保护来加强可执行对象之间的定时界限
- `DisableAllInterrupts`/`SuspendAllInterrupts` 等相关接口是不能关闭时间保护定时器中断的

## 故障后措施

关于时间保护检测到时间异常之后的细节，规范 `AUTOSAR_SWS_OS 7.7.2.2 Requirements` 里面已经说的非常清楚了，总的来说就是调用 ProtectionHook 函数，并根据实际情况传递错误码：

- `E_OS_PROTECTION_ARRIVAL`：触发间隔时间保护
- `E_OS_PROTECTION_TIME`：触发执行时间保护
- `E_OS_PROTECTION_LOCKED`：触发锁定时间保护

# ProtectionHook

`ProtectionHook` 的函数原型为 `ProtectionReturnType ProtectionHook(StatusType Fatalerror)`，该函数由用户提供，在时间保护检测到异常时，就会调用此函数并传递错误码。用户根据错误码来选择 `ProtectionHook` 的命令码进行返回（具体请看规范`AUTOSAR_SWS_OS 7.8.2 Requirements`），其命令码对应的功能为

- `PRO_IGNORE`： 忽略此次错误
- `PRO_TERMINATETASKISR`：强制终止故障任务/第二类 ISR
- `PRO_TERMINATEAPPL`：强制终止故障任务/第二类 ISR 所属的 APP
- `PRO_TERMINATEAPPL_RESTART`：强制重启故障任务/第二类 ISR 所属的 APP
- `PRO_SHUTDOWN`：关闭系统

由此，就可以实现一个时间保护检测到一个异常任务，报告给系统并调用 `ProtectionHook`，系统再根据 `ProtectionHook` 的返回值来选择实施具体的故障措施

# 硬件实现

这里我们以英飞凌的 `TriCore` 架构为例进行讲解，从手册 `Infineon-AURIX_TC3xx_Architecture_vol1-UserManual-v01_00-EN`中的 `11 Temporal Protection System` 章节我们可以得知，`TriCore` 架构自带时间保护定时器，这个定时器很简单，其功能如下：

- 单调以 `CPU` 主频的频率递减
- 写入非 0 值开始计时
- 写入 0 值关闭定时器
- 定时器到期后触发异常陷阱(`Class-4`, `Tin-7`)，这个是无法通过`DisableAllInterrupts`/`SuspendAllInterrupts` 等相关接口关闭的
- 每个核都有 3 个独立的时间保护定时器，以 `tc397` 为例，有 6 个核，也就代表着有 `6 * 3=18` 个时间保护定时器

![image-20240223105917478](AUTOSAR-CP-OS-%E6%97%B6%E9%97%B4%E4%BF%9D%E6%8A%A4/image-20240223105917478.png)

另外，`TriCore` 架构还提供了一个 `CPU_CCNT` 的寄存器，该寄存器在手册`Infineon-AURIX_TC3xx_Architecture_vol1-UserManual-v01_00-EN`中的 `12.11 Performance Counter Registers` 章节可以查阅到，该寄存器会记录当前的 `CPU` 时钟周期，这个寄存器非常适合用来进行间隔时间保护

![image-20240223111111580](AUTOSAR-CP-OS-%E6%97%B6%E9%97%B4%E4%BF%9D%E6%8A%A4/image-20240223111111580.png)

# 软件实现

待更新

# 参考

- https://zhuanlan.zhihu.com/p/430996183
