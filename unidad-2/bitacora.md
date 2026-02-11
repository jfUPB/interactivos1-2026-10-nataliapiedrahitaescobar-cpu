# Unidad 2

## Bitácora de proceso de aprendizaje

### Actividad 1 
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


class Pixel:
    def __init__(self,_x,_y,_interval):
        self.event_queue = []
        self.timers = []
        self.x = _x
        self.y = _y
        self.pixelState = 0
        self.myTimer = self.createTimer("Timeout",_interval)

        self.estado_actual = None
        self.transicion_a(self.estado_waitInON)

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

    def estado_waitInON(self, ev):
        if ev == "ENTRY":
            self.pixelState = 9
            display.set_pixel(self.x,self.y,self.pixelState)
            self.myTimer.start()
        elif ev == "Timeout":
            self.transicion_a(self.estado_waitInOFF)

    def estado_waitInOFF(self, ev):
        if ev == "ENTRY":
            self.pixelState = 0
            display.set_pixel(self.x,self.y,self.pixelState)
            self.myTimer.start()
        elif ev == "Timeout":
            self.transicion_a(self.estado_waitInON)

pixel1 = Pixel(0,0,1000)
pixel2 = Pixel(4,4,600)

while True:
    pixel1.update()
    pixel2.update()
    utime.sleep_ms(20)
```
### ¿Cuáles son los estados del programa?
Los estados son los que viven en los datos y pueden representar atributos booleanos o enums, variables que representan una condición o combinaciones de valores.
En este caso serían: estado_waitInOn y estado_waitInOff donde ambos representan una condición.
### ¿Cuáles son los eventos del programa?
Los eventos son los que disparan la ejecución de una lógica o sea sus cambios o acciones. (Entry, exit y timeout)
En este caso self.estado_actual("ENTRY") y self.estado_actual("EXIT") son eventos que suceden en el código.
### ¿Cuáles son las acciones del programa?
Las acciones son lo que el sistema hace internamente cuando ocurre un evento.
En este caso las acciones sería: Crear el timer t = Timer(self, event, duration), guardar el timer self.timers.append(t), Agregar un evento a la cola self.event_queue.append(ev), actualizar timers t.update() y sacar un evento de la cola ev = self.event_queue.pop(0).

## Bitácora de aplicación 



## Bitácora de reflexión


