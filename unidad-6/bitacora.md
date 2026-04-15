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

### **¿Qué variables mínimas necesitarías extraer para poder construir una visualización útil?**

Para hacer una visualización útil se necesita:

1. El tipo de sonido para decidir qué elemento visual mostrar.
2. Un timestamp para sincronizar el momento en que aparece la animación.
3. El delta, para controlar cuánto dura o cómo se comporta la animación.

### **¿Qué problema resuelve la cola de eventos?**

La cola de eventos soluciona el problema de la desincronización. Si los eventos se ejecutan apenas llegan, podrían atrasarse o adelantarse por cosas como la conexión o el rendimiento del computador, pero si se guardan en una cola, el sistema puede esperar al momento exacto (timestamp) para ejecutarlo.

Esto logra que:
- Las animaciones esten en ritmo.
- Todo ocurra en el momento adecuado.
- No se depende de la velocidad de llegada del mensaje.

### **¿Por qué esta capa no pertenece al bridge sino al lado que interpreta el evento?**

Porque el bridge solo se encarga de transportar los datos, no de decidir cuando se usan.

El momento en el que se ejecuta un evento depende de cómo se quiere interpretar y eso es lo que hace el fronted.

### **¿Qué papel cumple el Adapter en U4 y U5?**

En las unidades 4 y 5 el adapter se encarga de recibir los datos del micro:bit y traducirlos a un formato claro para el sistema (ASCII en la unidad 4 y binario en la unidad 5). En estos casos, su función principal era tomar datos crudos y convertirlos en información organizada (movimiento en x, y, botones) para que el resto del sistema lo pueda utilizar sin preocuparse por el formato.

### **¿Qué Adapter necesitas ahora para que los eventos de Strudel no entren “crudos” al sistema visual?**

Para esta actividad se necesita un StrudelAdapter que debe recibir los datos desde Strudel por medio del WebSocket, interpretar el JSON que llega, extraer los datos importantes que son el sonido, timestamp y duración que es el delta y convertirlos en un formato claro para el sistema.

## Bitácora de aplicación 

### **ACTIVIDAD 2**

1. Se creó un StrudelAdapter que se encarga de de recibir los eventos musicales de Strudel por medio de WebSocket, extraer los parámetros relevantes y transformarlos en un formato en el que el sistema pueda utilizarlos sin depender de la estructura original de Strudel.

```
const WebSocket = require("ws"); //Tipo de conexión que permite enviar datos, recibir datos en tiempo real sin recargar la página.
const BaseAdapter = require("./BaseAdapter");

class StrudelAdapter extends BaseAdapter{  //Es el adaptador que se conecta al servidor de Strudel.
    constructor({ url =  "ws://localhost:8080", verbose = false } = {}) {
        super();
        this.url = url; //Dirección donde el Strudel envía datos.
        this.ws = null; //Conexión WebSocket que inicialmente está desconectada.
        this.verbose = verbose;
    }

    async connect() { //Crea la conexión con Strudel y define los eventos para recibir datos.
        if (this.connected) return;

        this.ws = new WebSocket(this.url);

        //Cuando se conecta, se establece la conexión y se notifica al cliente que está conectado.
        this.ws.on("open", () => {
            this.connected = true;
            this.onConnected?.(`connected to ${this.url}`);
        })

        this.ws.on("message", (message) => { //Llega un mensaje del Strudel.
            try {
                const msg = JSON.parse(message); //Se parsea el mensaje, se espera que sea un JSON con una estructura específica.

                //Extraer datos importantes: El sonido y el delta (cambio en el sonido).
                const args = msg.args || [];

                const getArg = (key) => {
                    const i = args.indexOf(key);
                    return i >= 0 ? args[i + 1] : null;
                };

                const s = getArg("s");
                const delta = getArg("delta");

                //Normalizar evento, transforma el mensaje en un formato limpio para que el cliente pueda usarlo fácilmente.
                this.onData?.({ 
                    type: "strudel",
                    timestamp: msg.timestamp,
                    payload: {
                        s,
                        delta
                    }
                });
            } catch (e) { //Reporta errores de parseo o mensajes mal formados, si el mensaje no es un JSON válido o no tiene la estructura esperada, se captura el error y se muestra un mensaje de advertencia.
                if(this.verbose) console.log("Bad strudel message", message);
            }
        });
        this.ws.on("error", (err) => this._fail(err));
        this.ws.on("close", () => this._closed()); 
    }
    
    async disconnect(){ //Cierra la conexión con Strudel, si está abierta, y  notifica al cliente que se ha desconectado.
        if (!this.connected) return;
        this.connected = false;

        if(this.ws) {
            this.ws.close();
        }

        this.ws = null;
        this.onDisconnected?.("ws closed");
    }

    getConnectionDetail() {
        return `ws ${this.url}`;
    }

    _fail(err) {
        this.onError?.(String(err?.message || err));
        this.disconnect();
    }

    _closed() {
        if (!this.connected) return;
        this.connected = false;
        this.ws = null;
        this.onDisconnected?.("ws closed (event)");
    }
}

module.exports = StrudelAdapter;
```




## Bitácora de reflexión
