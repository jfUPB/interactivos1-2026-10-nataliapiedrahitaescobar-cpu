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


## Bitácora de reflexión
