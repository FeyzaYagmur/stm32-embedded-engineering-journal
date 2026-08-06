# HC-05 Bluetooth Wireless Multi-Channel Command Control & Error Feedback

This project demonstrates bi-directional wireless hardware control using the **HC-05 Bluetooth TTL Transceiver Module** interfaced with an STM32 microcontroller. The firmware receives ASCII single-byte control frames wirelessly over USART, parses multi-channel output requests (`PB0` and `PB1` pins), and responds with error status feedback (`"Yanlis Girdiniz\r\n"`) upon receiving invalid commands.

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **Wireless Peripheral:** HC-05 Bluetooth 2.0 Module (TTL SPP)
- **Peripherals Used:** USART1 / USART2, GPIOB Output Drivers
- **Active Pins:** `PA10` (USART1_RX), `PB0` (Output Channel 1), `PB1` (Output Channel 2)
- **Method:** Blocking Multi-State Command Parsing via `HAL_UART_Receive()`

## 🔍 Key Concepts Covered
- **Multi-Channel Wireless Actuation:** Processing individual byte directives (`'0'`, `'1'`, `'2'`, `'3'`) to manipulate discrete GPIO output pins (`PB0` / `PB1`) independently.
- **Wireless Error Handling & Telemetry Feedback:** Trapping invalid user terminal inputs inside an `else` condition and returning diagnostic feedback strings over Bluetooth.
- **Status Machine Execution:** Integrating `HAL_OK` return status verification on `HAL_UART_Receive` to ensure reliable payload processing.

## 💻 Complete Source Code (`main.c` / Defterdeki Receive Kodu ile Birebir)

Below is the complete C implementation written in STM32CubeIDE using native HAL drivers:

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
