# Step 9: Multi-LED State Control via Modulo Operations

In this project, I designed a multi-state control system using an external push-button and three external LEDs (Red, Yellow, and Blue) connected to `GPIOB` (`PB0`, `PB1`, `PB2`). The project introduces sequential state logic controlled dynamically by mathematical operations.

The objective is to master dynamic multi-peripheral control, logic isolation using conditional state machines, and implementation of modular arithmetic (`%`) in embedded systems.

## Hardware Configuration
* **Input Pin:** `PC6` (GPIOC, Pin 6) connected to an external push-button.
* **Output Pins:** `PB0` (Red LED), `PB1` (Yellow LED), `PB2` (Blue LED) on `GPIOB`.
* **Working Principle:** The system evaluates the activation state when the push-button is pressed. Based on the calculated state index, the microcontroller isolates the current operation path to light up a specific LED combination while holding a stable software debounce lock.

## Code Architecture & State Logic
The execution architecture utilizes global parameter storage and handles sequential output paths within the primary system runtime loop:

```c
/* USER CODE BEGIN PV */
uint32_t counter = 0; // Global state tracking parameter
/* USER CODE END PV */

/* Infinite loop */
/* USER CODE BEGIN WHILE */
while (1)
{
    if (HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_6) == GPIO_PIN_SET)
    {
        counter++;
        HAL_Delay(1000); // Debounce delay and state window control
        
        if (counter % 3 == 1)
        {
            HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, GPIO_PIN_SET);
            HAL_GPIO_WritePin(GPIOB, GPIO_PIN_1 | GPIO_PIN_2, GPIO_PIN_RESET);
        }
        else if (counter % 3 == 2)
        {
            HAL_GPIO_WritePin(GPIOB, GPIO_PIN_1, GPIO_PIN_SET);
            HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0 | GPIO_PIN_2, GPIO_PIN_RESET);
        }
        else if (counter % 3 == 0)
        {
            // State 3: All external LEDs illuminated simultaneously
            HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0 | GPIO_PIN_1 | GPIO_PIN_2, GPIO_PIN_SET);
        }
    }
    else
    {
        // Default State: Turn off all outputs when button is unpressed
        HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0 | GPIO_PIN_1 | GPIO_PIN_2, GPIO_PIN_RESET);
    }
    
    /* USER CODE END WHILE */
    /* USER CODE BEGIN 3 */
}
