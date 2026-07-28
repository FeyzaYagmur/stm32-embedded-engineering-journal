# Seven Segment Display Decimal Counter (Hardware Emulation)

This project implements a 0-to-9 numerical decimal counter using a single-digit 7-Segment Display interfaced with an STM32 Nucleo platform via the Wokwi online simulation environment. Due to physical display module unavailability, the embedded hardware configuration and logic mapping are simulated using dynamic pin array driving and lookup table (LUT) architectures.

## ⚙️ Hardware & Configuration
- **MCU / Platform:** STM32L031K6 (ARM Cortex-M0+) / Wokwi Simulation Layer
- **Display Component:** Single-Digit 7-Segment Display (Emulated)
- **Active Pins:** `D2`, `D3`, `D4`, `D5`, `D6`, `D7`, `D8` (Segment Pins A through G)
- **Method:** 2D Lookup Array Mapping with Iterative Pin-State Execution

## 🔍 Key Concepts Covered
- **Hardware Simulation & Emulation:** Verifying peripheral logic, multiplexing patterns, and segment driving through virtual prototyping tools prior to physical PCB integration.
- **Segment Look-Up Tables (LUT):** Designing a 2D matrix structure to map raw numeric values ($0-9$) directly into corresponding physical segment states ($a, b, c, d, e, f, g$).
- **Array-Based Output Bus Driving:** Programmatically looping through initialized digital output pins inside modular functions (`showDigit()`) to maintain scalable, clean firmware architecture.

## 💻 Complete Simulation Source Code (`main.cpp` / Wokwi Engine)

Below is the complete hardware control logic and the segment mapping matrix used in the simulation environment:

```cpp
#include <Arduino.h>

const int segmentPins[] = {D2, D3, D4, D5, D6, D7, D8};

// 7-segment logic map for digits 0 to 9 (Segments A-G)
const int digits[10][7] = {
  {1, 1, 1, 1, 1, 1, 0}, // 0
  {0, 1, 1, 0, 0, 0, 0}, // 1
  {1, 1, 0, 1, 1, 0, 1}, // 2
  {1, 1, 1, 1, 0, 0, 1}, // 3
  {0, 1, 1, 0, 0, 1, 1}, // 4
  {1, 0, 1, 1, 0, 1, 1}, // 5
  {1, 0, 1, 1, 1, 1, 1}, // 6
  {1, 1, 1, 0, 0, 0, 0}, // 7
  {1, 1, 1, 1, 1, 1, 1}, // 8
  {1, 1, 1, 1, 0, 1, 1}  // 9
};

void showDigit(int num) {
  for (int seg = 0; seg < 7; seg++) {
    digitalWrite(segmentPins[seg], digits[num][seg]);
  }
}

void setup() {
  for (int i = 0; i < 7; i++) {
    pinMode(segmentPins[i], OUTPUT);
    digitalWrite(segmentPins[i], LOW);
  }
}

void loop() {
  for (int num = 0; num < 10; num++) {
    showDigit(num);
    delay(1000);
  }
}
