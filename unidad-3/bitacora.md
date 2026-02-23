# Unidad 3

## Bitácora de proceso de aprendizaje
## Actividad 3 
**¿Cómo funciona el código?**
**1.Index.HTML:** 
Es el archivo que abre el navegador, carga el p5.js, carga el código y es el contenedor donde se dibuja el proyecto. O sea, es el archivo que muestra lo que es necesario agregar en el código para que este funcione correctamente en p5.js y pueda ser pasado a Micro:bit.

En este caso en la línea de código 15 se indica que el código debe tener un archivo fsm.js en el cuerpo del código para que pueda funcionar.
```
<script src="fsm.js"></script>
```
Lo mismo sucede en la línea de código 17 pero esta vez se pide es el sketch.js.
```
 <script src="sketch.js"></script>
```
 
## Bitácora de aplicación 

## Actividad 1 --- Código arreglado
```py
from microbit import *
import utime

class Timer:
    def __init__(self, owner, event_to_post, duration):
        self.owner = owner
        self.event = event_to_post
        self.duration = duration

        self.start_time = 0
        self.active = False

    def start(self, new_duration=None):
        if new_duration is not None:
            self.duration = new_duration
        self.start_time = utime.ticks_ms()
        self.active = True

    def stop(self):
        self.active = False

    def update(self):
        if self.active:
            if utime.ticks_diff(utime.ticks_ms(), self.start_time) >= self.duration:
                self.active = False
                self.owner.post_event(self.event)


class Semaforo:
    def __init__(self,_x,_y,_timeInRed,_timeInGreen,_timeInYellow):
        self.event_queue = []
        self.timers = []
        self.x = _x
        self.y = _y
        self.timeInRed = _timeInRed
        self.timeInGreen = _timeInGreen
        self.timeInYellow = _timeInYellow
        self.myTimer = self.createTimer("Timeout",self.timeInRed)
        self.led_on = False
        self.estado_actual = None
        self.transicion_a(self.estado_waitInRed)

    def createTimer(self,event,duration):
        t = Timer(self, event, duration)
        self.timers.append(t)
        return t

    def post_event(self, ev):
        self.event_queue.append(ev)

    def update(self):
        # 1. Actualizar todos los timers internos automáticamente
        for t in self.timers:
            t.update()

        # 2. Procesar la cola de eventos resultante
        while len(self.event_queue) > 0:
            ev = self.event_queue.pop(0)
            if self.estado_actual:
                self.estado_actual(ev)

    def transicion_a(self, nuevo_estado):
        if self.estado_actual: self.estado_actual("EXIT")
        self.estado_actual = nuevo_estado
        self.estado_actual("ENTRY")

    def clear(self):
        display.set_pixel(self.x,self.y,0)
        display.set_pixel(self.x,self.y+1,0)
        display.set_pixel(self.x,self.y+2,0)

    def estado_waitInRed(self, ev):
        if ev == "ENTRY":
            self.clear()
            display.set_pixel(self.x,self.y,9)
            self.myTimer.start(self.timeInRed)
        if ev == "Timeout":
            display.set_pixel(self.x,self.y,0)
            self.transicion_a(self.estado_waitInGreen)
        if ev == "A":
            pass
        if ev == "B":
            self.transicion_a(self.estado_nocturno)

    def estado_waitInGreen(self, ev):
        if ev == "ENTRY":
            self.clear()
            display.set_pixel(self.x,self.y+2,9)
            self.myTimer.start(self.timeInGreen)

        if ev == "Timeout":
            display.set_pixel(self.x,self.y+2,0)
            self.transicion_a(self.estado_waitInYellow)

        if ev == "A":
            display.set_pixel(self.x,self.y+2,0)
            self.transicion_a(self.estado_waitInYellow)

        if ev == "B":
            self.transicion_a(self.estado_nocturno)

    def estado_waitInYellow(self, ev):
        if ev == "ENTRY":
            self.clear()
            display.set_pixel(self.x,self.y+1,9)
            self.myTimer.start(self.timeInYellow)
        if ev == "Timeout": 
            display.set_pixel(self.x,self.y+1,0)
            self.transicion_a(self.estado_waitInRed)
            
        if ev == "A":
            pass

        if ev == "B":
            self.transicion_a(self.estado_nocturno)
            
    def estado_nocturno(self, ev):
        if ev == "Entry":
            if self.led_on:
                display.set_pixel(self.x,self.y+1,0)
                self.led_on = False
        else:
            if self.led_on:
                display.set_pixel(self.x,self.y+9,0)
                self.led_on = True
        self.myTimer.start(300)
        if ev == "TimeOut":
            self.transicion_a(self.estado_nocturno)
        if  ev == "A":
            display.set_pixel(self.x,self.y+1,0)
            self.transicion_a(self.estado_waitInRed)
           
semaforo1 = Semaforo(0,0,2000,1000,500)

while True:
    if button_a.was_pressed():
        semaforo1.post_event("A")
    if button_b.was_pressed():
        semaforo1.post_event("B")

    semaforo1.update()
    utime.sleep_ms(20)
```
## Actividad 2 --- Código arreglado.
```py
from microbit import *
from fsm import FSMTask, ENTRY, EXIT
from utils import FILL
import utime
import music

class Temporizador(FSMTask):
    def __init__(self):
        super().__init__()
        self.event_queue = []
        self.timers = []
        self.counter = 20
        self.paused = False
        self.sequence = ""
        self.myTimer = self.add_timer("Timeout",1000)
        self.estado_actual = None
        self.transition_to(self.estado_config)


    def estado_config(self, ev):
        if ev == ENTRY:
            self.counter = 20
            display.show(FILL[self.counter])
            self.myTimer.start()
        if ev == "A":
            if self.counter > 15:
                self.counter -= 1
            display.show(FILL[self.counter])
        if ev == "B":
            if self.counter < 25:
                self.counter += 1
            display.show(FILL[self.counter])
        if ev == "S":
            self.transition_to(self.estado_armed)

    def estado_armed(self, ev):
        if ev == ENTRY:
            self.myTimer.start()
        if ev == "Timeout":
            if self.counter > 0:
                self.counter -= 1
                display.show(FILL[self.counter])
                if self.counter == 0:
                    self.transition_to(self.estado_timeout)
                else:
                    self.myTimer.start()
        if ev == "A":
            if not self.paused:
                self.myTimer.stop()
                self.paused = True
            else:
                self.myTimer.start()
                self.paused = False

    def estado_timeout(self, ev):
        if ev == ENTRY:
            display.show(Image.SKULL)
            music.play(music.FUNERAL)
        if ev == "A":
            music.stop()
            self.transition_to(self.estado_config)

temporizador = Temporizador()

while True:

    if button_a.was_pressed():
        temporizador.sequence += "A"
        temporizador.post_event("A")
    if button_b.was_pressed():
        temporizador.sequence += "B"
        temporizador.post_event("B")
    if accelerometer.was_gesture("shake"):
        temporizador.post_event("S")

    temporizador.update()
    if temporizador.sequence.endswith("ABA"):
        temporizador.transition_to(temporizador.estado_config)
        temporizador.sequence = ""
    utime.sleep_ms(20)
```
## Bitácora de reflexión
## Actividad 1
**Paso a Paso: Creando el botón A**
El botón A se agregó para podeer crear el modo peatonal donde al apretar el botón A el semáforo se pone en rojo, pero antes de debe pasar por el amarillo.
Creación del botón: 
```py
if button_a.was_pressed():
    semaforo1.post_event("A")
```
Luego se modificó el estado verde para que cuando se aprete el botón A, esta información se reciba y realice la transición al botón amarillo: (Verde --- Amarillo --- Rojo)
```py
if ev == "A":
    self.transicion_a(self.estado_waitInYellow)
```
Para que múltiples presiones del botón no afecten el proceso del modo peatonal, se coloca la instrucción pass en los estados rojo y amarillo.

