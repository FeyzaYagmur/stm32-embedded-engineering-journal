# Step 8: Stateful Input Tracking and Dynamic Button Counter

In this project, I implemented volatile variable structures to track input states dynamically. Beyond simple state polling, this firmware stores a historical state count inside the memory architecture, incrementing a software counter each time a physical input event is validated.

The objective is to master memory-variable declarations (`uint32_t`), runtime arithmetic increment operations, and dynamic execution feedback.

## Hardware Configuration
* **Input Pin:** `PC6` (GPIOC, Pin 6) monitored via an external push-button.
* **Output Pin:** `PB0` (GPIOB, Pin 0) driving an external Blue LED.
* **Working Principle:** When the button is pressed, `PC6` reads a logic HIGH (`GPIO_PIN_SET`). The software catches this state, increments the `counter` variable in memory by one (`counter++`), and flashes the blue LED for 200 milliseconds to provide immediate visual confirmation of the registered event.

## Code Architecture & Variable Mapping
The code utilizes globally declared variables to store state values outside the main execution loop, preventing them from being reset during cyclic execution:

```c
/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */
uint32_t counter = 0;      // Event accumulator register
uint8_t buttonState = 0;   // Instantaneous button status buffer
/* USER CODE END 0 */

/* Infinite loop */
/* USER CODE BEGIN WHILE */
while (1)
{
    // Read and buffer the current pin state
    buttonState = HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_6);
    
    // Condition check for button press
    if (buttonState == GPIO_PIN_SET)
    {
        counter++; // Increment the software counter
        
        // Visual feedback routine
        HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, GPIO_PIN_SET);   // Turn LED ON
        HAL_Delay(200);                                       // Maintain state
        HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, GPIO_PIN_RESET); // Turn LED OFF
    }
    
    /* USER CODE END WHILE */
    /* USER CODE BEGIN 3 */
}
