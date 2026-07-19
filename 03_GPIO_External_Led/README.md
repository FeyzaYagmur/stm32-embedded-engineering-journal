# External LED Control

This project marks the transition from using on-board indicators to interfacing with external hardware components. It demonstrates how to configure a GPIO pin to drive a single external LED connected via a breadboard, utilizing a current-limiting resistor to ensure safe hardware operations.

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **External Components:** 1x Red LED, 1x 220Ω or 330Ω Resistor
- **Active Pin:** `PD1` (Configured as External Output)
- **Method:** Active-High Output Control

## 🔍 Key Concepts Covered
- **Breadboard Interfacing:** Establishing physical signal and common ground (GND) connections between the STM32 Discovery board and external circuits.
- **Hardware Protection:** Understanding the necessity of current-limiting resistors to protect both the MCU pin and the external diode from overcurrent.
- **HAL Output Drivers:** Utilizing `HAL_GPIO_WritePin()` to set the output state explicitly high (`GPIO_PIN_SET`) and low (`GPIO_PIN_RESET`) within timed state transitions.

## 💻 Complete Source Code (`main.c` & GPIO Init)

Below is the complete hardware control logic and the peripheral configuration for driving the external LED:

```c
/* GPIO Initialization Function */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  /* GPIO Ports Clock Enable */
  __HAL_RCC_GPIOH_CLK_ENABLE();
  __HAL_RCC_GPIOD_CLK_ENABLE();

  /* Configure GPIO pin Output Level */
  HAL_GPIO_WritePin(GPIOD, GPIO_PIN_1, GPIO_PIN_RESET);

  /* Configure GPIO pin: PD1 (External LED Connection) */
  GPIO_InitStruct.Pin = GPIO_PIN_1;
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

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Turn ON the external LED */
    HAL_GPIO_WritePin(GPIOD, GPIO_PIN_1, GPIO_PIN_SET);
    HAL_Delay(500);
    
    /* Turn OFF the external LED */
    HAL_GPIO_WritePin(GPIOD, GPIO_PIN_1, GPIO_PIN_RESET);
    HAL_Delay(500);

    /* USER CODE END WHILE */
  }
}
