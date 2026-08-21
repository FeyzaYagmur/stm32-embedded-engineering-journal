# HC-05 Bluetooth Wireless Serial Data Transmission (TTL)

This project demonstrates wireless serial data transmission using the **HC-05 Bluetooth TTL Transceiver Module** interfaced with an STM32 microcontroller via USART peripheral. The firmware transmits structured wireless telemetry strings (`"TTL uzerinden veri aktariliyor\r\n"`) to a paired host Bluetooth terminal (Termite / Mobile Terminal) using polling-based HAL UART routines.

##  Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **Wireless Component:** HC-05 Bluetooth 2.0 SPP Transceiver (TTL Level)
- **Peripheral Used:** USART1 / USART2 (Asynchronous Mode)
- **Baud Rate:** 9600 bps / 115200 bps (8 Data Bits, 1 Stop Bit, No Parity)
- **Active Pins:** `PA9` (USART1_TX / Bluetooth RX Connection)
- **Method:** Wireless Transmit Loop via `HAL_UART_Transmit()`

##  Key Concepts Covered
- **Wireless Serial Pass-Through (TTL):** Utilizing the transparent SPP (Serial Port Profile) of HC-05 to bridge microcontroller USART pins wirelessly with host terminals.
- **UART Buffer Precision:** Applying `sizeof(testMesaji) - 1` and explicit type casting `(uint8_t*)` to transmit exact string boundaries without sending trailing null bytes (`\0`).
- **Telemetry Streaming:** Establishing periodic ($1000\text{ms}$) wireless telemetry reporting for remote monitoring applications.

##  Complete Source Code (`main.c` / Defterdeki Transmit Kodu ile Birebir)

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
  __HAL_RCC_GPIOA_CLK_ENABLE();
}

int main(void)
{
  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();
  MX_USART1_UART_Init();

  /* Custom Transmit Buffer Declaration */
  char testMesaji[] = "TTL uzerinden veri aktariliyor\r\n";

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Transmit string wirelessly over HC-05 Bluetooth module */
    HAL_UART_Transmit(&huart1, (uint8_t *)testMesaji, sizeof(testMesaji) - 1, 100);

    /* Delay interval between transmissions */
    HAL_Delay(1000);

    /* USER CODE END WHILE */
  }
}
