

<img width="1408" height="768" alt="image" src="https://github.com/user-attachments/assets/b7bfc58d-c556-43bf-b306-55f5768bca11" />

3. Understand STM32 timers
STM32F446RE has several hardware timers.

### Important timer registers:

| Register | Purpose |
| :--- | :--- |
| `TIM2_CR1` | Timer control |
| `TIM2_PSC` | Prescaler |
| `TIM2_ARR` | Auto-reload value |
| `TIM2_CCMR1` | Capture/compare mode |
| `TIM2_CCER` | Channel enable |
| `TIM2_CCR1` | Compare value |
| `TIM2_CNT` | Counter |
| `TIM2_EGR` | Event generation |


<p>
To visualize how your settings make this happen, here is how those pieces work together in a chain reaction:
1. The Base Speed: The internal clock gets slowed down by your Prescaler (PSC) to tick at exactly 1 MHz (1 million times a second).
2. The Total Loop (ARR): The counter counts those 1 MHz ticks up to your Counter Period (ARR) of 100, then resets. This defines the frequency (how fast the loop repeats).
3. The Duty Cycle (CH1 / CCR1): Channel 1 watches that count. It decides how much of that 100-tick loop the pin stays HIGH versus LOW based on your Pulse (CCR1) value.
So if you change Pulse (CCR1) to 50 in your code later, Channel 1 keeps the pin HIGH for 50 ticks and LOW for 50 ticks, giving you a perfect 50% duty cycle.
</p>


Step 1: Start the PWM (Put in main.c before your loop)

Add this line right after your peripherals initialize to turn the signal on:

```
c

HAL_TIM_PWM_Start(&htim1, TIM_CHANNEL_1);
```



Step 2: Change the Duty Cycle (Inside your while(1) loop)

Use this specific macro to change the Pulse (CCR1) value dynamically. Since your ARR is 100, the value you type here directly equals your duty cycle percentage (0 to 100):
```
c
// Set duty cycle to 25% (ON for 25 ticks, OFF for 75 ticks)
__HAL_TIM_SET_COMPARE(&htim1, TIM_CHANNEL_1, 25);
```



Complete Code:
```
/* Initialize all configured peripherals */
MX_GPIO_Init();
MX_TIM1_Init();

/* 1. Start the PWM signal on Channel 1 */
HAL_TIM_PWM_Start(&htim1, TIM_CHANNEL_1);

while (1)
{
    /* 2. Change duty cycle to 10% (Dim) */
    __HAL_TIM_SET_COMPARE(&htim1, TIM_CHANNEL_1, 10);
    HAL_Delay(1000); // Wait 1 second
    
    /* 3. Change duty cycle to 90% (Bright) */
    __HAL_TIM_SET_COMPARE(&htim1, TIM_CHANNEL_1, 90);
    HAL_Delay(1000); // Wait 1 second
}
```


5. Enable timer and GPIO clocks
TIM2 is connected to APB1.


c
#define RCC_AHB1ENR (*(volatile unsigned int*)0x40023830)
#define RCC_APB1ENR (*(volatile unsigned int*)0x40023840)
```
RCC_AHB1ENR |= (1 << 0); // GPIOA clock
RCC_APB1ENR |= (1 << 0); // TIM2 clock
6. Configure PA0 as alternate function
PA0 uses AF1 for TIM2_CH1.
```

c
```
#define GPIOA_MODER (*(volatile unsigned int*)0x40020000)
#define GPIOA_AFRL  (*(volatile unsigned int*)0x40020020)

GPIOA_MODER &= ~(3 << 0);
GPIOA_MODER |=  (2 << 0);

GPIOA_AFRL &= ~(0xF << 0);
GPIOA_AFRL |=  (1 << 0);
```

Any time a microchip needs to move data from the CPU's memory (parallel data) to a single wire (serial data), it must use a shift register. Because of this, you will always see:
1. A TX Empty flag (like TXE) to prevent the CPU from overwriting data.
2. An RX Not Empty flag (like RXNE) to prevent the CPU from reading garbage data before it fully arrives.

<img width="691" height="536" alt="image" src="https://github.com/user-attachments/assets/4c8cc7e1-a0a2-4eef-a11c-1e9b92ffe72d" />

Note: STM32 timer clocks can be different from PCLK1 when the APB1 prescaler is not 1. For this initial setup, we'll use the HSI clock with APB1 prescaler set to 1.

8. Configure PWM mode
The main settings are:

TIM2_CH1 enabled.

PWM mode 1.

Output compare preload enabled.

Auto-reload preload enabled.

Timer enabled.

The PWM mode is selected using TIM2_CCMR1.

PWM mode 1 uses OC1M = 110.

Create the driver files:


text
drivers/
    pwm/
        pwm.c
        pwm.h
pwm.h


c
#ifndef PWM_H
#define PWM_H

#include <stdint.h>

void PWM_Init(void);

void PWM_SetDutyCycle(uint16_t duty);

#endif
pwm.c should configure the timer and provide a duty-cycle update function.

9. Main application: LED fading
The application will gradually increase and decrease PWM duty cycle.

```
c
#include "pwm.h"

void delay(void)
{
    for (volatile unsigned int i = 0;
         i < 50000;
         i++);
}

int main(void)
{
    PWM_Init();

    while (1)
    {
        // Increase brightness
        for (int i = 0; i <= 1000; i += 10)
        {
            PWM_SetDutyCycle(i);
            delay();
        }

        // Decrease brightness
        for (int i = 1000; i >= 0; i -= 10)
        {
            PWM_SetDutyCycle(i);
            delay();
        }
    }
}
```
The LED should gradually fade in and fade out.


10. How PWM duty cycle is controlled
For PWM mode 1 with up-counting:

<img width="733" height="164" alt="image" src="https://github.com/user-attachments/assets/07b40238-77ab-48db-9e2b-936d63e4106f" />

If ARR = 999:
| CCR1 Value | Duty Cycle |
| :--- | :--- |
| `0` | 0% |
| `250` | 25% |
| `500` | 50% |
| `750` | 75% |
| `1000` | 100% |

For a 0–1000 input range, your driver can directly map the requested value to CCR1, with appropriate bounds checking.

5. Final project structure
Now your firmware project has four communication/control drivers.

```
stm32f446_baremetal/
│
├── startup/
│   └── startup_stm32f446xx.s
│
├── linker/
│   └── stm32f446re.ld
│
├── drivers/
│   ├── gpio/
│   │   ├── gpio.c
│   │   └── gpio.h
│   │
│   ├── uart/
│   │   ├── uart.c
│   │   └── uart.h
│   │
│   ├── i2c/
│   │   ├── i2c.c
│   │   └── i2c.h
│   │
│   ├── spi/
│   │   ├── spi.c
│   │   └── spi.h
│   │
│   └── pwm/
│       ├── pwm.c
│       └── pwm.h
│
├── src/
│   └── main.c
│
└── Makefile



```
