# External Knight Rider (Sequential External LED Chaser)

This project scales up external hardware interfacing by controlling a multi-LED sequential chaser circuit on a breadboard. It demonstrates synchronized multi-pin configuration, register-level array mapping, and continuous scanning logic applied to external components.

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **External Components:** 4x External LEDs, 4x Current-Limiting Resistors (220Ω/330Ω)
- **Active Pins:** `PD0`, `PD1`, `PD2`, `PD3` (Configured as External Outputs)
- **Method:** Sequenced Output Control via Array Iteration

## 🔍 Key Concepts Covered
- **Multi-Pin Bus Configurations:** Initializing continuous sequences of external pins within a single `GPIO_InitTypeDef` declaration utilizing bitwise OR (`|`) operations.
- **Hardware Tracking via Software:** Mapping active breadboard pin signals dynamically through index looping structures to generate flowing signal sweeps.
- **External Load Management:** Correctly distributing current pathways using discrete resistors for each external diode to safeguard internal MCU driver topologies.

## 💻 Complete Source Code (`main.c` & GPIO Init)

Below is the complete hardware control logic and the peripheral configuration for driving the external Knight Rider sequence:

```c
/* GPIO Initialization Function */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  /* GPIO Ports Clock Enable */
  __HAL_RCC_GPIOH_CLK_ENABLE();
  __HAL_RCC_GPIOD_CLK_ENABLE();

  /* Configure GPIO pin Output Level */
  HAL_GPIO_WritePin(GPIOD, GPIO_PIN_0|GPIO_PIN_1|GPIO_PIN_2|GPIO_PIN_3, GPIO_PIN_RESET);

  /* Configure GPIO pins : PD0 PD1 PD2 PD3 (External LED Connections) */
  GPIO_InitStruct.Pin = GPIO_PIN_0|GPIO_PIN_1|GPIO_PIN_2|GPIO_PIN_3;
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

  /* Array storing the Pin definitions of external breadboard LEDs */
  uint16_t externalLeds[4] = {GPIO_PIN_0, GPIO_PIN_1, GPIO_PIN_2, GPIO_PIN_3};

  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* Scan Forward: PD0 -> PD1 -> PD2 -> PD3 */
    for (int i = 0; i < 4; i++)
    {
      HAL_GPIO_WritePin(GPIOD, externalLeds[i], GPIO_PIN_SET);
      HAL_Delay(100);
      HAL_GPIO_WritePin(GPIOD, externalLeds[i], GPIO_PIN_RESET);
    }

    /* Scan Backward: PD2 -> PD1 */
    for (int i = 2; i > 0; i--)
    {
      HAL_GPIO_WritePin(GPIOD, externalLeds[i], GPIO_PIN_SET);
      HAL_Delay(100);
      HAL_GPIO_WritePin(GPIOD, externalLeds[i], GPIO_PIN_RESET);
    }

    /* USER CODE END WHILE */
  }
}
