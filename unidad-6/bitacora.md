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
    constructor({port = 8080, verbose = false } = {}) {
        super();
        this.port = port; //Puerto en el que se conecta al servidor de Strudel.
        this.ws = null; //Conexión WebSocket que inicialmente está desconectada.
        this.verbose = verbose;
    }

    async connect() { //Crea la conexión con Strudel y define los eventos para recibir datos.
        if (this.connected) return;

        this.wss = new WebSocket.Server({ port: this.port})

        this.connected = true;
        this.onConnected?.(`Strudel WS server on ${this.port}`);

        this.wss.on("connection", (ws) => {
            console.log("Strudel conectado al Bridge");

            ws.on("message", (message) => {
                try{
                    const msg = JSON.parse(message);

                    const args = msg.args || [];

                    const getArg = (key) => {
                        const i = args.indexOf(key);
                        return i >= 0 ? args[i + 1] : null;
                    };

                    const s = getArg("s");
                    const delta = getArg("delta");

                    this.onData?.({
                        type: "strudel",
                        timestamp: msg.timestamp,
                        payload: {
                            s,
                            delta
                        }
                    });

                } catch (e) {
                    if (this.verbose) console.log("Bad strudel message", message);
                }
            });
        });
       
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

}
    
module.exports = StrudelAdapter;
```
2. Conectar el StrudelAdpater al bridgeServer para que el programa pueda soportar una nueva fuente de datos (Strudel), que es el que envía eventos musicales.

Se importó el nuevo adapter que se conecta al WebSocket por el puerto 8080, que es donde Strudel emite los eventos.
```

//   Uso:
//     node bridgeServer.js
//     node bridgeServer.js --device microbit
//     node bridgeServer.js --device sim --wsPort 8081 --hz 30
//     node bridgeServer.js --device microbit --wsPort 8081 --serialPort COM5 --baud 115200
//     node bridgeServer.js --device microbit-v2 
//     node bridgeServer.js --device microbitBinary --serialPort COM --wsPort 8081
//     node bridgeServer.js --device strudel --wsPort 8082
//  Activar el Verbose: node bridgeServer.js --device microbitBinary --verbose
//   WS contract:
//    * bridge To client:
//        {type:"status", state:"ready|connected|disconnected|error", detail:"..."}
//        {type:"microbit", x:int, y:int, btnA:bool, btnB:bool, t:ms}
//    * client To bridge:
//        {cmd:"connect"} | {cmd:"disconnect"}
//        {cmd:"setSimHz", hz:30}
//        {cmd:"setLed", x:2, y:3, value:9}


const { WebSocketServer } = require("ws"); //Comunicación con el navegador.
const { SerialPort } = require("serialport"); //Conexión con el microbit a través del puerto serial.
const SimAdapter = require("./adapters/SimAdapter"); //Simulador.
const MicrobitASCIIAdapter = require("./adapters/MicrobitASCIIAdapter"); //El hardware real.
// const MicrobitBinaryAdapter = require("./adapters/MicrobitBinaryAdapter");
const Microbit2ASCIIAdapter = require("./adapters/Microbit2ASCIIAdapter"); 
const MicrobitBinaryAdapter = require ("./adapters/MicrobitBinaryAdapter.js"); //El hardware con protocolo binario.
const StrudelAdapter = require ("./adapters/StrudelAdapter"); //El adapter para conectar con Strudel.
const log = {
  info: (...args) => console.log(`[${new Date().toISOString()}] [INFO]`, ...args),
  warn: (...args) => console.warn(`[${new Date().toISOString()}] [WARN]`, ...args),
  error: (...args) => console.error(`[${new Date().toISOString()}] [ERROR]`, ...args)
};


function getArg(name, def = null) {
  const i = process.argv.indexOf(`--${name}`);
  if (i >= 0 && i + 1 < process.argv.length) return process.argv[i + 1];
  return def;
}

function hasFlag(name) {
  return process.argv.includes(`--${name}`);
}

function nowMs() { return Date.now(); }

function safeJsonParse(s) {
  try {
    return JSON.parse(s);

  } catch (e) {
    log.warn("Failed to parse JSON: ", s, e);
    return null;
  }
}

