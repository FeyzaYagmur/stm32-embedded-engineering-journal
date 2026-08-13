# L293D DC Motor Speed & Direction Control via PWM

This project demonstrates closed-loop speed and directional control of a DC motor using an **L293D H-Bridge Motor Driver IC** interfaced with an STM32 microcontroller. The setup utilizes General-Purpose Timer 2 (TIM2 Channel 3) to deliver variable Duty Cycle Pulse Width Modulation (PWM) signals to the driver's Enable pin, enabling dynamic multi-speed operation.

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **Motor Driver IC:** L293D Dual H-Bridge Driver
- **Actuator:** 3V-6V DC Brushed Motor
- **Active Pins:** `PA0`, `PA1` (Direction Control Direction Pins IN1/IN2), `PA2` (TIM2_CH3 PWM Output to EN1)
- **Method:** H-Bridge Direction Steering with Discrete PWM Speed Stepping (`__HAL_TIM_SET_COMPARE`)

## 🔍 Key Concepts Covered
- **H-Bridge Driver Architecture:** Controlling DC motor rotation polarity (Clockwise / Counter-Clockwise) via complementary logic outputs on L293D direction pins (`PA0` / `PA1`).
- **PWM Speed Modulation:** Regulating motor angular velocity by feeding variable duty cycle PWM pulses into the driver enable pin (EN1).
- **Multi-Level Speed Stepping:** Iterating through predefined Capture/Compare Register (CCR) values (`300`, `600`, `1000`) with $2000\text{ms}$ step intervals to demonstrate discrete speed ramps.

## 💻 Complete Source Code (`main.c` & TIM2 PWM Init)

Below is the complete C implementation written in STM32CubeIDE using native HAL drivers:

```c
#include "main.h"

TIM_HandleTypeDef htim2;

/* TIM2 Peripheral Initialization Function */
static void MX_TIM2_Init(void)
{
  TIM_MasterConfigTypeDef sMasterConfig = {0};
  TIM_OC_InitTypeDef sConfigOC = {0};

  htim2.Instance = TIM2;
  htim2.Init.Prescaler = 83;             // Clock prescaler for PWM frequency
  htim2.Init.CounterMode = TIM_COUNTERMODE_UP;
  htim2.Init.Period = 999;               // ARR Value (1000 total ticks per period)
  htim2.Init.ClockDivision = TIM_CLOCKDIVISION_DIV1;
  htim2.Init.AutoReloadPreload = TIM_AUTORELOAD_PRELOAD_DISABLE;
  HAL_TIM_PWM_Init(&htim2);

  sMasterConfig.MasterOutputTrigger = TIM_TRGO_RESET;
  sMasterConfig.MasterSlaveMode = TIM_MASTERSLAVEMODE_DISABLE;
  HAL_TIMEx_MasterConfigSynchronization(&htim2, &sMasterConfig);

  /* Configure PWM Channel 3 */
  sConfigOC.OCMode = TIM_OCMODE_PWM1;
  sConfigOC.Pulse = 0;                   // Initial duty cycle (0%)
  sConfigOC.OCPolarity = TIM_OCPOLARITY_HIGH;
  sConfigOC.OCFastMode = TIM_OCFAST_DISABLE;
  HAL_TIM_PWM_ConfigChannel(&htim2, &sConfigOC, TIM_CHANNEL_3);
}

/* GPIO Initialization Function */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  /* Enable GPIO Clocks */
  __HAL_RCC_GPIOH_CLK_ENABLE();
  __HAL_RCC_GPIOA_CLK_ENABLE();

  /* Set initial pin state to LOW */
  HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0|GPIO_PIN_1, GPIO_PIN_RESET);

  /* Configure GPIOA Pins 0 & 1 as Push-Pull Outputs (Direction Pins) */
  GPIO_InitStruct.Pin = GPIO_PIN_0|GPIO_PIN_1;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);
}

int main(void)
{
  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();
  MX_TIM2_Init();

  /* Start TIM2 PWM Output on Channel 3 */
  HAL_TIM_PWM_Start(&htim2, TIM_CHANNEL_3);

  /* Set Motor Direction: Forward (IN1 = HIGH, IN2 = LOW) */
  HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0, GPIO_PIN_SET);
  HAL_GPIO_WritePin(GPIOA, GPIO_PIN_1, GPIO_PIN_RESET);

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Speed Level 1: ~30% Duty Cycle */
    __HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_3, 300);
    HAL_Delay(2000);

    /* Speed Level 2: ~60% Duty Cycle */
    __HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_3, 600);
    HAL_Delay(2000);

    /* Speed Level 3: 100% Duty Cycle (Full Speed) */
    __HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_3, 1000);
    HAL_Delay(2000);

    /* USER CODE END WHILE */
  }
}
