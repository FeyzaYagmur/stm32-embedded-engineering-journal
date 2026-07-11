# Step 5: External Knight Rider LED Scanner via Bit-Shifting

In this project, I integrated advanced algorithmic control with external hardware circuitry. I built an external "Knight Rider" scanning light effect using 3 distinct LEDs (Red, Blue, and Yellow) connected via an external breadboard to pins `PA0`, `PA1`, and `PA2` on the STM32 microcontroller.

The goal is to master concurrent sequential multi-device driving on external ports using optimized register math instead of linear hardcoded routines.

## Hardware Configuration
* **Microcontroller Pins:** `PA0` (Red), `PA1` (Blue), `PA2` (Yellow) on `GPIOA`.
* **Circuit Components:** 3 External LEDs, Current-Limiting Resistors, Jumper Wires, and an External Breadboard.
* **Working Principle:** The software sequentially shifts logic high states across pins 0 to 2, outputting 3.3V to illuminate the target external LED while maintaining safe current drops via individual series resistors.

## Algorithmic Implementation & Code Explanation
By applying bitwise left-shifts `(1 << i)` dynamic addressing within constrained loop boundaries, the firmware seamlessly scans back and forth through the external GPIO pins:

```c
/* Infinite loop */
/* USER CODE BEGIN WHILE */
while (1)
{
    for (int i = 0; i <= 2; i++) {
        HAL_GPIO_WritePin(GPIOA, (uint16_t)(1 << i), GPIO_PIN_SET);
        HAL_Delay(150);
        HAL_GPIO_WritePin(GPIOA, (uint16_t)(1 << i), GPIO_PIN_RESET);
        HAL_Delay(150);
    }
    for (int i = 1; i >= 1; i--) {
        HAL_GPIO_WritePin(GPIOA, (uint16_t)(1 << i), GPIO_PIN_SET);
        HAL_Delay(150);
        HAL_GPIO_WritePin(GPIOA, (uint16_t)(1 << i), GPIO_PIN_RESET);
        HAL_Delay(150);
    }
    
    /* USER CODE END WHILE */
    /* USER CODE BEGIN 3 */
}
