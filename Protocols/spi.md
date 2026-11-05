### Step 3: Understand SPI registers
SPI2 is connected to the APB1 bus.

| Register | Purpose |
| :--- | :--- |
| `RCC_AHB1ENR` | Enable GPIOB clock |
| `RCC_APB1ENR` | Enable SPI2 clock |
| `GPIOB_MODER` | Configure pins |
| `GPIOB_AFRH`  | Select alternate function |
| `SPI2_CR1`    | Configure SPI |
| `SPI2_CR2`    | Configure additional SPI options |
| `SPI2_SR`     | Check transmission status |
| `SPI2_DR`     | Transmit/receive data |

<P>
Configure GPIO pins
Configure:

PB12 as normal GPIO output for CS.

PB13 as AF5.

PB14 as AF5.

PB15 as AF5.

Configure SPI pins as alternate function, push-pull, with appropriate output speed.

Keep CS HIGH when the flash is not selected.
  
</P>
