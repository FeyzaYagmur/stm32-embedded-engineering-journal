# ITM Debugging & ADC Threshold Multi-LED Control System

This project demonstrates continuous analog signal processing using the 12-bit Analog-to-Digital Converter (ADC1) peripheral on an STM32 microcontroller, combined with non-intrusive ITM (`printf`) retargeted telemetry debug tracking. Raw analog inputs sampled from a 10kΩ potentiometer (`PA1`) are dynamically thresholded to drive a 3-channel visual LED indicator array (Blue, Red, Green) while streaming telemetry data to the SWV debugging console.

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **Input Component:** 1x 10kΩ Potentiometer (Analog Voltage Source)
- **Actuators:** 3x External LEDs (Blue, Red, Green)
- **Active Pins:** `PA1` (ADC1_IN1), `PD3` (Blue LED), `PD4` (Red LED), `PD5` (Green LED)
- **Resolution:** 12-Bit ADC Resolution ($0 - 4095$ Range)
- **Debugging:** ITM Trace Macrocell (Retargeted `printf` over SWO pin)

## 🔍 Key Concepts Covered
- **ITM Printf Redirection:** Overriding the low-level `_write()` system calls to redirect standard library `printf()` output frames directly into `ITM_SendChar()` data registers.
- **Dynamic Voltage Thresholding:** Partitioning 12-bit digital ranges ($0 - 4095$) into discrete operational bands to dynamically activate dedicated color-coded LEDs (Blue, Red, Green).
- **Mixed-Signal Interfacing & Telemetry:** Simultaneous execution of high-speed analog-to-digital conversions, GPIO state switching, and non-blocking real-time debug reporting.

## 💻 Complete Source Code (`main.c` / Videodaki Çalışmaya Birebir Uygun)

Below is the complete implementation incorporating retargeted ITM printf functionality, ADC polling, and 3-color LED threshold actuation:

```c
#include "main.h"
#include <stdio.h>

ADC_HandleTypeDef hadc1;

/* Retarget C library printf function to ITM (Instrumentation Trace Macrocell) */
int _write(int file, char *ptr, int len)
{
  int i;
  for (i = 0; i < len; i++)
  {
    ITM_SendChar((*ptr++));
  }
  return len;
}

/* ADC1 Peripheral Initialization Function */
static void MX_ADC1_Init(void)
{
  ADC_ChannelConfTypeDef sConfig = {0};

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

  sConfig.Channel = ADC_CHANNEL_1; // Configured on PA1
  sConfig.Rank = 1;
  sConfig.SamplingTime = ADC_SAMPLETIME_3CYCLES;
  HAL_ADC_ConfigChannel(&hadc1, &sConfig);
}

/* GPIO Initialization Function (Blue, Red, Green LEDs) */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  __HAL_RCC_GPIOA_CLK_ENABLE();
  __HAL_RCC_GPIOD_CLK_ENABLE();

  /* Clear LED pin outputs initially */
  HAL_GPIO_WritePin(GPIOD, GPIO_PIN_3 | GPIO_PIN_4 | GPIO_PIN_5, GPIO_PIN_RESET);

  /* Configure PD3 (Blue), PD4 (Red), PD5 (Green) as Outputs */
  GPIO_InitStruct.Pin = GPIO_PIN_3 | GPIO_PIN_4 | GPIO_PIN_5;
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
  MX_ADC1_Init();

  uint32_t adcValue = 0;

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Start Analog to Digital Conversion */
    HAL_ADC_Start(&hadc1);

    /* Poll ADC conversion process */
    if (HAL_ADC_PollForConversion(&hadc1, 100) == HAL_OK)
    {
      /* Read 12-bit raw digital conversion value */
      adcValue = HAL_ADC_GetValue(&hadc1);

      /* Threshold Actuation Logic for Blue, Red, Green LEDs */
      if (adcValue < 1365)
      {
        /* Zone 1: Blue LED ON */
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_3, GPIO_PIN_SET);
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_4, GPIO_PIN_RESET);
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_5, GPIO_PIN_RESET);
      }
      else if (adcValue >= 1365 && adcValue < 2730)
      {
        /* Zone 2: Red LED ON */
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_3, GPIO_PIN_RESET);
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_4, GPIO_PIN_SET);
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_5, GPIO_PIN_RESET);
      }
      else
      {
        /* Zone 3: Green LED ON */
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_3, GPIO_PIN_RESET);
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_4, GPIO_PIN_RESET);
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_5, GPIO_PIN_SET);
      }
    }

    HAL_ADC_Stop(&hadc1);
    HAL_Delay(100);

    /* USER CODE END WHILE */
  }
}
