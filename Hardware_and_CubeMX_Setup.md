# Hardware Setup & STM32CubeMX Configuration

This guide details the physical wiring and the microcontroller peripheral configuration required to build the Redundant Brake-by-Wire (BBW) Controller.

## 1. Hardware Components
* **Microcontroller:** STM32L476RG Nucleo-64 Board
* **Sensors:** 2x Linear Rotary Potentiometers (10kΩ recommended)
* **Actuators/Indicators:** 
  * 1x Blue LED (Simulated Brake Actuator)
  * 1x Red LED (External Fault Indicator)
  * 1x Green LED (On-board LD2, Normal Operation Indicator)
* **Miscellaneous:** Breadboard, jumper wires, and current-limiting resistors (e.g., 220Ω - 330Ω) for the LEDs.

## 2. Breadboard Wiring & Pin Mapping

| Component | STM32 Pin | Peripheral / Function | Breadboard Connection |
| :--- | :--- | :--- | :--- |
| **Potentiometer A** | `PA0` | `ADC1_IN5` | Outer pins to 3.3V and GND*. Middle wiper pin to PA0. |
| **Potentiometer B** | `PA1` | `ADC2_IN6` | Outer pins to 3.3V and GND*. Middle wiper pin to PA1. |
| **Blue LED (Actuator)** | `PA8` | `TIM1_CH1` (PWM) | Anode (long leg) to PA8. Cathode (short leg) to GND via resistor. |
| **Red LED (Fault)** | `PB2` | `GPIO_Output` | Anode (long leg) to PB2. Cathode (short leg) to GND via resistor. |
| **Green LED (Status)**| `PA5` | `GPIO_Output` | Internal to Nucleo board (LD2). No wiring needed. |

> **Wiring Note on Sensor Polarity:** Ensure that when turning the potentiometers "up" (e.g., simulating a pedal press), the mechanical wiper moves toward the 3.3V rail. If the Blue LED dims when you rotate the knob up, swap the 3.3V and GND wires on the outer pins of the potentiometers to reverse the electrical gradient.

---

## 3. STM32CubeMX Configuration

### System Core
* **SYS:** Debug set to `Serial Wire`. Timebase Source set to `TIM6` (essential for FreeRTOS).
* **RCC:** High-Speed Clock (HSE) / Low-Speed Clock (LSE) as required by your board, usually bypassed or internal MSI.

### Analog (Sensors)
* **ADC1:** 
  * `IN5` Enabled. 
  * Resolution: `12-bit`. 
  * Number of Conversions: `1`.
* **ADC2:** 
  * `IN6` Enabled. 
  * Resolution: `12-bit`. 
  * Number of Conversions: `1`.

### Timers (Actuator)
* **TIM1:** 
  * Clock Source: `Internal Clock`.
  * Channel 1: `PWM Generation CH1`.
  * Prescaler: `79` (For an 80MHz clock, this gives a 1MHz timer clock).
  * Counter Period (AutoReload Register): `1000`. (This provides a 0-1000 PWM duty cycle range).

### Middleware (FreeRTOS)
* **API:** `CMSIS_V2`
* **Tasks Created:**
  * `defaultTask`: Normal priority.
  * `Task_Acquisition`: Real-time priority (Suspended in final monolithic architecture to prevent HAL locks).
  * `Task_SafetyVoting`: High priority. Handles HAL ADC polling, safety math, and PWM generation.
  * `Task_Actuator`: Normal priority.

### GPIO Config
* `PA5`: Output Level LOW, Mode: Output Push Pull, Pull-up/Pull-down: No Pull.
* `PB2`: Output Level LOW, Mode: Output Push Pull, Pull-up/Pull-down: No Pull.