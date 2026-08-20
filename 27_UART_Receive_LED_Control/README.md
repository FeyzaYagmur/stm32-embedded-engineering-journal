# UART Bi-Directional Serial Receive & Command Control System

This project demonstrates two-way (full-duplex / bi-directional) serial communication using the Universal Synchronous Asynchronous Receiver Transmitter (USART2) peripheral on an STM32 microcontroller. The firmware listens for incoming ASCII command characters (`'1'` and `'0'`) sent from a host PC serial terminal (Termite) via blocking `HAL_UART_Receive()` routines, toggles an onboard LED, and transmits confirmation status messages back to the host interface.

##  Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **Peripheral:** USART2 (Asynchronous Full-Duplex Mode)
- **Baud Rate:** 115200 bps (8 Data Bits, 1 Stop Bit, No Parity)
- **Active Pins:** `PA2` (USART2_TX), `PA3` (USART2_RX), `PD12` (Onboard Green LED Output)
- **Host Terminal:** Termite 3.4 Serial Monitor
- **Method:** Blocking Serial Reception via `HAL_UART_Receive(&huart2, &rxData, 1, HAL_MAX_DELAY)`

##  Key Concepts Covered
- **Bi-Directional Serial Command Processing:** Parsing incoming ASCII byte values to execute hardware state changes and returning status confirmation strings over the same bus.
- **Blocking Mode RX Handling:** Utilizing `HAL_MAX_DELAY` inside polling-based reception calls to halt thread execution until a valid byte arrives in the UART RX buffer register.
- **Hardware Feedback Integration:** Driving onboard status LEDs conditionally (`HAL_GPIO_WritePin`) based on serial terminal command inputs.

##  Complete Source Code (`main.c` & USART2 Command Handling)

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
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  __HAL_RCC_GPIOH_CLK_ENABLE();
  __HAL_RCC_GPIOA_CLK_ENABLE();
  __HAL_RCC_GPIOD_CLK_ENABLE();

  /* Reset Onboard Green LED */
  HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12, GPIO_PIN_RESET);

  /* Configure GPIOD Pin 12 as Output */
  GPIO_InitStruct.Pin = GPIO_PIN_12;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(GPIOD, &GPIO_InitStruct);
}

int main(void)
{
  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();
  MX_USART2_UART_Init();

  uint8_t rxData = 0;

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Wait continuously until 1 byte is received over USART2 */
    HAL_UART_Receive(&huart2, &rxData, 1, HAL_MAX_DELAY);

    /* Command Processing: Turn ON LED when '1' is received */
    if (rxData == '1')
    {
      HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12, GPIO_PIN_SET);
      HAL_UART_Transmit(&huart2, (uint8_t *)"Led Yandi\r\n", 11, 1000);
    }
    /* Command Processing: Turn OFF LED when '0' is received */
    else if (rxData == '0')
    {
      HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12, GPIO_PIN_RESET);
      HAL_UART_Transmit(&huart2, (uint8_t *)"Led Sondo\r\n", 11, 1000);
    }

    /* USER CODE END WHILE */
  }
}
