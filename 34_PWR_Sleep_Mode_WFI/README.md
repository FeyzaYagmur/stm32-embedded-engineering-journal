# STM32 Power Management: Sleep Mode & WFI (Wait For Interrupt)

This project demonstrates the fundamental implementation of microcontroller power management by placing the STM32 ARM Cortex-M4 core into **Sleep Mode**. Using the `WFI` (Wait For Interrupt) instruction architecture via HAL libraries, the main execution thread is suspended to save power. A hardware timer (TIM3) is configured to generate periodic interrupts, acting as a wake-up source to temporarily resume core operations, toggle a status LED inside the callback, and immediately return to sleep.

##  Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **Power Mode:** Sleep Mode (`PWR_MAINREGULATOR_ON`)
- **Wake-Up Source:** TIM3 Hardware Interrupt (Period Elapsed)
- **Active Pins:** `PD12` (Onboard Green LED)
- **Method:** Core Suspension via `HAL_PWR_EnterSLEEPMode()` and `WFI` Instruction

##  Key Concepts Covered
- **Low-Power State Execution:** Halting the core CPU clock while keeping peripheral clocks (like TIM3 and GPIO) active to significantly reduce power consumption without completely shutting down the system.
- **Wait For Interrupt (WFI):** Utilizing the ARM Cortex instruction set to suspend the main `while(1)` loop. The MCU remains idle until a hardware interrupt request (IRQ) is detected.
- **Asynchronous ISR Wake-Up:** Handling the timer interrupt inside `HAL_TIM_PeriodElapsedCallback()`. Once the ISR finishes execution, the core returns to the `while(1)` loop and immediately re-enters sleep mode.

##  Complete Source Code (`main.c` / Defterdeki Yapı ile Birebir)

Below is the complete C implementation demonstrating the power management logic and interrupt callback routines:

```c
#include "main.h"

TIM_HandleTypeDef htim3;

/* Timer Period Elapsed Callback (ISR - Wake-up Source) */
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
  /* Check if the interrupt was triggered by TIM3 */
  if (htim->Instance == TIM3)
  {
    /* Toggle LED to indicate MCU has momentarily woken up */
    HAL_GPIO_TogglePin(GPIOD, GPIO_PIN_12);
  }
}

/* TIM3 Initialization Function (Interrupt Mode) */
static void MX_TIM3_Init(void)
{
  htim3.Instance = TIM3;
  htim3.Init.Prescaler = 8399;           // Scale timer clock
  htim3.Init.CounterMode = TIM_COUNTERMODE_UP;
  htim3.Init.Period = 9999;              // Define interrupt period
  htim3.Init.ClockDivision = TIM_CLOCKDIVISION_DIV1;
  htim3.Init.AutoReloadPreload = TIM_AUTORELOAD_PRELOAD_DISABLE;
  HAL_TIM_Base_Init(&htim3);
}

/* GPIO Initialization Function */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  __HAL_RCC_GPIOD_CLK_ENABLE();

  /* Configure PD12 as Output */
  GPIO_InitStruct.Pin = GPIO_PIN_12;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(GPIOD, &GPIO_InitStruct);
}

int main(void)
{
  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();
  MX_TIM3_Init();

  /* Start TIM3 in Interrupt Mode to act as the wake-up trigger */
  HAL_TIM_Base_Start_IT(&htim3);

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Enter Sleep Mode. The core will halt here until TIM3 fires an interrupt */
    HAL_PWR_EnterSLEEPMode(PWR_MAINREGULATOR_ON, PWR_SLEEPENTRY_WFI);
    
    /* USER CODE END WHILE */
  }
}
