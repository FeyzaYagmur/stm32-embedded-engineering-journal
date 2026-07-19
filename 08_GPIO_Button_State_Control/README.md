# Button State Control (Multi-State Switching)

This project implements a practical multi-state controller using software-based state machine principles. By tracking button press transitions and utilizing modulo arithmetic, a single push-button cycles the system through distinct operational modes, dynamically changing the blinking behavior of an external LED output.

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **External Components:** 1x Push-Button, 1x External LED, 2x Resistors
- **Active Pins:** `PD2` (Digital Input from Button), `PD3` (Digital Output to LED)
- **Method:** Multi-State Selection via Modulo Arithmetic (`%`)

## 🔍 Key Concepts Covered
- **Software State Machines:** Managing system behaviors by organizing code into distinct logical states (Modes 0, 1, and 2).
- **Modulo Control Sequences:** Using the `%` operator to wrap counter variables automatically within bounded operational limits (e.g., `currentState % 3`).
- **Dynamic Logic Toggling:** Shifting peripheral execution paths dynamically based on volatile control parameters updated in real-time by user inputs.

## 💻 Complete Source Code (`main.c` & GPIO Init)

Below is the complete hardware control logic and the peripheral configuration for the multi-state control implementation:

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

  uint8_t systemMode = 0;     // 0: LED OFF, 1: Fast Blink, 2: Slow Blink
  uint8_t lastButtonState = 0; // Tracks the previous button read

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    uint8_t currentButtonState = HAL_GPIO_ReadPin(GPIOD, GPIO_PIN_2);

    /* Edge Detection: Catch button press transition (Low to High) */
    if (currentButtonState == GPIO_PIN_SET && lastButtonState == 0)
    {
      systemMode = (systemMode + 1) % 3; // Cycle through modes: 0 -> 1 -> 2 -> 0
      HAL_Delay(50);                     // Debounce delay
    }
    
    lastButtonState = currentButtonState; // Save state for next cycle

    /* Execute behavior based on the current active system mode */
    switch (systemMode)
    {
      case 0:
        /* Mode 0: LED Completely OFF */
        HAL_GPIO_WritePin(GPIOD, GPIO_PIN_3, GPIO_PIN_RESET);
        break;

      case 1:
        /* Mode 1: Fast Blinking (100ms) */
        HAL_GPIO_TogglePin(GPIOD, GPIO_PIN_3);
        HAL_Delay(100);
        break;

      case 2:
        /* Mode 2: Slow Blinking (400ms) */
        HAL_GPIO_TogglePin(GPIOD, GPIO_PIN_3);
        HAL_Delay(400);
        break;

      default:
        systemMode = 0;
        break;
    }

    /* USER CODE END WHILE */
  }
}
