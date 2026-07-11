# Step 3: Simultaneous Multi-LED Drive Configuration

In this project, I expanded the basic input/output operations to control all 4 on-board LEDs (Green `GPIO_PIN_12`, Orange `GPIO_PIN_13`, Red `GPIO_PIN_14`, Blue `GPIO_PIN_15`) simultaneously on the STM32 Discovery board. 

The core objective is to execute concurrent bit operations using a single HAL command structure instead of sequential, line-by-line pin manipulation.

## Hardware Configuration
* **Microcontroller Pins:** `PD12`, `PD13`, `PD14`, `PD15`.
* **Working Principle:** By using bitwise OR operations, multiple pins are targeted in a single execution cycle, supplying 3.3V to all four onboard LEDs at the exact same time.

## Algorithmic Implementation & Code Explanation
Instead of calling `HAL_GPIO_WritePin` four separate times, the system optimizes register outputs by grouping the pin definitions together using the bitwise OR (`|`) operator:

```c
/* Infinite loop */
/* USER CODE BEGIN WHILE */
while (1)
{
    HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12 | GPIO_PIN_13 | GPIO_PIN_14 | GPIO_PIN_15, GPIO_PIN_SET);
    HAL_Delay(1000);
    HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12 | GPIO_PIN_13 | GPIO_PIN_14 | GPIO_PIN_15, GPIO_PIN_RESET);
    HAL_Delay(1000);
    
    /* USER CODE END WHILE */
    /* USER CODE BEGIN 3 */
}
