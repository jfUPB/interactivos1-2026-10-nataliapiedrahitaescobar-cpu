# Unidad 5
## Bitácora de proceso de aprendizaje
### **¿Qué ventajas y desventajas ves en usar un formato binario en lugar de texto ASCII?**
**Ventajas**

1. El formato biniario usa menos espacio y más rápido de enviar porque los datos van directamente en bytes.
2. El formato binario es más eficiente de utilizar con el microbit.

**Desventajas**

1. El formato binario no se puede leer fácilmente, porque su formato no es textual como el ASCII.
2. Es más complicado de programar y comprender porque se deben convertir los datos para poder interpretarlos.

### **Si xValue=500, yValue=524, aState=True, bState=False, ¿cómo se vería el paquete en hexadecimal? (Pista: convierte cada valor según su tipo y anota los bytes en orden.)**

'>2h2B' (Formato a utilizar)
> Big Endian: Lo bytes van en orden de mayor a menor.
2h xValue, yValue: Enteros de dos bytes con signo.
2B aState, bState: 1 byte cada uno.

**Convertir cada valor:**
xValue = 500
Hexadecimal: 1F4
Big Endian: 01 F4

yValue = 524
Hexadecimal: 20C
Big Endian: 02 0C

aState = True
True = 1
Big Endian: 01

bState = False
False = 0 
Big Endian = 00

**Resultado Final:**
01 F4 02 OC 01 00

### **¿Por qué el protocolo ASCII de la unidad anterior no tenía este problema de sincronización?**

Porque en ASCII cada mensaje termina con el caracter especial \n, el cual funciona como delimitador. Entonces aunque los datos lleguen corridos, el receptor simpre va a leer los datos hasta que llegue a \n.

### **¿Por qué en binario no podemos usar \n como delimitador?**

No se puede utilizar porque el protocolo binario usa bytes puros (números de 0 a 255) y no texto.
\n no es un caracter especial, sino un valor que puede aparecer dentro de los datos (00 0A) lo que puede causar errores al separar los paquetes.

### **¿Cuántos bytes tiene el paquete completo con framing?**

Cada framing cuenta con 8 bytes: 1 byte del header, los datos que cuentan con 6 bytes y 1 byte del checksum.

### **¿Cuántos más que sin framing?**

Un paquete con framing tiene 2 bytes adicionales que serían el header y el checksum y sin framing, el paquete solo contaría con 6 bytes que serían los datos.

### **¿Qué pasa si un byte de datos tiene el valor 0xAA (170 en decimal)? ¿Podría el receptor confundirlo con un header? ¿Cómo ayuda el checksum en este caso?** 

Un byte con valor 0xAA si puede confundirse con un header si el receptor no está sincronizado, pero el checksum permite detectar que los datos son incorrectos, evitando que se interprete un paquete inválido como válido.

### **ACTIVIDAD 3**

