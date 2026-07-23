# Active-Low Button Counter

This project explores an inverted digital input topology using an **Active-Low** push-button circuit configuration. Unlike standard active-high configurations, the input pin is continuously pulled high (to $V_{DD}$) via a pull-up resistor when idle, and transitions to low ($GND$) upon user activation.

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **External Components:** 1x Push-Button, 1x External/Internal Pull-Up Resistor
- **Active Pin:** `PD2` (Digital Input with Active-Low Topology)
- **Method:** Falling-Edge Active State Polling with Software Debounce

## 🔍 Key Concepts Covered
- **Active-Low Topology:** Understanding circuit designs where $GND$ represents the active signal condition (`GPIO_PIN_RESET`) and $V_{DD}$ represents the idle state (`GPIO_PIN_SET`).
- **Internal/External Pull-Up Configuration:** Utilizing pull-up resistors to maintain floating digital lines at a known stable logic level when switches are disengaged.
- **Inverted State Logic:** Adapting software edge-detection algorithms to register count increments on falling transitions (`SET` to `RESET`).

## 💻 Complete Source Code (`main.c` & GPIO Init)

Below is the complete hardware control logic and the peripheral configuration for the active-low button counter implementation:

```c
#include "main.h"

/* GPIO Initialization Function */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  /* Enable GPIO Port Clocks */
  __HAL_RCC_GPIOH_CLK_ENABLE();
  __HAL_RCC_GPIOD_CLK_ENABLE();

  /* Configure GPIO pin : PD2 (Active-Low Button Input with Internal Pull-Up) */
  GPIO_InitStruct.Pin = GPIO_PIN_2;
  GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
  GPIO_InitStruct.Pull = GPIO_PULLUP; // Active-Low: Default state is HIGH (3.3V)
  HAL_GPIO_Init(GPIOD, &GPIO_InitStruct);
}

/* Infinite Loop inside main() */
int main(void)
{
  /* Reset of all peripherals, Initializes the Flash interface and the Systick. */
  HAL_Init();

  /* Configure the system clock */
  SystemClock_Config();

  /* Initialize all configured peripherals */
  MX_GPIO_Init();

  uint32_t activeLowPressCount = 0;
  uint8_t buttonWasPressed = 0;

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Active-Low Read: Pin goes LOW (GPIO_PIN_RESET) when button is pressed */
    if (HAL_GPIO_ReadPin(GPIOD, GPIO_PIN_2) == GPIO_PIN_RESET)
    {
      /* Edge Detection: Catch transition from idle (HIGH) to active (LOW) */
      if (buttonWasPressed == 0)
      {
        activeLowPressCount++;     // Increment counter on press
        buttonWasPressed = 1;      // Set flag to avoid continuous incrementing
        HAL_Delay(50);             // Debounce window
      }
    }
    else
    {
      /* Button is released: Pin returns to idle state (GPIO_PIN_SET / HIGH) */
      buttonWasPressed = 0;
    }

    /* USER CODE END WHILE */
  }
}
