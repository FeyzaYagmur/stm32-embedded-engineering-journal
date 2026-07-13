# Step 10: Active-Low Input Configuration and Dynamic Pulse Counting

In this project, I implemented an active-low button control sequence using an external pull-up resistor configuration. By shifting from standard active-high polling to active-low logical inversion, the microcontroller increments a state parameter (`buttonCounter`) and triggers output channel `PB0` upon grounding the input pin `PC6`.

The objective is to master input logic inversion (Active-Low configuration), optimized debounce timing, and input data register filtering.

## Hardware Configuration
* **Input Pin:** `PC6` (GPIOC, Pin 6) connected to an external push-button in a pull-up circuit configuration.
* **Output Pin:** `PB0` (GPIOB, Pin 0) connected to an external Red LED.
* **Working Principle:** In an active-low (pull-up) setup, the input pin `PC6` is default-held at VCC (3.3V / `SET`) through a resistor. Pressing the button closes the loop to ground (GND / `RESET`). The microcontroller reads this transition, detects the logic level drop, and interprets the inverted state as a user-initiated activation.

## Code Architecture & Logical Inversion
The program configures local memory parameters and polls the input register using bitwise negation (`!`) to filter signal states:

```c
/* USER CODE BEGIN PV */
uint8_t buttonState = 1;      // Default high level state storage
uint16_t buttonCounter = 0;   // Inverted transition activation counter
/* USER CODE END PV */

/* Infinite loop */
/* USER CODE BEGIN WHILE */
while (1)
{
    // Read the current state of the active-low button
    buttonState = HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_6);
    
    // Optimized 50ms software debounce filter window
    HAL_Delay(50);
    
    // Logical negation: True (1) when buttonState is RESET (0)
    if (!buttonState)
    {
        HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, GPIO_PIN_SET); // Drive the LED on active low
        buttonCounter++;                                    // Increment activation count
    }
    else
    {
        HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, GPIO_PIN_RESET);
    }
    
    /* USER CODE END WHILE */
    /* USER CODE BEGIN 3 */
}
