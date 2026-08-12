# ADC Discrete LED Zone Selector

This project implements an analog channel parsing selector using the Analog-to-Digital Converter (ADC) peripheral. By sampling the continuous output voltage from an external potentiometer, the firmware evaluates the digitized 12-bit result against strictly bounded range matrices to activate exactly one discrete LED at a time, creating an analog-driven position selector system.

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **External Components:** 1x 10kΩ Potentiometer, 3x External LEDs (Green, Red, Blue), 3x 220Ω/330Ω Current-Limiting Resistors
- **Active Pins:** `PA1` (ADC1_IN1 Analog Input), `PD0` (Blue LED Output), `PD1` (Red LED Output), `PD2` (Green LED Output)
- **Method:** Exclusive Single-Zone Switching via Polled ADC Processing

## 🔍 Key Concepts Covered
- **Exclusive Multi-Zone Parsing:** Splitting the continuous analog voltage channel ($0$ - $4095$) into discrete sectors where only one conditional state evaluates as true, preventing overlapping signals.
- **Mutual Exclusion Output Control:** Ensuring that when a new voltage zone is selected, all other output pins are actively reset to low logic levels, isolating the active visual indicator.
- **Dynamic Variable State Routing:** Mapping physical control modifications from human interfaces directly to indexed state configurations without hardware-level toggle noise.

## 💻 Complete Source Code (`main.c` & ADC/GPIO Init)

Below is the complete hardware control logic and the specific configuration routines for this discrete zone selector project:

```c
#include "main.h"

ADC_HandleTypeDef hadc1;

/* ADC1 Peripheral Initialization Function */
static void MX_ADC1_Init(void)
{
  ADC_ChannelConfTypeDef sConfig = {0};

  hadc1.Instance = ADC1;
  hadc1.Init.ClockPrescaler = ADC_CLOCK_SYNC_PCLK_DIV4;
  hadc1.Init.Resolution = ADC_RESOLUTION_12B;
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
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  /* Enable GPIO Port Clocks */
  __HAL_RCC_GPIOH_CLK_ENABLE();
  __HAL_RCC_GPIOA_CLK_ENABLE();
  __HAL_RCC_GPIOD_CLK_ENABLE();

  /* Configure GPIO pin Output Level */
  HAL_GPIO_WritePin(GPIOD, GPIO_PIN_0|GPIO_PIN_1|GPIO_PIN_2, GPIO_PIN_RESET);

  /* Configure GPIO pins : PD0 PD1 PD2 (External LED Outputs) */
  GPIO_InitStruct.Pin = GPIO_PIN_0|GPIO_PIN_1|GPIO_PIN_2;
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
  MX_ADC1_Init();

  uint32_t adcSelectorValue = 0;

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Start Analog to Digital Conversion */
    HAL_ADC_Start(&hadc1);

    /* Poll for conversion completion */
    if (HAL_ADC_PollForConversion(&hadc1, 10) == HAL_OK)
    {
      /* Read digitized value (Range: 0 - 4095) */
      adcSelectorValue = HAL_ADC_GetValue(&hadc1);

      /* Zone 0: Lower Spectrum (0 - 1300) -> Only Green LED ON */
      if (adcSelectorValue < 1300)
      {
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_2, GPIO_PIN_SET);   // Green ON
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_0|GPIO_PIN_1, GPIO_PIN_RESET);
      }
      /* Zone 1: Mid Spectrum (1300 - 2700) -> Only Red LED ON */
      else if (adcSelectorValue >= 1300 && adcSelectorValue < 2700)
      {
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_1, GPIO_PIN_SET);   // Red ON
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_0|GPIO_PIN_2, GPIO_PIN_RESET);
      }
      /* Zone 2: Upper Spectrum (2700 - 4095) -> Only Blue LED ON */
      else
      {
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_0, GPIO_PIN_SET);   // Blue ON
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_1|GPIO_PIN_2, GPIO_PIN_RESET);
      }
    }

    /* Stop ADC conversion and small delay for stabilization */
    HAL_ADC_Stop(&hadc1);
    HAL_Delay(50);

    /* USER CODE END WHILE */
  }
}
