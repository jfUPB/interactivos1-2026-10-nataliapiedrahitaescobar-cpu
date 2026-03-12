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



## Bitácora de reflexión

