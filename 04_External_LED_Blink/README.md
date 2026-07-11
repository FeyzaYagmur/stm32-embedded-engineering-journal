# Step 4: Interfacing and Driving an External LED via GPIO

In this project, I moved beyond the on-board peripherals to interface an external hardware component. I configured a generic red LED connected via a breadboard circuit to the `PA0` pin on the STM32 microcontroller.

The objective is to master external hardware interfacing, output current sourcing, and basic breadboard circuit building techniques.

## Hardware Configuration
* **Microcontroller Pin:** `PA0` (GPIOA, Pin 0)
* **Circuit Components:** External Red LED, Current-Limiting Resistor, Jumper Wires, External Breadboard.
* **Working Principle:** The `PA0` pin is configured as a GPIO Output. When set to `SET` (1), it outputs 3.3V, driving current through the resistor and illuminating the LED. Setting it to `RESET` (0) cuts the voltage loop.

## Code Architecture
The implementation targets the `GPIOA` peripheral directly using the standard HAL API configuration:

```c
/* Infinite loop */
/* USER CODE BEGIN WHILE */
while (1)
{
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0, GPIO_PIN_SET);
    HAL_Delay(1000);
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0, GPIO_PIN_RESET);
    HAL_Delay(1000);
    
    /* USER CODE END WHILE */
    /* USER CODE BEGIN 3 */
}