function broadcast(wss, obj) {
  const text = JSON.stringify(obj);
  for (const client of wss.clients) {
    if (client.readyState === 1) client.send(text);
  }
}

function status(wss, state, detail = "") {
  broadcast(wss, { type: "status", state, detail, t: nowMs() });
}

//Configuración del sketch de p5.js para recibir los datos del microbit a través del bridgeServer y dibujar en el canvas según esos datos.
const DEVICE = (getArg("device", "sim") || "sim").toLowerCase();
const WS_PORT = parseInt(getArg("wsPort", "8081"), 10);
const SERIAL_PATH = getArg("serialPort", null);
const BAUD = parseInt(getArg("baud", "115200"), 10);
const SIM_HZ = parseInt(getArg("hz", "30"), 10);
const VERBOSE = hasFlag("verbose");

//Buscar microbit automáticamente, si no se encuentra, usar el simulador.
async function findMicrobitPort() { //Detecta el microbit automáticamente buscando en los puertos seriales un dispositivo con el vendorId del microbit.
  const ports = await SerialPort.list();
  const microbit = ports.find(p =>
    p.vendorId && parseInt(p.vendorId, 16) === 0x0D28
  );
  return microbit?.path ?? null;
}

async function createAdapter() { //Crea el adapter y decide si se conecta al microbit real o al simulador.
  //Microbit v1
  if (DEVICE === "microbit") {
    const path = SERIAL_PATH ?? await findMicrobitPort();
    if (!path) {
      log.error("micro:bit not found. Use --serialPort to specify manually.");
      process.exit(1);
    }
    log.info(`micro:bit (v1) found at ${path}`);
    return new MicrobitASCIIAdapter({ path, baud: BAUD, verbose: VERBOSE}); //Path es la dirección del puerto donde está conectado el microbit.
    //baud es la velocidad de comunicación serial
    //VERBOSE muestra un mensaje extra en consola con los datos crudos recibidos del microbit.
  }

  //Microbit v2 
  if (DEVICE === "microbit-v2") {
    const path = SERIAL_PATH ?? await findMicrobitPort();

    if(!path) {
      log.error("micro:bit not found. Use --serialPort to specify manually");
      process.exit(1); //Se cierra el programa porque no se encontró el microbir
    }
    log.info(`micro:bit (v2) found at ${path})`);
    return new Microbit2ASCIIAdapter({ path, baud: BAUD, verbose: VERBOSE});
  }

//Microbit con protocolo binario
if (DEVICE === "microbitbinary") {
  const path = SERIAL_PATH ?? "COM8"; //Busca el microbit automáticamente, si no se encuentra, usa el simulador.
  if (!path) {
    log.error("Micro:bit not found. Use --serialPort to specify manually.");
    process.exit(1); //Se cierra el programa porque no se encontró el microbit.
  }
 log.info(`micro:bit (binary) found at ${path}`);
 return new MicrobitBinaryAdapter({ path, baud: BAUD, verbose: VERBOSE}); //Path es la dirección del puerto donde está conectado el microbit. 
}

//Strudel
if (DEVICE === "strudel") {
  return new StrudelAdapter({
    url: "ws://localhost:8080", //Dirección del servidor de Strudel
    verbose: VERBOSE 
  });
}
  

  // if (DEVICE === "microbit-bin") {
  //   const path = SERIAL_PATH ?? await findMicrobitPort();
  //   if (!path) {
  //     log.error("micro:bit not found. Use --serialPort to specify manually.");
  //     process.exit(1);
  //   }
  //   return new MicrobitBinaryAdapter({ path, baud: BAUD });
  // }
  log.info("Using Simulator");
  return new SimAdapter({ hz: SIM_HZ });
}

