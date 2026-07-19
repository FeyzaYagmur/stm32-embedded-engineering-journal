# ITM Debug & ADC Control (Analog Sampling with ITM Redirection)

This project introduces analog signal processing alongside advanced non-intrusive debugging methodologies. It demonstrates how to initialize the Analog-to-Digital Converter (ADC) peripheral to sample external voltages while concurrently redirecting the standard `printf()` output stream directly to the Keil SWV (Serial Wire Viewer) console via hardware ITM (Instrumentation Trace Macrocell) structures.

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **External Components:** 1x 10kΩ Potentiometer (Analog Voltage Source)
- **Active Pins:** `PA1` (Configured as ADC1_IN1 Analog Input)
- **Debugging Bus:** SWD (Serial Wire Debug) with `SWO` pin enabled for real-time trace telemetry.
- **Method:** Polling-based ADC Acquisition with Hardware-Assisted ITM Instrumentation

## 🔍 Key Concepts Covered
- **ITM Printf Redirection:** Overriding the low-level `_write()` syscall framework to stream character buffers directly into the ARM Cortex core's `ITM->PORT[0]` data registers.
- **Analog-to-Digital Conversion (ADC):** Configuring peripheral resolutions (12-bit), alignment parameters, and channel sampling cycles to digitize variable external voltages.
- **Hardware Telemetry Monitoring:** Tracking digitized voltage behaviors dynamically in real-time inside the Keil debugger without halting CPU loop cycles.

## 💻 Complete Source Code (`main.c` & ADC/ITM Init)

Below is the complete hardware control logic, ITM printf redirection architecture, and peripheral configuration:

```c
#include "main.h"
#include <stdio.h>

ADC_HandleTypeDef hadc1;

/* Retargets the C library printf function to the ITM (Instrumentation Trace Macrocell) */
int _write(int file, char *ptr, int len)
{
  int i;
  for (i = 0; i < len; i++)
  {
    ITM_SendChar((*ptr++));
  }
  return len;
}

/* ADC1 Peripherals Initialization Function */
static void MX_ADC1_Init(void)
{
  ADC_ChannelConfTypeDef sConfig = {0};

  /** Configure the global features of the ADC (Clock, Resolution, Data Alignment) */
  hadc1.Instance = ADC1;
  hadc1.Init.ClockPrescaler = ADC_CLOCK_SYNC_PCLK_DIV4;
  hadc1.Init.Resolution = ADC_RESOLUTION_12B; // 0 to 4095 digital range
  hadc1.Init.ScanConvMode = DISABLE;
  hadc1.Init.ContinuousConvMode = DISABLE;
  hadc1.Init.DiscontinuousConvMode = DISABLE;
  hadc1.Init.ExternalTrigConvEdge = ADC_EXTERNALTRIGCONVEDGE_NONE;
  hadc1.Init.ExternalTrigConv = ADC_SOFTWARE_START;
  hadc1.Init.DataAlign = ADC_DATAALIGN_RIGHT;
  hadc1.Init.NbrOfConversion = 1;
  hadc1.Init.DMAContinuousRequests = DISABLE;
  hadc1.Init.EOCSelection = ADC_EOC_SINGLE_CONV;
  HAL_ADC_Init(&hadc1);

  /** Configure for the selected ADC regular channel (PA1) */
  sConfig.Channel = ADC_CHANNEL_1;
  sConfig.Rank = 1;
  sConfig.SamplingTime = ADC_SAMPLETIME_3CYCLES;
  HAL_ADC_ConfigChannel(&hadc1, &sConfig);
}

/* GPIO Initialization Function */
static void MX_GPIO_Init(void)
{
  /* Enable GPIO Port Clocks */
  __HAL_RCC_GPIOH_CLK_ENABLE();
  __HAL_RCC_GPIOA_CLK_ENABLE();
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
  MX_ADC1_Init();

  uint32_t adcValue = 0;

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Start the Analog to Digital Conversion */
    HAL_ADC_Start(&hadc1);

    /* Poll for the conversion process completion */
    if (HAL_ADC_PollForConversion(&hadc1, 100) == HAL_OK)
    {
      /* Read the digitized raw ADC conversion result value */
      adcValue = HAL_ADC_GetValue(&hadc1);

      /* Transmit telemetry tracking data over the hardware SWO pin via ITM printf */
      printf("Raw ADC Value monitored via ITM: %lu\n", adcValue);
    }

    /* Stop conversion and delay the loop cycle */
    HAL_ADC_Stop(&hadc1);
    HAL_Delay(500);

    /* USER CODE END WHILE */
  }
}
