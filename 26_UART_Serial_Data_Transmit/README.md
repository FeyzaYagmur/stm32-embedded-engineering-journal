# UART Serial Communication & Data Transmission

This project demonstrates asynchronous serial data communication using the Universal Synchronous Asynchronous Receiver Transmitter (USART2) peripheral on an STM32 microcontroller. The setup transmits structured text strings to a host PC serial terminal (Termite) via an onboard USB-to-UART bridge using blocking HAL transmission routines.

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **Peripheral:** USART2 (Asynchronous Mode)
- **Baud Rate:** 115200 bps (8 Data Bits, 1 Stop Bit, No Parity)
- **Active Pins:** `PA2` (USART2_TX Output Pin)
- **Host Terminal:** Termite 3.4 Serial Monitor
- **Method:** Polling-Based Data Transmission via `HAL_UART_Transmit()`

## 🔍 Key Concepts Covered
- **Asynchronous Serial Framing:** Configuring standard UART packet structures (Baud Rate, Frame Bits, Parity) for seamless inter-device serial communication.
- **Buffer Management & Null Termination Handling:** Utilizing `sizeof(mesaj) - 1` and `\r\n` (CRLF) formatting to strip null-terminators (`\0`) while formatting multi-line serial outputs.
- **Blocking Mode Transmission:** Implementing timeout-managed data transfer via `HAL_UART_Transmit()` to ensure complete packet transmission to the host receiver.

## 💻 Complete Source Code (`main.c` & USART2 Init)

Below is the complete C implementation written in STM32CubeIDE using native HAL drivers:

```c
#include "main.h"
#include <string.h>

UART_HandleTypeDef huart2;

/* USART2 Peripheral Initialization Function */
static void MX_USART2_UART_Init(void)
{
  huart2.Instance = USART2;
  huart2.Init.BaudRate = 115200;
  huart2.Init.WordLength = UART_WORDLENGTH_8B;
  huart2.Init.StopBits = UART_STOPBITS_1;
  huart2.Init.Parity = UART_PARITY_NONE;
  huart2.Init.Mode = UART_MODE_TX_RX;
  huart2.Init.HwFlowCtl = UART_HWCONTROL_NONE;
  huart2.Init.OverSampling = UART_OVERSAMPLING_16;
  HAL_UART_Init(&huart2);
}

/* GPIO Initialization Function */
static void MX_GPIO_Init(void)
{
  __HAL_RCC_GPIOH_CLK_ENABLE();
  __HAL_RCC_GPIOA_CLK_ENABLE();
}

int main(void)
{
  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();
  MX_USART2_UART_Init();

  /* Custom Transmit Buffer Declaration */
  uint8_t mesaj[] = "Feyza Yagmur - UART \r\n";

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Transmit message over USART2 bus (excluding null terminator) */
    HAL_UART_Transmit(&huart2, mesaj, sizeof(mesaj) - 1, 1000);

    /* Transmit Delay Interval */
    HAL_Delay(1000);

    /* USER CODE END WHILE */
  }
}