async function main() {
  const wss = new WebSocketServer({ port: WS_PORT }); //Permite que el navegador (sketch.js) se conecte
  log.info(`WS listening on ws://127.0.0.1:${WS_PORT} device=${DEVICE}`);

  const adapter = await createAdapter();
   
  //Eventos del adapter
  adapter.onConnected = (detail) => {
    log.info(`[ADAPTER] Device Connected: ${detail}`);
    status(wss, "connected", detail);
  };

  adapter.onDisconnected = (detail) => {
    log.warn(`[ADAPTER] Device Disconnected: ${detail}`);
    status(wss, "disconnected", detail);
  };

  adapter.onError = (detail) => {
    log.error(`[ADAPTER] Device Error: ${detail}`);
    status(wss, "error", detail);
  };

  adapter.onData = (d) => { //Son los eventos del adapter, es donde se reciben los datos del parser, los coniverte a JSON y los envía al navegador.
     
    //Caso 1: Datos del Strudel
    if(d.type === "strudel") {
      broadcast(wss, d); //Envía los datos del Strudel al navegador.
      return;
    }
    
    //Caso 2: Datos del microbit
    broadcast(wss, {
      type: "microbit",
      x: d.x,
      y: d.y,
      btnA: !!d.btnA, //Se convierte el botón A en valor booleano.
      btnB: !!d.btnB,
      t: nowMs() 

    })
  };

  status(wss, "ready", `bridge up (${DEVICE})`);

  //Conexión con el cliente. 
  wss.on("connection", (ws, req) => { //Conexión con el cliente. Aquí se conecta el navegador y recibe los estados.
    log.info(`[NETWORK] Remote Client connected from ${req.socket.remoteAddress}. Total clients: ${wss.clients.size}`);

    const state = adapter.connected ? "connected" : "ready";

    const detail = adapter.connected
      ? adapter.getConnectionDetail()
      : `bridge (${DEVICE})`;

    ws.send(JSON.stringify({ type: "status", state, detail, t: nowMs() }));

    ws.on("message", async (raw) => {
      const msg = safeJsonParse(raw.toString("utf8"));
      if (!msg) return;

      if (msg.cmd === "connect") {
        log.info(`[NETWORK] Client requested adapter connect`);

        if (adapter.connected) {
         ws.send(JSON.stringify({
          type: "status",
          state: "connected",
          detail: adapter.getConnectionDetail(),
          t: nowMs()
        }));
        return;
      }

        
        try {
          await adapter.connect();
        } catch (e) {
          const detail = `connect failed: ${e.message || e}`;
          log.error(`[ADAPTER] ` + detail);
          status(wss, "error", detail);
        }
      }
        

      if (msg.cmd === "disconnect") {
        log.info(`[NETWORK] Client requested adapter disconnect`);
     
        try{
          await adapter.disconnect();
        } catch (e) {
          const detail = `disconnect failed: ${e.message || e}`;
          log.error(detail);
          status (wss, "error", detail);
        }
      }

      if (msg.cmd === "setSimHz" && adapter instanceof SimAdapter) {
        log.info(`Setting Sim Hz to ${msg.hz}`); 
        await adapter.handleCommand(msg);
        status(wss, "connected", `sim hz=${adapter.hz}`);
      }
      
      if (msg.cmd === "setLed") {
        try {
          await adapter.handleCommand?.(msg);
          } catch (e) {
            const detail = `command failed: ${e.message || e}`;
            log.error(`[ADAPTER] ` + detail);
            status(wss, "error", detail);
          }
        }
      });
             
      ws.on("close", () => {
        log.info(`[NETWORK] Remote Client disconnected. Total clients left: ${wss.clients.size}`);
        if (wss.clients.size === 0) {
          log.info("[HW-POLICY] No more remote clients. Auto-disconnecting adapter device to free resources...");
          adapter.disconnect();
        }
      });
    });
    
    //Auto-connect solo para el simulador.
    if (DEVICE === "sim") {
      await adapter.connect();
    }
  }
  
  main().catch((e) => {
    log.error("Fatal:", e);
    process.exit(1);
  });
```

**Código bridgeClient completo:**

```
class BridgeClient {
  constructor(url = "ws://127.0.0.1:8081") { //Define en donde se conecta el bridge, por defecto es en el localHost en el puerto 8081.
    this._url = url;
    //Muestra el estado de la conexión con el bridge, si está abierto o cerrado.
    this._ws = null;
    this._isOpen = false;
    
    //Callbacks para manejar eventos de datos, conexión, dexconexión y estado.
    this._onData = null;
    this._onConnect = null;
    this._onDisconnect = null;
    this._onStatus = null;
  }

  get isOpen() {
    return this._isOpen;
  }

