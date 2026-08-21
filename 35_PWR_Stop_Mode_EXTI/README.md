# STM32 Power Management: Stop Mode & EXTI External Hardware Wake-Up

This project implements advanced power optimization using the STM32 ARM Cortex-M4 **Stop Mode**. By placing the microcontroller into Stop Mode (`HAL_PWR_EnterSTOPMode`), all high-speed core clocks (HSI/HSE) and PLLs are halted while SRAM and register contents are preserved. An External Interrupt line (`EXTI0` connected to the onboard blue USER button `PA0`) serves as the hardware wake-up event. Upon waking, the system clock is reconfigured via `SystemClock_Config()`, flashes a status LED (`PD12`), and re-enters low-power Stop Mode.

##  Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **Power Mode:** Stop Mode (`PWR_LOWPOWERREGULATOR_ON`)
- **Wake-Up Source:** `EXTI0` Line (PA0 Onboard Blue USER Button)
- **Active Pins:** `PA0` (GPIO_EXTI0 Input), `PD12` (Onboard Green LED Output)
- **Clock Management:** Re-initializing system clock tree using `SystemClock_Config()` post-wake-up

##  Key Concepts Covered
- **Stop Mode Mechanics:** Halting all internal clock domains while maintaining core voltage regulator states to minimize static power consumption while preserving register states.
- **External Interrupt (EXTI) Wake-Up:** Harnessing hardware edge triggers on EXTI line 0 (`PA0`) to instantly transition the MCU from deep sleep to active running state.
- **Clock Re-initialization Handling:** Calling `SystemClock_Config()` immediately after exiting Stop Mode to restore system clock frequency (168MHz) before executing user code, as the MCU defaults to internal HSI oscillator upon wake-up.

##  Complete Source Code (`main.c` / Videodaki Yapı ile Birebir)

Below is the complete C implementation written in STM32CubeIDE using native HAL drivers:

```c
#include "main.h"

/* GPIO Initialization Function */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  /* Enable GPIO Clocks */
  __HAL_RCC_GPIOA_CLK_ENABLE();
  __HAL_RCC_GPIOD_CLK_ENABLE();

  /* Configure PD12 as Output (Green LED) */
  HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12, GPIO_PIN_RESET);
  GPIO_InitStruct.Pin = GPIO_PIN_12;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(GPIOD, &GPIO_InitStruct);

  /* Configure PA0 as EXTI Line 0 Interrupt (USER Button) */
  GPIO_InitStruct.Pin = GPIO_PIN_0;
  GPIO_InitStruct.Mode = GPIO_MODE_IT_RISING;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);

  /* Enable EXTI Line 0 Interrupt in NVIC */
  HAL_NVIC_SetPriority(EXTI0_IRQn, 0, 0);
  HAL_NVIC_EnableIRQ(EXTI0_IRQn);
}

int main(void)
{
  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Enter Stop Mode. MCU halts completely until EXTI0 (PA0) button press */
    HAL_PWR_EnterSTOPMode(PWR_LOWPOWERREGULATOR_ON, PWR_STOPENTRY_WFI);

    /* --- WAKE-UP EVENT OCCURRED VIA BUTTON PRESS --- */

    /* Re-configure System Clock back to full speed (168MHz) */
    SystemClock_Config();

    /* Toggle Status LED to signal successful wake-up */
    HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12, GPIO_PIN_SET);
    HAL_Delay(1000);
    HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12, GPIO_PIN_RESET);

    /* USER CODE END WHILE */
  }
}
