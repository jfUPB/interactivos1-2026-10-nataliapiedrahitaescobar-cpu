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

## Bitácora de aplicación 
### **ACTIVIDAD 2**
**MicrobitBinaryAdapter.js**
1. Tomé como base el código del ASCII Adpater y lo modifiqué para que pudiera trabajar con los datos binarios, usando un buffer para ir guardando la información que llega.
2. Luego hice que el programa leyeran los datos binarios y los separara para obtener x, y, botón A y botón B.
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

## Bitácora de reflexión