  onData(callback) { this._onData = callback; } //Cuando llegan datos, se ejecuta la función.
  onConnect(callback) { this._onConnect = callback; } //Cuando se conecta, se ejecuta la función.
  onDisconnect(callback) { this._onDisconnect = callback; } //Cuando se desconecta.
  onStatus(callback) { this._onStatus = callback; } //Mensajes de estado, como errores o cambios de estado del microbit.

  open() { //Se conecta al bridgeServer.
    if (this._ws && this._ws.readyState === WebSocket.OPEN) {
      if (!this._isOpen) this.send({ cmd: "connect" });
      return;
    }

    if (this._ws) {
      this.close();
    }

    this._ws = new WebSocket(this._url);

    this._ws.onopen = () => {
      this.send({ cmd: "connect" });
    };

    this._ws.onmessage = (event) => { //Es donde llegan todos los mensajes del bridgeServer
      // Esperamos JSON normalizado desde el bridge
      let msg;
      try {
        msg = JSON.parse(event.data); //Parsear a JSON, convierte el mensaje en un objeto.
      } catch (e) {
        console.warn("WS message is not JSON:", event.data);
        return;
      }

      // Convención mínima:
      // - {type:"status", state:"...", detail:"..."}
      // - {type:"microbit", x:..., y:..., btnA:..., btnB:...}
      if (msg.type === "status") {
        this._onStatus?.(msg);

        if (msg.state === "connected") {
          this._isOpen = true;
          this._onConnect?.();
        }

        if (msg.state === "disconnected" || msg.state === "error" || msg.state === "ready") {
          this._isOpen = false; 
          this._onDisconnect?.();
          if (msg.state === "error") {
            this._ws?.close();
            this._ws = null;
          }          
        }
        return;
      }

      if (msg.type === "microbit") {
        // payload ya normalizado
        this._onData?.(msg);
        return;
      }

      if (msg.type === "strudel") {
        this._onData?.(msg); //Si el mensaje es del tipo strudel, se envía a la función de datos. Esto es para que el cliente pueda recibir los datos del strudel.
        return;
      }
    };

    this._ws.onerror = (err) => { //Muestra los errores de conexión con el bridgeServer.
      console.warn("WS error:", err);
    };

    this._ws.onclose = () => { //Si se cae la conexión con el bridgeServer, se muestra un mensaje de advertencia y se ejecuta la función de desconexión.
      this._handleDisconnect();
    };
  }

  close() {
    if (!this._ws || this._ws.readyState !== WebSocket.OPEN) return;

    try {
      this.send({ cmd: "disconnect" }); //Le dice al bridgeServer que se desconecte.
      this._isOpen = false;
    } catch (e) {
      console.warn("Failed to send disconnect command:", e);
    }
  }

  send(obj) {
    if (!this._ws || this._ws.readyState !== WebSocket.OPEN) return;
    this._ws.send(JSON.stringify(obj));
  }

  _handleDisconnect() {
    this._isOpen = false;
    this._ws = null;
    this._onDisconnect?.();
  }
}
```
4. Se agregó una nueva clase (PainterTask) en el FSM.js que crea una lógica de eventos donde:
- Los eventos de Strudel se guardan en una cola (eventQueue).
- Los eventos no se ejecutan de inmediato.
- En cada frame se revisa si el tiempo actual ya alcanzó el timestamp y cuando esto pasa, se convierte en una animación.

Esta parte del código hace que no se dibuje la animación cuando llega el mensaje, sino cuando debe ocurrir según su tiempo, lo que genera que se haga una sincronización con la música.

```
const ENTRY = Object.freeze({ type: "ENTRY" });
const EXIT = Object.freeze({ type: "EXIT" });

const EVENTS = {
  CONNECT: "CONNECT",
  DISCONNECT: "DISCONNECT",
  DATA: "DATA", //microbit data
  STRUDEL: "STRUDEL", //strudel data


}
class Timer {
  constructor(owner, eventToPost, duration) {
    this.owner = owner;
    this.event = eventToPost;
    this.duration = duration;
    this.startTime = 0;
    this.active = false;
  }

  start(newDuration = null) {
    if (newDuration !== null) this.duration = newDuration;
    this.startTime = millis();
    this.active = true;
  }

  stop() {
    this.active = false;
  }

