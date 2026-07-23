# Toggle SWV Button LED (Real-Time Debugging & State Toggle)

This project integrates real-time hardware tracing with toggle-switch state logic. It demonstrates how to toggle an external LED using a single push-button while simultaneously outputting state-change telemetry to the Keil SWV (Serial Wire Viewer) trace console via ITM data channels.

## ⚙️ Hardware & Configuration
- **MCU:** STM32F407VGT6 (ARM Cortex-M4)
- **External Components:** 1x Push-Button, 1x Red LED, 2x Resistors
- **Active Pins:** `PD2` (Digital Input from Button), `PD3` (Digital Output to LED)
- **Debugging Bus:** SWD with `SWO` pin active for ITM telemetry redirection
- **Method:** Edge-Triggered State Inversion with Real-Time SWV Telemetry Logging

##
