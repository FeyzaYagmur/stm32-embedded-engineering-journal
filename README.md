# STM32F407 Discovery Development Journal

This repository serves as my professional engineering journal, documenting a structured, step-by-step development journey into ARM Cortex-M microcontrollers using the STM32 platform (specifically targetting the STM32F407VG Discovery). 

The core focus is on mastering peripheral configurations, hardware-software interfacing, and protocol implementations through **HAL (Hardware Abstraction Layer)** libraries, verified with real-time physical testing and simulation.

---

## 🗺️ Technical Roadmap & Completed Modules

### Phase 1: GPIO, Debugging & Hardware Basics
* **Step 1: On-Board LED Drive Implementation (Blink)**
    * Core configuration of GPIO outputs, CPU loop cycles, and basic timing/delay abstractions using HAL libraries.
* **Step 2: External Interrupts (EXTI) & Push-Button Interfacing**
    * Asynchronous event handling, EXTI line configuration, and software-based debouncing methodologies.
* **Step 3: Seven-Segment Displays & Keil IDE Debugging**
    * Utilizing Keil uVision debugger (breakpoints, watch windows) and driving multi-digit Seven-Segment Displays.

### Phase 2: Actuators, Drivers & Analog Interfacing
* **Step 4: Transistor Switching & DC Motor Control**
    * Understanding transistor saturation, flyback diode protection, and basic DC motor driving.
* **Step 5: Dedicated Motor Drivers (L293D & ULN2003AN)**
    * Interfacing with **L293D H-Bridge** for DC motor bi-directional control and **ULN2003AN Darlington pair** for Step Motor sequence generation.
* **Step 6: Analog-to-Digital Interfacing (ADC & NTC Thermistors)**
    * Voltage sampling, resolution tuning, and converting raw ADC data to physical temperature values using **NTC Thermistors**.
* **Step 7: Digital-to-Analog Conversion (DAC)**
    * Generating analog signals (sine, triangle, sawtooth waves) from digital inputs.

### Phase 3: Advanced Timers, Signal Generation & Sensors
* **Step 8: Hardware Timers & Precise Interrupts (ISR)**
    * Configuring internal clock prescalers and counter registers for time-critical embedded tasks.
* **Step 9: Pulse Width Modulation (PWM)**
    * Duty cycle calculations and capture/compare register manipulation for dimming and motor speed control.
* **Step 10: Ultrasonic Distance Sensing (HC-SR04)**
    * Using Input Capture mode of Timers to measure echo pulse-width and calculate real-time distance.

### Phase 4: Communication Protocols & Data Visualization
* **Step 11: UART/USART Serial Communications & Bluetooth (HC-05)**
    * Establishing serial data pipelines between the MCU and PC. Wireless control implementation via HC-05 Bluetooth module.
* **Step 12: Real-Time Data Visualization (MATLAB Integration)**
    * Streaming real-time sensor data over UART to MATLAB to generate real-time graphical plots.
* **Step 13: Synchronous Serial Protocols (I2C & SPI)**
    * Master-Slave architecture, bus timing analysis, and reading/writing to external EEPROMs or sensors.
* **Step 14: Direct Memory Access (DMA) Integration**
    * Offloading high-speed peripheral data transfer operations (ADC to RAM / Memory to UART) from the CPU.

### Phase 5: System Reliability, Power & Low-Level Control
* **Step 15: Watchdog Timers (IWDG & WWDG)**
    * Implementing Independent and Window Watchdog timers to prevent system lockups in mission-critical loops.
* **Step 16: Power Control (PWR) & Backup Registers**
    * Configuring low-power modes (Sleep, Stop, Standby) and utilizing Backup Data Registers to preserve critical state data during power loss.

---

## 📚 Academic & Reference Materials
* **Datasheet & Reference Manual Analysis:** A dedicated guide on how to read and navigate the STM32F407VG datasheet and RM0090 Reference Manual.
* **University Coursework & Exam Prep:** Selected midterm/final practice questions, laboratory assignments, and conceptual Q&A sections.
* **Further Reading & Resources:** Curated list of recommended textbooks, online courses, and technical websites for advanced ARM development.