  update() {
    if (this.active && millis() - this.startTime >= this.duration) {
      this.active = false;
      this.owner.postEvent(this.event);
    }
  }
}

class FSMTask { //No se modifica esta clase porque es la base para crear las máquinas de estados.
  constructor() {
    this.queue = [];
    this.timers = [];
    this.state = null;
  }

  postEvent(ev) {
    this.queue.push(ev);
  }

  addTimer(event, duration) {
    let t = new Timer(this, event, duration);
    this.timers.push(t);
    return t;
  }

  transitionTo(newState) {
    if (this.state) this.state(EXIT);
    this.state = newState;
    this.state(ENTRY);
  }

  update() {
    for (let t of this.timers) {
      t.update();
    }
    while (this.queue.length > 0) {
      let ev = this.queue.shift();
      if (this.state) this.state(ev);
    }
  }
}

class PainterTask extends FSMTask {
  constructor() {
    super();

    this.eventQueue = []; //Cola de eventos musicales.

    this.activeAnimations = []; //Animaciones activas, cada una con su propio estado interno.

    this.transitionTo(this.estado_esperando); 
  }

  //Estado esperando.
  estado_esperando = (ev) => {
    if (ev.type === "ENTRY") {
      console.log("Esperando conexión...");
    }

    else if (ev.type === EVENTS.CONNECT) {
      this.transitionTo(this.estado_corriendo); 
    }
  };

  //Estado corriendo.
  estado_corriendo = (ev) => {
    if (ev.type === ENTRY) {
      console.log("Sistema listo");

      this.eventQueue = [];
      this.activeAnimations = [];
    }

    else if(ev.type === EVENTS.DISCONNECT) {
      this.transitionTo(this.estado_esperando);
    }

    //Eventos de Strudel
    else if (ev.type === EVENTS.STRUDEL) {
      this.updateLogic(ev);
    }
  };

  //Guardar eventos de Strudel sin dibujar
updateLogic(ev) {

if (!ev.payload) return; //Si no hay payload, no hacer nada

let sound = null;
let delta = 0.25;

//Caso 1: Viene normalizado
if (ev.payload.s) {
  sound = ev.payload.s;
  delta = ev.payload.delta || 0.25;
}

//Caso 2: Viene crudo desde Strudel
else if (ev.payload.args) {
  let args = ev.payload.args;

  for (let i = 0; i < args.length; i++) {
    if (args[i] === "s") {
      delta = args[i + 1];
    }
  }
}

//Si no hay sonido, ignorar
if (!sound) return;

//Guardar evento en la cola
this.eventQueue.push({
  timestamp: Date.now(),
  s: sound,
  delta: delta
});

//Ordenar por tiempo
this.eventQueue.sort((a, b) => a.timestamp - b.timestamp); //El evento más próximo al tiempo actual queda al principio de la cola.
  
}

  //Ejecutar eventos en su tiempo
  processEvents() {
    let now = Date.now();

    while(
      this.eventQueue.length > 0 &&
      now >= this.eventQueue[0].timestamp
    ) {
      let ev = this.eventQueue.shift();

      this.activeAnimations.push({
        startTime: ev.timestamp,
        duration: ev.delta * 1000,
        type: ev.s,
        x: random(width),
        y: random(height)
      });
    }
  }
}
```

5. Se integró strudel en el código del sketch, donde el sketch recibe los datos del bridgeClient y los envía como eventos.

El sketch convierte los eventos musicales en visuales en 4 pasos.
1. Recibe los eventos de Strudel: Desde el bridgeClient, el sketch recibe mensajes ya normalizados tipo strudel, no los interpreta directamente, sino que los manda a una máquina de estados.
2. Guarda los eventos en una cola, o sea que cuando llega un evento de strudel, no se dibuja de inmediato sino que se guarda en la cola (Queue) con su timestamp y datos (sonido y duración).
3. Controla el tiempo de ejecución, es donde ocurre la sincronización de la música.
4. Dibuja la animación.

```
let painter;
let bridge;
let connectBtn;
let renderer = new Map();

