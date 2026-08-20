# HC-SR04 Ultrasonic Distance Sensor Modular Driver & Software Microsecond Timing

This project implements physical distance measurement using the **HC-SR04 Ultrasonic Ranging Module** interfaced with an STM32 microcontroller. The firmware features a custom software-based microsecond delay routine (`delayUS`), a modular driver function (`HCSR04_Read()`), and acoustic wave time-to-distance conversion ($Distance = \frac{Time}{58.0}$) with upper/lower out-of-range safety filtering ($2\text{cm} - 400\text{cm}$).

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **Sensor Component:** HC-SR04 Ultrasonic Ranging Sensor
- **Active Pins:** `PA1` (Trig Output Pin), `PA0` (Echo Input Pin)
- **Timing:** Software `__NOP()` Assembly Cycle-Based Microsecond Delay (`delayUS`)
- **Conversion Equation:** $Distance_{\text{cm}} = \frac{Echo_{\text{us}}}{58.0}$

## 🔍 Key Concepts Covered
- **Software Microsecond Delay (`delayUS`):** Implementing calibrated microsecond delays using CPU instruction cycles (`__NOP()`) based on clock frequency.
- **Modular Driver Architecture:** Encapsulating trigger pulsing, echo high-state polling, and math conversion inside an isolated driver routine (`HCSR04_Read()`).
- **Out-of-Range Bound Filtering:** Clamping valid measurement windows between $2.0\text{cm}$ and $400.0\text{cm}$ to reject erroneous acoustic readings.

## 💻 Complete Source Code (`main.c` / Defterdeki Yapı ile Birebir)

Below is the complete C implementation written in STM32CubeIDE using native HAL drivers:

```c
#include "main.h"

uint32_t echo_time = 0;
float distance = 0.0f;

/* Software Microsecond Delay using Assembly NOP Instruction */
void delayUS(uint32_t us)
{
  uint32_t count = us * 8; // Calibrated for STM32 clock speed
  while (count--)
  {
    __NOP(); // Empty CPU clock cycle execution
  }
}

/* Modular HC-SR04 Ultrasonic Sensor Driver Function */
float HCSR04_Read(void)
{
  uint32_t local_time = 0;
  float tempDistance = 0.0f;

  /* Step 1: Send 10us HIGH pulse to Trigger Pin (PA1) */
  HAL_GPIO_WritePin(GPIOA, GPIO_PIN_1, GPIO_PIN_RESET);
  delayUS(2);
  HAL_GPIO_WritePin(GPIOA, GPIO_PIN_1, GPIO_PIN_SET);
  delayUS(10);
  HAL_GPIO_WritePin(GPIOA, GPIO_PIN_1, GPIO_PIN_RESET);

  /* Step 2: Wait until Echo Pin (PA0) goes HIGH and measure pulse duration */
  while (!HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_0));
  while (HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_0))
  {
    local_time++;
    delayUS(2);
  }

  /* Step 3: Convert Echo duration to centimeters (Distance = Time / 58.0) */
  tempDistance = (float)local_time / 58.0f;
  
  HAL_Delay(50);

  return tempDistance;
}

/* GPIO Initialization Function */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  __HAL_RCC_GPIOA_CLK_ENABLE();

  /* Configure PA1 as Trigger Output */
  GPIO_InitStruct.Pin = GPIO_PIN_1;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);

  /* Configure PA0 as Echo Input */
  GPIO_InitStruct.Pin = GPIO_PIN_0;
  GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);
}

int main(void)
{
  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Read distance from sensor */
    distance = HCSR04_Read();

    /* Boundary Filter: Out-of-range checks */
    if (distance > 400.0f)
    {
      distance = 400.0f;
    }
    if (distance < 2.0f)
    {
      distance = 2.0f;
    }

    /* USER CODE END WHILE */
  }
}
