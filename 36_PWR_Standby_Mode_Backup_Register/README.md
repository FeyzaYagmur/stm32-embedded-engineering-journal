# STM32 Power Management: Deep Standby Mode, WakeUp Pin & RTC Backup Registers (BDR)

This project demonstrates ultra-low-power embedded architecture using the STM32 ARM Cortex-M4 **Standby Mode** combined with **RTC Backup Domain Registers (BDR)**. The firmware places the microcontroller into its lowest power consumption state (`HAL_PWR_EnterSTANDBYMode()`), where the internal voltage regulator and clock domains are shut down. Upon detection of a high-level pulse on the dedicated hardware WakeUp pin (`PA0` / `PWR_WAKEUP_PIN1`), the core undergoes a system reset, increments an persistent wake-up counter stored in Non-Volatile Backup RAM (`RTC_BKP_DR1`), and signals state resumption via status LED indicators (`PD12`).

##  Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **Power Mode:** Standby Mode (`HAL_PWR_EnterSTANDBYMode()`)
- **Wake-Up Source:** Hardware `PWR_WAKEUP_PIN1` (`PA0` Button Edge Trigger)
- **Persistent Memory:** RTC Backup Domain Register 1 (`RTC_BKP_DR1`)
- **Active Pins:** `PA0` (WakeUp Pin 1 Input), `PD12` (Onboard Green LED Output)

##  Key Concepts Covered
- **Deep Standby Mode Architecture:** Achieving sub-microamp power consumption by de-energizing the core logic domain, main SRAM, and internal clock trees.
- **RTC Backup Domain (BDR) Persistence:** Utilizing battery-backed register blocks (`HAL_RTCEx_BKUPRead` / `HAL_RTCEx_BKUPWrite`) to retain critical system parameters across CPU power-down and system reset cycles.
- **Hardware WakeUp Pin Triggering:** Configuring dedicated silicon wakeup lines (`HAL_PWR_EnableWakeUpPin`) to trigger cold-start processor resets directly out of deep sleep modes.

##  Complete Source Code (`main.c` / Videodaki Yapı ile Birebir)

Below is the complete C implementation written in STM32CubeIDE using native HAL drivers:

```c
#include "main.h"

RTC_HandleTypeDef hrtc;
uint32_t reset_sayaci = 0;

/* RTC Peripheral Initialization Function */
static void MX_RTC_Init(void)
{
  hrtc.Instance = RTC;
  hrtc.Init.HourFormat = RTC_HOURFORMAT_24;
  hrtc.Init.AsynchPrediv = 127;
  hrtc.Init.SynchPrediv = 255;
  hrtc.Init.OutPut = RTC_OUTPUT_DISABLE;
  HAL_RTC_Init(&hrtc);
}

/* GPIO Initialization Function */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  __HAL_RCC_GPIOA_CLK_ENABLE();
  __HAL_RCC_GPIOD_CLK_ENABLE();

  /* Configure PD12 as Output (Green LED) */
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
  MX_RTC_Init();

  /* --- STEP 1: BACKUP REGISTER (BDR) READ & INCREMENT --- */
  
  /* Enable access to the RTC Backup Domain Registers */
  HAL_PWR_EnableBkUpAccess();

  /* Read previous reset/wake-up count from BKP_DR1 */
  reset_sayaci = HAL_RTCEx_BKUPRead(&hrtc, RTC_BKP_DR1);

  /* Increment wake-up counter and update Backup Register */
  reset_sayaci++;
  HAL_RTCEx_BKUPWrite(&hrtc, RTC_BKP_DR1, reset_sayaci);

  /* Disable access to prevent unintended writes */
  HAL_PWR_DisableBkUpAccess();

  /* --- STEP 2: WAKE-UP / BOOT INDICATOR (PD12 LED) --- */
  
  /* Turn ON status LED for 2 seconds upon boot/wake-up */
  HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12, GPIO_PIN_SET);
  HAL_Delay(2000);
  HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12, GPIO_PIN_RESET);

  /* --- STEP 3: ENTER STANDBY MODE --- */

  /* Clear WakeUp Flag if set previously */
  __HAL_PWR_CLEAR_FLAG(PWR_FLAG_WU);

  /* Enable Hardware WakeUp Pin 1 (PA0) */
  HAL_PWR_EnableWakeUpPin(PWR_WAKEUP_PIN1);

  /* Enter Standby Mode (MCU shuts down here until PA0 WakeUp pulse) */
  HAL_PWR_EnterSTANDBYMode();

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Code execution will never reach here as Standby triggers a system reset */
  }
}
