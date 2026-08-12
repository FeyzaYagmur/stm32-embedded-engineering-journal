# Active-Low Button Controlled LED Sequence Pause System

This project demonstrates an active-low digital input configuration used to interactively halt a multi-stage GPIO output sequence. Under idle conditions, an internal pull-up resistor holds the input line HIGH (`PD2`), allowing an incremental LED pattern (Red -> Red+Blue -> Red+Blue+Green) to cycle continuously. Driving the input LOW via an active-low push-button immediately freezes the sequential execution state.

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **Actuators:** 3x External LEDs (Red, Blue, Green)
- **Input Component:** 1x Push-Button (Active-Low Topology with Internal Pull-Up)
- **Active Pins:** `PD2` (Digital Input), `PD3` (Red LED), `PD4` (Blue LED), `PD5` (Green LED)
- **Method:** Active-Low Input Sensing (`GPIO_PULLUP`) with Sequence Freeze Logic

## 🔍 Key Concepts Covered
- **Active-Low Input Topology:** Configuring `GPIO_PULLUP` resistors so that the default unpressed idle state reads logic HIGH (`GPIO_PIN_SET`) and pressing the switch connects the line to GND (`GPIO_PIN_RESET`).
- **Sequential Animation Freeze:** Utilizing active-low logical condition checks to gate execution clock delays (`HAL_Delay`), freezing the active state machine on demand.
- **Progressive Multi-Channel Output Control:** Sequential step-wise activation of GPIO ports to generate cumulative visual indication arrays.

## 💻 Complete Source Code (`main.c`)

```c
#include "main.h"

static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  __HAL_RCC_GPIOH_CLK_ENABLE();
  __HAL_RCC_GPIOD_CLK_ENABLE();

  /* Clear LED outputs initially */
  HAL_GPIO_WritePin(GPIOD, GPIO_PIN_3 | GPIO_PIN_4 | GPIO_PIN_5, GPIO_PIN_RESET);

  /* Configure PD3 (Red), PD4 (Blue), PD5 (Green) as Outputs */
  GPIO_InitStruct.Pin = GPIO_PIN_3 | GPIO_PIN_4 | GPIO_PIN_5;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(GPIOD, &GPIO_InitStruct);

  /* Configure PD2 as Active-Low Digital Input with Pull-Up */
  GPIO_InitStruct.Pin = GPIO_PIN_2;
  GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
  GPIO_InitStruct.Pull = GPIO_PULLUP;
  HAL_GPIO_Init(GPIOD, &GPIO_InitStruct);
}

int main(void)
{
  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();

  uint8_t step = 0;

  while (1)
  {
    /* If button is NOT pressed (Active-Low idle HIGH state), advance sequence */
    if (HAL_GPIO_ReadPin(GPIOD, GPIO_PIN_2) == GPIO_PIN_SET)
    {
      switch (step)
      {
        case 0:
          HAL_GPIO_WritePin(GPIOD, GPIO_PIN_3, GPIO_PIN_SET);   // Red
          HAL_GPIO_WritePin(GPIOD, GPIO_PIN_4, GPIO_PIN_RESET); // Blue
          HAL_GPIO_WritePin(GPIOD, GPIO_PIN_5, GPIO_PIN_RESET); // Green
          break;

        case 1:
          HAL_GPIO_WritePin(GPIOD, GPIO_PIN_3, GPIO_PIN_SET);   // Red
          HAL_GPIO_WritePin(GPIOD, GPIO_PIN_4, GPIO_PIN_SET);   // Blue
          HAL_GPIO_WritePin(GPIOD, GPIO_PIN_5, GPIO_PIN_RESET); // Green
          break;

        case 2:
          HAL_GPIO_WritePin(GPIOD, GPIO_PIN_3, GPIO_PIN_SET);   // Red
          HAL_GPIO_WritePin(GPIOD, GPIO_PIN_4, GPIO_PIN_SET);   // Blue
          HAL_GPIO_WritePin(GPIOD, GPIO_PIN_5, GPIO_PIN_SET);   // Green
          break;
      }

      step = (step + 1) % 3;
      HAL_Delay(400);
    }
    else
    {
      /* Button Pressed (LOW state): Freeze sequence */
      HAL_Delay(50);
    }
  }
}
