---
title: STM32控制直流电机
date: 2022-03-09 21:50:55
tags:
---
# 通过STM32CUBEMX来配置PWM控制电机

## 1.基本配置

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e2adc9c2-dc3b-4479-8f27-5ec41b1d9e2c/Untitled.png)

STM32CUBEMX的包等文件的配置在这里不做过多的叙述。

**前置的知识**

**1..所有外设初始化：HAL_Init()**

**2.系统时钟配置**

- RCC振荡器初始化：HAL_RCC_OsConfig()
- RCC时钟初始化：HAL_RCC_ClockConfig()
- 系统滴答定时器初始化：HAL_SYSTICK_Config()

**3.高级定时器初始化**

- 基本环境初始化：HAL_TIM_Base_Init()
- 时钟源配置：HAL_TIM_ConfigClockSource()
- 比较输出初始化（包含mcu硬件初始化）：HAL_TIM_PWM_Init()
    - 初始化：HAL_TIM_Base_MspInit()
    - 反初始化：HAL_TIM_Base_MspDeInit()
- 主输出模式同步：HAL_TIMEx_MasterConfigSynchronization()
- 刹车和死区时间配置：HAL_TIMEx_ConfigBreakDeadTime()
- 通道比较输出配置：HAL_TIM_PWM_ConfigChannel()
    - 定时器实际时钟频率为：72MHz/(定时器预分频+1)=36MHz
    - PWM频率为：定时器实际时钟频率/(定时器周期+1)，[定时器周期为900的话，36MHz/(900+1)=40Khz]
    - 实际PWM频率为：PWM频率/(高级定时器重复计数寄存器值+1)，[高级定时器重复计数寄存器值为0的话，40KHz/0+1=40KHz]
    - **PWM占空比为：定时器通道比较输出脉冲/定时器周期*100%，占空比为50%时电机不转。**[定时器通道比较输出脉冲为450，定时器周期为900的话，则占空比为50%，电机不转]
- GPIO引脚初始化
    - a.GPIO端口时钟使能：__HAL_RCC_GPIOA_CLK_ENABLE()
    - b.设置IO引脚：Pin
    - c.设置工作模式：Mode
    - d.设置速度：Speed
    - e.完成初始化：HAL_GPIO_Init()

**4.启动定时器：HAL_TIM_BASE_Start()**

**5.启动定时器通道和互补通道PWM输出：HAL_TIM_PWM_Start() 、 HAL_TIMEx_PWMN_Start()**

**处理流程：**

1.一定要选择对应的芯片，不要选错

2.设置RCC（系统时钟）

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c2b8425f-5e9d-4323-a9ba-e9f53ec2b78a/Untitled.png)

选择外部晶磁外部时钟源

3.设置串口

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1048db71-99b6-4fe6-99f9-0f0179405366/Untitled.png)

选择USART1，初始的波特率设置成115200

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8547ca99-1813-4d5e-b960-58b82d8f6e65/Untitled.png)

然后开启CH1和CH4通道，频率设置为10khz

**PWM频率计算公式 f = 定时器时钟频率 / [ (Period+1) *(Prescaler) ]= 72M / (7199+1) *1 = 10k HZ**

**占空比计算公式 = P = pulse/per *100%**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0f721aef-653b-41a2-a7bf-2eedb3c51f76/Untitled.png)

这里的TIM1时钟是72MHZ

5.使用定时器2设置中断（周期是10ms）

每10ms进行一次定时器中断，

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bd3294e8-05df-4c61-a1e4-f135448ac05c/Untitled.png)

打开更新定时器中断

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/dd0ecf92-6e9b-4268-9124-f69c802e722c/Untitled.png)

****7、编码器配置****

**STM32自带编码器配置，使用定时器4（TIM4_CH1和TIM4_CH2），打开更新定时器中断**

8****、中断优先级配置****

**因为编码器中断要发生在定时10ms中断内，故编码器中断的抢占优先级要大于定时10ms**

9.****配置时钟****

F**1系列芯片系统时钟为72MHzs**