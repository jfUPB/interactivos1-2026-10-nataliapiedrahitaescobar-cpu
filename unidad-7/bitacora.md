# Unidad 7

## Bitácora de proceso de aprendizaje

## **Actividad 1**

### **¿Qué diferencia hay entre un evento musical y un mensaje de control?**

La diferencia es que los eventos musicales ocurren en un momento específico en el tiempo (timestamp), son eventos instantáneos y disparan una acción actual. 
Mientras que un mensaje de control (Open Stage Control) no depende de un momento puntual, representan un valor contínuo y modifican el estado del sistema.

Evento musical: Sucede y se va.

Evento de control: Cambia algo y se queda.

### **¿Qué quiere decir que un parámetro del sistema sea persistente?**

Un parámetro pesistente es un valor que se mantiene en el tiempo, no desaparece después de usarse y sigue afectando el sistema hasta que alguien lo cambie.

### **¿Qué partes del sistema de la unidad 6 permanecen intactas en este nuevo caso?**

Se mantiene intacto el pipeline de Strudel (eventQueue → Process Strudel → ActiveAnimations), el sistema de animación (ActiveAnimations,drawRunning y dibujarElemento), la lógica del tiempo (Timestamp, sincronización y la duración de animaciones) y el bridge.

El sistema ahora separa correctamente dos responsabilidades:

**Eventos:** Son los que generan acciones en el tiempo.

**Parámetros:** Son los que definen los estados del sistema.

Esto permite que múltiples fuentes de datos convivan sin interferir entre sí, manteniendo una arquitectura modular, desacoplada y extendible.


## Bitácora de aplicación 


## Bitácora de reflexión
