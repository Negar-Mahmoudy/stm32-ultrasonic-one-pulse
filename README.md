# stm32-ultrasonic-one-pulse
This repository contains two versions of an STM32 project for measuring distance using an ultrasonic sensor. Both versions use PWM and input capture for measurement but differ in their operating mode and power efficiency.

---

## **Version 1: Continuous PWM Measurement**

### **Description**
In this version, the distance is measured continuously using a PWM signal. A pulse with a minimum high time of 10µs is sent to the sensor's **Trigger** pin. The sensor outputs a response on the **Echo** pin, which is captured using a timer in PWM input mode to calculate the distance.

- Continuous measurement
- Suitable for real-time distance monitoring
- Higher power consumption due to continuous PWM generation
- 
- **Timer 3 (PWM output)**: Generates a 12µs high pulse every 50ms on the Trigger pin.
- **Timer 4 (Input Capture)**: Measures the duration of the high pulse on the Echo pin.
- **Distance Calculation**: `distance = duty_cycle * 340 * 100 / 2`
- **UART Output**: Prints measured distance every second.

---

## **Version 2: One-Pulse Mode with Key Trigger**

### **Description**
This version optimizes power consumption by measuring distance only when a button is pressed. The timer operates in **One-Pulse Mode**, acting as a slave that triggers measurement upon receiving a pulse from the external pin (ETR).

- Event-based measurement
- Low power consumption
- One-Pulse mode prevents multiple triggers due to button noise

- **Timer 2 (One-Pulse Mode, PWM output)**: Generates a pulse when the button is pressed. ARR = 50ms, OCR ~12µs.
- **Trigger Source**: Button input via ETR pin (external trigger for the slave timer).
- **Timer 4 (Input Capture)**: Measures the high time on Echo pin.
- **Distance Calculation**: Same as Version 1.
- **UART Output**: Prints distance immediately after measurement.

---

## **Key Differences Between Versions**

| Feature | Version 1 | Version 2 |
|---------|-----------|-----------|
| Measurement Mode | Continuous PWM | One-Pulse Mode with Key Trigger |
| Power Consumption | Higher | Lower |
| Timer Usage | Timer3 PWM + Timer4 Input Capture | Timer2 One-Pulse + Timer4 Input Capture |
| Trigger Source | Continuous PWM | External (button) |
| Response to Trigger Noise |    | Handled by One-Pulse mode |
| Use Case | Continuous monitoring | Event-based / battery-efficient monitoring |

---

## **Common Code Highlights**
```c
// Input Capture Callback
void HAL_TIM_IC_CaptureCallback(TIM_HandleTypeDef *htim) {
    if(htim->Channel == HAL_TIM_ACTIVE_CHANNEL_1) {
        uint32_t icValue1 = HAL_TIM_ReadCapturedValue(htim, TIM_CHANNEL_1);
        uint32_t icValue2 = HAL_TIM_ReadCapturedValue(htim, TIM_CHANNEL_2);
        uint32_t diff = icValue1 - icValue2;
        float duty = diff / 1000000.0;
        float dist = duty * 340 * 100 / 2;
        printf("distance = %0.3f cm\r\n", dist);
    }
}
````
![UltraSonic Datasheet](https://github.com/Negar-Mahmoudy/stm32-ultrasonic-one-pulse/blob/main/images/ultrasomicsensor.png?raw=true)
