# Ejercicio: Servidor de Consulta de Temperaturas
## Objetivo

Desarrollar una aplicación cliente-servidor utilizando sockets TCP en Python.

## Descripción

Una estación meteorológica mantiene información de temperatura de distintas ciudades.

Los clientes podrán conectarse al servidor y consultar la temperatura de una ciudad.

## Protocolo

El cliente envía:

GET Buenos Aires

El servidor responde:

25

Si la ciudad no existe:

ERROR Ciudad no encontrada

Para finalizar la conexión, el cliente puede enviar:

EXIT

y el servidor responderá:

BYE

## Datos iniciales

El servidor debe almacenar en memoria:

| Ciudad | Temperatura |
|--------|-------------|
| Buenos Aires	| 25 |
| Rosario	| 23 |
| Córdoba	| 20 |
| Mendoza	| 18 |
| Bariloche	| 10 |

## Requisitos
### Servidor
Escuchar en el puerto 5000.
- Aceptar múltiples consultas durante la misma conexión.
- Mostrar en pantalla:
  - IP del cliente.
  - Comando recibido.
  - Respuesta enviada.
- Manejar errores de comandos inválidos.
### Cliente
- Solicitar al usuario un comando.
- Enviar el comando al servidor.
- Mostrar la respuesta recibida.
- Permitir realizar múltiples consultas sin reconectarse.

## Ejemplo de ejecución
### Cliente

Ingrese comando:

GET Rosario

Respuesta:

23

Ingrese comando:

GET Salta

Respuesta:

ERROR Ciudad no encontrada

Ingrese comando:

EXIT

Respuesta:
BYE
