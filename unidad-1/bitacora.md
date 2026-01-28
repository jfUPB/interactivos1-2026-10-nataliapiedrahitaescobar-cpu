# Unidad 1

## Bitácora de proceso de aprendizaje

### Actividad 01

### ¿Qué es un sistema físico interactivo?
Es la interacción entre un mundo físico y ficticio generado para ser exprimentado en tiempo real por varias personas.
### ¿Cómo podrías aplicar lo que has visto en tu perfil profesional?
Me gustaría irme por la línea de animación en mi perfil profesional y creo que los sistemas físicos interactivos se pueden aplicar en una animación para volverla más real, o sea que la animación se sienta con vida, como si estuviera sucediendo de verdad al frente de las personas que la están viendo, ya sea por medio de reacciones que hace el personaje que se sienten reales, logrando que los entornos de la animación se sientan potentes y reales, etc.

### Actividad 02

### ¿Qué es el diseño de arte generativo?
El diseño de arte generativo es una forma donde el diseñador crea reglas, datos y procesos que permiten a una computadora genear resultados visuales o narrativos, produciendo variaciones y soluciones que no serían fáciles de hacer manualmente.
### ¿Cómo puedo aplicar lo visto en mi perfil profesional?
Por la línea de animación, puedo usar el arte generativo creando sistemas de movimiento y animación basado en reglas, lo que me permite crear variaciones visuales y animaciones interactivas sin depender únicamente del formato frame by frame lo que me permite ampliar mis posibilidades creativas y las técnicas que puedo usar en mi trabajo.

### Actividad 03

código micro:bit
```py
from microbit import *

uart.init(baudrate=115200)
display.show(Image.BUTTERFLY)

while True:
    if button_a.is_pressed():
        uart.write('A')
        sleep(500)
    if button_b.is_pressed():
        uart.write('B')
        sleep(500)
    if accelerometer.was_gesture('shake'):
        uart.write('C')
        sleep(500)
    if uart.any():
        data = uart.read(1)
        if data:
            if data[0] == ord('h'):
                display.show(Image.HEART)
                sleep(500)
                display.show(Image.HAPPY)
```

código p5.js
``` js
let port;
let connectBtn;

function setup() {
    createCanvas(400, 400);
    background(220);
    port = createSerial();
    connectBtn = createButton('Connect to micro:bit');
    connectBtn.position(80, 300);
    connectBtn.mousePressed(connectBtnClick);
    let sendBtn = createButton('Send Love');
    sendBtn.position(220, 300);
    sendBtn.mousePressed(sendBtnClick);
    fill('white');
    ellipse(width / 2, height / 2, 100, 100);
}

function draw() {

    if(port.availableBytes() > 0){
        let dataRx = port.read(1);
        if(dataRx == 'A'){
            fill('red');
        }
        else if(dataRx == 'B'){
            fill('yellow');
        }
        else{
            fill('green');
        }
        background(220);
        ellipse(width / 2, height / 2, 100, 100);
        fill('black');
        text(dataRx, width / 2, height / 2);
    }


    if (!port.opened()) {
        connectBtn.html('Connect to micro:bit');
    }
    else {
        connectBtn.html('Disconnect');
    }
}

function connectBtnClick() {
    if (!port.opened()) {
        port.open('MicroPython', 115200);
    } else {
        port.close();
    }
}

function sendBtnClick() {
    port.write('h');
}
```

### Sacude el micro:bit ¿Qué pasa?
Cuando sacudo el microbit el círculo que se encuentra en p5.js se pone de color verde y en el centro del círculo aparece la letra C.
### Presiona el botón "Send Love" ¿Qué pasa?
Cuando apreto el botón "Send Love" en el micro:bit aparece una carita feliz al principio y al volver a apretar el botón aparece un corazón por unos segundos y después vuelve a aparecer la carita feliz.

### Actividad 04

