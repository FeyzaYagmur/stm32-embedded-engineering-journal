# L293D H-Bridge Bidirectional DC Motor Direction & State Control

This project demonstrates bidirectional speed/direction control of a DC motor using an L293D dual H-bridge motor driver IC interfaced with an STM32 microcontroller. The firmware toggles discrete GPIO logic lines connected to the input channels (IN1, IN2) while keeping the driver enable line (EN1) active, sequentially transitioning the actuator between Forward, Active Brake/Stop, and Reverse operational states.

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **Motor Driver IC:** L293D Push-Pull Four-Channel Driver (H-Bridge)
- **Actuator:** 3V-6V Brushed DC Toy Motor
- **Active Pins:** 
  - `PA0` (L293D Pin 2 / IN1 - Logic Input 1)
  - `PA1` (L293D Pin 7 / IN2 - Logic Input 2)
  - `PA2` (L293D Pin 1 / EN1 - Channel 1 Enable)
- **Method:** Periodic Digital State Switching via `HAL_GPIO_WritePin()`

## 🔌 L293D Pin Interfacing Overview
- **Pin 1 (1,2EN):** Driven HIGH via `PA2` to activate the driver half-bridges.
- **Pin 2 (1A / IN1):** Connected to `PA0` (Direction Line 1).
- **Pin 7 (2A / IN2):** Connected to `PA1` (Direction Line 2).
- **Pins 3 & 6 (1Y, 2Y):** Wired directly across the DC motor terminals.
- **Pins 4, 5, 12, 13 (GND):** Tied to a common ground rail (MCU GND & external power GND).
- **Pin 8 (VCC2):** External motor power supply rail.
- **Pin 16 (VCC1):** Logic supply rail (5V / 3.3V).

## 🔍 Key Concepts Covered
- **H-Bridge Logic Truth Table:** Alternating polarities across driver inputs (`SET/RESET` for Forward, `RESET/RESET` for Inertial Stop, and `RESET/SET` for Reverse).
- **Inductive Back-EMF Protection:** Relying on internal clamp diodes within the L293D to suppress switching transients during directional polarity inversions.
- **Timed State Sequences:** Implementing structured execution delays (`HAL_Delay(2000)`) between state changes to prevent shoot-through transitions during reverse switching.

## 💻 Complete Source Code (`main.c` & GPIO Init)

```c
#include "main.h"

/* GPIO Initialization Function */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  /* Enable GPIO Clocks */
  __HAL_RCC_GPIOH_CLK_ENABLE();
  __HAL_RCC_GPIOA_CLK_ENABLE();

  /* Set initial pin state to LOW */
  HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0 | GPIO_PIN_1 | GPIO_PIN_2, GPIO_PIN_RESET);

  /* Configure GPIOA Pins 0, 1 (Direction) and 2 (Enable) as Push-Pull Outputs */
  GPIO_InitStruct.Pin = GPIO_PIN_0 | GPIO_PIN_1 | GPIO_PIN_2;
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

  /* Enable L293D Channel 1 Driver Stage */
  HAL_GPIO_WritePin(GPIOA, GPIO_PIN_2, GPIO_PIN_SET);

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* 1. Forward Direction (IN1 = HIGH, IN2 = LOW) */
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0, GPIO_PIN_SET);
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_1, GPIO_PIN_RESET);
    HAL_Delay(2000);

    /* 2. Stop / Brake State (IN1 = LOW, IN2 = LOW) */
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0, GPIO_PIN_RESET);
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_1, GPIO_PIN_RESET);
    HAL_Delay(1000);

    /* 3. Reverse Direction (IN1 = LOW, IN2 = SET) */
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0, GPIO_PIN_RESET);
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_1, GPIO_PIN_SET);
    HAL_Delay(2000);

    /* 4. Stop / Brake State (IN1 = LOW, IN2 = LOW) */
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0, GPIO_PIN_RESET);
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_1, GPIO_PIN_RESET);
    HAL_Delay(1000);

    /* USER CODE END WHILE */
  }
}
