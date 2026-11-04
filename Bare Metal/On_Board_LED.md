# Project Name: Bare Metal On-Board LED

<P> Basic concept
To blink an LED, we need to:

Enable the GPIO peripheral clock.

Configure a GPIO pin as an output.

Set the GPIO pin HIGH.

Add a delay.

Set the GPIO pin LOW.

Repeat forever.
</P>

. STM32F446RE GPIO register addresses
For GPIOA and RCC:


| Register | Address |
| :--- | :--- |
| `RCC_AHB1ENR` | `0x40023830` |
| `GPIOA_MODER` | `0x40020000` |
| `GPIOA_OTYPER` | `0x40020004` |
| `GPIOA_OSPEEDR` | `0x40020008` |
| `GPIOA_PUPDR` | `0x4002000C` |
| `GPIOA_ODR` | `0x40020014` |
| `GPIOA_BSRR` | `0x40020018` |


2. Enable GPIOA clock
The GPIOA peripheral clock is controlled by bit 0 of RCC_AHB1ENR.


```
#define RCC_AHB1ENR (*(volatile unsigned int*)0x40023830)

RCC_AHB1ENR |= (1 << 0);
```

3. Configure PA5 as output
GPIO pins use two MODER bits per pin.

| MODER Value (2-bit) | Configuration |
| :--- | :--- |
| `00` | Input mode (reset state) |
| `01` | General-purpose output mode |
| `10` | Alternate function mode |
| `11` | Analog mode |

PA5 uses bits 11:10.

```
#define GPIOA_MODER (*(volatile unsigned int*)0x40020000)

GPIOA_MODER &= ~(3 << 10);
GPIOA_MODER |=  (1 << 10);
```

4. Complete bare-metal LED blinking code

```
#define RCC_AHB1ENR (*(volatile unsigned int*)0x40023830)

#define GPIOA_MODER (*(volatile unsigned int*)0x40020000)
#define GPIOA_ODR   (*(volatile unsigned int*)0x40020014)
#define GPIOA_BSRR  (*(volatile unsigned int*)0x40020018)

void delay(void)
{
    for (volatile unsigned int i = 0; i < 1000000; i++)
    {
        // Software delay
    }
}

int main(void)
{
    // 1. Enable GPIOA clock
    RCC_AHB1ENR |= (1 << 0);

    // 2. Configure PA5 as output
    GPIOA_MODER &= ~(3 << 10);
    GPIOA_MODER |=  (1 << 10);

    while (1)
    {
        // LED ON
        GPIOA_BSRR = (1 << 5);

        delay();

        // LED OFF
        GPIOA_BSRR = (1 << (5 + 16));

        delay();
    }
}
```


PA5 LED blinking
5. Understand BSRR register
The BSRR register is useful because it allows atomic GPIO set/reset operations.

Operation

Register operation

Set PA5 HIGH

GPIOA_BSRR = (1 << 5);

Set PA5 LOW

GPIOA_BSRR = (1 << 21);

The onboard LD2 LED is active HIGH:

PA5 = HIGH → LED ON

PA5 = LOW → LED OFF

6. GPIO configuration flow
[STM32 學習紀錄] GPIO 操作相關暫存器. [文章摘要 Abstract] | by RUU | Medium
 

text
        STM32F446RE
             |
             v
       RCC_AHB1ENR
       Enable GPIOA
             |
             v
       GPIOA_MODER
       Configure PA5
        as output
             |
             v
         GPIOA_BSRR
             |
       +-----+-----+
       |           |
       v           v
    Set PA5     Reset PA5
       |           |
       v           v
    LED ON      LED OFF
7. Important: Building the firmware
To run this on the actual STM32F446RE, you'll also need:

Startup assembly file (startup_stm32f446xx.s)

Linker script (.ld)

Reset handler and vector table

ARM GCC compiler

ST-Link for flashing


