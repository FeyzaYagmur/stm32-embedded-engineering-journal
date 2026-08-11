# Button Level-Triggered Sequential Multi-LED Stepper Pattern

This project implements a continuous level-triggered sequential LED pattern generator on an STM32 microcontroller. As long as the external push-button input remains active (pressed state), the firmware executes a repeating time-delay loop that incrementally energizes multi-channel GPIO output pins (`1 LED ON -> 2 LEDs ON -> 3 LEDs ON -> Cycle Reset`), demonstrating state-machine pattern generation based on held input conditions.

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **Actuators:** 3x External LEDs (Red, Yellow, Green)
- **Input:** 1x Push-Button with External Pull-Down Resistor
- **Active Pins:** `PD2` (Button Input), `PD3` (LED 1 Output), `PD4` (LED 2 Output), `PD5` (LED 3 Output)
- **Method:** Level-Triggered High State Active Loop (`HAL_GPIO_ReadPin`)

## 🔍 Key Concepts Covered
- **Level-Triggered State Execution:** Executing continuous sequence patterns as long as an input pin retains its active HIGH logic level, as opposed to edge-triggered single events.
- **Sequential Step Pattern Generation:** Step-by-step activation of multi-channel GPIO outputs in a circular loop structure (`0 -> 1 -> 2 -> 3 -> 0`).
- **Timing Gated Output Transitions:** Controlling step pattern transition speeds using delay intervals (`HAL_Delay(300)`) inside active polling loops.

## 💻 Complete Source Code (`main.c` / Videodaki Çalışmaya Birebir Uygun)

Below is the complete C implementation written in STM32CubeIDE using native HAL drivers:

```c
#include "main.h"

/* GPIO Initialization Function */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  /* GPIO Ports Clock Enable */
  __HAL_RCC_GPIOH_CLK_ENABLE();
  __HAL_RCC_GPIOD_CLK_ENABLE();

  /* Set initial pin levels to RESET */
  HAL_GPIO_WritePin(GPIOD, GPIO_PIN_3 | GPIO_PIN_4 | GPIO_PIN_5, GPIO_PIN_RESET);

  /* Configure GPIO pins : PD3, PD4, PD5 (LED Outputs) */
  GPIO_InitStruct.Pin = GPIO_PIN_3 | GPIO_PIN_4 | GPIO_PIN_5;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(GPIOD, &GPIO_InitStruct);

  /* Configure GPIO pin : PD2 (External Button Input) */
  GPIO_InitStruct.Pin = GPIO_PIN_2;
  GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
  GPIO_InitStruct.Pull = GPIO_NOPULL; // Externally pulled down via hardware resistor
  HAL_GPIO_Init(GPIOD, &GPIO_InitStruct);
}

int main(void)
{
  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();

  uint8_t patternStep = 0; // Steps: 0 = 1 LED, 1 = 2 LEDs, 2 = 3 LEDs

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Check if button is held active (High Level Trigger) */
    if (HAL_GPIO_ReadPin(GPIOD, GPIO_PIN_2) == GPIO_PIN_SET)
    {
      /* Execute pattern based on active step count */
      switch (patternStep)
      {
        case 0:
          /* Step 0: 1 LED Active */
          HAL_GPIO_WritePin(GPIOD, GPIO_PIN_3, GPIO_PIN_SET);
          HAL_GPIO_WritePin(GPIOD, GPIO_PIN_4, GPIO_PIN_RESET);
          HAL_GPIO_WritePin(GPIOD, GPIO_PIN_5, GPIO_PIN_RESET);
          break;

        case 1:
          /* Step 1: 2 LEDs Active */
          HAL_GPIO_WritePin(GPIOD, GPIO_PIN_3, GPIO_PIN_SET);
          HAL_GPIO_WritePin(GPIOD, GPIO_PIN_4, GPIO_PIN_SET);
          HAL_GPIO_WritePin(GPIOD, GPIO_PIN_5, GPIO_PIN_RESET);
          break;

        case 2:
          /* Step 2: All 3 LEDs Active */
          HAL_GPIO_WritePin(GPIOD, GPIO_PIN_3, GPIO_PIN_SET);
          HAL_GPIO_WritePin(GPIOD, GPIO_PIN_4, GPIO_PIN_SET);
          HAL_GPIO_WritePin(GPIOD, GPIO_PIN_5, GPIO_PIN_SET);
          break;
      }

      /* Advance to next step in sequence */
      patternStep = (patternStep + 1) % 3;

      /* Pattern step transition delay */
      HAL_Delay(300);
    }
    else
    {
      /* Reset state and turn OFF all LEDs when button is released */
      HAL_GPIO_WritePin(GPIOD, GPIO_PIN_3 | GPIO_PIN_4 | GPIO_PIN_5, GPIO_PIN_RESET);
      patternStep = 0;
    }

    /* USER CODE END WHILE */
  }
}
