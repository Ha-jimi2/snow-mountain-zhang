# 电控组第二次培训作业
- **贾政**
- ***20263001720***

流水灯实现思路：

在CubeMX中将A0命名为LED0，将A3命名为LED1,将A7命名为LED2，并配置为‘GPIO_Output’

生成了宏定义：
```c
#define LED0_Pin GPIO_PIN_0
#define LED0_GPIO_Port GPIOA
#define LED1_Pin GPIO_PIN_3
#define LED1_GPIO_Port GPIOA
#define LED2_Pin GPIO_PIN_7
#define LED2_GPIO_Port GPIOA
```
定义一个变量`step`记录当前亮灯的状态，通过`if-else`分支判断当前状态，将引脚输出设置为高电平以点亮LED，设置为低电平以熄灭LED，随后更新`step`进入下一个阶段

非阻塞延时原理说明；

基于`HAL_GetTick()`函数实现了时间戳差值法

首先定义
```c
#define LED_0 0
#define LED_1 1
#define LED_2 2
```
定义`lastTick`记录上一次切换LED的时间，定义`step`记录当前的状态

初始 此时时间为0，LED0亮，step==LED_0
```c
  uint32_t lastTick=0;
  uint8_t step = LED_0;
  HAL_GPIO_WritePin(LED0_GPIO_Port, LED0_Pin, GPIO_PIN_SET);
  HAL_GPIO_WritePin(LED1_GPIO_Port, LED1_Pin, GPIO_PIN_RESET);
  HAL_GPIO_WritePin(LED2_GPIO_Port, LED2_Pin, GPIO_PIN_RESET);
```
通过`HAL_GetTick()`获取现在的时间
```c
uint32_t currentTick=HAL_GetTick()
```
判断现在与上一次的时间差是否大于等于500ms
```c
if (currentTick-lastTick>=500)
```
条件满足，则更新触发时间
```c
lastTick=currentTick
```
然后根据当前step切换对应的LED状态
```c
if (LED_0==step)
      {
        HAL_GPIO_WritePin(LED0_GPIO_Port, LED0_Pin, GPIO_PIN_RESET);
        HAL_GPIO_WritePin(LED1_GPIO_Port, LED1_Pin, GPIO_PIN_SET);
        HAL_GPIO_WritePin(LED2_GPIO_Port, LED2_Pin, GPIO_PIN_RESET);
        step=LED_1;
      }
      else if (LED_1==step)
      {
        HAL_GPIO_WritePin(LED0_GPIO_Port, LED0_Pin, GPIO_PIN_RESET);
        HAL_GPIO_WritePin(LED1_GPIO_Port, LED1_Pin, GPIO_PIN_RESET);
        HAL_GPIO_WritePin(LED2_GPIO_Port, LED2_Pin, GPIO_PIN_SET);
        step=LED_2;
      }
      else if (LED_2==step)
      {
        HAL_GPIO_WritePin(LED0_GPIO_Port, LED0_Pin, GPIO_PIN_SET);
        HAL_GPIO_WritePin(LED1_GPIO_Port, LED1_Pin, GPIO_PIN_RESET);
        HAL_GPIO_WritePin(LED2_GPIO_Port, LED2_Pin, GPIO_PIN_RESET);
        step=LED_0;
      }
```
只要时间差没有达到500ms，cpu会略过这段代码去执行循环中的其他任务，摆脱了HAL_Delay()死等的方式，实现非阻塞延时
