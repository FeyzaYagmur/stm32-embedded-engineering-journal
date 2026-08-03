# Seven Segment Display Decimal Counter (Hardware Emulation & Production HAL Implementation)

> **Note on Implementation:** Due to the physical display component being temporarily unavailable, the functional simulation and visual demonstration were executed via the **Wokwi online simulator**. To ensure full portability and alignment with production standards, the complete **pure STM32 C / HAL driver codebase** is provided below alongside the simulation code.

This project implements a 0-to-9 numerical decimal counter using a single-digit 7-Segment Display. It demonstrates Common Anode segment driver logic, LUT (Look-Up Table) matrix mapping, and iterative GPIO output pin control.

## ⚙️ Hardware & Configuration
- **MCU / Target:** STM32C031C6 / STM32 Platform
- **Display Component:** Single-Digit 7-Segment Display (Common Anode)
- **Active Pins:** `PA2`, `PA3`, `PA4`, `PA5`, `PA6` (Segments A-E) and `PB9`, `PB10` (Segments F-G)
- **Method:** 2D Lookup Array Mapping with Active-Low Common Anode Driving

## 🔍 Key Concepts Covered
- **Simulated Hardware Prototyping:** Validating display driver logic, pin mappings, and visual timing intervals using virtual simulation environments prior to physical assembly.
- **Common Anode Active-Low Driving:** Configuring digital pin output logic where low logic level (`GPIO_PIN_RESET` / `0`) energizes the corresponding LED segment.
- **Pure STM32 HAL Array Iteration:** Looping through custom mapped `GPIO_TypeDef*` and `GPIO_Pins` arrays inside custom driver routines (`Show_Digit()`).

## 💻 Complete Production Source Code (`main.c` / Pure STM32 HAL)

Below is the production-ready C codebase utilizing native STM32 HAL drivers:

```c
#include "main.h"

// Map GPIO ports and pins corresponding to 7-segment pins (a, b, c, d, e, f, g)
GPIO_TypeDef* GPIO_Ports[] = {GPIOA, GPIOA, GPIOA, GPIOA, GPIOA, GPIOB, GPIOB};
const uint16_t GPIO_Pins[] = {GPIO_PIN_2, GPIO_PIN_3, GPIO_PIN_4, GPIO_PIN_5, GPIO_PIN_6, GPIO_PIN_9, GPIO_PIN_10};

// Common Anode Truth Table for digits 0 to 9 (0: ON / GPIO_PIN_RESET, 1: OFF / GPIO_PIN_SET)
const uint8_t digits[10][7] = {
  {0, 0, 0, 0, 0, 0, 1}, // 0
  {1, 0, 0, 1, 1, 1, 1}, // 1
  {0, 0, 1, 0, 0, 1, 0}, // 2
  {0, 0, 0, 0, 1, 1, 0}, // 3
  {1, 0, 0, 1, 1, 0, 0}, // 4
  {0, 1, 0, 0, 1, 0, 0}, // 5
  {0, 1, 0, 0, 0, 0, 0}, // 6
  {0, 0, 0, 1, 1, 1, 1}, // 7
  {0, 0, 0, 0, 0, 0, 0}, // 8
  {0, 0, 0, 0, 1, 0, 0}  // 9
};

/* Display Digit Function */
void Show_Digit(uint8_t num) {
  for (int seg = 0; seg < 7; seg++) {
    HAL_GPIO_WritePin(GPIO_Ports[seg], GPIO_Pins[seg], digits[num][seg] ? GPIO_PIN_SET : GPIO_PIN_RESET);
  }
}

/* GPIO Initialization Function */
void MX_GPIO_Init(void) {
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  __HAL_RCC_GPIOA_CLK_ENABLE();
  __HAL_RCC_GPIOB_CLK_ENABLE();

  // Configure GPIOA Pins (A-E)
  GPIO_InitStruct.Pin = GPIO_PIN_2 | GPIO_PIN_3 | GPIO_PIN_4 | GPIO_PIN_5 | GPIO_PIN_6;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);

  // Configure GPIOB Pins (F-G)
  GPIO_InitStruct.Pin = GPIO_PIN_9 | GPIO_PIN_10;
  HAL_GPIO_Init(GPIOB, &GPIO_InitStruct);

  // Turn off all segments initially
  for (int i = 0; i < 7; i++) {
    HAL_GPIO_WritePin(GPIO_Ports[i], GPIO_Pins[i], GPIO_PIN_SET);
  }
}

int main(void) {
  HAL_Init();
  MX_GPIO_Init();

  while (1) {
    for (uint8_t num = 0; num < 10; num++) {
      Show_Digit(num);
      HAL_Delay(1000);
    }
  }
}
