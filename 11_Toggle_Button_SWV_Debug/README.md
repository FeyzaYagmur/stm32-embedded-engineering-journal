# Step 11: Toggle Button State Logic and SWV Debugging via ITM

In this project, I designed a latching toggle switch mechanism using an external button. Additionally, I implemented real-time in-circuit diagnostics using the Instrumentation Trace Macrocell (ITM) to redirect the standard `printf` output library straight to the Serial Wire Viewer (SWV) debug console.

The objective is to master latching toggle state-machines, edge-blocking state loops (wait-on-release), and custom ITM-based hardware printf redirection.

## Hardware Configuration
* **Input Pin:** `PA0` (GPIOA, Pin 0) connected to an external push-button in a pull-up circuit configuration.
* **Output Pin:** `PD12` (GPIOD, Pin 12) connected to the on-board Green LED.
* **Working Principle:** The software evaluates the transition of `PA0` (Active-Low). When activated, it toggles the state of `PD12` and uses a blocking `while` loop to wait for the button release before incrementing the diagnostic counter and transmitting log data via the serial debug pipeline.

## Code Architecture & SWV Redirection
By overriding the low-level `_write` syscall to feed the ITM Send Character register, standard output functions like `printf` and `fflush` write directly to the debugger terminal:

```c
/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */
#include "stdio.h"

// Redirect printf output to the SWV ITM Data Console
int _write(int file, char *ptr, int len)
{
    int i = 0;
    for (i = 0; i < len; i++)
    {
        ITM_SendChar((*ptr++));
    }
    return len;
}
/* USER CODE END 0 */

/* USER CODE BEGIN PV */
uint8_t buttonState = 1;
uint8_t buttonFlag = 0;
uint16_t buttonCounter = 0;
/* USER CODE END PV */

/* Infinite loop */
/* USER CODE BEGIN WHILE */
while (1)
{
    buttonState = HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_0);
    HAL_Delay(200); // Initial debounce window
    
    if (!buttonState)
    {
        // Toggle State Logic
        if (buttonFlag == 0)
        {
            HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12, GPIO_PIN_SET);
            buttonFlag = 1;
        }
        else
        {
            HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12, GPIO_PIN_RESET);
            buttonFlag = 0;
        }
        
        buttonCounter++;
        
        // Output debugging telemetry data
        printf("Counter Value: %d \r\n", buttonCounter);
        fflush(stdout);
        
        // Blocking wait-on-release loop to prevent counter runaway
        while(!HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_0));
    }
    
    /* USER CODE END WHILE */
}
