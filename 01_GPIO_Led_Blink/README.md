# On-Board LED Blink

This is the foundational "Hello World" application of embedded systems. The goal of this project is to configure the on-board Green LED of the STM32F407VG Discovery board and control its state with a simple time delay.

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **Active Pin:** `PD12` (On-Board Green LED)
- **Method:** Active-High Output Control

## 🔍 Key Concepts Covered
- **GPIO Initialization:** Enabling peripheral clocks for Port D and setting Pin 12 as a digital output in push-pull mode.
- **Precision Delays:** Utilizing `HAL_Delay()` driven by the internal SysTick timer for deterministic timing control.
- **Pin State Toggle:** Using `HAL_GPIO_TogglePin()` to easily invert the output state without manually tracking the current logical level.

## 💻 Complete Source Code (`main.c` & GPIO Init)

Below is the complete hardware control logic and the system configuration for the LED blink execution:

```c
/* GPIO Initialization Function */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  /* GPIO Ports Clock Enable */
  __HAL_RCC_GPIOH_CLK_ENABLE();
  __HAL_RCC_GPIOA_CLK_ENABLE();
  __HAL_RCC_GPIOD_CLK_ENABLE();

  /* Configure GPIO pin Output Level */
  HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12, GPIO_PIN_RESET);

  /* Configure GPIO pin: PD12 (On-Board Green LED) */
  GPIO_InitStruct.Pin = GPIO_PIN_12;
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
    /* Toggle the state of the On-board Green LED (PD12) */
    HAL_GPIO_TogglePin(GPIOD, GPIO_PIN_12);
    
    /* Delay of 500 milliseconds */
    HAL_Delay(500);

    /* USER CODE END WHILE */
  }
}

The physical hardware execution and test output for this project can be viewed inside the main repository video library.

https://drive.google.com/drive/folders/1QEF27VpM2UhSrB0GhmENcJqSesYxxM8_?usp=sharing
