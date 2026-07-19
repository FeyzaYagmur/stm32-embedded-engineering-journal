# ADC Potentiometer LED Control

This project focuses on executing real-time output control based on continuous analog inputs. By sampling a variable voltage from a potentiometer using the ADC peripheral, the firmware maps the digitized values against specific voltage thresholds to dynamically drive an external LED array.

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **External Components:** 1x 10kΩ Potentiometer, 2x External LEDs, 2x 220Ω/330Ω Resistors
- **Active Pins:** `PA1` (ADC1_IN1 Analog Input), `PD0` (LED 1 Output), `PD1` (LED 2 Output)
- **Method:** Multi-Threshold Analog Voltage Mapping via Polled ADC Conversion

## 🔍 Key Concepts Covered
- **Analog Input Mapping:** Translating a continuous hardware voltage signal into 12-bit digital values ($0$ to $4095$) to drive multi-state logical outputs.
- **Threshold-Based Switching:** Setting operational software boundaries (thresholds) to trigger discrete external hardware actions based on an analog range.
- **Signal Control Logic:** Interfacing analog sensing circuits with multiple digital output channels concurrently within a single processing cycle.

## 💻 Complete Source Code (`main.c` & ADC/GPIO Init)

Below is the complete hardware control logic and the peripheral configuration for the potentiometer-based LED control:

```c
#include "main.h"

ADC_HandleTypeDef hadc1;

/* ADC1 Peripheral Initialization Function */
static void MX_ADC1_Init(void)
{
  ADC_ChannelConfTypeDef sConfig = {0};

  hadc1.Instance = ADC1;
  hadc1.Init.ClockPrescaler = ADC_CLOCK_SYNC_PCLK_DIV4;
  hadc1.Init.Resolution = ADC_RESOLUTION_12B; // 12-bit Resolution
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
  HAL_GPIO_WritePin(GPIOD, GPIO_PIN_0|GPIO_PIN_1, GPIO_PIN_RESET);

  /* Configure GPIO pins : PD0 PD1 (External LED Outputs) */
  GPIO_InitStruct.Pin = GPIO_PIN_0|GPIO_PIN_1;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(GPIOD, &GPIO_InitStruct);
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

  uint32_t analogRaw = 0;

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Start Analog to Digital Conversion */
    HAL_ADC_Start(&hadc1);

    /* Poll for conversion completion */
    if (HAL_ADC_PollForConversion(&hadc1, 10) == HAL_OK)
    {
      /* Read digitized value (Range: 0 - 4095) */
      analogRaw = HAL_ADC_GetValue(&hadc1);

      /* Mode 0: Low Voltage (0 - 1500) -> Both LEDs OFF */
      if (analogRaw < 1500)
      {
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_0, GPIO_PIN_RESET);
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_1, GPIO_PIN_RESET);
      }
      /* Mode 1: Mid Voltage (1500 - 3000) -> Turn ON First LED */
      else if (analogRaw >= 1500 && analogRaw < 3000)
      {
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_0, GPIO_PIN_SET);
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_1, GPIO_PIN_RESET);
      }
      /* Mode 2: High Voltage (3000 - 4095) -> Turn ON Both LEDs */
      else
      {
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_0, GPIO_PIN_SET);
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_1, GPIO_PIN_SET);
      }
    }

    /* Stop ADC conversion and small delay for stabilization */
    HAL_ADC_Stop(&hadc1);
    HAL_Delay(50);

    /* USER CODE END WHILE */
  }
}
