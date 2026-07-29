# Timer Base Interrupts & Callback Driven LED Toggling

This project advances hardware timing execution by implementing **Timer Interrupts (IT)** with the General-Purpose Timer 2 (TIM2) peripheral. By offloading time-keeping completely from the main execution loop (`while(1)`), the firmware executes background LED toggling asynchronously inside the hardware-triggered `HAL_TIM_PeriodElapsedCallback()` routine upon counter overflow events.

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **Peripherals Used:** TIM2 (32-bit Timer in Interrupt Mode), Onboard LEDs
- **Active Pins:** `PD12` (Green LED Output), `PD13` (Orange LED Output)
- **Method:** Asynchronous Interrupt Handling via `HAL_TIM_PeriodElapsedCallback()`

## 🔍 Key Concepts Covered
- **Asynchronous Interrupt Processing:** Offloading periodic routine execution from the CPU's main polling loop into hardware-triggered Interrupt Service Routines (ISR).
- **HAL Timer Callback Architecture:** Utilizing the weak-bound `HAL_TIM_PeriodElapsedCallback()` function to safely execute user code when a specific timer instance (`htim->Instance == TIM2`) triggers a Period Elapsed event.
- **Zero-CPU-Load Delay Architecture:** Achieving completely non-blocking LED state transitions while keeping the main application loop (`while(1)`) entirely empty for power efficiency and multitasking.

## 💻 Complete Source Code (`main.c` & TIM2 Interrupt Logic)

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
  htim2.Init.Prescaler = 8399;           // Prescale 84MHz clock down to 10kHz
  htim2.Init.CounterMode = TIM_COUNTERMODE_UP;
  htim2.Init.Period = 9999;              // Trigger interrupt every 1 second (10000 ticks)
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
  HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12|GPIO_PIN_13, GPIO_PIN_RESET);

  /* Configure GPIOD Pins 12, 13 as Push-Pull Outputs */
  GPIO_InitStruct.Pin = GPIO_PIN_12|GPIO_PIN_13;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(GPIOD, &GPIO_InitStruct);
}

/* Timer Period Elapsed Callback (Hardware Interrupt Handler) */
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
  /* Verify if the interrupt source is TIM2 */
  if (htim->Instance == TIM2)
  {
    /* Toggle Green and Orange LEDs asynchronously */
    HAL_GPIO_TogglePin(GPIOD, GPIO_PIN_12 | GPIO_PIN_13);
  }
}

/* Infinite Loop inside main() */
int main(void)
{
  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();
  MX_TIM2_Init();

  /* Start TIM2 in Interrupt Mode */
  HAL_TIM_Base_Start_IT(&htim2);

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Main loop is empty; execution handled completely by hardware interrupts */
    
    /* USER CODE END WHILE */
  }
}
