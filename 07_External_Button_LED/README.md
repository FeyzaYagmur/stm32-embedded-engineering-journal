# Step 7: External Button Interfacing and Input/Output Logic

In this project, I implemented basic input/output hardware interaction. I interfaced an external momentary push-button as an input device on pin `PC6` and paired it with an external blue LED configured as an output device on pin `PB0`.

The objective is to master runtime input polling (`HAL_GPIO_ReadPin`), conditional flow control, and simple peripheral synchronization.

## Hardware Configuration
* **Input Pin:** `PC6` (GPIOC, Pin 6) connected to an external push-button.
* **Output Pin:** `PB0` (GPIOB, Pin 0) connected to an external Blue LED.
* **Working Principle:** The input pin `PC6` monitors the state of the button. In this configuration, when the button is pressed, it pulls the pin state to logic HIGH (`SET` or 3.3V). The software polls this pin continuously; if a HIGH state is detected, it triggers `PB0` to output 3.3V, lighting up the blue LED. Releasing the button breaks the state, turning the LED off.

## Code Architecture & Polling Logic
The execution logic relies on an infinite while loop that continuously queries the state of the input data register (IDR) via the HAL framework:

```c
/* Infinite loop */
/* USER CODE BEGIN WHILE */
while (1)
{
    if (HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_6) == GPIO_PIN_SET)
    {
        HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, GPIO_PIN_SET);
    }
    else
    {
        HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, GPIO_PIN_RESET);
    }
    
    /* USER CODE END WHILE */
    /* USER CODE BEGIN 3 */
}
