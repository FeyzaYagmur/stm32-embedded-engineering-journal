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
