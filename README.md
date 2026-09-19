# 🚀 STM32F407 Embedded Systems Engineering Journal

This repository serves as a personal engineering journal documenting practical embedded firmware implementations developed on the **STM32F407VG Discovery Board** using **STM32CubeIDE** and the **STM32Cube HAL** library.

It encompasses **38 modular laboratory modules**, bridging low-level GPIO operations with high-speed serial communication protocols, sensor interfacing, and advanced low-power configurations.

---

## 📺 Hardware Demonstrations & Video Archive

Physical hardware execution recordings, oscilloscope/multimeter verification captures, and circuit setups for all modules are documented in the Google Drive archive:  
🔗 **[Click Here to Access STM32 Project Videos Archive](https://drive.google.com/drive/folders/1QEF27VpM2UhSrB0GhmENcJqSesYxxM8_?usp=sharing)**

---

## 🛠️ Completed Laboratory Modules

### 🔹 GPIO & Foundational Peripherals
* **GPIO Operations:** Digital output actuation (LED, Buzzer) and digital input handling (Push Buttons).
* **State Control & Counters:** Active-Low and Active-High input decoding, software debouncing filters, and counter mechanisms.
* **Live Tracing & Debugging:** Real-time variable inspection and program state profiling via STM32CubeIDE SWV / ITM Live Data Trace.

### 🔹 Analog Interfacing (ADC & DAC)
* **Analog-to-Digital Conversion (ADC):** Analog sensor polling with 10kΩ Potentiometers and NTC Thermistors; linear voltage scaling and temperature derivation.
* **Digital-to-Analog Conversion (DAC):** Analog voltage synthesis and waveform generation.

### 🔹 Timers & Asynchronous Interrupts
* **External Interrupts (EXTI):** Hardware-level external event handling and asynchronous edge-triggered inputs.
* **Hardware Timers:** Periodic non-blocking time-base generation via Timer Update Interrupts (TIM IT).
* **PWM Signal Generation:** Hardware PWM duty-cycle modulation for dynamic LED dimming and motor speed throttling.
* **SysTick:** Dedicated system tick interrupt configuration and custom timekeeping routines.

### 🔹 Motor Drivers, Displays & Actuators
* **Motor Driving:** Discrete transistor and relay-based DC motor switching; bidirectional PWM speed control using the L293D H-Bridge driver.
* **Stepper Motor Actuation:** Step-angle driving and sequencing using the ULN2003AN Darlington array.
* **Seven-Segment Displays:** Direct multiplexing and driving of dual seven-segment display units.

### 🔹 Sensors & Wireless Telemetry
* **HC-SR04 Ultrasonic Sensor:** Pulse-width timing capture for distance measurement and dynamic proximity threshold gating.
* **HC-05 Bluetooth Module:** Wireless UART serial telemetry streaming and remote command-based hardware control.

### 🔹 Industrial Serial Communication Protocols
* **UART / USART:** Asynchronous serial data streaming, framing, and interrupt-driven (RX Interrupt) command processing.
* **I2C Protocol:**
  * SSD1306 OLED display driver integration.
  * Modular driver implementation for 16x2 HD44780 Character LCD via PCF8574 I2C backpack and real-time ADC voltmeter dashboard.
* **SPI Protocol:** Full-Duplex synchronous data transmission/reception and loopback verification.

### 🔹 Power Control & Low-Power Modes (PWR)
* **Sleep Mode (WFI):** Processor core clock gating using Wait-For-Interrupt to minimize dynamic power dissipation.
* **Stop Mode (EXTI):** Halting high-speed clocks and core execution with asynchronous external interrupt wake-up.
* **Standby Mode & Backup Registers:** Deepest power-down mode and data preservation across power loss using Backup Data Registers (BKP).

---

## 👩‍💻 Developer Profile

👩‍💻 **Developer:** Feyza Yağmur Arat  
🎓 **Department:** Mersin University — Electrical and Electronics Engineering  
🛠️ **Development Environment:** STM32CubeIDE | STM32Cube HAL  
🎯 **Hardware:** STM32F407VG Discovery Board  

---

## Status
*Completed & Fully Documented Archive*
