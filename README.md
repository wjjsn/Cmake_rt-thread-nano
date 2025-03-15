# rt-thread nano 4.1.1 cmake version
 - 目前完美支持小华单片机`hc32f4a0xi`,其它单片机要自行验证
 - 只测试过arm-none-eabi-gcc编译,去除了很多警告
## 需要自己实现的函数
 - `void hardware_init(void)`该函数用于初始化硬件资源,下面是初始化示例,该函数中的`SysTick_Configuration`函数也很重要,是`rt-thread`系统的tick,如果写错操作系统会无法运行,非小华单片机需要自己去rt-thread查找他们的写法,`rt_hw_us_delay`是rt-thread用到的一个函数
 - `void rt_hw_console_output(const char *str)` 该函数是`rt_kprintf`输出信息的端口,可以自己重定向到串口输出
 - `char rt_hw_console_getchar(void)`该函数是finsh输入口的函数,可以自己重定向到串口输入
 ```bash
 static void  SysTick_Configuration(void);

void hardware_init(void)
{
  /* Peripheral registers write unprotected */
  LL_PERIPH_WE(LL_PERIPH_ALL);
  BSP_CLK_Init();
  BSP_XTAL32_Init();
  init_gpio();
  init_debug();
  i2c1_init();
  i2c2_init();
  i2c3_init();
  spi4_init();
  /* Configure the SysTick */
  SystemCoreClockUpdate();
  SysTick_Configuration();

}

/*******************************************************************************
 * Function Name  : SysTick_Configuration
 * Description    : Configures the SysTick for OS tick.
 * Input          : None
 * Output         : None
 * Return         : None
 *******************************************************************************/
static void  SysTick_Configuration(void)
{
    stc_clock_freq_t stcClkFreq;
    rt_uint32_t cnts;

    CLK_GetClockFreq(&stcClkFreq);

    cnts = (rt_uint32_t)stcClkFreq.u32HclkFreq / RT_TICK_PER_SECOND;

    SysTick_Config(cnts);
}

void rt_hw_us_delay(rt_uint32_t us)
{
    uint32_t start, now, delta, reload, us_tick;
    start = SysTick->VAL;
    reload = SysTick->LOAD;
    us_tick = SystemCoreClock / 1000000UL;

    do
    {
        now = SysTick->VAL;
        delta = start > now ?  start - now : reload + start - now;
    }
    while (delta < us_tick * us);
}

void rt_hw_console_output(const char *str)
{
    extern void rt_enter_critical(void);
    rt_enter_critical(); 
	while (*str!='\0'){ 
        if (*str=='\n') { 
            while (RESET == USART_GetStatus(UART_DEBUG_UNIT, USART_FLAG_TX_EMPTY)){}
	        USART_WriteData(UART_DEBUG_UNIT, '\r'); 
        } 
        while (RESET == USART_GetStatus(UART_DEBUG_UNIT, USART_FLAG_TX_EMPTY)){}
        USART_WriteData(UART_DEBUG_UNIT, *str++); 
	} 
    extern void rt_exit_critical(void);
    rt_exit_critical();
}

char rt_hw_console_getchar(void)
{
    /* Note: the initial value of ch must < 0 */
    int ch = -1;
    if (SET == USART_GetStatus(UART_DEBUG_UNIT, USART_FLAG_RX_FULL)){
        
        ch = USART_ReadData(UART_DEBUG_UNIT);
    }
    
    return ch;
}
 ```
## 使用方法
- 可以使用子模块方式管理
```bash
git submodule add https://github.com/nczyw/rt-thread.git rt-thread
```
- 在顶层cmake中添加,即可使用
```bash
add_subdirectory("fatfs")
```