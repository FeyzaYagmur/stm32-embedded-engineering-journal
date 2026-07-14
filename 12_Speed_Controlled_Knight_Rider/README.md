# Step 12: Dynamic Speed-Controlled Knight Rider Scanner

In this project, I designed a dynamically modulated LED scanning sequence using the on-board User Button (`PA0`) and the four on-board LEDs (`PD12`, `PD13`, `PD14`, `PD15`) of the STM32 Discovery board. The application uses an array-based structure to handle sequential outputs and dynamically adjusts the timing loop on button transition.

The objective is to master array-based peripheral mapping, on-board active-high button polling, and conditional runtime delay scaling.

## Hardware Configuration
* **Input Interface:** On-Board Blue User Button connected to `PA0` (GPIOA, Pin 0).
* **Output Peripherals:** On-Board LEDs mapped to `PD12` (Green), `PD13` (Orange), `PD14` (Red), and `PD15` (Blue).
* **Working Principle:** The on-board User Button is hardware-wired as an Active-High input. By default, it reads logic LOW (`RESET`). When pressed, it connects to VCC and registers a logic HIGH (`SET`). The software constantly polls this pin to toggle between a fast-scan state (50ms execution delay) and an idle slow-scan state (300ms execution delay).

## Code Architecture & Dynamic Timing Control
The firmware utilizes memory arrays to cleanly loop back and forth through the output pins without redundant line-by-line function blocks:

```c
/* Infinite loop */
/* USER CODE BEGIN WHILE */
while (1)
{
    // Local array mapping for the 4 on-board LED pins
    uint16_t ledArray[4] = {GPIO_PIN_12, GPIO_PIN_13, GPIO_PIN_14, GPIO_PIN_15};
    
    // Check if the on-board User Button (PA0) is pressed (Active-High)
    if (HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_0) == GPIO_PIN_SET)
    {
        // Fast Scanning State (50ms Delay)
        for (int i = 0; i < 4; i++)
        {
            HAL_GPIO_WritePin(GPIOD, ledArray[i], GPIO_PIN_SET);
            HAL_Delay(50);
            HAL_GPIO_WritePin(GPIOD, ledArray[i], GPIO_PIN_RESET);
        }
        for (int i = 2; i > 0; i--)
        {
            HAL_GPIO_WritePin(GPIOD, ledArray[i], GPIO_PIN_SET);
            HAL_Delay(50);
            HAL_GPIO_WritePin(GPIOD, ledArray[i], GPIO_PIN_RESET);
        }
    }
    else
    {
        // Slow Scanning State (300ms Delay)
        for (int i = 0; i < 4; i++)
        {
            HAL_GPIO_WritePin(GPIOD, ledArray[i], GPIO_PIN_SET);
            HAL_Delay(300);
            HAL_GPIO_WritePin(GPIOD, ledArray[i], GPIO_PIN_RESET);
        }
        for (int i = 2; i > 0; i--)
        {
            HAL_GPIO_WritePin(GPIOD, ledArray[i], GPIO_PIN_SET);
            HAL_Delay(300);
            HAL_GPIO_WritePin(GPIOD, ledArray[i], GPIO_PIN_RESET);
        }
    }
    
    /* USER CODE END WHILE */
}
