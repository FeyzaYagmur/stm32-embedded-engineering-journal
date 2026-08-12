# Edge-Triggered GPIO LED Toggle Switch with Software Debouncing

This project demonstrates a bistable (latching/toggle) output control system using an edge-triggered external push-button input on an STM32 microcontroller. Rather than requiring continuous button pressure, each discrete press event (rising edge) flips the output logic state of an external LED (`OFF -> ON` or `ON -> OFF`), incorporating software debouncing to ensure single-trigger precision.

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **Actuator:** 1x External Red LED with Limiting Resistor
- **Input Component:** 1x Push-Button with External Pull-Down Resistor
- **Active Pins:** `PD2` (Digital Input from Button), `PD3` (Digital Output to LED)
- **Method:** Rising-Edge Detection with Software Debouncing Window

## 🔍 Key Concepts Covered
- **Edge-Triggered Toggle State Logic:** Converting momentary tactile switch inputs into persistent, bistable output states (`HAL_GPIO_TogglePin`).
- **Software Debounce Filtering:** Suppressing mechanical switch contact noise through time-delay gating (`HAL_Delay(50)`) to prevent multiple false trigger events.
- **State Memory Tracking:** Maintaining previous input pin states in RAM (`lastButtonState`) to isolate low-to-high logical transitions.

## 💻 Complete Source Code (`main.c` & GPIO Init)

Below is the complete C implementation written in STM32CubeIDE using native HAL drivers:

```c
#include "main.h"

/* GPIO Initialization Function */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  /* Enable GPIO Port Clocks */
  __HAL_RCC_GPIOH_CLK_ENABLE();
  __HAL_RCC_GPIOD_CLK_ENABLE();

  /* Clear LED pin output level initially */
  HAL_GPIO_WritePin(GPIOD, GPIO_PIN_3, GPIO_PIN_RESET);

  /* Configure PD3 (Red LED) as Output */
  GPIO_InitStruct.Pin = GPIO_PIN_3;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(GPIOD, &GPIO_InitStruct);

  /* Configure PD2 (External Button) as Input */
  GPIO_InitStruct.Pin = GPIO_PIN_2;
  GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
  GPIO_InitStruct.Pull = GPIO_NOPULL; // Pull-down resistor handled via hardware
  HAL_GPIO_Init(GPIOD, &GPIO_InitStruct);
}

int main(void)
{
  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();

  uint8_t lastButtonState = 0; // Tracks previous button state

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    uint8_t currentButtonState = HAL_GPIO_ReadPin(GPIOD, GPIO_PIN_2);

    /* Edge Detection: Detect Low to High transition (Button Press) */
    if (currentButtonState == GPIO_PIN_SET && lastButtonState == 0)
    {
      /* Invert the current LED state */
      HAL_GPIO_TogglePin(GPIOD, GPIO_PIN_3);

      /* Software debounce delay */
      HAL_Delay(50);
    }

    /* Save current button state for the next loop iteration */
    lastButtonState = currentButtonState;

    /* USER CODE END WHILE */
  }
}
