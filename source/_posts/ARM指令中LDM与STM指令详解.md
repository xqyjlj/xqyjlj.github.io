---
title: ARM指令中LDM与STM指令详解
date: 2022-10-10 18:08:00
tags:
    - 汇编
    - ARM
    - LDM
    - STM
categories:
    - 汇编
cover: 101733334_p0_master1200.jpg
---

## 简介:

ARM指令中多数据传输共有两种:

`LDM`:(load much)多数据加载,将地址上的值加载到寄存器上

`STM`:(store much)多数据存储,将寄存器上的值加载到地址上

其主要用途有：现场保护、数据复制、参数传送等，共有8种模式 (前面4种用于数据块的传输，后面4种是堆栈操作）具体如下：

1. `IA`: (Increase After） 每次传送后地址加 4,其中的寄存器从左到右执行,例如:`STMIA R0,{R1,LR}` 先存`R1`,再存`LR`

1. `IB`: (Increase Before）每次传送前地址加 4,同上

1. `DA`: (Decrease After） 每次传送后地址减 4,其中的寄存器从右到左执行,例如:`STMDA R0,{R1,LR}` 先存`LR`,再存`R1`

1. `DB`: (Decrease Before）每次传送前地址减 4,同上

1. `FD`: 满递减堆栈 (每次传送前地址减 4)

1. `FA`: 满递增堆栈 (每次传送后地址加 4)

1. `ED`: 空递减堆栈 (每次传送前地址减 4)

1. `EA`: 空递增堆栈 (每次传送后地址加 4)

> 注意：
>
> 在数据块的传输是`STMDB`和`LDMIA`对应，`STMIA`和`LDMDB`对应
>
> 在堆栈中的操作是`STMFD`和`LDMFD`对应，`STMFA`和`LDMFA`对应

## 格式:

```apl
LDM{cond} mode Rn{!}, reglist{^}

STM{cond} mode Rn{!}, reglist{^}
```

其中：

​	`Rn`：基址寄存器，装有传送数据的起始地址，`Rn`不允许为`R15`；

​	`!`：表示最后的地址写回到`Rn`中；

​	`reglist`：可包含多于一个寄存器范围，用`,`隔开，如`{R1，R2，R6-R9}`，寄存器由小到大顺序排列；

​	`^`：不允许在用户模式和系统模式下运行

## 举例

```assembly
STMFD  SP，｛R0-R3｝

；执行伪指令大致是：
；SP-4  = R3
；SP-8  = R2
；SP-12 = R1
；SP-16 = R0
；SP 的值未修改。

LDMFD  SP，｛R0-R3｝
；执行伪指令大致是：
；R3 = SP-4  
；R2 = SP-8  
；R1 = SP-12
；R0 = SP-16 
；SP 的值未修改。

STMFD  SP！，｛R0-R3｝
；执行伪指令大致是：
；SP -= 4 
；SP = R3
；SP -= 4 
；SP = R2
；SP -= 4 
；SP = R1
；SP -= 4 
；SP = R0
；SP -= 4 
；SP 的值已修改。

STMED  SP！，｛R0-R3｝
；执行伪指令大致是：
；SP = R3
；SP -= 4 
；SP = R2
；SP -= 4 
；SP = R1
；SP -= 4 
；SP = R0
；SP -= 4 
；SP 的值已修改。
```

## 详解

### IA

#### STMIA R0!,{R1,R2, R3,R14}
​          先传后增,寄存器→RAM
​          效果图:

![img](ARM%E6%8C%87%E4%BB%A4%E4%B8%ADLDM%E4%B8%8ESTM%E6%8C%87%E4%BB%A4%E8%AF%A6%E8%A7%A3/28458801_1373266870ojDd.png)

#### LDMIA R0!,{R1,R2, R3,R14}
​          先传后增, RAM →寄存器
​          效果图:

![img](ARM%E6%8C%87%E4%BB%A4%E4%B8%ADLDM%E4%B8%8ESTM%E6%8C%87%E4%BB%A4%E8%AF%A6%E8%A7%A3/28458801_1373267167skGt.png)

### IB
#### STMIB R0!,{R1,R2, R3,R14}
​          先增后传,寄存器→RAM
​          效果图:

![img](ARM%E6%8C%87%E4%BB%A4%E4%B8%ADLDM%E4%B8%8ESTM%E6%8C%87%E4%BB%A4%E8%AF%A6%E8%A7%A3/28458801_13732675001eeY.png)

#### LDMIB R0!,{R1,R2, R3,R14}
​          先增后传, RAM →寄存器
​          效果图:

![img](ARM%E6%8C%87%E4%BB%A4%E4%B8%ADLDM%E4%B8%8ESTM%E6%8C%87%E4%BB%A4%E8%AF%A6%E8%A7%A3/28458801_13732676888U86.png)

### DA
#### STMDA R0!,{R1,R2, R3,R14}
​          先传后减, 寄存器→ RAM
​          效果图:

![img](ARM%E6%8C%87%E4%BB%A4%E4%B8%ADLDM%E4%B8%8ESTM%E6%8C%87%E4%BB%A4%E8%AF%A6%E8%A7%A3/28458801_13732677555jox.png)

#### LDMDA R0!,{R1,R2, R3,R14}
​          先传后减, RAM → 寄存器
​          效果图:

![img](ARM%E6%8C%87%E4%BB%A4%E4%B8%ADLDM%E4%B8%8ESTM%E6%8C%87%E4%BB%A4%E8%AF%A6%E8%A7%A3/28458801_13732679412hwL.png)

### DB
#### STMDB R0!,{R1,R2, R3,R14}
​          先减后传,寄存器→ RAM
​          效果图:

![img](ARM%E6%8C%87%E4%BB%A4%E4%B8%ADLDM%E4%B8%8ESTM%E6%8C%87%E4%BB%A4%E8%AF%A6%E8%A7%A3/28458801_137326812866pz.png)

#### LDMDB R0!,{R1,R2, R3,R14}
​          先减后传, RAM → 寄存器
​          效果图:

![img](ARM%E6%8C%87%E4%BB%A4%E4%B8%ADLDM%E4%B8%8ESTM%E6%8C%87%E4%BB%A4%E8%AF%A6%E8%A7%A3/28458801_13732683358578.png)

### FA
#### STMFA SP!,{R0,R1,R2,R14}
​           满递增入栈,R13为基址地址
​                 效果图:

![img](ARM%E6%8C%87%E4%BB%A4%E4%B8%ADLDM%E4%B8%8ESTM%E6%8C%87%E4%BB%A4%E8%AF%A6%E8%A7%A3/28458801_13732685373tJq.png)

#### LDMFA SP!,{R0,R1,R2,R14}
​         满递增出栈,R13为基址地址
​         效果图:

![img](ARM%E6%8C%87%E4%BB%A4%E4%B8%ADLDM%E4%B8%8ESTM%E6%8C%87%E4%BB%A4%E8%AF%A6%E8%A7%A3/28458801_1373268638AZCe.png)

### FD
#### STMFD SP!,{R0,R1,R2,R14}
​         满递减入栈,R13为基址地址
​          效果图:

![img](ARM%E6%8C%87%E4%BB%A4%E4%B8%ADLDM%E4%B8%8ESTM%E6%8C%87%E4%BB%A4%E8%AF%A6%E8%A7%A3/28458801_1373268720rnOs.png)

#### LDMFD SP!,{R0,R1,R2,R14}
​          满递减出栈,R13为基址地址
​          效果图:

![img](ARM%E6%8C%87%E4%BB%A4%E4%B8%ADLDM%E4%B8%8ESTM%E6%8C%87%E4%BB%A4%E8%AF%A6%E8%A7%A3/28458801_1373268790DHOz.png)

### EA
#### STMEA SP!,{R0,R1,R2,R14}
​         空递增入栈,R13为基址地址
​         效果图:

![img](ARM%E6%8C%87%E4%BB%A4%E4%B8%ADLDM%E4%B8%8ESTM%E6%8C%87%E4%BB%A4%E8%AF%A6%E8%A7%A3/28458801_1373268854LaKz.png)

#### LDMEA SP!,{R0,R1,R2,R14}
​        空递增出栈,R13为基址地址
​         效果图:

![img](ARM%E6%8C%87%E4%BB%A4%E4%B8%ADLDM%E4%B8%8ESTM%E6%8C%87%E4%BB%A4%E8%AF%A6%E8%A7%A3/28458801_1373268937wCaZ.png)

### ED
#### STMED SP!,{R0,R1,R2,R14}
​         空递减入栈,R13为基址地址
​         效果图:

![img](ARM%E6%8C%87%E4%BB%A4%E4%B8%ADLDM%E4%B8%8ESTM%E6%8C%87%E4%BB%A4%E8%AF%A6%E8%A7%A3/28458801_1373269012Yqnw.png)

#### LDMED SP!,{R0,R1,R2,R14}
​         空递减出栈,R13为基址地址
​         效果图:

![img](ARM%E6%8C%87%E4%BB%A4%E4%B8%ADLDM%E4%B8%8ESTM%E6%8C%87%E4%BB%A4%E8%AF%A6%E8%A7%A3/28458801_13732690909cnn.png)
