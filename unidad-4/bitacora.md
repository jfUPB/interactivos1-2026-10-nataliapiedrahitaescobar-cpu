# Unidad 4

## Bitácora de proceso de aprendizaje
### **Diagrama de funcionanmiento**
<img width="602" height="292" alt="image" src="https://github.com/user-attachments/assets/f8d80ce9-27e9-429e-b5e8-59318441193b" />


## Bitácora de aplicación 
**PROCESO**
**1.Creación del nuevo adaptor:**
Se creó un nuevo adaptador tipos ASCII que se encaragaría de leer los nuevos datos del sistema serial para que funcionara bien el nuevo arte generativo pedido. Para esto se hizo un cambio en la parte de parseCsv(Line) donde se agregaron los nuevos valores de la suma de verificación y el chk (const chk, const calc) y se cambió el const value que pasó de ser (",") a ser ("|") como se pedía en el enunciado.
**CÓDIGO**
```
const { SerialPort } = require("serialport");
const BaseAdapter = require("./BaseAdapter");

class ParseError extends Error { }

//Interpreta el texto del hardware 
function parseCsvLine(line) {
  const values = line.trim().split("|");
  if (values.length !== 6) throw ne
 ParseError(`Expected 6 values, got ${values.length}`);
 
  const x = Number(values[1].split(":")[1]); //: Es la separación de la clave del valor.
  const y = Number(values[2].split(":")[1]);
  const btnA = String(values[3].split(":")[1]);
  const btnB = String(values[4].split(":")[1]);
  const chk = Number(values[5].split(";")[1]);
// values[1,2,3,4,5] acceden a una posiciión específica del arreglo, split(":")[1] obtiene el valor después de los dos puntos, y trim() elimina espacios en blanco.
 const calc = Math.abs(x) + Math.abs(y) + Number(btnA) + Number(btnB);
 if (calc !== chk){
  throw new ParseError("Checksum mismatch");
 }
  if (!Number.isFinite(x) || !Number.isFinite(y)) throw new ParseError("Invalid numeric data");
  if (x < -2048 || x > 2047 || y < -2048 || y > 2047) throw new ParseError("Out of expected range");
  if (!["true", "false"].includes(btnA) || !["true", "false"].includes(btnB)) throw new ParseError("Invalid button data");

console.log(parseCsvLine("$T:45020|X:-245|Y:12|A:1|B:0|CHK:258"))

  return { x: x | 0, y: y | 0, btnA: btnA === "true", btnB: btnB === "true" };
}


class MicrobitAsciiAdapter extends BaseAdapter {
  constructor({ path, baud = 115200, verbose = false } = {}) {
    super();
    this.path = path;
    this.baud = baud;
    this.port = null;
    this.buf = "";
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
    this.buf = "";
    this.onDisconnected?.("serial closed");
  }

  getConnectionDetail() {
    return `serial open ${this.path}`;
  }

  _onChunk(chunk) {
    this.buf += chunk.toString("utf8");

    let idx;
    while ((idx = this.buf.indexOf("\n")) >= 0) {
      const line = this.buf.slice(0, idx).trim();
      this.buf = this.buf.slice(idx + 1);

      if (!line) continue;

      try {
        const parsed = parseCsvLine(line);
        this.onData?.(parsed);
      } catch (e) {
        if (e instanceof ParseError) {
          if (this.verbose) console.log("Bad data:", e.message, "raw:", line);
        } else {
          this._fail(e);
        }
      }
    }

    if (this.buf.length > 4096) this.buf = "";
  }

  _fail(err) {
    this.onError?.(String(err?.message || err));
    this.disconnect();
  }

  _closed() {
    if (!this.connected) return;
    this.connected = false;
    this.port = null;
    this.buf = "";
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

module.exports = MicrobitAsciiAdapter;

```
<img width="569" height="346" alt="image" src="https://github.com/user-attachments/assets/675f921c-7820-4a77-b01c-6556fe8e066f" />

Luego se cerificó el código para saber si funcionaba bien el adaptador por medio de view---terminal---newterminal-code bridgeServer.js y como se puede ver en la imagen se ve que el adaptador está conectado, lo que demuestra que el código funciona bien y de la manera esperada.
<img width="960" height="540" alt="image" src="https://github.com/user-attachments/assets/00b02b8b-4be5-4fb5-99ad-8f60e58ce311" />