**Paso a Paso: Creando el botón B**
El botón B se creó para activar es estado de modo nocturno donde la luz amarilla del semáforo parpadea constantemente.
Creación del botón: 
``` py
if button_b.was_pressed():
    semaforo1.post_event("B")
```
Se crea una transición de la luces del semáforo al estado nocturno por medio de:
```py
if ev == "B":
    self.transicion_a(self.estado_nocturno)
```
Se crea el estado nocturno y dentro de este se coloca el parpadeo constante de la luz amarilla usando el temporizador de myTimer que está creado y se crea una variable booleana: El bool permite mirar si el LED amarillo está prendido o apagado.
```py
self.led_on
```
Cuando se acaba el temporizador se genera el evento Timeout, lo que genera que sistema entre al modo nocturno: Este proceso sucede constantemente el efecto del parpadeo.
```py
if ev == "Timeout":
    self.transicion_a(self.estado_nocturno)
```
Para salir del estado nocturno se preciona el botón A para que el semáforo regrese a su estadano normal:
```py
if ev == "A":
    self.transicion_a(self.estado_waitInRed)
```
## Actividad 2
1. El self.paused se crea porque el código inicial no se puede pausar solo y con el self.paused se le puede recordar ese comando al código para que lo haga.
2. El self.sequence va guardando los botones que se presionen y lo guarda como una secuencia (ABA).
3. Lograr que el botón A pause o continúé el comando realizado de esta manera:
   ```py
   if ev == "A":
    if not self.paused: //Si el timer no está pausado
        self.myTimer.stop() //Se detiene el timer
        self.paused = True//Se marca que ahora está pausado
    else:
        self.myTimer.start()
        self.paused = False
   ```
4. Iniciar el timer cuando ya está pausado:
     ```py
     self.myTimer.start() //Se reinicia el Timer
     self.paused = False //Se marca que ya no está pausado el Timer
     ```
5. Guardar la secuencia (ABA)
   En el while True se agrega:
   ```py
   temporizador.sequence += "A"
   temporizador.post_event("A")
   ```
   // Este código agrega una A a lo que ya estaba guardado y lo mismo se hace con el botón B
6. Detectar (ABA)
   Se agrega:
   ```py
   if temporizador.sequence.endswith("ABA"):
    temporizador.transition_to(temporizador.estado_config) //Si sí se presionaron las teclas se regresa al modo de configuración 
    temporizador.sequence = "" //Se borra la secuencia para empezar otra vez
   ```
   Esta parte del código se encarga de verificar que las últimas teclas presionadas si generan la secuencia (ABA).




