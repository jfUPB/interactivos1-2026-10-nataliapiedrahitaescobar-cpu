# Unidad 3

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 

## Actividad 1
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
