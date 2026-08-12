# DAC Sawtooth Wave Generator (Analog Waveform Generation)

This project introduces real analog signal synthesis using the Digital-to-Analog Converter (DAC) peripheral. It demonstrates how to generate a periodic Sawtooth (Ramp) waveform using 8-bit right-aligned DAC output values, driving an external LED with linear voltage ramping transitions.

##  Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **External Components:** 1x External LED, 1x 220Ω Current-Limiting Resistor
- **Active Pins:** `PA4` (DAC_OUT1 Channel 1 Output Pin)
- **Method:** Iterative 8-Bit Right-Aligned Value Loading (`DAC_ALIGN_8B_R`)

##  Key Concepts Covered
- **Analog Signal Synthesis:** Converting digital numeric increments ($0$ to $255$) into a continuous, linear voltage ramp ($0\text{V}$ to $3.3\text{V}$) via hardware DAC registers.
- **Sawtooth Wave Generation:** Structuring iterative loop counts to produce periodic ramp signals that reset instantly upon reaching maximum DAC resolution boundaries.
- **8-Bit Resolution Alignment:** Utilizing `DAC_ALIGN_8B_R` alignment modes to optimize memory bus utilization during real-time analog wave generation.

##  Complete Source Code (`main.c` & DAC Init)

Below is the complete hardware control logic and the peripheral configuration for the DAC Sawtooth Wave Generator:

```c
#include "main.h"

DAC_HandleTypeDef hdac;

/* DAC Peripheral Initialization Function */
static void MX_DAC_Init(void)
{
  DAC_ChannelConfTypeDef sConfig = {0};

  /* Enable DAC Peripheral Clock */
  __HAL_RCC_DAC_CLK_ENABLE();

  /* Initialize DAC Peripheral */
  hdac.Instance = DAC;
  HAL_DAC_Init(&hdac);

  /** Configure DAC Channel 1 */
  sConfig.DAC_Trigger = DAC_TRIGGER_NONE;
  sConfig.DAC_OutputBuffer = DAC_OUTPUTBUFFER_ENABLE;
  HAL_DAC_ConfigChannel(&hdac, &sConfig, DAC_CHANNEL_1);
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
  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();
  MX_DAC_Init();

  /* Start DAC Channel 1 */
  HAL_DAC_Start(&hdac, DAC_CHANNEL_1);

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Sawtooth Waveform Generation Loop: Ramp up from 0 to 255 */
    for (int i = 0; i < 256; i++)
    {
      /* Write 8-bit value to DAC Channel 1 */
      HAL_DAC_SetValue(&hdac, DAC_CHANNEL_1, DAC_ALIGN_8B_R, i);
      
      /* Voltage ramp delay interval */
      HAL_Delay(10);
    }

    /* USER CODE END WHILE */
  }
}
