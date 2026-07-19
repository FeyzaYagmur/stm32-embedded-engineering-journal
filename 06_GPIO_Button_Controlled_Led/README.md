# Button Controlled LED

This project introduces digital input configurations into the embedded firmware architecture. It demonstrates how to poll the logical state of an external push-button switch using a GPIO input pin and actively trigger an external LED output based on that real-time state.

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **External Components:** 1x Push-Button, 1x Red LED, 2x Current-Limiting/Pull-Down Resistors
- **Active Pins:** `PD2` (Digital Input from Button), `PD3` (Digital Output to LED)
- **Method:** Digital State Polling (Input Reading & Direct Mapping)

## 🔍 Key Concepts Covered
- **GPIO Input Configuration:** Setting up a digital pin mode to read voltages rather than driving them, focusing on floating vs. referenced electrical levels.
- **External Pull-Down Circuitry:** Utilizing a physical pull-down resistor to tie the input pin safely to ground (`GND`) when the button is open, preventing noise.
- **Real-Time Polling Loops:** Using `HAL_GPIO_ReadPin()` inside the infinite processing cycle to instantly catch hardware state changes.

## 💻 Complete Source Code (`main.c` & GPIO Init)

Below is the complete hardware control logic and the peripheral configuration for the button-controlled LED implementation:

```c
/* GPIO Initialization Function */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  /* GPIO Ports Clock Enable */
  __HAL_RCC_GPIOH_CLK_ENABLE();
  __HAL_RCC_GPIOD_CLK_ENABLE();

  /* Configure GPIO pin Output Level */
  HAL_GPIO_WritePin(GPIOD, GPIO_PIN_3, GPIO_PIN_RESET);

  /* Configure GPIO pin : PD3 (External LED Output) */
  GPIO_InitStruct.Pin = GPIO_PIN_3;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(GPIOD, &GPIO_InitStruct);

  /* Configure GPIO pin : PD2 (External Button Input) */
  GPIO_InitStruct.Pin = GPIO_PIN_2;
  GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
  GPIO_InitStruct.Pull = GPIO_NOPULL; // Externally pulled down via hardware resistor
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
    /* Check if the external button on PD2 is pressed (High state) */
    if (HAL_GPIO_ReadPin(GPIOD, GPIO_PIN_2) == GPIO_PIN_SET)
    {
      /* Turn ON the external LED on PD3 */
      HAL_GPIO_WritePin(GPIOD, GPIO_PIN_3, GPIO_PIN_SET);
    }
    else
    {
      /* Turn OFF the external LED on PD3 */
      HAL_GPIO_WritePin(GPIOD, GPIO_PIN_3, GPIO_PIN_RESET);
    }

    /* USER CODE END WHILE */
  }
}
