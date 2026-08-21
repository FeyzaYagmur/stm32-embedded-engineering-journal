# HC-05 Bluetooth Multi-Channel Command Control & Error Feedback System

> **Hardware & Setup Note:** This project demonstrates UART/USART communication protocol logic using STM32 HAL libraries. Although the curriculum demonstrates data transmission via an HC-05 Bluetooth module, the underlying protocol remains standard UART. Therefore, physical hardware verification was successfully conducted using a USB-to-TTL converter connected directly to the STM32 USART TX/RX pins, communicating with a PC serial terminal (Termite).

This project demonstrates bidirectional serial hardware actuation using an STM32 microcontroller. The firmware receives single-byte ASCII command frames over USART1, independently manages multi-channel GPIO outputs (`PB0` and `PB1`), and provides dynamic error status telemetry (`Yanlis Girdiniz\r\n`) upon receiving unmapped instructions.

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **Serial Interface:** USB-to-TTL Converter (CP2102 / CH340) / HC-05 Bluetooth SPP
- **Peripheral:** USART1 (115200 bps, 8 Data Bits, 1 Stop Bit, No Parity - 8N1)
- **Active Pins:** 
  - `PA10` (USART1_RX), `PA9` (USART1_TX)
  - `PB0` (Output Channel 1 - LED 1)
  - `PB1` (Output Channel 2 - LED 2)
- **Host Terminal:** Termite 3.4 Serial Monitor
- **Method:** Polling Reception with Return Status Validation (`HAL_UART_Receive`)

## 🔍 Key Concepts Covered
- **Multi-Channel Actuation:** Parsing discrete command bytes (`'0'`, `'1'`, `'2'`, `'3'`) to control independent output lines.
- **Wireless Error Handling & Telemetry:** Catching invalid command inputs in an `else` branch and transmitting formatted error feedback over UART.
- **State Validation:** Ensuring robust frame reception through `HAL_OK` status checks before processing buffer payloads.

## 💻 Complete Source Code (`main.c` & Init)

```c
#include "main.h"
#include <string.h>

UART_HandleTypeDef huart1;

/* USART1 Peripheral Initialization Function */
static void MX_USART1_UART_Init(void)
{
  huart1.Instance = USART1;
  huart1.Init.BaudRate = 115200;
  huart1.Init.WordLength = UART_WORDLENGTH_8B;
  huart1.Init.StopBits = UART_STOPBITS_1;
  huart1.Init.Parity = UART_PARITY_NONE;
  huart1.Init.Mode = UART_MODE_TX_RX;
  huart1.Init.HwFlowCtl = UART_HWCONTROL_NONE;
  huart1.Init.OverSampling = UART_OVERSAMPLING_16;
  HAL_UART_Init(&huart1);
}

/* GPIO Initialization Function */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  __HAL_RCC_GPIOA_CLK_ENABLE();
  __HAL_RCC_GPIOB_CLK_ENABLE();

  /* Configure PB0 and PB1 as Push-Pull Outputs */
  GPIO_InitStruct.Pin = GPIO_PIN_0 | GPIO_PIN_1;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(GPIOB, &GPIO_InitStruct);

  /* Set initial pin states to LOW */
  HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0 | GPIO_PIN_1, GPIO_PIN_RESET);
}

int main(void)
{
  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();
  MX_USART1_UART_Init();

  uint8_t gelenVeri;

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Receive 1 byte wirelessly over Bluetooth with status check */
    if (HAL_UART_Receive(&huart1, &gelenVeri, 1, HAL_MAX_DELAY) == HAL_OK)
    {
      /* Command '0': Set PB0 HIGH */
      if (gelenVeri == '0')
      {
        HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, GPIO_PIN_SET);
      }
      /* Command '1': Set PB1 HIGH */
      else if (gelenVeri == '1')
      {
        HAL_GPIO_WritePin(GPIOB, GPIO_PIN_1, GPIO_PIN_SET);
      }
      /* Command '2': Reset PB0 LOW */
      else if (gelenVeri == '2')
      {
        HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, GPIO_PIN_RESET);
      }
      /* Command '3': Reset PB1 LOW */
      else if (gelenVeri == '3')
      {
        HAL_GPIO_WritePin(GPIOB, GPIO_PIN_1, GPIO_PIN_RESET);
      }
      /* Invalid Command Handling */
      else
      {
        char hata[] = "Yanlis Girdiniz\r\n";
        HAL_UART_Transmit(&huart1, (uint8_t *)hata, sizeof(hata) - 1, 100);
      }
    }

    /* USER CODE END WHILE */
  }
}
