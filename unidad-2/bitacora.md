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
En este caso self.estado_actual("ENTRY") es el evento que sucede en el código.
### ¿Cuáles son las acciones del programa?
Las acciones son lo que el sistema hace internamente cuando ocurre un evento.
En este caso las acciones sería: Crear el timer t = Timer(self, event, duration), guardar el timer self.timers.append(t), Agregar un evento a la cola self.event_queue.append(ev), actualizar timers t.update() y sacar un evento de la cola ev = self.event_queue.pop(0).


## Actividad 3
## ¿Cómo es posible estructurar una aplicación usando una máquina de estados para poder atender varios eventos de manera concurrente?
Se puede estructurar teniendo en cuenta los posibles estados del sistema y los eventos que pueden ocurrir. Cuando sucede un evento, se guarda en una lista de eventos y el sistema los procesa según el estado actual realizando acciones o cambios de estado según se mande la instrucción. 

## ¿Cómo haces para probar que el programa está correcto?
Se puede probar que un programa está correcto, mirando que el sistema si pase por los estados planeados en el código según los eventos que ocurran. Si ante cada evento se realiza la indicación o transición adecuada y el comportamiento es el mismo que se diseño en la máquina de estados, se puede decir que el programa funciona bien.
## Bitácora de aplicación 

## Actividad 2, punto 1 (Código con botón agregado)
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

    def estado_waitInYellow(self, ev):
        if ev == "ENTRY":
            self.clear()
            display.set_pixel(self.x,self.y+1,9)
            self.myTimer.start(self.timeInYellow)
        if ev == "Timeout":
            display.set_pixel(self.x,self.y+1,0)
            self.transicion_a(self.estado_waitInRed)

semaforo1 = Semaforo(0,0,2000,1000,500)
while True:
    if button_a.was_pressed():
        semaforo1.post_event("A")
    
    semaforo1.update()
    utime.sleep_ms((20))
```
## Actividad 2, punto 2 (Máquina de estados)
@startuml
[*] --> WaitInRed

WaitInRed --> WaitInGreen : TimeOut
WaitInGreen --> WaitInYellow : TimeOut
WaitInGreen --> WaitInYellow : A 
WaitInYellow --> WaitInRed : TimeOut
@enduml

<img width="234" height="396" alt="image" src="https://github.com/user-attachments/assets/44e605ed-3648-4f21-b107-163c85e17041" />

## Actividad 4
## 1. Máquina de estado
@startuml
[*] --> Configurando (Aumenta o disminuye el tiempo con A y B)

Configurando --> Configurando : A
Configurando --> Configurando : B
Configurando --> Contando : S (Shake, arma el temporizador)

Contando --> Contando : TimeOut (Cada TimeOut apaga un 1 pixel)
Contando --> Terminado : TimeOut [Tiempo == 0] (Cuando el tiempo queda en 0 pasa a terminado)

Terminado --> Configurando : A
(Se muestra la calavera, Botón A reinicia y se vuelve a configuración)
@enduml

<img width="303" height="396" alt="image" src="https://github.com/user-attachments/assets/fa22e613-57ac-4f0c-9466-5d05c094783c" />

## 2. Código micro:bit
```py
//Primero se hacen las importaciones necesarias que piede el ejercicio: utime que mide el tiempo en milisegundos y music
from microbit import *
import utime
import music
//Segundo se crean las imágenes del display
def make_fill_images(on='9', off='0'):
    imgs = []
    for n in range(26):
        rows = []
        k = 0
        for y in range(5):
            row = []
            for x in range(5):
                row.append(on if k < n else off)
                k += 1
            rows.append(''.join(row))
        imgs.append(Image(':'.join(rows)))
    return imgs
   
FILL = make_fill_images()
# Para mostrar usas display.show(FILL[n]) donde n será
# un valor de 0 a 25
//Tercero se crea el temporizador
class Timer:
    def __init__(self, owner, event_to_post, duration):
        self.owner = owner
        self.event = event_to_post
        self.duration = duration
        self.start_time = 0
        self.active = False
        self.tiempo = 20

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
//Cuarto se coloca el comportamiento del programa, se crea la variable inicial tiempo que se temporiza en 20 segundos y se crean los estados (Configurando, contando y terminando)
class Task:
    def __init__(self):
        self.event_queue = []
        self.timers = []
        # Personalizas el nombre del evento y la duración
        self.myTimer = self.createTimer("Timeout",1000)

        self.tiempo = 20
        self.estado_actual = None
        self.transicion_a(self.estado_configurando)

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


    def estado_configurando(self, ev):
        if ev == "ENTRY":
            display.show(FILL[self.tiempo])
        if ev == "A" and self.tiempo < 25:
            self.tiempo += 1
            display.show(FILL[self.tiempo])
        if ev == "B"   and self.tiempo < 15:
            self.tiempo -= 1
            display.show(FILL[self.tiempo])
            pass
        
            
        if  ev == "Timeout":
            if ev == "S":
                self.transicion_a(self.estado_contando)
            pass

    def estado_contando(self, ev):
        if ev == "ENTRY":
            self.myTimer.start()
            pass
        if ev == "Timeout":
            self.tiempo -= 1
            display.show(FILL[self.tiempo])
            if self.tiempo == 0:
                self.transicion_a(self.estado_terminado)
            pass

    def estado_terminado(self,ev):
        if ev == "ENTRY":
            display.show(Image.SKULL)
            music.play(music.BA_DING)
            
        if ev == "A":
            self.tiempo = 20
            self.transicion_a(self.estado_configurando)
//Quinto la máquina de estados decide qué hacer
task = Task()
while True:
    # Aquí generas los eventos de los botones y el gesto
    if button_a.was_pressed():
        task.post_event("A")
    if button_b.was_pressed():
        task.post_event("B")
    if accelerometer.was_gesture("shake"):
        task.post_event("S")

    task.update()
    utime.sleep_ms(20)
            
```
## Bitácora de reflexión

## Actividad 3
## ¿Hay algo que aún no comprendes completamente?
- Todavía no comprendo muy bien cómo encontrar en un código  los eventos, acciones y estados.
- No sé muy bien cómo empezar a editar un código, especialmente saber en dóndo debo editarlo precisamente.








