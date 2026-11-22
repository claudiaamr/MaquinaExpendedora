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

## 2. Funcionamiento

El programa se organiza en tres modos principales.

### Modo Arranque

Al encender Arduino, la pantalla muestra `CARGANDO...` mientras el led rojo parpadea tres veces con intervalos de un segundo. Los parpadeos se controlan mediante TimerOne. Al terminar, la máquina pasa automáticamente al modo Servicio.

### Modo Servicio

El sensor HC-SR04 mide continuamente la distancia. Cuando detecta un cliente a menos de 100 cm, la pantalla muestra la temperatura y humedad durante 5 segundos.
Después aparece el catálogo de productos:

| Producto | Precio |
|---|---|
| Café Solo | 1,00 € |
| Café Cortado | 1,10 € |
| Café Doble | 1,25 € |
| Café Premium | 1,50 € |
| Chocolate | 2,00 € |

El joystick permite navegar entre los productos y su botón confirma la selección. La preparación dura entre 4 y 8 segundos, utilizando `random()`. Durante este tiempo, el led verde aumenta progresivamente su brillo mediante PWM, indicando el porcentaje de preparación. Al finalizar aparece `RETIRE BEBIDA` durante 3 segundos y la máquina vuelve a esperar un cliente. El botón externo permite cancelar la operación mediante una pulsación de entre 2 y 3 segundos.

### Modo Administración

Se accede manteniendo pulsado el botón externo durante al menos 5 segundos. En este modo ambos leds permanecen encendidos.
El menú permite:
- Consultar temperatura y humedad.
- Consultar la distancia del sensor ultrasónico.
- Consultar los segundos transcurridos desde el encendido.
- Modificar los precios en incrementos de 0,05 €.

La navegación se realiza mediante el joystick. Su botón confirma y el movimiento hacia la izquierda permite retroceder o descartar cambios. Para salir de Administración se mantiene pulsado el botón externo durante 5 segundos.
