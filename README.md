# Redundant Brake-by-Wire (BBW) Controller

An ISO 26262-inspired, FreeRTOS-based embedded system that models a redundant Brake-by-Wire (BBW) controller using an STM32 microcontroller.

## Extended Overview & System Description

In modern automotive engineering, Drive-by-Wire (DbW) systems replace traditional mechanical control linkages—such as the steel pushrod connecting a brake pedal to a hydraulic master cylinder—with purely electronic sensors and electromechanical actuators. While this allows for advanced features like autonomous emergency braking and regenerative blending, it introduces a critical vulnerability: a single point of failure in an electronic sensor could result in a total loss of braking capability.

To mitigate this and comply with functional safety standards (such as ISO 26262), engineers must implement strict hardware and software redundancy.

This project is a functional, small-scale implementation of that exact safety paradigm. Built on an **STM32L476RG Nucleo-64** development board running a deterministic **FreeRTOS** environment, the system reads a single mechanical brake pedal input using two entirely independent analog sensors (dual potentiometers).

At the core of the software architecture is a high-priority RTOS safety-voting loop. This loop continuously polls both independent ADC channels and calculates the absolute deviation between them.

* **Normal Operation:** If the sensor readings match within an acceptable engineering tolerance, the system calculates the average demand, scales the 12-bit ADC value, and drives a 10-bit PWM signal to a physical actuator (visualized in this project by a proportionally dimming/brightening Blue LED). A Green status LED indicates system health.

* **Fail-Safe Trigger:** If the sensors diverge—simulating a short circuit, physical damage, or carbon track wear on one of the potentiometers—the voting algorithm instantly intercepts the command. It overrides the PWM output to 0% (cutting brake torque to avoid unpredictable actuation) and latches a digital Red LED fault flag to indicate a critical hardware failure.

By integrating mechanical deadband filtering to eliminate resting electrical noise and utilizing hardware-level timers, this project demonstrates a highly responsive, fault-tolerant embedded system design tailored for mission-critical automotive applications.

## Features

* **Hardware Redundancy:** Utilizes dual independent ADCs (`ADC1` and `ADC2`) to sample redundant potentiometers simultaneously.

* **Safety-Voting Logic:** Calculates the absolute deviation between sensors (`abs(SensorA - SensorB) > TOLERANCE`).

* **Mechanical Deadband Filtering:** Ignores micro-voltage electrical noise when the simulated pedal is at 0%.

* **Deterministic Execution:** Built on FreeRTOS (CMSIS v2) to guarantee cyclic execution of the safety constraints.

* **PWM Actuation:** Dynamically scales the 12-bit ADC input (0-4095) into a 10-bit PWM output (0-1000) to control a physical actuator.

* **Binary Fault Latching:** Instantly kills actuator output and latches a digital Red LED indicator if a sensor mismatch is detected.

## Software Architecture

The system relies on a monolithic safety loop inside `Task_SafetyVoting` (High Priority).

1. **Acquisition:** Commands hardware-level polling via `HAL_ADC_Start` and `HAL_ADC_PollForConversion`.

2. **Deadband Check:** Evaluates if inputs are below the mechanical noise floor.

3. **Redundancy Validation:** Checks if the deviation exceeds the `ADC_TOLERANCE` threshold (configurable, currently set to accommodate manual breadboard testing).

4. **Actuation:** If safe, calculates the average of both sensors and updates the `TIM1_CH1` PWM compare register directly.

## Getting Started

1. Clone this repository.

2. Open the `.ioc` file in **STM32CubeIDE**.

3. Wire the breadboard according to the `Hardware_and_CubeMX_Setup.md` guide.

4. Build the project and flash it to your STM32L476RG Nucleo-64 board.

5. Rotate both potentiometers together to control the Blue LED brightness. Desync the potentiometers to trigger the Red LED safe state.

## Technologies Used

* **C (gnu11)**

* **FreeRTOS (CMSIS v2)**

* **STM32 HAL (Hardware Abstraction Layer)**

* **STM32CubeMX & STM32CubeIDE**

* **ARM Cortex-M4 Architecture**
