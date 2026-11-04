
| Register | Purpose |
| :--- | :--- |
| `RCC_APB1ENR` | Enable I2C1 clock |
| `RCC_AHB1ENR` | Enable GPIOB clock |
| `GPIOB_MODER` | Configure alternate function |
| `GPIOB_AFRH` | Select I2C alternate function |
| `GPIOB_OTYPER`| Configure open-drain |
| `I2C_CR1`     | Enable and control I2C |
| `I2C_CR2`     | Configure peripheral clock frequency |
| `I2C_CCR`     | Configure I2C clock |
| `I2C_TRISE`   | Configure rise time |
| `I2C_SR1`     | Check communication status |
| `I2C_SR2`     | Check bus status |
| `I2C_DR`      | Transmit and receive data |


| Function | Responsibility |
| :--- | :--- |
| `I2C1_Init()` | Initialize GPIO and I2C peripherals |
| `I2C1_Start()` | Generate the I2C START condition on the bus |
| `I2C1_Stop()` | Generate the I2C STOP condition on the bus |
| `I2C1_Write()` | Send and write register data to the slave device |
| `I2C1_Read()` | Read register data from the slave device |
| `I2C1_WaitFlag()` | Monitor hardware status flags safely with a timeout mechanism |

