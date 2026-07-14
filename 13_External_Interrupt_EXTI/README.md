# Step 13: Hardware External Interrupts (EXTI) and Callback Control

In this project, I moved away from inefficient software polling routines to implement a hardware-driven External Interrupt (EXTI) configuration. Using the on-board blue User Button (`PA0`), I triggered an asynchronous hardware event that overrides the main background execution loop to flash a specific combination of on-board LEDs via the EXTI service pipeline.

The objective is to master External Interrupt lines (EXTI), Interrupt Service Routines (ISR), Callback management, and hardware priority switching.

## Hardware Configuration
* **Interrupt Source Pin:** On-Board User Button wired to `PA0` (configured in `GPIO_MODE_EVT_RISING` / `EXTI_Line0`).
* **Background & Target Peripherals:** On-board LEDs on `GPIOD` (`PD12`, `PD13`, `PD14`).
* **Working Principle:** The microcontroller runs a sequential scanning loop in the background. When the button on `PA0` transitions from low to high, it causes a hardware rising edge. The Nested Vectored Interrupt Controller (NVIC) immediately pauses the central processing pipeline, executes the custom handler callback, and then drops back into the background sequence exactly where it left off.

## Code Architecture & Interrupt Handling
The background sequential loop runs inside the infinite execution block, while the high-priority alert routine is isolated cleanly inside the asynchronous callback framework:

```c
/* Infinite loop */
/* USER CODE BEGIN WHILE */
while (1)
{
    // Background Task: Continuous low-priority LED scanning array
    uint16_t leds[3] = {GPIO_PIN_12, GPIO_PIN_13, GPIO_PIN_14};
    
    for (int i = 0; i < 3; i++)
    {
        HAL_GPIO_WritePin(GPIOD, leds[i], GPIO_PIN_SET);
        HAL_Delay(300);
        HAL_GPIO_WritePin(GPIOD, leds[i], GPIO_PIN_RESET);
    }
    for (int i = 1; i > 0; i--)
    {
        HAL_GPIO_WritePin(GPIOD, leds[i], GPIO_PIN_SET);
        HAL_Delay(300);
        HAL_GPIO_WritePin(GPIOD, leds[i], GPIO_PIN_RESET);
    }
    
    /* USER CODE END WHILE */
}

/* USER CODE BEGIN 4 */
// Asynchronous External Interrupt Callback Service Routine
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
    // Verify that the trigger source originated from EXTI Line 0 (PA0)
    if (GPIO_Pin == GPIO_PIN_0)
    {
        // High-Priority Task: Flash all three target LEDs simultaneously 4 times
        for (int k = 0; k < 4; k++)
        {
            HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12 | GPIO_PIN_13 | GPIO_PIN_14, GPIO_PIN_SET);
            HAL_Delay(200);
            HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12 | GPIO_PIN_13 | GPIO_PIN_14, GPIO_PIN_RESET);
            HAL_Delay(200);
        }
    }
}
/* USER CODE END 4 */
