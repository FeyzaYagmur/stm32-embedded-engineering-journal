🔴 Step 1: On-Board Green LED Blinking (Blink)

In this project, I configured the on-board Green LED (`PD12`) on my STM32 Discovery board to blink at 1-second intervals.

## 🔌 Hardware Explanation
* **Active Pin:** `PD12` (On-board Green LED)
* **Working Principle:** When the microcontroller output is set to `SET` (1), it supplies 3.3V to the LED. Setting it to `RESET` (0) cuts the power.

## 💻 Code & Physical Board Demonstration
Below is the video showing both the STM32CubeIDE code compilation, loading process, and the physical green LED blinking in real-time:

<video src="blink_demo.mp4" controls width="100%"></video>

## 🧠 Engineering Insights
* **What happens if we remove the last `HAL_Delay(1000);`?**
  Without a delay after turning the LED off, the loop restarts in microseconds. The LED turns back on so quickly that the human eye cannot perceive the off-state. The LED appears constantly on.
