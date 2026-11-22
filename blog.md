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

## 3. Diseño e implementación de la solución

### 3.1. Organización del programa

He dividido el código en siete archivos `.ino` para separar las diferentes funcionalidades de la máquina expendedora:

- **MaquinaExpendedora.ino:** contiene la configuración de los componentes, la inicialización de ArduinoThread, el bucle principal y el control de los estados.
- **arranque.ino:** gestiona los tres parpadeos iniciales mediante TimerOne.
- **presencia.ino:** realiza las mediciones de distancia con el sensor HC-SR04.
- **monitorizacion.ino:** muestra la temperatura y la humedad durante cinco segundos.
- **navegacion.ino:** gestiona la navegación por el catálogo mediante el joystick, la selección de productos y la simulación de la preparación.
- **administracion.ino:** controla el menú de administración, la consulta de sensores y la modificación de precios.
- **reinicio.ino:** gestiona las pulsaciones del botón externo para cancelar operaciones o entrar y salir de Administración.

Esta división permite mantener separadas las distintas funcionalidades, facilitando la comprensión del código.

El funcionamiento se controla mediante la variable `estadoMaquina`, que identifica el estado actual. En `tareaControlEstados()` se comprueba su valor para ejecutar la función correspondiente.

### 3.2. Gestión de tareas con ArduinoThread

He utilizado **ArduinoThread** para organizar el programa en dos tareas periódicas:

- **tareaBoton (10 ms):** ejecuta `procesarBotonExterno()`, que comprueba la duración de las pulsaciones.
- **tareaEstados (1 ms):** ejecuta `tareaControlEstados()`, encargada de actualizar el funcionamiento de la máquina según su estado.

Ambas tareas se añaden a un `ThreadController`, que se ejecuta desde `loop()`.

He separado estas tareas porque el botón externo debe poder atenderse mientras la máquina realiza otras operaciones, como mostrar productos o preparar una bebida.

La medición ultrasónica se mantiene fuera de ArduinoThread y se comprueba directamente desde `loop()`, ya que necesita detectar rápidamente los cambios de la señal ECHO.

### 3.3. Interrupciones hardware

Para detectar las pulsaciones del botón externo he utilizado la interrupción **INT0**, correspondiente al pin digital 2 de Arduino UNO.

La interrupción se configura con `FALLING`, porque el pulsador tiene una resistencia pull-up externa y su señal pasa de HIGH a LOW al presionarlo.

La ISR únicamente modifica la variable `volatile` `eventoBotonPendiente`. El procesamiento de la pulsación se realiza posteriormente en `procesarBotonExterno()`.

He utilizado esta solución para detectar el inicio de las pulsaciones sin tener que comprobar continuamente el botón y evitar operaciones largas dentro de la ISR.

Mediante `millis()` se diferencia entre una pulsación de 2-3 segundos, que cancela la operación, y otra de al menos 5 segundos, que permite entrar o salir de Administración.

### 3.4. Control de tiempos y medición ultrasónica

He utilizado **`millis()` y `micros()`** para controlar los diferentes intervalos de tiempo sin recurrir a `delay()`.

`millis()` se utiliza para controlar la monitorización ambiental, la navegación con el joystick, la preparación de las bebidas y las pulsaciones del botón.

En cambio, para el sensor HC-SR04 utilizo `micros()`, porque necesito generar un pulso TRIGGER de al menos 10 microsegundos y medir la duración de ECHO.

He dividido la medición en cuatro fases: activar TRIGGER, finalizar el pulso, esperar a que ECHO pase a HIGH y esperar a que vuelva a LOW.

Esta organización evita utilizar `pulseIn()`, que podría mantener el programa esperando una respuesta del sensor.

También he establecido un tiempo máximo de espera de 25 ms para cancelar una medición si no se recibe la señal esperada y permitir un nuevo intento.

La distancia se calcula mediante:

**Distancia (cm) = tiempo de ECHO (µs) × 0,0343 / 2**

Se divide entre dos porque el sonido realiza un recorrido de ida y vuelta.

### 3.5. TimerOne y PWM

He utilizado **TimerOne** para gestionar los parpadeos del LED1 durante el arranque. El temporizador genera interrupciones cada segundo, permitiendo controlar el LED sin detener el programa.

Durante la preparación de bebidas utilizo **PWM** para representar visualmente el progreso mediante el LED2.

La duración de cada preparación se genera con `random()` entre 4 y 8 segundos. A partir del tiempo transcurrido calculo un valor de PWM entre 0 y 255, que aplico mediante `analogWrite()`.

He conectado el LED2 al pin 3 porque permite utilizar PWM sin interferir con TimerOne.

### 3.6. Watchdog

He incorporado un **Watchdog de 8 segundos** como mecanismo de seguridad.

Se configura mediante `wdt_enable(WDTO_8S)` y se refresca en cada iteración de `loop()` utilizando `wdt_reset()`.

Si el programa se bloquea y deja de ejecutar esta instrucción durante aproximadamente 8 segundos, el microcontrolador se reinicia.

He elegido este intervalo para disponer de un margen de seguridad antes de provocar el reinicio.
