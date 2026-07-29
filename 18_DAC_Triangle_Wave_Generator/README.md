# DAC Triangle Wave Generator (Symmetric Analog Waveform Synthesis)

This project expands on analog signal synthesis by generating a continuous, symmetric Triangle Waveform using the Digital-to-Analog Converter (DAC) peripheral. It demonstrates how to combine sequential voltage ramping (upward slope) and voltage decay (downward slope) to drive an external LED with smooth, breathing-style linear intensity modulation.

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **External Components:** 1x External LED, 1x 220Ω Current-Limiting Resistor
- **Active Pins:** `PA4` (DAC_OUT1 Channel 1 Output Pin)
- **Method:** Dual-Phase Loop Iteration (Ramp-Up and Ramp-Down Synthesis)

## 🔍 Key Concepts Covered
- **Symmetric Waveform Synthesis:** Programming bi-directional loop transitions ($0 \rightarrow 255 \rightarrow 0$) to form linear upward and downward voltage slopes.
- **Continuous Analog Modulation:** Utilizing 8-bit DAC resolution modes (`DAC_ALIGN_8B_R`) to ensure smooth visual intensity transitions without step-wise quantization noise.
- **Signal Frequency Calibration:** Controlling signal period and symmetry through precise delay loop parameters (`HAL_Delay`).

## 💻 Complete Source Code (`main.c` & DAC Init)

Below is the complete C implementation written in STM32CubeIDE using native HAL drivers:

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

  /* Start DAC Channel 1 Output */
  HAL_DAC_Start(&hdac, DAC_CHANNEL_1);

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Phase 1: Ramp Up (Rising Slope) -> 0 to 255 */
    for (int i = 0; i < 256; i++)
    {
      HAL_DAC_SetValue(&hdac, DAC_CHANNEL_1, DAC_ALIGN_8B_R, i);
      HAL_Delay(5);
    }

    /* Phase 2: Ramp Down (Falling Slope) -> 255 down to 0 */
    for (int i = 255; i >= 0; i--)
    {
      HAL_DAC_SetValue(&hdac, DAC_CHANNEL_1, DAC_ALIGN_8B_R, i);
      HAL_Delay(5);
    }

    /* USER CODE END WHILE */
  }
}
