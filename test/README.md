# Práctica 2 - Interrupciones (Apartado B)

## Descripción

En esta práctica se implementa una **interrupción mediante un temporizador hardware en el ESP32**.  
El temporizador genera una interrupción de forma periódica (cada 1 segundo). Cada vez que ocurre la interrupción se incrementa un contador y se muestra el número total de interrupciones a través del **monitor serie**.

---

## Hardware utilizado

- ESP32
- Comunicación serie (115200 baudios)

No se requiere hardware externo ya que se utilizan **temporizadores internos del microcontrolador**.

---

## Conceptos utilizados

### Temporizadores (Timers)

Un temporizador es un **contador interno del microcontrolador** que genera una interrupción cuando alcanza un valor determinado.

El ESP32 dispone de varios **timers hardware de 64 bits**, que permiten generar interrupciones periódicas sin bloquear la ejecución del programa principal.

---

## Código

```cpp
#include <Arduino.h>

volatile int interruptCounter;
int totalInterruptCounter;

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