# SPI Full-Duplex TransmitReceive Communication & Hardware Loopback Verification

This project demonstrates full-duplex serial communication using the **Serial Peripheral Interface 1 (SPI1)** peripheral on an STM32 microcontroller. Utilizing the simultaneous blocking routine `HAL_SPI_TransmitReceive()`, the firmware sends data packets over the MOSI line while concurrently capturing incoming bytes from the MISO line via hardware loopback wire jumpers, validating payload integrity (`0x55`) to toggle status indicators.

##  Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **Peripheral:** SPI1 (Full-Duplex Master Mode)
- **Active Pins:** `PA5` (SPI1_SCK), `PA6` (SPI1_MISO), `PA7` (SPI1_MOSI), `PD12` (Onboard Green LED Output)
- **Test Setup:** Hardware Loopback (Physical connection jumper between `PA6` MISO and `PA7` MOSI)
- **Method:** Synchronous Blocking Bidirectional Transfer via `HAL_SPI_TransmitReceive()`

##  Key Concepts Covered
- **Full-Duplex SPI Architecture:** Simultaneous bidirectional shift register data clocking (tx/rx) driven by Master Clock (SCK) pulses.
- **Hardware Loopback Bus Verification:** Directing MOSI output signals physically back into MISO input pins to test driver stack reliability without external slave chips.
- **Payload Validation & GPIO Toggling:** Comparing received SPI byte buffers (`gelenVeri == 0x55`) to drive conditional LED outputs (`PD12`).

##  Complete Source Code (`main.c` / Videodaki Kod ile Birebir)

Below is the complete C implementation written in STM32CubeIDE using native HAL drivers:

```c
#include "main.h"

SPI_HandleTypeDef hspi1;

/* SPI1 Peripheral Initialization Function */
static void MX_SPI1_Init(void)
{
  hspi1.Instance = SPI1;
  hspi1.Init.Mode = SPI_MODE_MASTER;
  hspi1.Init.Direction = SPI_DIRECTION_2LINES;
  hspi1.Init.DataSize = SPI_DATASIZE_8BIT;
  hspi1.Init.CLKPolarity = SPI_POLARITY_LOW;
  hspi1.Init.CLKPhase = SPI_PHASE_1EDGE;
  hspi1.Init.NSS = SPI_NSS_SOFT;
  hspi1.Init.BaudRatePrescaler = SPI_BAUDRATEPRESCALER_16;
  hspi1.Init.FirstBit = SPI_FIRSTBIT_MSB;
  hspi1.Init.TIMode = SPI_TIMODE_DISABLE;
  hspi1.Init.CRCCalculation = SPI_CRCCALCULATION_DISABLE;
  hspi1.Init.CRCPolynomial = 10;
  HAL_SPI_Init(&hspi1);
}

/* GPIO Initialization Function */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  __HAL_RCC_GPIOA_CLK_ENABLE();
  __HAL_RCC_GPIOD_CLK_ENABLE();

  /* Configure Onboard Green LED (PD12) */
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
  MX_SPI1_Init();

  uint8_t gidenVeri = 0x55;
  uint8_t gelenVeri = 0x00;

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Simultaneous Transmit and Receive over SPI1 */
    HAL_SPI_TransmitReceive(&hspi1, &gidenVeri, &gelenVeri, 1, 100);

    /* Validate incoming SPI loopback data payload */
    if (gelenVeri == 0x55)
    {
      HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12, GPIO_PIN_SET);   // Turn ON LED
    }
    else
    {
      HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12, GPIO_PIN_RESET); // Turn OFF LED
    }

    HAL_Delay(500);

    /* USER CODE END WHILE */
  }
}
