# ADC LDR Light Sensor Simulation & Multi-Threshold Control

This project simulates an environmental LDR (Light Dependent Resistor) sensing system using a potentiometer-driven ADC input channel. Due to hardware component emulation techniques, physical ambient light transitions are simulated via analog voltage division, driving an automatic multi-level night-light LED system based on ambient luminance thresholds.

##  Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **External Components:** 1x 10kΩ Potentiometer (Emulating LDR Sensor), 3x External LEDs (Blue, Green, Red), 3x Resistors
- **Active Pins:** `PA1` (ADC1_IN1 Analog Sensor Input), `PA3` (Blue LED Output), `PA1` (Green LED Output), `PA2` (Red LED Output)
- **Method:** Sensor Emulation & Threshold-Based Automatic Lighting Control

##  Key Concepts Covered
- **Hardware Sensor Emulation:** Utilizing variable potentiometer voltage sources to mirror resistive sensor behaviors (e.g., LDRs) for firmware verification prior to physical sensor integration.
- **Luminance Threshold Categorization:** Translating raw ADC data into environmental lighting zones (Darkness, Dim Light, Full Daylight) to control automated lighting logic.
- **Inverted Ambient Control Logic:** Designing automated safety responses where lowering the simulated luminance triggers additional visual indicator outputs.

##  Complete Source Code (`main.c` & ADC/GPIO Init)

Below is the complete hardware control logic and the specific configuration routines for this LDR simulation project:

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

  /* Configure GPIO pin Output Level */
  HAL_GPIO_WritePin(GPIOA, GPIO_PIN_1|GPIO_PIN_2|GPIO_PIN_3, GPIO_PIN_RESET);

  /* Configure GPIO pins : PA1 PA2 PA3 (External LED Outputs) */
  GPIO_InitStruct.Pin = GPIO_PIN_1|GPIO_PIN_2|GPIO_PIN_3;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);
}

/* Infinite Loop inside main() */
int main(void)
{
  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();
  MX_ADC1_Init();

  uint32_t ldr_degeri = 0;

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Start Analog to Digital Conversion */
    HAL_ADC_Start(&hadc1);

    /* Poll for conversion completion */
    if (HAL_ADC_PollForConversion(&hadc1, 1000) == HAL_OK)
    {
      /* Read digitized value from emulated sensor */
      ldr_degeri = HAL_ADC_GetValue(&hadc1);
    }
    HAL_ADC_Stop(&hadc1);

    /* Mode 1: Low Light/Darkness (< 1000) -> All LEDs ON (Blue, Green, Red) */
    if (ldr_degeri < 1000)
    {
      HAL_GPIO_WritePin(GPIOA, GPIO_PIN_3, GPIO_PIN_SET); // Blue ON
      HAL_GPIO_WritePin(GPIOA, GPIO_PIN_1, GPIO_PIN_SET); // Green ON
      HAL_GPIO_WritePin(GPIOA, GPIO_PIN_2, GPIO_PIN_SET); // Red ON
    }
    /* Mode 2: Medium/Dim Light (1000 - 2500) -> Blue & Green ON, Red OFF */
    else if (ldr_degeri >= 1000 && ldr_degeri < 2500)
    {
      HAL_GPIO_WritePin(GPIOA, GPIO_PIN_3, GPIO_PIN_SET); // Blue ON
      HAL_GPIO_WritePin(GPIOA, GPIO_PIN_1, GPIO_PIN_SET); // Green ON
      HAL_GPIO_WritePin(GPIOA, GPIO_PIN_2, GPIO_PIN_RESET); // Red OFF
    }
    /* Mode 3: Daylight/Bright Environment (>= 2500) -> All LEDs OFF */
    else
    {
      HAL_GPIO_WritePin(GPIOA, GPIO_PIN_3, GPIO_PIN_RESET); // Blue OFF
      HAL_GPIO_WritePin(GPIOA, GPIO_PIN_1, GPIO_PIN_RESET); // Green OFF
      HAL_GPIO_WritePin(GPIOA, GPIO_PIN_2, GPIO_PIN_RESET); // Red OFF
    }

    HAL_Delay(100);

    /* USER CODE END WHILE */
  }
}
