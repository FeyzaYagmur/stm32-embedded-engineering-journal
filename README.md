# 🛠️ STM32 Embedded Systems Developer Journal

Welcome to my personal learning journal! In this repository, I document my journey into the world of embedded systems using the STM32 Discovery board. My goal is to understand the core physics, hardware architecture, and code logic of every single component from scratch.

## 📚 Table of Contents (Step-by-Step Projects)

### 🔴 Phase 1: Basic Input/Output (GPIO) Fundamentals
* **[Step 1: On-Board Green LED Blinking (Blink)](./01_Dahili_LED_Blink)**
  * *Concepts Learned:* `HAL_GPIO_WritePin`, `HAL_Delay`, CPU execution speed vs. human eye perception limits.
* **[Step 2: LED Control via Toggle Logic](./02_LED_Toggle_Mantigi)**
  * *Concepts Learned:* Code optimization and minimizing instructions using `HAL_GPIO_TogglePin`.
* **[Step 3: Momentary Pushbutton LED Control](./03_Anlik_Buton_LED)**
  * *Concepts Learned:* `HAL_GPIO_ReadPin`, understanding current flow paths in a Pull-Up resistor circuit configuration.

### 🟡 Phase 2: Advanced Peripheral Applications & Debugging
* **[Step 4: Latching (Toggle) Button & SWV Debugging](./04_Kalici_Buton_SWV_Debug)**
  * *Concepts Learned:* Redirecting `printf` via the `_write` function to the SWV ITM Data Console, mechanical switch debouncing, and locking the program flow using `while` loops.
* **[Step 5: Knight Rider (Kara Şimşek) Speed Adjustment via Button](./05_Kara_Simsek_Hiz_Ayari)**
  * *Concepts Learned:* Variable-based `HAL_Delay` modification and building state-dependent algorithms.

---
*Each project folder contains the respective CubeIDE source files (`main.c`), detailed breadboard circuit schematics, and my engineering notes explaining the inner workings.*
