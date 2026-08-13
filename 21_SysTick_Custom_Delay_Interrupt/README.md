# SysTick Core Timer Custom Delay & Interrupt Handling

This project implements a bare-metal, custom time-base generator by directly configuring the ARM Cortex-M core's internal SysTick (System Tick) core timer without relying on standard blocking library abstraction functions (`HAL_Delay`). By configuring the SysTick timer to issue hardware interrupts every 1ms (`SystemCoreClock / 1000`), the firmware executes non-blocking atomic counter decrements using volatile memory qualifiers inside the vector table interrupt service routine.

##  Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4 Core)
- **Core Peripheral:** SysTick (24-bit Down-Counting System Tick Timer)
- **Active Pins:** `PD12` (Onboard Green LED Output)
- **Method:** Custom `SysTick_Config()` Setup with ISR-based volatile Counter Decrement

##  Key Concepts Covered
- **SysTick Core Architecture:** Configuring the 24-bit internal system tick timer directly coupled with the Cortex-M core for high-precision time base generation.
- **Volatile Qualifier Usage:** Employing the `volatile` keyword (`volatile uint32_t ms_counter`) to prevent compiler optimization loops when shared variables are modified asynchronously inside Interrupt Service Routines (ISRs).
- **Custom Hardware Delay Implementation:** Designing an atomic, non-blocking software blocking routine (`Custom_Delay_ms()`) driven by core SysTick hardware interrupt pulses.

##  Complete Source Code (main.c & stm32f4xx_it.c)

Below is the implementation logic across the system tick configuration, custom delay function, and vector table interrupt service handler:

```c
#include "main.h"

/* Volatile variable declaration to prevent compiler optimizations */
volatile uint32_t ms_counter = 0;

/* Custom Millisecond Delay Function Driven by SysTick Interrupts */
void Custom_Delay_ms(uint32_t ms)
{
  ms_counter = ms;
  /* Wait until the SysTick_Handler decrements the counter to 0 */
  while (ms_counter > 0);
}

/* GPIO Initialization Function */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  __HAL_RCC_GPIOD_CLK_ENABLE();

  HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12, GPIO_PIN_RESET);

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

  /* Configure SysTick to generate 1ms interrupts (SystemCoreClock / 1000) */
  SysTick_Config(SystemCoreClock / 1000);

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Toggle LED using custom SysTick interrupt-driven delay */
    HAL_GPIO_TogglePin(GPIOD, GPIO_PIN_12);
    Custom_Delay_ms(500);

    /* USER CODE END WHILE */
  }
}

/* ==================================================================== */
/* Interrupt Service Routine (stm32f4xx_it.c)                          */
/* ==================================================================== */

extern volatile uint32_t ms_counter;

/**
  * @brief This function handles System tick timer.
  */
void SysTick_Handler(void)
{
  /* USER CODE BEGIN SysTick_IRQn 0 */




  if (ms_counter > 0)
  {
    ms_counter--;
  }




  /* USER CODE END SysTick_IRQn 0 */
  HAL_IncTick();
  /* USER CODE BEGIN SysTick_IRQn 1 */

  /* USER CODE END SysTick_IRQn 1 */
}