código micro:bit
```py
from microbit import *

uart.init(baudrate=115200)

while True:

    if button_a.is_pressed():
        uart.write('A')
    else:
        uart.write('N')

    sleep(100)
```
código 5p.js
```js
  let port;
  let connectBtn;
  let connectionInitialized = false;

  function setup() {
    createCanvas(400, 400);
    background(220);
    port = createSerial();
    connectBtn = createButton("Connect to micro:bit");
    connectBtn.position(80, 300);
    connectBtn.mousePressed(connectBtnClick);
  }

  function draw() {
    background(220);

    if (port.opened() && !connectionInitialized) {
      port.clear();
      connectionInitialized = true;
    }

    if (port.availableBytes() > 0) {
      let dataRx = port.read(1);
      if (dataRx == "A") {
        fill("red");
      } else if (dataRx == "N") {
        fill("green");
      }
    }

    rectMode(CENTER);
    rect(width / 2, height / 2, 50, 50);

    if (!port.opened()) {
      connectBtn.html("Connect to micro:bit");
    } else {
      connectBtn.html("Disconnect");
    }
  }

  function connectBtnClick() {
    if (!port.opened()) {
      port.open("MicroPython", 115200);
      connectionInitialized = false;
    } else {
      port.close();
    }
  }
```
### ¿Por qué la aplicación no funcióno?
El programa no funcionaba porque solo se le colocaba el color al cuadrado cuando se apretaba la tecla A pero cuando al programa le llegaban otros datos diferentes, el color del cuadrado no cambiaba y dejaba los mismos datos que se mandaban al presionar A,o sea que el cuadrado se quedaba con el color rojo. Esto sucedía porque no estaba definido el valor visual en cada frame.
### ¿Por qué no funcionaba el programa con was_pressed() y por qué funciona con is_pressed()? 
Porque was_pressed() solo funciona cuando detecta el momento exacto en el que se presiona el botón y al mandar la señal a p5.ps esta se perdía rápido porque el código trabaja con el sistema while True y envía información constantemente y no solo presión por presión. (Para detectar eventos puntuales)
is_pressed() si funcionó porque el micro:bit revisa constantemente si el botón está presionado y esto hace que funcione la función True de forma continua. (Para interacciones que necesitan mantenerse en el tiempo)

---Código que no funciona---
```
let port;
let connectBtn;

function setup() {
    createCanvas(400, 400);
    background(220);
    port = createSerial();
    connectBtn = createButton('Connect to micro:bit');
    connectBtn.position(80, 300);
    connectBtn.mousePressed(connectBtnClick);
    let sendBtn = createButton('Send Love');
    sendBtn.position(220, 300);
    sendBtn.mousePressed(sendBtnClick);
    fill('white');
    ellipse(width / 2, height / 2, 100, 100);
}

function draw() {

    if(port.availableBytes() > 0){
        let dataRx = port.read(1);
        if(dataRx == 'A'){
            fill('red');
        }
        else if(dataRx == 'B'){
            fill('yellow');
        }
        else{
            fill('green');
        }
        background(220);
        ellipse(width / 2, height / 2, 100, 100);
        fill('black');
        text(dataRx, width / 2, height / 2);
    }


    if (!port.opened()) {
        connectBtn.html('Connect to micro:bit');
    }
    else {
        connectBtn.html('Disconnect');
    }
}

function connectBtnClick() {
    if (!port.opened()) {
        port.open('MicroPython', 115200);
    } else {
        port.close();
    }
}

function sendBtnClick() {
    port.write('h');
}
```
### ¿Por qué no funciona el programa con was_pressed() y por qué si funciona con is_pressed?


### Actividad 05

### Explicación del sistema físico interactivo:
Por medio del microbit la persona tiene la opción de presionar las teclas A o B y la información de la tecla seleccionada pasa a ser interpretada o leída por el programa p5.js. Si la tecla presionada es la A, el programa hace que el círculo se mueva hacia la izquierda y si la tecla presionada es la B, el programa hace que el círculo se mueva a la derecha. 

## Bitácora de aplicación 

### Actividad 05
**-Código programa 5P.js**

```js
let port;
let connectBtn;
let posX;

function setup() {
  createCanvas(400, 400);
  background(220);

  port = createSerial();

  connectBtn = createButton('Connect to micro:bit');
  connectBtn.position(160, 300);
  connectBtn.mousePressed(connectBtnClick);

  posX = width / 2;
}

function draw() {
  if (port.availableBytes() > 0) {
    let dataRx = port.read(1);

    if (dataRx == 'A') {
      posX = posX - 20;
    } else if (dataRx == 'B') {
      posX = posX + 20;
    }

    background(220);
    ellipse(posX, height / 2, 100, 100);
  }

  if (!port.opened()) {
    connectBtn.html('Connect to micro:bit');
  } else {
    connectBtn.html('Disconnect');
  }
}

function connectBtnClick() {
  if (!port.opened()) {
    port.open('MicroPython', 115200);
  } else {
    port.close();
  }
}
```

**-Código micro:bit**
```py
from microbit import *

while True:
    if button_a.is_pressed():
        uart.write('A')
        sleep(200)

    if button_b.is_pressed():
        uart.write('B')
        sleep(200)
```

## Bitácora de reflexión