**2.Modificación del archivo Sketch:**
Aquí se hicieron los ajustes en el archivo sketh.js para unir los datos del micro:bit con el nuevo código de arte generativo pedido.
Se modificó la función drawRunning() para que utilizara los valores del acelerómetro en lugar de usa el mouse. De esta forma el eje X controla el tamaño de la figura y el eje Y controla la cantidad de vértices del polígono. También se corrigieron algunos errores en el acceso a los datos recibido (rxData) y en la condición que verifica si los datos están listos para ser utilizados.
**CÓDIGO:**
```
// Maneja la conexión con el microbit, Recibe los datos del microbit y dibuja en pantalla con estos datos. Utiliza una máquina de estados para manejar la lógica de la aplicación.

const EVENTS = {
    //Definición de los eventos.
    CONNECT: "CONNECT",
    DISCONNECT: "DISCONNECT",
    DATA: "DATA",
    KEY_PRESSED: "KEY_PRESSED",
    KEY_RELEASED: "KEY_RELEASED",
};

class PainterTask extends FSMTask {
    constructor() {
        super();

        this.c = color(181, 157, 0); //Color inicial del dibujo. se actualiza al soltar el botón B del microbit, con un color aleatorio.
        this.lineSize = 100;
        this.angle = 0;
        this.clickPosX = 0;
        this.clickPosY = 0;

        this.rxData = { //Objeto donde se guardan los datos del microbit, se actualizan en la función updateLogic.
            x: 0,
            y: 0,
            btnA: false,
            btnB: false,
            prevA: false,
            prevB: false,
            ready: false
        };
        //Es donde se guardan los datos recibidos del microbit, se actualizan en la función updateLogic.
        this.transitionTo(this.estado_esperando);
    }
  
    //Estado donde todavía no hay conexión con el microbit.
    estado_esperando = (ev) => {
        if (ev.type === "ENTRY") {
            cursor();
            console.log("Waiting for connection...");
        } else if (ev.type === EVENTS.CONNECT) {
            this.transitionTo(this.estado_corriendo);
        }
    };

    //El microbit ya está conectado, se reciben los datos y se dibuja en pantalla.
    estado_corriendo = (ev) => {
        if (ev.type === "ENTRY") {
            noCursor();
            strokeWeight(0.75);
            background(255);
            console.log("Microbit ready to draw");
            this.rxData = {
                x: 0,
                y: 0,
                btnA: false,
                btnB: false,
                prevA: false,
                prevB: false,
                ready: false
            };
        }

        else if (ev.type === EVENTS.DISCONNECT) { //Cuando el bridge recibe los datos de desconexión, se vuelve al estado de espera.
            this.transitionTo(this.estado_esperando);
        }

        else if (ev.type === EVENTS.DATA) {
            this.updateLogic(ev.payload); //Se procesan los datos del microbit.
        }

        else if (ev.type === EVENTS.KEY_PRESSED) {
            this.handleKeys(ev.keyCode, ev.key);
        }

        else if (ev.type === EVENTS.KEY_RELEASED) {
            this.handleKeyRelease(ev.keyCode, ev.key);
        }

        else if (ev.type === "EXIT") {
            cursor();
        }
    };
    //Convierte los valores del acelerómetro a coordenadas de la pantalla y maneja la lógica de los botones.
    updateLogic(data) {
        this.rxData.ready = true;
        this.rxData.x = map(data.x,-2048,2047,0,width);
        this.rxData.y = map(data.y,-2048,2047,0,height);
        this.rxData.btnA = data.btnA;
        this.rxData.btnB = data.btnB;

        if (this.rxData.btnA && !this.prevA) {
            this.lineSize = random(50, 160);
            this.clickPosX = this.rxData.x;
            this.clickPosY = this.rxData.y;
            console.log("A pressed");
        }

        if (!this.rxData.btnB && this.prevB) {
            this.c = color(random(255), random(255), random(255), random(80, 100));
            console.log("B released");
        }

        this.prevA = this.rxData.btnA;
        this.prevB = this.rxData.btnB;
    }
}

let painter;
let bridge;
let connectBtn;
const renderer = new Map();

function setup() { //Crea el canvas, usa todo el tamaño de la ventana y pinta el fondo de blanco (255).
    createCanvas(windowWidth, windowHeight);
    background(255);
    painter = new PainterTask(); //Es el objeto principal de la aplicación.
    bridge = new BridgeClient(); //Puente de comunicación con el microbit.

    bridge.onConnect(() => {
        connectBtn.html("Disconnect");
        painter.postEvent({ type: EVENTS.CONNECT });
    });

    bridge.onDisconnect(() => {
        connectBtn.html("Connect");
        painter.postEvent({ type: EVENTS.DISCONNECT });
    });

    bridge.onStatus((s) => {
        console.log("BRIDGE STATUS:", s.state, s.detail ?? "");
    });

    bridge.onData((data) => { //Se ejecuta cada vez que llegan datos al microbit.
        painter.postEvent({
            type: EVENTS.DATA, payload: {
                x: data.x,
                y: data.y,
                btnA: data.btnA,
                btnB: data.btnB
            }
        });
    });

    connectBtn = createButton("Connect");
    connectBtn.position(10, 10);
    connectBtn.mousePressed(() => {
        if (bridge.isOpen) bridge.close();
        else bridge.open();
    });

    renderer.set(painter.estado_corriendo, drawRunning); //Si el estado está corriendo, ejecuta el drawRunning.
}

function draw() {//Se ejecuta 60 veces por segundo.
    painter.update(); //Actualiza la máquina de estados.
    renderer.get(painter.state)?.();
}

function drawRunning() { //Ejecuta cada frame mientras la máquina de estados esté en estado_corriendo.
   let mb = painter.rxData;//Busca que función dibuja el estado actual, se obtiene  los datos que llegaron del microbit.
   
   
   if (!mb || !mb.ready) return;//Verifica si llegaron los datos.

   if(true){
       push();//Guarda la configuración actual del dibujo.
       translate(width / 2, height / 2);
       
       //Resolución del polígono según inclinación en y
       let circleResolution = int(map(mb.y, 0, height, 2, 10));
       
       //Radio según resolución en x
       let radius = map(mb.x, 0, width, 10, width / 2);
       
       let angle = TAU / circleResolution;
       
       //Botón b activa relleno
       if(mb.btnB){
           fill(34,45,122,50);
        }else {
            noFill();
        }
        
        beginShape();
        for (let i = 0; i <= circleResolution; i++){
         let x = cos(angle * i) * radius;
         let y = sin(angle * i) * radius;
         vertex(x,y);
        }

     endShape();

     pop();
   } 
}

function windowResized() {
    resizeCanvas(windowWidth, windowHeight);
}
```
<img width="960" height="509" alt="image" src="https://github.com/user-attachments/assets/953d65c5-e231-4fcf-986c-0c814946853e" />

-La página ejecutada cuando el polígono estaba muy pequeño:
<img width="960" height="479" alt="image" src="https://github.com/user-attachments/assets/2d419682-12b0-4ad7-8664-ba059d9a6c58" />

-La página con las medidas del polígono corregidas:
<img width="960" height="512" alt="image" src="https://github.com/user-attachments/assets/2c56aa4a-88bc-4422-8b69-3121021f94e8" />

**3.Código del Micro:bit para que funcionaran los controles:**
```
# Imports go at the top
from microbit import *

while True: 
    x = accelerometer.get_x()
    y = accelerometer.get_y()

    btnA = button_a.is_pressed()
    btnB = button_b.is_pressed()

    calc = (x + y + btnA + btnB) % 256
    chk = calc
    
    print("{}|{}|{}|{}".format(x, y, int(btnA), int(btnB), chk, calc))

    sleep(50)
```
## Bitácora de reflexión
**¿Por qué el botón de connect en el html no se activaba después de corregir el código?**
No funcionaba porque no tenía ejecutado el servidor del nodo, para solucionarlo solo debía ir al terminal y activar node bridgeServer.js, dejar la terminal abierta, abrir el HTML por medio del Go Live y presionar el connect. Ahí ya funcionó la página.

