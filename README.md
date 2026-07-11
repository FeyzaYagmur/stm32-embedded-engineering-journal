# STM32 Embedded Software Engineering Journal

This repository serves as my professional engineering journal, documenting a structured, step-by-step development journey into ARM Cortex-M microcontrollers using the STM32 platform. 

The core focus is on mastering peripheral configurations, hardware-software interfacing, and protocol implementations through practical, verified code and real-time physical testing.

---

## Technical Roadmap & Completed Modules

### Phase 1: GPIO & Fundamental Peripherals
* **Step 1: On-Board LED Drive Implementation (Blink)**
  * Core configuration of GPIO outputs, CPU loop cycles, and basic timing/delay abstractions.
* **Step 2: Optimized LED Scanning via Bit-Shifting Operations**
  * Advanced register addressing using bitwise operations `(1 << i)` to reduce software instructions and optimize loop boundaries.
* **Step 3: External Interrupts (EXTI) & Hardware Interfacing**
  * Asynchronous event handling, push-button interface configuration, and software-based debouncing methodologies.

### Phase 2: Analog-to-Digital Interfacing & Advanced Timers
* **Step 4: Hardware Timers & Precise Interrupt Service Routines (ISR)**
  * Configuring internal clock prescalers and counter registers for time-critical embedded tasks.
* **Step 5: Pulse Width Modulation (PWM) & Signal Generation**
  * Duty cycle calculations and capture/compare register manipulation for dimming and actuator controls.
* **Step 6: Analog Interfacing (ADC & DAC)**
  * Voltage sampling, data conversion sequences, resolution tuning, and generating analog signals from digital inputs.

### Phase 3: Communication Protocols & Data Transfer
* **Step 7: Universal Asynchronous Receiver-Transmitter (UART/USART)**
  * Establishing serial data transmission pipelines between the microcontroller and external terminals.
* **Step 8: Synchronous Serial Protocols (I2C & SPI)**
  * Master-Slave architecture implementation, bus timing analysis, and reading/writing to external sensors and displays.
* **Step 9: Direct Memory Access (DMA) Integration**
  * Offloading peripheral data transfer operations from the CPU to maximize execution efficiency.

---
*Note: Each directory contains fully isolated, compilation-ready C source modules, clock configurations, and corresponding physical hardware verification clips.*
