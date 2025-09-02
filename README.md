forked from [nczyw/rt-thread](https://github.com/nczyw/rt-thread)
---
![RT-Thread的启动流程](https://www.rt-thread.org/document/site/rt-thread-version/rt-thread-nano/nano-port-principle/figures/startup1.png)

黄底的部分在[libcpu](https://github.com/RT-Thread/rt-thread/tree/master/libcpu)下选择对应的汇编文件，这个文件实现了开关中断、上下文切换等函数。
## 需要自己实现的函数
绿底部分主要实现两个函数。
其他部分均已完成，唯一要做的就是为系统提供心跳。
```c
void hardware_init(void)
{
    /*初始化其他硬件*/
	led_init_config(&LED1);
    /*初始化systick，1ms中断一次，系统频率为1000Hz*/
	SystemCoreClockUpdate();
	SysTick_Config(SystemCoreClock / 1000);
}
void SysTick_Handler(void)
{
    rt_interrupt_enter();
    rt_tick_increase();
    rt_interrupt_leave();
}
```
## 使用方法
添加该子模块

`git submodule add https://github.com/wjjsn/Cmake_rt-thread-nano.git rt-thread`

在顶层CMakeLists.txt中添加

`add_subdirectory("rt-thread")`