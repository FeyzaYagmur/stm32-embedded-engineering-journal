# STM32F407 Embedded Systems Engineering Journal

This repository serves as my professional engineering journal, documenting my structured, theoretical and practical development journey into ARM Cortex-M microcontrollers using the STM32 platform (specifically targeting the STM32F407VG Discovery board).

The core focus of this study is mastering hardware-software interfacing, peripheral configurations, and bare-metal microcontroller programming concepts using the STM32Cube HAL framework.

---

## 📺 Project Hardware Demonstrations
All physical hardware verifications, laboratory measurements, and running application videos are stored in a dedicated Google Drive folder. You can access the entire video library directly through the link below:

🔗 [**Click Here to Access the STM32 Project Videos Folder**] https://drive.google.com/drive/folders/1QEF27VpM2UhSrB0GhmENcJqSesYxxM8_?usp=sharing

---

## 📚 Theoretical & Practical Curriculum Overview

### Phase 1: Microcontroller Basics, GPIO & Debugging
In this foundational phase, the study covers the architecture of general-purpose inputs and outputs (GPIO), configuring peripheral clocks, and understanding register-level behaviors. It also focuses on real-time debugging operations using Keil uVision and ST-LINK, utilizing tools like Serial Wire Viewer (SWV) to track internal variables, state changes, and hardware-software synchronization without halting the CPU.

### Phase 2: Actuators, Motor Drivers & Analog Interfacing
This phase bridges the gap between digital control and analog/physical hardware. It involves learning transistor switching principles, flyback diode protection, and interfacing with dedicated drivers like the **L293D H-Bridge** and **ULN2003AN Darlington pair** to control DC and stepper motors. On the analog side, it covers analog-to-digital conversion (ADC) parameters, sampling rates, and temperature measurement calculations using NTC thermistors, as well as digital-to-analog conversion (DAC) for waveform generation.

### Phase 3: Hardware Timers, PWM & Sensors
Focuses on the internal timing engines of the ARM Cortex-M core. This covers setting up hardware timer prescalers, counter registers, and period values to trigger precise internal interrupts (ISRs). It also includes implementing Pulse Width Modulation (PWM) for motor speed and LED brightness control, alongside reading precise echo timings from sensors like the HC-SR04 ultrasonic distance sensor using input capture modes.

### Phase 4: Serial Communication Protocols & DMA
Explores the communication pipelines of modern embedded systems. This includes establishing asynchronous pipelines via UART/USART (including wireless communication over HC-05 Bluetooth), plotting real-time telemetry data directly in MATLAB, and mastering synchronous serial buses like I2C and SPI. Additionally, Direct Memory Access (DMA) is explored to offload high-speed data streams directly to memory without loading the CPU.

### Phase 5: System Reliability & Power Management
Dedicated to building robust, fail-safe industrial embedded applications. This covers configuring Independent (IWDG) and Window (WWDG) Watchdog timers to reset the system during software locks. It also explores configuring the low-power modes (Sleep, Stop, Standby) of the STM32 Power Control (PWR) peripheral and managing Backup Registers to preserve critical state parameters during power failures.

---
*Developed with by Feyza Yağmur Arat. Electrical-Electronics Engineering Student at Mersin University.*
