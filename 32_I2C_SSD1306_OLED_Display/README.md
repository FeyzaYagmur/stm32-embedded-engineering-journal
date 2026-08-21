# SSD1306 OLED Display Driver & Dynamic Text Rendering via I2C

This project demonstrates hardware interfacing and dynamic buffer rendering for a **0.96-inch SSD1306 Monochrome OLED Display (128x64 resolution)** using the I2C serial communication protocol on an STM32 microcontroller. The firmware initializes display buffers, clears screen framebuffers (`SSD1306_Clear`), and updates multi-line text strings dynamically using custom font definitions.

##  Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **Display Component:** 0.96" SSD1306 OLED Display ($128 \times 64$ Resolution)
- **Protocol:** I2C (Inter-Integrated Circuit) Bus
- **I2C Address:** `0x78` / `0x3C`
- **Active Pins:** `PB6` (I2C1_SCL Clock Line), `PB7` (I2C1_SDA Data Line)
- **Method:** Memory Framebuffer Manipulation with Sequential Page Redraws

##  Key Concepts Covered
- **I2C Bus Protocol Verification:** Managing two-wire synchronous data transfers (SCL/SDA) to stream pixel buffer commands to graphics controller ICs.
- **Dynamic Screen Buffer Updating:** Executing buffer clears (`SSD1306_Clear()`), positioning coordinate pointers (`SSD1306_GotoXY()`), and pushing frame updates (`SSD1306_UpdateScreen()`).
- **Sequential String Transition Rendering:** Demonstrating dynamic display updates by sequencing system introduction strings (`WELCOME TO EMBEDDED SYSTEMS`) followed by personalized developer identification signatures (`FEYZA YAGMUR`).

##  Complete Source Code (`main.c` / Dual-State Sequence)

Below is the complete C code demonstrating the dynamic screen buffer update logic used in the execution:

```c
#include "main.h"
#include "ssd1306.h"

/* I2C1 Handle Declaration */
I2C_HandleTypeDef hi2c1;

/* I2C1 Initialization Function */
static void MX_I2C1_Init(void)
{
  hi2c1.Instance = I2C1;
  hi2c1.Init.ClockSpeed = 400000;         // Fast Mode 400kHz
  hi2c1.Init.DutyCycle = I2C_DUTYCYCLE_2;
  hi2c1.Init.OwnAddress1 = 0;
  hi2c1.Init.AddressingMode = I2C_ADDRESSINGMODE_7BIT;
  hi2c1.Init.DualAddressMode = I2C_DUALADDRESS_DISABLE;
  hi2c1.Init.GeneralCallMode = I2C_GENERALCALL_DISABLE;
  hi2c1.Init.NoStretchMode = I2C_NOSTRETCH_DISABLE;
  HAL_I2C_Init(&hi2c1);
}

/* GPIO Initialization Function */
static void MX_GPIO_Init(void)
{
  __HAL_RCC_GPIOH_CLK_ENABLE();
  __HAL_RCC_GPIOB_CLK_ENABLE();
}

int main(void)
{
  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();
  MX_I2C1_Init();

  /* Initialize SSD1306 OLED Display */
  SSD1306_Init();

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* --- State 1: System Welcome Banner --- */
    SSD1306_Clear();

    SSD1306_GotoXY(28, 5);
    SSD1306_Puts("WELCOME TO", &Font_7x10, 1);

    SSD1306_GotoXY(35, 25);
    SSD1306_Puts("EMBEDDED", &Font_7x10, 1);

    SSD1306_GotoXY(38, 45);
    SSD1306_Puts("SYSTEMS", &Font_7x10, 1);

    SSD1306_UpdateScreen(); // Flush buffer to display hardware
    HAL_Delay(2000);

    /* --- State 2: Dynamic Update / Developer Signature --- */
    SSD1306_Clear();

    SSD1306_GotoXY(25, 10);
    SSD1306_Puts("FEYZA:", &Font_7x10, 1);

    SSD1306_GotoXY(20, 30);
    SSD1306_Puts("YAGMUR", &Font_7x10, 1);

    SSD1306_GotoXY(30, 50);
    SSD1306_Puts("ARAT", &Font_7x10, 1);

    SSD1306_UpdateScreen(); // Flush buffer to display hardware
    HAL_Delay(2000);

    /* USER CODE END WHILE */
  }
}
