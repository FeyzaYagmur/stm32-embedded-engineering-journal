# ADC Multi-LED Threshold Control

This project implements an advanced multichannel analog threshold controller using the Analog-to-Digital Converter (ADC) peripheral. By continuous sampling of an external potentiometer's output voltage, the firmware maps digitized 12-bit results into distinct dynamic operating ranges, sequentially driving a 3-element external LED array based on physical input modifications.

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **External Components:** 1x 10kΩ Potentiometer, 3x External LEDs (Blue, Red, Green), 3x 220Ω/330Ω Current-Limiting Resistors
- **Active Pins:** `PA1` (ADC1_IN1 Analog Input), `PD0` (Blue LED Output), `PD1` (Red LED Output), `PD2` (Green LED Output)
- **Method:** Multi-Zone Boundary Evaluation via Continuous Polled ADC Sampling

## 🔍 Key Concepts Covered
- **Multi-Level Voltage Mapping:** Segmenting a continuous 0 to 3.3V analog input range into multiple distinct software voltage zones utilizing digitized data scales (0 - 4095).
- **Cascaded Peripheral Control:** Designing sequential control architectures where outputs cascade dynamically (turning pins ON and OFF one by one) depending on potentiometer calibration positions.
- **Signal Multi-plexing Logic:** Interfacing a single analog sensor channel to drive an isolated array of three output indicators asynchronously inside high-speed loops.

## 💻 Complete Source Code (`main.c` & ADC/GPIO Init)

Below is the complete hardware control logic and the specific configuration routines for this analog execution:

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

  uint32_t adcRawValue = 0;

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Start Analog to Digital Conversion */
    HAL_ADC_Start(&hadc1);

    /* Poll for conversion completion */
    if (HAL_ADC_PollForConversion(&hadc1, 10) == HAL_OK)
    {
      /* Read digitized value (Range: 0 - 4095) */
      adcRawValue = HAL_ADC_GetValue(&hadc1);

      /* Zone 0: Very Low (0 - 1000) -> All LEDs OFF */
      if (adcRawValue < 1000)
      {
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_0, GPIO_PIN_RESET); // Blue
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_1, GPIO_PIN_RESET); // Red
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_2, GPIO_PIN_RESET); // Green
      }
      /* Zone 1: Low-Mid (1000 - 2000) -> Blue ON, Others OFF */
      else if (adcRawValue >= 1000 && adcRawValue < 2000)
      {
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_0, GPIO_PIN_SET);
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_1, GPIO_PIN_RESET);
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_2, GPIO_PIN_RESET);
      }
      /* Zone 2: Mid-High (2000 - 3200) -> Blue & Red ON, Green OFF */
      else if (adcRawValue >= 2000 && adcRawValue < 3200)
      {
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_0, GPIO_PIN_SET);
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_1, GPIO_PIN_SET);
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_2, GPIO_PIN_RESET);
      }
      /* Zone 3: High (3200 - 4095) -> All LEDs ON */
      else
      {
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_0, GPIO_PIN_SET);
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_1, GPIO_PIN_SET);
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_2, GPIO_PIN_SET);
      }
    }

    /* Stop ADC conversion and small delay for stability */
    HAL_ADC_Stop(&hadc1);
    HAL_Delay(50);

    /* USER CODE END WHILE */
  }
}