function setup() {
    createCanvas(windowWidth, windowHeight);
    background(0);

    painter = new PainterTask(); //Crea una nueva tarea de pintura, que es la máquina de estados que controla la pintura en el canvas.
    bridge = new BridgeClient(); //Crea una nueva instancia del cliente del puente, que se conecta al servidor del puente para recibir los datos del microbit y el strudel.

    //Conexión con el bridge.
    bridge.onConnect(() => {
        connectBtn.html("Disconnect");
        painter.postEvent({ type: "CONNECT"});
    });

    bridge.onDisconnect(() => {
        connectBtn.html("Connect");
        painter.postEvent({ type: "DISCONNECT"});
    });

    bridge.onStatus((s) => {
        console.log("BRIDEGE STATUS:", s.state, s.detail ?? "");
    });

    //Aquí llegan todos los datos del microbit y el strudel.
    bridge.onData((data)=> {
         //Strudel
         if (data.type === "strudel") {
            painter.postEvent({
                type: "Strudel",
                timestamp: data.timestamp,
                payload: data.payload
            });
         }

         //Microbit
         else if (data.type === "microbit") {
            painter.postEvent({
                type: "DATA",
                payload: data
            });
         }
    });

    //Botón 
    connectBtn = createButton("Connect");
    connectBtn.position(10,10);
    connectBtn.mousePressed(() => {
        if (bridge.isOpen) bridge.close();
        else bridge.open();
    });

    renderer.set(painter.estado_corriendo, drawRunning);
}

function draw() {
    painter.update();
    renderer.get(painter.state)?.(); //Llama a la función de renderizado correspondiente al estado actual de la máquina de estados.
}

function drawRunning() {
    background(0, 40);

    //Activar eventos en el tiempo correcto
    painter.processEvents();

    //Dibujo de Strudel
    for (let anim of painter.ActiveAnimations) {

        let now = Date.now(); //Tiempo actual para calcular la animación.
        let progress = (now - anim.startTime) / anim.duration; //Progeso de la animación.

        if (progress > 1) continue; //Si la animación ha terminado, saltar a la siguiente.

        switch (anim.type) {
            
            case "tr909bd": //Bombo
            fill (255, 0, 80);
            noStroke();
            circle(width / 2, height / 2, lerp(50, 300, progress)); //Dibuja un círculo que crece con el tiempo.
            break;

            case "tr909sd": //Caja
            fill (0, 200, 255);
            rectMode(CENTER); 
            rect(width / 2, height / 2, lerp(width, 0, progress), 50); //Dibuja un rectángulo que se encoge con el tiempo.
            break; 

            case "tr909hh":
            case "tr909oh": //Hi-hat abierto y cerrado
            fill (255, 255, 0);
            rect(anim.x, anim.y, lerp(40, 0, progress), lerp(40, 0, progress)); //Dibuja un rectángulo que se encoge con el tiempo en la posición del hi-hat.
            break;

            default:
                fill(255);
                noFill();
                rect(anim.x, anim.y, lerp(100, 0, progress), lerp(100, 0, progress)); //Dibuja un rectángulo genérico para otros tipos de animaciones.
                break;
        }
    }

}

function windowResized() {
    resizeCanvas(windowWidth, windowHeight);
} 
```

## Bitácora de reflexión

**Error en el fsm.js**

Al inicio, el sistema sí recibía los datos desde strudel, pero las animaciones no aparecían en eñ html. El problema estaba en cómo se estaban interpretando los datos.

Strudel envía la información por medio de un formato específico, donde los datos importantes viene dentro del arreglo args. (args: ["s", "tr909bd", "delta", 0.5]) donde s indica el sonido, tr909bd es el sonido (bombo), delta es la duración, y 0.5 es el valor de esa duración.

El problema que había era que el código antes intentaba leer los datos así:
```
ev.payload.s
ev.payload.delta
```

Pero esos valores no existían directamente ahí porque están dentro del args y por eso s quedaba como undefined, delta también quedaba mal, los eventos no se guardaban bien y por eso no se veían las animaciones en pantalla. Para corregir este erros, se hizo un cambio en el updateLogic para que pudiera leer correctamente el arreglo args:

```
let args = ev.payload.args;

for (let i = 0; i < args.length; i++) {
  if (args[i] === "s") {
    sound = args[i + 1];
  }
  if (args[i] === "delta") {
    delta = args[i + 1];
  }
}
```
