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
### Actividad 05

### Explicación del sistema físico interactivo:
Por medio del microbit la persona tiene la opción de presionar las teclas A o B y la información de la tecla seleccionada pasa a ser interpretada o leída por el programa p5.js. Si la tecla presionada es la A, el programa hace que el círculo se mueva hacia la izquierda o si la tecla presionada es B, el programa hace que el círculo se mueva a la derecha. 
## Bitácora de aplicación 

### Actividad 05
**-Código programa 5P.js**
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


**-Código micro:bit**
from microbit import *

while True:
    if button_a.is_pressed():
        uart.write('A')
        sleep(200)

    if button_b.is_pressed():
        uart.write('B')
        sleep(200)
## Bitácora de reflexión






