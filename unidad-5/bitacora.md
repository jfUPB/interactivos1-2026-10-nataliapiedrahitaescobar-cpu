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
## Bitácora de aplicación 


## Bitácora de reflexión
