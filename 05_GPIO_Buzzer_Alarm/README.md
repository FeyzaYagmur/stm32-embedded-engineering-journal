# GPIO Buzzer Alarm System

This project introduces acoustic signaling peripherals into the embedded system layout. It demonstrates how to drive an external active buzzer using a GPIO output pin to create a structured sound-alert alarm system synchronized with timing delays.

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **External Components:** 1x Active Buzzer, 1x Protection/Current-Limiting Resistor
- **Active Pin:** `PD6` (Configured as External Output)
- **Method:** Active-High Acoustic Transduction

## 🔍 Key Concepts Covered
- **Acoustic Transducer Driving:** Configuring digital output signals to energize and de-energize an active buzzer's internal oscillator circuit.
- **Synchronized Visual-Audio Integration:** Structuring delays to orchestrate precise ON/OFF sound frequencies, mimicking industrial alarm conditions.
- **Inductive Load Considerations:** Learning the basic signal principles required when handling piezo/magnetic acoustic devices from sensitive microcontroller pins.

## 💻 Complete Source Code (`main.c` & GPIO Init)

Below is the complete hardware control logic and the peripheral configuration for running the Buzzer Alarm sequence:

```c
/* GPIO Initialization Function */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  /* GPIO Ports Clock Enable */
  __HAL_RCC_GPIOH_CLK_ENABLE();
  __HAL_RCC_GPIOD_CLK_ENABLE();

  /* Configure GPIO pin Output Level */
  HAL_GPIO_WritePin(GPIOD, GPIO_PIN_6, GPIO_PIN_RESET);

  /* Configure GPIO pin: PD6 (External Buzzer Connection) */
  GPIO_InitStruct.Pin = GPIO_PIN_6;
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
    /* Trigger Alarm: Sound ON */
    HAL_GPIO_WritePin(GPIOD, GPIO_PIN_6, GPIO_PIN_SET);
    HAL_Delay(200);
    
    /* Quiet State: Sound OFF */
    HAL_GPIO_WritePin(GPIOD, GPIO_PIN_6, GPIO_PIN_RESET);
    HAL_Delay(800);

    /* USER CODE END WHILE */
  }
}
