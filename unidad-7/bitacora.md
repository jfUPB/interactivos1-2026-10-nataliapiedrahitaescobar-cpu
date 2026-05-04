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

## **Paso 1**

### **Si Open Stage Control fuera “el dispositivo” de esta unidad, ¿Cuál sería su protocolo?**

El protocolo de Open Stage Control sería OSC, que es la forma en la que el programa  manda la información.

Manda mensajes que tienen: un nombre tipo (/rgb_1) y unos valores como ([255, 120, 30]) que es como decir: "Quiero cambiar esto... a estos valores..."

### **¿Qué parte de ese protocolo te interesa conservar y cuál te gustaría normalizar?**

Lo más importante que se debe conservar es saber qué cosa se está cambiando, como el color y con qué valores como los números RGB.

Lo que se puede normalizar es el /rgb_1 y el args porque son comando que al ser muy específicos del Open Stage Control, pueden confundir el código.

Ejemplo: 

En vez de usar esto:
```
address: "/rgb_1",
args: [255, 120, 30]
``` 

Se cambiaría a esto:
```
color = [255, 120, 30]
``` 
 Open Stage Control mana la información de una forma específica, pero no es necesario que el sistema la utlice tal cual, ya que lo mejor es traducir esos mensajes a un lenguaje más simple y claro , para que el código sea más fácil de entender y usar.

## **Paso 2**

### **¿Por qué no conviene procesar un mensaje OSC igual que un mensaje de Strudel?**

Porque ambos programas hacen cosas diferentes. Strudel manda eventos que pasan en un momento exacto mientras que Open Stage Control manda cambios que se quedan. Si se tratara un mensaje tipo OSC como si fuera Strudel el cambio solo pasaría un instante, no se mantendría en el tiempo y se perdería la idea del control contínuo.

### **¿Qué variables del sistema deberían vivir como estado persistente y no como evento efímero?**

Deberían ser persistentes todas las variables que definen cómo se ven o se comportan las visuales en general, no algo que suceda una sola vez. Como el color, el tamaño base, la velocidad y la transparencia o el brillo.

## **Paso 3**

### **¿Qué componentes de la arquitectura necesitas conservar obligatoriamente?**

Para que Open Stage Control no dañe lo que ya funciona, se deben mantener en el código los adapters que hacen que todo lo que entre al sistema se traduzca correctamente antes de usarse, el bridge que sigue siendo el intermediario que envía los datos sin lógica, las capas de estado donde se guarda lo que esté pasando en el sistema y el sketch o la capa de renderizado que es donde se dibujan las visuales.

### **¿Qué nuevas estructuras de estado necesitas introducir para soportar control paramétrico?**

Se debe introducir una nueva estructura que guarde los controles que vinen del OSC como por ejemplo:
```
this.params = {
  color: [255, 255, 255],
  size: 1,
  speed: 1
};
```

Estos son los datos del OSC que se encargan de guardar los valores actuales del sistema, se actualizan cuando llega OSC y se utilizan para el render de los dibujos.

## Bitácora de aplicación 
### **Actividad 2**
1. Se creó un nuevo adapter (OSCAdapter) que tiene de base al BaseAdapter que servirá para recibir los mensajes OSC crudos (Desde UDP/bridgeServer) y normalizarlos.

El OSCAdapter solo emite datos limpios al bridge mas no interpreta la lógica visual y no modifica directamente el sketch.


### **Código OSCAdapter**
```
//Importaciones
const BaseAdapter = require("./BaseAdapter"); //Clase vase obligatoria del sistema
const osc = require("osc"); //Librería que abre un puerto UDP para recibir los mensajes de OSC

//No hay conexión todavía, solo configuración
class OpenStageControlAdapter extends BaseAdapter {
    constructor(port = 9000) { //Es el puerto donde OpenStageControl enviará los datos
        super();//Activa la herencia del BaseAdapter

        this.port = port;
        this.udpPort = null; //Donde se guarda la la conexión OSC cuando se abre
    }

    async connect() { //Se abre el puerto OSC, se empieza a escuhar la red
        try {
            //Crear el servidor UDP
            this.udpPort = new osc.UDPPort({
                localAddress: "0.0.0.0", //Escucha cualquier dispositivo en red
                localPort: this.port //El puerto 9000
            });

            this.udpPort.on("ready", () => { //Cuando el puerto ya está abierto
                this.connected = true; //Se marca el adapter como conectado
                console.log("OSC Adapter conectado en puerto", this.port);

                if (this.onConnected) this.onConnected(); //Notifica al sistema bridge/UI
            });

            this.udpPort.on("message", (msg) => { //Donde llegan todos los mensajes del OSC
                try {
                 const normalized = { //Normalización del sistema, convierte los mensajes crudos en un formato estandar
                    type: "osc", //Identifica el origen
                    payload: {
                        address: msg.address, //Especifica qué control fue usado
                        args: msg.args || [] //Valores del control
                    }
                 };

                 //Enviar al sistema 
                 if (this.onData) this.onData(normalized); //El adapter entrega un mensaje al bridge
                } catch (err) { //Si algo falla se imprime un error y se notifica en el sistema
                    if (this.onError) this.onError(err);
                }
            });

            this.udpPort.on("error", (err) => {
                console.error("OSC error:", err);
                if (this.onError) this.onError(err);
            });

            this.udpPort.open(); //Activa todo el listener OSC
        }catch (err) {
            if (this.onError) this.onError
        }
    }

    async disconnect() {
        if(this.udpPort) {
            this.udpPort.close();
            this.udpPort = null;
        }

        this.connected = false; //Cierra conexión OSC, se marca desconectado

        if(this.onDisconnected) this.onDisconnected(); //Se notifica al sistema
    }

    getConnectionDetail() {
        //Devuelve información del estado como el tipo de conexión, el puerto usado y si está conectado o no.
        return {
            type: "osc",
            port: this.port,
            connected: this.connected
        };
    }
}

module.exports = OpenStageControlAdapter;
  
```

## Bitácora de reflexión
