# 🟢 Step 2: Knight Rider (Kara Şimşek) LED Scanner with Bit-Shifting

In this project, I developed a classic "Knight Rider" (Kara Şimşek) LED scanning effect using the 4 on-board LEDs (Green `PD12`, Orange `PD13`, Red `PD14`, Blue `PD15`) on the STM32 Discovery board. 

Instead of writing repetitive lines of code, I utilized a highly optimized algorithm using `for` loops and **Bit-Shifting** operations.

## 🔌 Hardware Configuration
* **LED Pins:** `PD12` (Green), `PD13` (Orange), `PD14` (Red), `PD15` (Blue).
* **Working Principle:** The microcontroller sequentially energizes these pins to create a fluid, continuous scanning light effect across the board.

## 💻 Algorithmic Optimization & Code Explanation
Instead of calling `HAL_GPIO_WritePin` manually for each LED, I used mathematical bit-shifting:

```c
for (int i = 12; i <= 15; i++) {
    HAL_GPIO_WritePin(GPIOD, (uint16_t)(1 << i), GPIO_PIN_SET);
    HAL_Delay(50);
    HAL_GPIO_WritePin(GPIOD, (uint16_t)(1 << i), GPIO_PIN_RESET);
    HAL_Delay(50);
}
for (int i = 14; i >= 13; i--) {
    HAL_GPIO_WritePin(GPIOD, (uint16_t)(1 << i), GPIO_PIN_SET);
    HAL_Delay(50);
    HAL_GPIO_WritePin(GPIOD, (uint16_t)(1 << i), GPIO_PIN_RESET);
    HAL_Delay(50);
}
