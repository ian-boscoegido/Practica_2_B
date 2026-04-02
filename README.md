# Practice 2 - Timer Interrupt (ESP32)

## 📌 Overview

This project demonstrates the use of **hardware timer interrupts on the ESP32** using the Arduino framework in PlatformIO.  
A hardware timer is configured to generate periodic interrupts, allowing the program to execute tasks at precise time intervals without blocking the main loop.

---

## ⚙️ Features

- Hardware timer configuration on ESP32  
- Periodic interrupt generation (1 second interval)  
- Use of **Interrupt Service Routine (ISR)**  
- Safe shared variable handling using critical sections  
- Real-time monitoring via serial output  

---

## 🧠 How It Works

1. A hardware timer is initialized with a prescaler to achieve microsecond precision.
2. The timer is configured to trigger an interrupt every **1 second**.
3. Each interrupt calls an ISR (`onTimer()`), which increments a counter.
4. The main loop checks for new interrupts and updates a total counter.
5. The result is printed to the serial monitor.

---

## 💻 Code

```cpp
#include <Arduino.h>

volatile int interruptCounter = 0;
int totalInterruptCounter = 0;

hw_timer_t * timer = NULL;
portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;

void IRAM_ATTR onTimer() {
  portENTER_CRITICAL_ISR(&timerMux);
  interruptCounter++;
  portEXIT_CRITICAL_ISR(&timerMux);
}

void setup() {

  Serial.begin(115200);

  timer = timerBegin(0, 80, true);
  timerAttachInterrupt(timer, &onTimer, true);
  timerAlarmWrite(timer, 1000000, true);
  timerAlarmEnable(timer);

}

void loop() {

  if (interruptCounter > 0) {

    portENTER_CRITICAL(&timerMux);
    interruptCounter--;
    portEXIT_CRITICAL(&timerMux);

    totalInterruptCounter++;

    Serial.print("An interrupt has occurred. Total number: ");
    Serial.println(totalInterruptCounter);

  }

}
