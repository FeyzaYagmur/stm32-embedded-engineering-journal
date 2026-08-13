# ADC to DAC Signal Passthrough & Loopback Control

This project combines both peripheral processing domains—Analog-to-Digital Conversion (ADC) and Digital-to-Analog Conversion (DAC)—into a real-time signal loopback system. The MCU continuously samples an analog voltage from a potentiometer via a 12-bit ADC, maps the digital result, and directly reconstructs the analog signal on the 12-bit DAC output to drive an external LED with full dynamic fidelity.

##  Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **External Components:** 1x 10kΩ Potentiometer, 1x External LED, 1x 220Ω Resistor
- **Active Pins:** `PA1` (ADC1_IN1 Input Pin), `PA4` (DAC_OUT1 Output Pin)
- **Method:** 12-Bit Direct Bus Passthrough (`ADC_RESOLUTION_12B` to `DAC_ALIGN_12B_R`)

##  Key Concepts Covered
- **Analog Loopback Processing:** Acquiring continuous environmental voltage inputs via ADC and immediately mirroring them to DAC hardware channels without signal degradation.
- **12-Bit Resolution Matching:** Synchronizing ADC raw output values ($0 - 4095$) directly with 12-bit right-aligned DAC write buffers (`DAC_ALIGN_12B_R`).
- **Real-Time Signal Conversion:** Executing low-latency hardware-level acquisition-to-generation pipeline inside execution loop cycles.

##  Complete Source Code (`main.c` & Peripheral Init)

Below is the complete C implementation written in STM32CubeIDE using native HAL drivers:

```c
#include "main.h"

ADC_HandleTypeDef hadc1;
DAC_HandleTypeDef hdac;

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

  sConfig.Channel = ADC_CHANNEL_1;
  sConfig.Rank = 1;
  sConfig.SamplingTime = ADC_SAMPLETIME_3CYCLES;
  HAL_ADC_ConfigChannel(&hadc1, &sConfig);
}

/* DAC Peripheral Initialization Function */
static void MX_DAC_Init(void)
{
  DAC_ChannelConfTypeDef sConfig = {0};

  __HAL_RCC_DAC_CLK_ENABLE();

  hdac.Instance = DAC;
  HAL_DAC_Init(&hdac);

  sConfig.DAC_Trigger = DAC_TRIGGER_NONE;
  sConfig.DAC_OutputBuffer = DAC_OUTPUTBUFFER_ENABLE;
  HAL_DAC_ConfigChannel(&hdac, &sConfig, DAC_CHANNEL_1);
}

/* GPIO Initialization Function */
static void MX_GPIO_Init(void)
{
  __HAL_RCC_GPIOH_CLK_ENABLE();
  __HAL_RCC_GPIOA_CLK_ENABLE();
}

/* Infinite Loop inside main() */
int main(void)
{
  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();
  MX_ADC1_Init();
  MX_DAC_Init();

  /* Start DAC Output Channel */
  HAL_DAC_Start(&hdac, DAC_CHANNEL_1);

  uint32_t pot_degeri = 0;

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Step 1: Start ADC Conversion & Read Analog Voltage */
    HAL_ADC_Start(&hadc1);

    if (HAL_ADC_PollForConversion(&hadc1, 100) == HAL_OK)
    {
      pot_degeri = HAL_ADC_GetValue(&hadc1);
    }
    HAL_ADC_Stop(&hadc1);

    /* Step 2: Write Digitized Value directly to 12-bit DAC Channel */
    HAL_DAC_SetValue(&hdac, DAC_CHANNEL_1, DAC_ALIGN_12B_R, pot_degeri);

    HAL_Delay(10);

    /* USER CODE END WHILE */
  }
}
