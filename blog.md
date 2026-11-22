# Práctica 3 — Máquina expendedora con Arduino UNO

**Asignatura:** Sistemas Empotrados y de Tiempo Real  
**Autor:** Claudia Morales Rojas

## 1. Objetivo y montaje

El objetivo de esta práctica es programar una máquina expendedora utilizando Arduino UNO. La máquina detecta la presencia de un cliente, muestra información ambiental, permite seleccionar una bebida y simula su preparación. También dispone de un modo de administración.

### Esquema del circuito

El circuito se ha diseñado utilizando Fritzing.

![Esquema del circuito](esquematico.png)

He utilizado una **resistencia pull-up externa** para el botón externo y `INPUT_PULLUP` para el joystick. Así evito lecturas incorrectas y consigo que ambos botones estén en HIGH sin pulsar y en LOW al pulsarlos. En el joystick aprovecho la resistencia interna de Arduino para no añadir más componentes.

También he colocado **resistencias en serie con los LEDs** para limitar la corriente y evitar que se dañen. En el **LCD**, he utilizado una resistencia para su correcto funcionamiento, evitando una conexión directa sin limitación de corriente.
