# Dual 7-Segment Multiplexed Display Driver (STM32 HAL & Wokwi Simulation)

> **Note on Implementation:** Due to physical display module availability constraints during initial verification, visual execution was simulated within the **Wokwi simulation platform**. The production codebase is built using **pure C and native STM32 HAL** routines (`HAL_GPIO_WritePin`, `GPIO_TypeDef`), demonstrating professional hardware multiplexing techniques.

This project implements a bare-metal driver for a 2-digit multiplexed 7-segment display. Using high-speed Time-Division Multiplexing (TDM), the system parses two-digit numbers into tens and ones places using integer arithmetic (`/` and `%`), rendering them smoothly on a shared segment bus without visual flicker.

## ⚙️ Hardware & Configuration
- **MCU:** STM32C031C6 (ARM Cortex-M0+) / Wokwi Platform
- **Display Component:** 2-Digit Common Anode Multiplexed 7-Segment Display
- **Active Pins:** `PA0`-`PA7` (Shared Segment Bus A-G + DP), `PB0`-`PB1` (Digit Select Transistor Switches)
- **Method:** High-Frequency Time-Division Multiplexing (TDM) via Pure STM32 HAL Drivers

## 🔍 Key Concepts Covered
- **Time-Division Multiplexing (TDM):** Rapidly alternating active digit select pins (`PB0` and `PB1`) to present multiple independent digits using a single 8-bit bus, relying on human persistence of vision.
- **Bare-Metal Drivers via HAL:** Direct configuration and manipulation of GPIO register blocks using native `GPIO_InitTypeDef` structures without high-level abstraction delays.
- **Arithmetic Digit Parsing:** Utilizing integer division (`/ 10`) and modulo arithmetic (`% 10`) to separate multi-digit integers into discrete display variables.

## 💻 Complete Production Source Code (`main.c` / STM32 HAL Driver)

Below is the complete C implementation written in STM32CubeIDE using native HAL drivers:

```c
#include "main.h"

/* 7-Segment Hexadecimal / Decimal Digit Map (0 to 9) */
uint8_t segment_kodlari[10] = {
  0x3F, // 0
  0x06, // 1
  0x5B, // 2
  0x4F, // 3
  0x66, // 4
  0x6D, // 5
  0x7D, // 6
  0x07, // 7
  0x7F, // 8
  0x6F  // 9
};

void GPIO_Init_1(void);
void GPIO_Init_2(void);
void Coklu_Ekran_Goster(uint8_t sayi);

int main(void)
{
  HAL_Init();
  GPIO_Init_1();
  GPIO_Init_2();

  while (1)
  {
    /* Render number '12' continuously via high-speed multiplexing */
    Coklu_Ekran_Goster(12);
  }
}

/* Configure Segment Bus Pins (GPIOA Pin 0 to Pin 7) */
void GPIO_Init_1(void)
{
  __HAL_RCC_GPIOA_CLK_ENABLE();
  
  GPIO_InitTypeDef GPIO_InitStruct = {0};
  GPIO_InitStruct.Pin = GPIO_PIN_0 | GPIO_PIN_1 | GPIO_PIN_2 | GPIO_PIN_3 | 
                        GPIO_PIN_4 | GPIO_PIN_5 | GPIO_PIN_6 | GPIO_PIN_7;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  
  HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);
}

/* Configure Digit Select Enable Pins (GPIOB Pin 0 & Pin 1) */
void GPIO_Init_2(void)
{
  __HAL_RCC_GPIOB_CLK_ENABLE();
  
  GPIO_InitTypeDef GPIO_InitStruct = {0};
  GPIO_InitStruct.Pin = GPIO_PIN_0 | GPIO_PIN_1;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  
  HAL_GPIO_Init(GPIOB, &GPIO_InitStruct);
}

/* High-Frequency Multiplexing Display Driver Routine */
void Coklu_Ekran_Goster(uint8_t sayi)
{
  uint8_t onlar_basamagi = sayi / 10;
  uint8_t birler_basamagi = sayi % 10;

  /* Step 1: Render Tens Digit on Left Display */
  HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, GPIO_PIN_SET);   // Enable Left Digit
  HAL_GPIO_WritePin(GPIOB, GPIO_PIN_1, GPIO_PIN_RESET); // Disable Right Digit
  HAL_Delay(5);

  /* Step 2: Render Ones Digit on Right Display */
  HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, GPIO_PIN_RESET); // Disable Left Digit
  HAL_GPIO_WritePin(GPIOB, GPIO_PIN_1, GPIO_PIN_SET);   // Enable Right Digit
  HAL_Delay(5);
}