**Realiza una tabla comparativa entre el adapter ASCII que creaste en la Unidad 4 (MicrobitV2Adapter.js) y el adapter binario de esta unidad (MicrobitBinaryAdapter.js).**
![Cuadro comparativo](https://github.com/user-attachments/assets/8b4b74ee-e2b7-4a6f-9825-57423af90f4d)


## Bitácora de aplicación 
### **ACTIVIDAD 2**
**MicrobitBinaryAdapter.js**
1. Tomé como base el código del ASCII Adpater y lo modifiqué para que pudiera trabajar con los datos binarios, usando un buffer para ir guardando la información que llega.
2. Luego hice que el programa leyera los datos binarios y los separara para obtener x, y, botón A y botón B.
3. Agregué la verificación del checksum para que verificara que los datos llegaran bien y no tuvieran errores.
```
const { SerialPort } = require("serialport");
const BaseAdapter = require("./BaseAdapter");

class ParseError extends Error { }



class MicrobitBinaryAdapter extends BaseAdapter {
  constructor({ path, baud = 115200, verbose = false } = {}) {
    super();
    this.path = path;
    this.baud = baud;
    this.port = null;
    this.buf = Buffer.alloc(0);
    this.verbose = verbose;
  }

  async connect() {
    if (this.connected) return;
    if (!this.path) throw new Error("serialPort is required for microbit device mode");

    this.port = new SerialPort({
      path: this.path,
      baudRate: this.baud,
      autoOpen: false,
    });

    await new Promise((resolve, reject) => {
      this.port.open((err) => (err ? reject(err) : resolve()));
    });

    this.connected = true;
    this.onConnected?.(`serial open ${this.path} @${this.baud}`);

    this.port.on("data", (chunk) => this._onChunk(chunk));
    this.port.on("error", (err) => this._fail(err));
    this.port.on("close", () => this._closed());
  }

  async disconnect() {
    if (!this.connected) return;
    this.connected = false;

    if (this.port && this.port.isOpen) {
      await new Promise((resolve, reject) => {
        this.port.close((err) => {
          if (err) reject(err);
          else resolve();
        });
      });
    }
    this.port = null;
    this.buf = Buffer.alloc(0); //Ya no es texto, es un buffer de bytes.
    this.onDisconnected?.("serial closed");
  }

  getConnectionDetail() {
    return `serial open ${this.path}`;
  }

  _onChunk(chunk) {
   //Donde se acumulan los bytes
   this.buf = Buffer.concat([this.buf, chunk]);

   //Procesa los paquetes
   while(this.buf.length >= 8) {

    //Buscar el header del paquete (0xAA)
    if(this.buf[0] !== 0xAA) {
        this.buf = this.buf.slice(1); //Si el dato no es el header, se descarta y se sigue buscando.
        continue;
    }

    //Tomar el paquete completo de 8 bytes
    const packet = this.buf.slice(0,8); 

    //Calcular el checksum
    let sum = 0;
    for(let i = 1; i <= 6; i++) {
        sum += packet[i];
    }
    const checksum = sum % 256;

    //Verificar el checksum
    {
        if(checksum !== packet[7]) {
            console.warn("Checksum inválido");
            this.buf = this.buf.slice(1); //Descartar el byte del header y seguir buscando.
            continue;
        }
    }

    //Parsear los datos del paquete
    const x = packet.readInt16BE(1); //Bytes 1 y 2
    const y = packet.readInt16BE(3); //Bytes 3 y 4
    const btnA = packet[5] === 1; //Byte 5
    const btnB = packet[6] === 1; //Byte 6

    //Enviar los datos al navegador
    this.onData?.({ x, y, btnA, btnB});

    //Limpiar el buffer de los bytes procesados 
    this.buf = this.buf.slice(8); //Se descartan los 8 bytes del paquete procesado y se sigue con el siguiente paquete.
   }
    
   //Evitar crecimiento infinito del buffer en caso de datos corruptos.
   if (this.buf.length > 4096) this.buf = Buffer.alloc(0); //Si el buffer crece demasiado sin encontrar paquetes válidos, se reinicia el buffer para evitar problemas de memoria.
  }

  _fail(err) {
    this.onError?.(String(err?.message || err));
    this.disconnect();
  }

  _closed() {
    if (!this.connected) return;
    this.connected = false;
    this.port = null;
    this.buf = Buffer.alloc(0);
    this.onDisconnected?.("serial closed (event)");
  }

  async writeLine(line) {
    if (!this.port || !this.port.isOpen) return;
    await new Promise((resolve, reject) => {
      this.port.write(line, (err) => (err ? reject(err) : resolve()));
    });
  }

  async handleCommand(cmd) {
    if (cmd?.cmd === "setLed") {
      const x = Math.max(0, Math.min(4, Math.trunc(cmd.x)));
      const y = Math.max(0, Math.min(4, Math.trunc(cmd.y)));
      const v = Math.max(0, Math.min(9, Math.trunc(cmd.value)));
      await this.writeLine(`LED,${x},${y},${v}\n`);
    }
  }
}

module.exports = MicrobitBinaryAdapter;
```
4. Integré el nuevo adaptador al BridgeServer para poder usarlo desde el terminal.
**bridgeServer.js**
```

//   Uso:
//     node bridgeServer.js
//     node bridgeServer.js --device microbit
//     node bridgeServer.js --device sim --wsPort 8081 --hz 30
//     node bridgeServer.js --device microbit --wsPort 8081 --serialPort COM5 --baud 115200
//     node bridgeServer.js --device microbit-v2 
//     node bridgeServer.js --device microbitBinary --serialPort COM5 --baud 115200
//  Probar enconsola: node bridgeServer.js --device microbitBinary --wsPort 8081
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
const MicrobitAsciiAdapter = require("./adapters/MicrobitASCIIAdapter"); //El hardware real.
// const MicrobitBinaryAdapter = require("./adapters/MicrobitBinaryAdapter");
const Microbit2ASCIIAdapter = require("./adapters/Microbit2ASCIIAdapter"); 
const MicrobitBinaryAdapter = require ("./adapters/Microbit2ASCIIAdapter"); //El hardware con protocolo binario.
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
    return new Microbit2ASCIIAdapter({ path, baud: BAUD, verbose: VERBOSE}); //Path es la dirección del puerto donde está conectado el microbit.
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
  const path = SERIAL_PATH ?? await findMicrobitPort(); //Busca el microbit automáticamente, si no se encuentra, usa el simulador.
  if (!path) {
    log.error("Micro:bit not found. Use --serialPort to specify manually.");
    process.exit(1); //Se cierra el programa porque no se encontró el microbit.
  }
 log.info(`micro:bit (binary) found at ${path}`);
 return new MicrobitBinaryAdapter({ path, baud: BAUD, verbose: VERBOSE}); //Path es la dirección del puerto donde está conectado el microbit. 
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
    broadcast(wss, {
      type: "microbit",
      x: d.x,
      y: d.y,
      btnA: !!d.btnA,
      btnB: !!d.btnB,
      t: nowMs(),
    });
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

## Bitácora de reflexión
- Tuve dudas en la parte del código del Adapter porque al calcular el checksum se utilizaban 6 datos en el código
```
let sum = 0;
    for(let i = 1; i <= 6; i++) {
        sum += packet[i];
    }
    const checksum = sum % 256;
```
mientras que al verificar el chekcsum se utilizaban 7 datos y pensaba que eso era un error.
```
   {
        if(checksum !== packet[7]) {
            console.warn("Checksum inválido");
            this.buf = this.buf.slice(1); //Descartar el byte del header y seguir buscando.
            continue;
        }
    }
```
Pero entendí que al verificar el checksum se usan 7 datos porque aquí se comparan dos cosas distintas: Los 6 bytes que se utilizan para calcular el el checksum y el checksum que ya viene en el paquete.

- Pensé que tenía un error en el adapter del microbit binario porque en esta parte del código todavía se estaban mandando comandos en texto, lo cual no se debería hacer en binario.
```
if (cmd?.cmd === "setLed") { const x = Math.max(0, Math.min(4, Math.trunc(cmd.x))); const y = Math.max(0, Math.min(4, Math.trunc(cmd.y))); const v = Math.max(0, Math.min(9, Math.trunc(cmd.value))); await this.writeLine(LED,${x},${y},${v}\n);
```
pero mantuve el envío de comandos en formato ASCII para simplificar la comunicación con el microbit; ya que el código del dispositivo no implementa recepción en binario.
