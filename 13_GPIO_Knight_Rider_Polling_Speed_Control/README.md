# Knight Rider Polling Speed Control

This project controls the speed of a sequential Knight Rider LED chaser using digital polling on an external push-button switch. It demonstrates how to update timing parameters dynamically during execution, altering scanning intervals across multi-pin sequences.

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **External Components:** 4x External LEDs, 1x Push-Button, 5x Resistors
- **Active Pins:** `PD0`, `PD1`, `PD2`, `PD3` (LED Bus Outputs), `PA0` (Button Input)
- **Method:** Polling-Based Dynamic Delay Speed Selection

## 🔍 Key Concepts Covered
- **Dynamic Speed Modulation:** Altering execution delay values (`speedDelay`) in real-time based on physical button inputs.
- **In-Loop Polling Execution:** Interrogating input pin states across sequential output cycles to ensure responsiveness during pattern execution.
- **Timing Boundaries:** Switching between predefined step delays (e.g., Fast: 50ms, Medium: 150ms, Slow: 300ms) using state iteration.

## 💻 Complete Source Code (`main.c` & GPIO Init)

Below is the complete hardware control logic and peripheral configuration for the speed-controlled Knight Rider chaser:

```c
#include "main.h"

/* GPIO Initialization Function */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  /* Enable Clocks */
  __HAL_RCC_GPIOH_CLK_ENABLE();
  __HAL_RCC_GPIOA_CLK_ENABLE();
  __HAL_RCC_GPIOD_CLK_ENABLE();

  /* Configure Output Level for PD0, PD1, PD2, PD3 */
  HAL_GPIO_WritePin(GPIOD, GPIO_PIN_0|GPIO_PIN_1|GPIO_PIN_2|GPIO_PIN_3, GPIO_PIN_RESET);

  /* Configure Pins PD0, PD1, PD2, PD3 as LED Outputs */
  GPIO_InitStruct.Pin = GPIO_PIN_0|GPIO_PIN_1|GPIO_PIN_2|GPIO_PIN_3;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(GPIOD, &GPIO_InitStruct);

  /* Configure Pin PA0 as Button Input */
  GPIO_InitStruct.Pin = GPIO_PIN_0;
  GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
  GPIO_InitStruct.Pull = GPIO_NOPULL; // External pull-down
  HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);
}

/* Infinite Loop inside main() */
int main(void)
{
  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();

  uint16_t leds[4] = {GPIO_PIN_0, GPIO_PIN_1, GPIO_PIN_2, GPIO_PIN_3};
  uint32_t speedDelay = 150; // Default speed
  uint8_t speedState = 0;
  uint8_t lastBtn = 0;

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    uint8_t currentBtn = HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_0);

    /* Check speed toggle button on PA0 */
    if (currentBtn == GPIO_PIN_SET && lastBtn == 0)
    {
      speedState = (speedState + 1) % 3;
      if (speedState == 0) speedDelay = 150;      // Medium
      else if (speedState == 1) speedDelay = 50;  // Fast
      else speedDelay = 300;                      // Slow
      
      HAL_Delay(50); // Debounce
    }
    lastBtn = currentBtn;

    /* Scan Forward */
    for (int i = 0; i < 4; i++)
    {
      HAL_GPIO_WritePin(GPIOD, leds[i], GPIO_PIN_SET);
      HAL_Delay(speedDelay);
      HAL_GPIO_WritePin(GPIOD, leds[i], GPIO_PIN_RESET);
    }

    /* Scan Backward */
    for (int i = 2; i > 0; i--)
    {
      HAL_GPIO_WritePin(GPIOD, leds[i], GPIO_PIN_SET);
      HAL_Delay(speedDelay);
      HAL_GPIO_WritePin(GPIOD, leds[i], GPIO_PIN_RESET);
    }

    /* USER CODE END WHILE */
  }
}
