# 01 - LED Blinking with GPIO

In this project, I configured the onboard Green LED (PD12) on the STM32F407VG Discovery board. I implemented a simple blinking sequence with a 1000 ms delay using the `HAL_Delay` function to understand GPIO output configuration and basic hardware delays.

## Hardware & Configuration
- **Microcontroller:** STM32F407VGT6 (ARM Cortex-M4)
- **Green LED:** PD12 (GPIO_Output)

## Source Code (main.c)

```c
/* USER CODE BEGIN WHILE */
while (1)
{
    // Turn on Green LED (PD12) and wait for 1000 ms
    HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12, GPIO_PIN_SET);
    HAL_Delay(1000);

    // Turn off Green LED (PD12) and wait for 1000 ms
    HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12, GPIO_PIN_RESET);
    HAL_Delay(1000);

    /* USER CODE END WHILE */
}
