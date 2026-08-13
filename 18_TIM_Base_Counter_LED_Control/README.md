# Timer Counter Hardware Polling & Output Control

This project introduces hardware timing functionality using the General-Purpose Timer 2 (TIM2) peripheral. It demonstrates how to initialize the timer in basic time-base counter mode and poll the hardware counter register (`CNT`) in real-time to trigger multi-channel onboard LED outputs without relying on blocking delay functions.

##  Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **Peripherals Used:** TIM2 (General Purpose 32-bit Timer), Onboard LEDs
- **Active Pins:** `PD12` (Green LED), `PD13` (Orange LED), `PD14` (Red LED), `PD15` (Blue LED)
- **Method:** Non-Blocking Hardware Timer Register Polling via `__HAL_TIM_GET_COUNTER()`

##  Key Concepts Covered
- **Hardware Time-Base Generation:** Utilizing internal clock sources and prescaler/auto-reload register (ARR) configurations to establish precise tick intervals.
- **Counter Register Interrogation:** Direct extraction of the instantaneous counter value from hardware registers using the `__HAL_TIM_GET_COUNTER()` macro.
- **Non-Blocking Logic Execution:** Decoupling timing constraints from CPU blocking routines (`HAL_Delay`), allowing simultaneous background task processing.

##  Complete Source Code (`main.c` & TIM2 Init)

Below is the complete C implementation written in STM32CubeIDE using native HAL drivers:

```c
#include "main.h"

TIM_HandleTypeDef htim2;

/* TIM2 Peripheral Initialization Function */
static void MX_TIM2_Init(void)
{
  TIM_ClockConfigTypeDef sClockSourceConfig = {0};
  TIM_MasterConfigTypeDef sMasterConfig = {0};

  htim2.Instance = TIM2;
  htim2.Init.Prescaler = 8399;           // Prescale 84MHz clock to 10kHz tick frequency
  htim2.Init.CounterMode = TIM_COUNTERMODE_UP;
  htim2.Init.Period = 1999;              // Auto-reload value (ARR)
  htim2.Init.ClockDivision = TIM_CLOCKDIVISION_DIV1;
  htim2.Init.AutoReloadPreload = TIM_AUTORELOAD_PRELOAD_DISABLE;
  HAL_TIM_Base_Init(&htim2);

  sClockSourceConfig.ClockSource = TIM_CLOCKSOURCE_INTERNAL;
  HAL_TIM_ConfigClockSource(&htim2, &sClockSourceConfig);

  sMasterConfig.MasterOutputTrigger = TIM_TRGO_RESET;
  sMasterConfig.MasterSlaveMode = TIM_MASTERSLAVEMODE_DISABLE;
  HAL_TIMEx_MasterConfigSynchronization(&htim2, &sMasterConfig);
}

/* GPIO Initialization Function */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  /* Enable GPIO Clocks */
  __HAL_RCC_GPIOH_CLK_ENABLE();
  __HAL_RCC_GPIOD_CLK_ENABLE();

  /* Reset Onboard LED Pins */
  HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12|GPIO_PIN_13|GPIO_PIN_14|GPIO_PIN_15, GPIO_PIN_RESET);

  /* Configure GPIOD Pins 12, 13, 14, 15 as Push-Pull Outputs */
  GPIO_InitStruct.Pin = GPIO_PIN_12|GPIO_PIN_13|GPIO_PIN_14|GPIO_PIN_15;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(GPIOD, &GPIO_InitStruct);
}

/* Infinite Loop inside main() */
int main(void)
{
  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();
  MX_TIM2_Init();

  /* Start TIM2 Base Unit */
  HAL_TIM_Base_Start(&htim2);

  uint16_t counter = 0;

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Read the instant 16-bit/32-bit hardware counter register value */
    counter = __HAL_TIM_GET_COUNTER(&htim2);

    /* Upper Threshold Check: Turn ON all onboard LEDs */
    if (counter == 1559)
    {
      HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12 | GPIO_PIN_13 | GPIO_PIN_14 | GPIO_PIN_15, GPIO_PIN_SET);
    }
    /* Reset Threshold Check: Turn OFF all onboard LEDs */
    else if (counter == 0)
    {
      HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12 | GPIO_PIN_13 | GPIO_PIN_14 | GPIO_PIN_15, GPIO_PIN_RESET);
    }

    /* USER CODE END WHILE */
  }
}
