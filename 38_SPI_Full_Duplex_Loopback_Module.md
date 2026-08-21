# SPI Full-Duplex Transmit/Receive & Modular Hardware Loopback Driver

This project implements a modular SPI loopback driver on an STM32 microcontroller using the Serial Peripheral Interface 1 (SPI1) peripheral. The driver abstracts full-duplex hardware verification routines (`HAL_SPI_TransmitReceive()`), enabling single-byte testing and multi-byte buffer comparison (`memcmp()`) across physical MOSI-to-MISO loopback jumpers with onboard status LED feedback (`PD12`).

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4 Core)
- **Peripheral:** SPI1 (Full-Duplex Master Mode, 8-Bit Data Frame)
- **Active Pins:** 
  - `PA5` (SPI1_SCK - Serial Clock)
  - `PA6` (SPI1_MISO - Master In Slave Out)
  - `PA7` (SPI1_MOSI - Master Out Slave In)
  - `PD12` (Onboard Green LED Output)
- **Hardware Setup:** Single Jumper Loopback bridging `PA6` (MISO) and `PA7` (MOSI)
- **Verification Method:** Physical LED Indicator & Live Expressions Watch (`gelenVeri = 0x55 / 85`)

## 🔍 Key Concepts Covered
- **Modular Driver Architecture:** Encapsulating peripheral handles and output indicators within a custom C structure (`SPI_Loopback_t`).
- **Synchronous Full-Duplex Clocking:** Simultaneously transmitting from MOSI while latching bytes from MISO on SCK edge transitions.
- **Payload Integrity Validation:** Comparing captured frame buffers against source payloads to verify hardware integrity before driving GPIO states.

---

## 💻 Complete Production Source Code

```c
/* ==============================================================================
 * 1. DOSYA: Core/Inc/spi_loopback.h (Sürücü Başlık Dosyası)
 * ============================================================================== */

#ifndef INC_SPI_LOOPBACK_H_
#define INC_SPI_LOOPBACK_H_

#include "main.h"

/* SPI Loopback Cihaz Yapısı */
typedef struct {
    SPI_HandleTypeDef *hspi;
    GPIO_TypeDef      *led_port;
    uint16_t           led_pin;
} SPI_Loopback_t;

/* Fonksiyon Prototipleri */
void SPI_Loopback_Init(SPI_Loopback_t *dev, SPI_HandleTypeDef *hspi, GPIO_TypeDef *led_port, uint16_t led_pin);
HAL_StatusTypeDef SPI_Loopback_TestByte(SPI_Loopback_t *dev, uint8_t tx_val, uint8_t *rx_val);
uint8_t SPI_Loopback_TestBuffer(SPI_Loopback_t *dev, uint8_t *pTx, uint8_t *pRx, uint16_t size);

#endif /* INC_SPI_LOOPBACK_H_ */


/* ==============================================================================
 * 2. DOSYA: Core/Src/spi_loopback.c (Sürücü Gövde Dosyası)
 * ============================================================================== */

#include "spi_loopback.h"
#include <string.h>

/* Modülü Başlatma */
void SPI_Loopback_Init(SPI_Loopback_t *dev, SPI_HandleTypeDef *hspi, GPIO_TypeDef *led_port, uint16_t led_pin)
{
    dev->hspi = hspi;
    dev->led_port = led_port;
    dev->led_pin = led_pin;
}

/* Tek Bayt Gönderip TransmitReceive İle Okuma */
HAL_StatusTypeDef SPI_Loopback_TestByte(SPI_Loopback_t *dev, uint8_t tx_val, uint8_t *rx_val)
{
    return HAL_SPI_TransmitReceive(dev->hspi, &tx_val, rx_val, 1, 100);
}

/* Blok Tampon Doğrulama */
uint8_t SPI_Loopback_TestBuffer(SPI_Loopback_t *dev, uint8_t *pTx, uint8_t *pRx, uint16_t size)
{
    HAL_StatusTypeDef status;
    
    status = HAL_SPI_TransmitReceive(dev->hspi, pTx, pRx, size, 500);
    
    if (status == HAL_OK && memcmp(pTx, pRx, size) == 0)
    {
        /* Veri birebir eşleşti -> LED'i Yak */
        HAL_GPIO_WritePin(dev->led_port, dev->led_pin, GPIO_PIN_SET);
        return 1;
    }
    else
    {
        /* Hata -> LED'i Söndür */
        HAL_GPIO_WritePin(dev->led_port, dev->led_pin, GPIO_PIN_RESET);
        return 0;
    }
}


/* ==============================================================================
 * 3. DOSYA: Core/Src/main.c (Ana Uygulama Dosyası)
 * ============================================================================== */

#include "main.h"
#include "spi_loopback.h"

SPI_HandleTypeDef hspi1;
SPI_Loopback_t spi_test;

uint8_t gidenVeri = 0x55;  /* Test verisi */
uint8_t gelenVeri = 0x00;
uint8_t loopback_success = 0;

void SystemClock_Config(void);
static void MX_GPIO_Init(void);
static void MX_SPI1_Init(void);

int main(void)
{
    HAL_Init();
    SystemClock_Config();
    MX_GPIO_Init();
    MX_SPI1_Init();

    /* SPI Loopback modülünü PD12 (Yeşil LED) ile bağla */
    SPI_Loopback_Init(&spi_test, &hspi1, GPIOD, GPIO_PIN_12);

    while (1)
    {
        /* 1. TransmitReceive ile MOSI->MISO hattından veriyi oku */
        SPI_Loopback_TestByte(&spi_test, gidenVeri, &gelenVeri);

        /* 2. Doğrulama mantığı: gelenVeri == 0x55 ise PD12'yi yak */
        if (gelenVeri == 0x55)
        {
            HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12, GPIO_PIN_SET);
            loopback_success = 1;
        }
        else
        {
            HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12, GPIO_PIN_RESET);
            loopback_success = 0;
        }

        HAL_Delay(500);
    }
}

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

static void MX_GPIO_Init(void)
{
    GPIO_InitTypeDef GPIO_InitStruct = {0};

    __HAL_RCC_GPIOA_CLK_ENABLE();
    __HAL_RCC_GPIOD_CLK_ENABLE();

    /* Configure PD12 as Push-Pull Output */
    GPIO_InitStruct.Pin = GPIO_PIN_12;
    GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
    GPIO_InitStruct.Pull = GPIO_NOPULL;
    GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
    HAL_GPIO_Init(GPIOD, &GPIO_InitStruct);
}
