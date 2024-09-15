<div align='center'>
    <h1>灵活, 小巧, 可靠</h1>
</div>

<div align='center'>
    <h2>RM2024香港科技大学ENTERPRIZE</h2>
</div>

<div align='center'>
    <h1>G4主控板硬件开源</h1>
</div>


> 设计者：
>
> 文档撰写：
>
> 香港科技大学ENTERPRIZE战队


## 方案简介

整套主控板分为核心板(下板)和扩展板(上板), 两板之间使用80pin BTB接口连接; 核心板包含所有核心芯片, 而扩展板一般只用于引出接口, 需要特殊功能时则用作功能扩展

- MCU: STM32G473VET6
- IMU: ICM-42688P
- 磁力计: LIS3MDLTR
- TTL (硬件流控): CH343P *2 
- FDCAN *3
- RS485 (硬件流控) *1

## 项目背景 & 应用情况

在 RoboMaster 2024 赛季, 由于队内自研的上一代主控板 (F407) 已经无法满足对机器人控制的需求, 于是产生了设计新型主控板的需求

在经过反复考虑和对比后, 我们选择了STM32G473VET6作为主控芯片, 主要原因如下

- 共有三路FDCAN外设, 可以控制更多电机
- 支持RS485硬件流控功能, 提升板间通讯的稳定性
- 相较于H7系列复杂度较低, 开发难度与F4系列近似

同时, 新一代主控板也有缩小体积的需求, 整个主控板大小控制在 60×40×21mm, 与大疆C板相近, 便于机械安装

最终, G4主控板倍用于控制ENTERPRIZE战队本赛季所有新研发的机器人

## 技术细节 & 特殊设计

### UART

虽然STM32G473的FDCAN等外设数量较多, 但UART的数量却只有5个, 如果用传统的方法对UART和相应功能进行一一绑定, 那么UART数量则不足以实现所有需要的功能

为了解决此问题, G4主控板在设计时利用了MCU外设内部的MUX, 比如 UART1 可以配置为 PC4/PC5 或 PE0/PE1, 再与相应功能进行绑定, 以此来实现在不同应用场景中适应不同需求的能力

同时, 对于同一功能, G4主控板也提供了可以从多个外设的访问的选项; 比如对于接收机, 可以通过软件配置来使用 UART1 或 UART3 来接受数据

下面是具体的UART引脚配置, 以A/B/C后缀来表示同一UART的不同配置方式:

<div style="margin-left: 20px">

|        | TX   | RX   | DE   | CTS  | RTS  | 功能          |
| -----  | ---- | ---- | ---- | ---- | ---- | ----          |
| UART1A | PC4  | PC5  | -    | -    | -    | 裁判系统/图传  |
| UART1B | PE0  | PE1  | -    | -    | -    | 通用          |
| UART1C | -    | PA10 | -    | -    | -    | 接收机        |
| UART2A | PA2  | PA3  | -    | PA0  | PA1  | TTL1          |
| UART2B | PD5  | PD6  | -    | -    | -    | 通用          |  
| UART3A | PD8  | PD9  | -    | PD11 | PD12 | TTL2          |
| UART3B | PB10 | PE15 | -    | -    | -    | 通用          |  
| UART3C | -    | PB11 | -    | -    | -    | 接收机        | 
| UART4  | PC10 | PC11 | PA15 | -    | -    | RS485        |   
| UART5  | PC12 | PD2  | -    | -    | -    | 裁判系统/图传 |
</div>

下面是部分机器人上的UART功能分配

<div style="margin-left: 20px">

|                   | UART1 | UART2 | UART3 | UART4 | UART5 |
| -----             | ----  | ----  | ----  | ----  | ----  |
| 步兵云台 | [C] 接收机 | [A] TTL1 | RESV | RS485 | [A] 图传 |
| 英雄云台 | [C] 接收机 | [A] TTL1 | RESV | RS485 | [A] 图传 |
| 哨兵云台 | [C] 接收机 | [A] TTL1 | [A] TTL2 | RS485 | [A] 图传 |
| 工程(单板) | [A] 裁判系统 | [B] 总线舵机  |  [C] 接收机 | RS485 | [A] 图传|
| 通用底盘  | RESV | RESV | RESV | RS485 | 裁判系统 |
</div>

## IMU恒温

在测试G板原型板经过测试发现












【RM2024-G4主控板硬件开源】香港科技大学-ENTERPRIZE战队