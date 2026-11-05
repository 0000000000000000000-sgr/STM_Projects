```
        Computer
            |
            | USB
            |
         ST-Link
            |
            | UART
            |
      STM32F446RE
            |
            v
       USART2
            |
            v
       main.c
```
2. Understand UART
UART stands for Universal Asynchronous Receiver Transmitter.

Unlike I2C and SPI, UART doesn't require a clock wire.

It generally uses two communication lines:

| Signal | Description |
| :--- | :--- |
| `TX` | Transmit data |
| `RX` | Receive data |
| `GND` | Common ground |

UART communication requires both devices to agree on:

Baud rate

Data bits

Parity

Stop bits


| Parameter | Configuration |
| :--- | :--- |
| **Baud rate** | `115200` |
| **Data bits** | `8` |
| **Parity** | `None` |
| **Stop bits** | `1` |
| **Mode** | `TX + RX` |

This is commonly called **115200 8N1**.
4. Understand USART registers
USART2 is connected to the APB1 bus.

Important register addresses:

| Register | Address | Purpose |
| :--- | :--- | :--- |
| `RCC_AHB1ENR` | `0x40023830` | GPIO clock |
| `RCC_APB1ENR` | `0x40023840` | USART2 clock |
| `GPIOA_MODER` | `0x40020000` | GPIO mode |
| `GPIOA_AFRL`  | `0x40020020` | Alternate function |
| `USART2_SR`   | `0x40004400` | Status |
| `USART2_DR`   | `0x40004404` | Data |
| `USART2_BRR`  | `0x40004408` | Baud rate |
| `USART2_CR1`  | `0x4000440C` | Control 1 |
| `USART2_CR2`  | `0x40004410` | Control 2 |
| `USART2_CR3`  | `0x40004414` | Control 3 |

For the register definitions:
'''
#define RCC_AHB1ENR (*(volatile unsigned int*)0x40023830)
#define RCC_APB1ENR (*(volatile unsigned int*)0x40023840)

#define GPIOA_MODER (*(volatile unsigned int*)0x40020000)
#define GPIOA_AFRL  (*(volatile unsigned int*)0x40020020)

#define USART2_SR   (*(volatile unsigned int*)0x40004400)
#define USART2_DR   (*(volatile unsigned int*)0x40004404)
#define USART2_BRR  (*(volatile unsigned int*)0x40004408)
#define USART2_CR1  (*(volatile unsigned int*)0x4000440C)
#define USART2_CR2  (*(volatile unsigned int*)0x40004410)
'''

5. Step-by-step UART configuration
Step 1: Enable clocks

'''
RCC_AHB1ENR |= (1 << 0);  // GPIOA clock
RCC_APB1ENR |= (1 << 17); // USART2 clock
'''

Step 2: Configure PA2 and PA3
Both pins use alternate function AF7.

Configure:

PA2 as alternate function.

PA3 as alternate function.

PA2 and PA3 as AF7.

Push-pull output for PA2.

The relevant register operations:


c
GPIOA_MODER &= ~(0xF << 4);
GPIOA_MODER |=  (0xA << 4);

// PA2 = AF7, PA3 = AF7
GPIOA_AFRL &= ~(0xFF << 8);
GPIOA_AFRL |=  (0x77 << 8);
Step 3: Configure baud rate
The USART baud-rate register depends on the APB1 peripheral clock.

For oversampling by 16:

U
S
A
R
T
D
I
V
=
P
C
L
K
1
16
×
B
a
u
d
R
a
t
e
USARTDIV= 
16×BaudRate
PCLK1
​
 
Assuming PCLK1 = 16 MHz:

U
S
A
R
T
D
I
V
=
16000000
16
×
115200
USARTDIV= 
16×115200
16000000
​
 
U
S
A
R
T
D
I
V
≈
8.6806
USARTDIV≈8.6806
The corresponding BRR value is approximately:


c
USART2_BRR = 0x008B;
This assumes the peripheral clock is actually 16 MHz.

Step 4: Configure USART control register
We need:

UE = 1: USART enabled.

TE = 1: Transmitter enabled.

RE = 1: Receiver enabled.

M = 0: Eight-bit data.

PCE = 0: No parity.


c
USART2_CR1 = 0;

USART2_CR1 |= (1 << 13); // USART enable
USART2_CR1 |= (1 << 3);  // Transmitter enable
USART2_CR1 |= (1 << 2);  // Receiver enable
Eight-bit data, no parity, and one stop bit are the defaults after reset.

6. Create the UART driver
Create these files:


text
drivers/
    uart/
        uart.c
        uart.h
uart.h


c
#ifndef UART_H
#define UART_H

#include <stdint.h>

void UART2_Init(void);

void UART2_SendChar(char data);

char UART2_ReceiveChar(void);

void UART2_SendString(const char *str);

#endif
uart.c


c
#include "uart.h"

// UART2_Init:
// Enable clocks
// Configure PA2 and PA3
// Configure baud rate
// Enable USART2

void UART2_SendChar(char data)
{
    // Wait until TXE is set
    while (!(USART2_SR & (1 << 7)));

    // Write data
    USART2_DR = data;
}

char UART2_ReceiveChar(void)
{
    // Wait until RXNE is set
    while (!(USART2_SR & (1 << 5)));

    return (char)(USART2_DR & 0xFF);
}

void UART2_SendString(const char *str)
{
    while (*str)
    {
        UART2_SendChar(*str++);
    }
}
This is the driver structure. Add the register definitions and timeout/error handling in the implementation.

7. Main application

c
#include "uart.h"

int main(void)
{
    UART2_Init();

    UART2_SendString("STM32F446RE Bare Metal\r\n");

    UART2_SendString("UART Initialized\r\n");

    while (1)
    {
        char received;

        received = UART2_ReceiveChar();

        UART2_SendString("Received: ");

        UART2_SendChar(received);

        UART2_SendString("\r\n");
    }
}
Expected behavior:

Serial terminal

Expected output
115200 baud · 8N1

STM32F446RE Bare Metal
UART Initialized
Received: H
Received: e
Received: l
Received: l
Received: o

Waiting for input...

Important: Add timeouts to the transmit and receive functions before using them in a production application. Also handle UART overrun and framing errors.
