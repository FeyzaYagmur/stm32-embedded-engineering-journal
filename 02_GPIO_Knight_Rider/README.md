# On-Board Knight Rider (Sequential LED Chaser)

This project implements a classic "Knight Rider" (sequential scanner) effect using the four on-board LEDs of the STM32F407VG Discovery board. It demonstrates handling arrays of pins, looping structures, and sequential GPIO driving.

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **Active Pins:** `PD12` (Green), `PD13` (Orange), `PD14` (Red), `PD15` (Blue)
- **Method:** Sequenced Output Control via Array Iteration

## 🔍 Key Concepts Covered
- **Pin Arrays:** Storing multiple GPIO pins in a structured array (`uint16_t ledArray[4]`) to allow clean iteration inside loops.
- **Bi-directional Looping:** Using sequential loops to scan forward (0 to 3) and then backward (2 down to 1) to create the continuous bouncing effect.
- **Dynamic Timing Control:** Utilizing `HAL_Delay()` to set the scan speed, showing how sequential states shift systematically over clock cycles.

## 💻 Complete Source Code (`main.c` & GPIO Init)

Below is the complete hardware control logic and the system configuration for the Knight Rider execution:

```c
/* GPIO Initialization Function */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  /* GPIO Ports Clock Enable */
  __HAL_RCC_GPIOH_CLK_ENABLE();
  __HAL_RCC_GPIOD_CLK_ENABLE();

  /* Configure GPIO pin Output Level */
  HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12|GPIO_PIN_13|GPIO_PIN_14|GPIO_PIN_15, GPIO_PIN_RESET);

  /* Configure GPIO pins : PD12 PD13 PD14 PD15 (On-Board LEDs) */
  GPIO_InitStruct.Pin = GPIO_PIN_12|GPIO_PIN_13|GPIO_PIN_14|GPIO_PIN_15;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(GPIOD, &GPIO_InitStruct);
}

/* Infinite Loop inside main() */
int main(void)
{
  /* Reset of all peripherals, Initializes the Flash interface and the Systick. */
  HAL_Init();

  /* Configure the system clock */
  SystemClock_Config();

  /* Initialize all configured peripherals */
  MX_GPIO_Init();

  /* Array storing the Pin definitions of on-board LEDs */
  uint16_t ledArray[4] = {GPIO_PIN_12, GPIO_PIN_13, GPIO_PIN_14, GPIO_PIN_15};

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Scan Forward: Green -> Orange -> Red -> Blue */
    for (int i = 0; i < 4; i++)
    {
      HAL_GPIO_WritePin(GPIOD, ledArray[i], GPIO_PIN_SET);
      HAL_Delay(100);
      HAL_GPIO_WritePin(GPIOD, ledArray[i], GPIO_PIN_RESET);
    }

    /* Scan Backward: Red -> Orange */
    for (int i = 2; i > 0; i--)
    {
      HAL_GPIO_WritePin(GPIOD, ledArray[i], GPIO_PIN_SET);
      HAL_Delay(100);
      HAL_GPIO_WritePin(GPIOD, ledArray[i], GPIO_PIN_RESET);
    }

    /* USER CODE END WHILE */
  }
}
