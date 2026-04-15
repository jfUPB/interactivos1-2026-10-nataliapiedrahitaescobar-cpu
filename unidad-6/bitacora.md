# Unidad 6

## Bitácora de proceso de aprendizaje
### **¿Cuál es la diferencia entre recibir un mensaje y ejecutarlo?**

Recibir un mensaje es cuando la información llega al sistema y ejecutarlo es cuando el sistema hace algo con la información recibida del mensaje.

### **¿Por qué un sistema audiovisual puede necesitar timestamp además de los datos del evento?**

Porque el timestamp ayuda a que se sepa cuándo sucede un evento.

El timestamp sirve para que los eventos planeados en el código sucedan en el momento correcto, dan sincronización al sonido a una animación.

### **¿Qué aspectos de la arquitectura de las unidades 4 y 5 permanecen intactos aunque ahora la fuente de datos ya no sea hardware?**

Aunque en esta unidad se utiliza Strudel y no un microbit, la estructura del sistema se mantiene, o sea que, la estructura del sistema no cambia, sino que solo cambia el origen y el tipo de datos que entran al sistema.

Todavía hay un Adapter que se encarga de recibir y traducir los datos a un formato que el sistema entienda. También se mantiene el bridgeServer que sirve trabajando como intermediario para enviar datos al fronted, el bridgeClient sigue recibiendo los datos y enviándolos a la lógica de la aplicación, y se mantiene la separación entre recepción de datos, updateLogic y draw.

### **Si Strudel fuera “el dispositivo” de esta unidad, ¿Cuál sería su protocolo?**

El protocolo de Strudel sería el formato que envía los mensajes por WebSocket, usando datos tipo JSON.

Cada mensaje representaría un evento musical y tendría información como:

- Qué sonido se reproduce.
- El timestamp, o sea cuando debe ocurrir el sonido.
- El delta o cuánto dura el ritmo.

El protocolo define cómo están organizados los datos que Strudel envía para que otro sistema los pueda leer de forma correcta.

## **¿Qué variables mínimas necesitarías extraer para poder construir una visualización útil?**

Para hacer una visualización útil se necesita:

1. El tipo de sonido para decidir qué elemento visual mostrar.
2. Un timestamp para sincronizar el momento en que aparece la animación.
3. El delta, para controlar cuánto dura o cómo se comporta la animación.





## Bitácora de aplicación 


## Bitácora de reflexión
