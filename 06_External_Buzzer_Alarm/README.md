# Step 6: Interfacing an External Active Buzzer for Acoustic Telemetry

In this project, I expanded my external hardware interfacing by introducing an acoustic output device. I connected an external active buzzer module to the `PA5` pin on the STM32 microcontroller via a breadboard to build a state-based audio alarm system.

The goal is to master timed frequency-triggering, signal loops, and audio telemetry in embedded development.

## Hardware Configuration
* **Microcontroller Pin:** `PA5` (GPIOA, Pin 5)
* **Circuit Components:** External Active Buzzer, Jumper Wires, External Breadboard.
* **Working Principle:** An active buzzer contains an internal oscillating source. Unlike a passive buzzer that requires a PWM frequency sequence, this component requires a simple DC voltage loop. When `PA5` is set to `SET` (1), it outputs 3.3V, triggering the internal circuit to generate a continuous tone. Setting it to `RESET` (0) mutes the device.

## Code Architecture & Alarm Logic
The software executes a dynamic sequence: a sustained 1-second continuous alert state followed by a series of rapid, high-frequency alarm pulses:

```c
/* Infinite loop */
/* USER CODE BEGIN WHILE */
while (1)
{
    // Continuous Alert Phase
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);
    HAL_Delay(1000);
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET);
    HAL_Delay(1000);
    
    // Rapid Pulse Alarm Phase
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);
    HAL_Delay(150);
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET);
    HAL_Delay(150);
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);
    HAL_Delay(150);
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET);
    HAL_Delay(1000);
    
    /* USER CODE END WHILE */
    /* USER CODE BEGIN 3 */
}
