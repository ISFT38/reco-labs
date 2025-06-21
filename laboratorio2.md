# Laboratorio 2: Análisis de la Capa de Red con GNS3 y Wireshark
## Objetivos
1. Comprender el funcionamiento de la capa de red (modelo OSI).

2. Observar cómo se enrutan los paquetes entre subredes.

3. Analizar encabezados IP con Wireshark.

4. Verificar tablas de ruteo y funcionamiento del protocolo ICMP.

## Requisitos

1. GNS3 instalado con acceso a imágenes de routers (por ejemplo, Cisco IOSv o routers VPCS).

2. Wireshark instalado.

3. Conocimiento básico de IP, subredes, ping y traceroute.

## Topología del laboratorio
```bash
 PC1 -------- R1 -------- R2 -------- PC2
```
* PC1: 192.168.10.10/24

* R1:

  * Fa0/0: 192.168.10.1/24

  * Fa0/1: 10.0.0.1/30

* R2:

  * Fa0/0: 10.0.0.2/30

  * Fa0/1: 192.168.20.1/24

* PC2: 192.168.20.10/24

## Pasos de configuración
1. Crear la topología en GNS3
* Agregá dos routers (R1 y R2) y dos PCs (podés usar VPCS).

* Conectalos según la topología.

* Asigná direcciones IP y activá las interfaces:

```bash
R1(config)# interface fa0/0
R1(config-if)# ip address 192.168.10.1 255.255.255.0
R1(config-if)# no shutdown

R1(config)# interface fa0/1
R1(config-if)# ip address 10.0.0.1 255.255.255.252
R1(config-if)# no shutdown

R2(config)# interface fa0/0
R2(config-if)# ip address 10.0.0.2 255.255.255.252
R2(config-if)# no shutdown

R2(config)# interface fa0/1
R2(config-if)# ip address 192.168.20.1 255.255.255.0
R2(config-if)# no shutdown
```
2. Configurar las PCs (VPCS)
```bash
PC1> ip 192.168.10.10 255.255.255.0 192.168.10.1
PC2> ip 192.168.20.10 255.255.255.0 192.168.20.1
```
3. Configurar rutas estáticas
```bash
R1(config)# ip route 192.168.20.0 255.255.255.0 10.0.0.2
R2(config)# ip route 192.168.10.0 255.255.255.0 10.0.0.1
```
## Actividades prácticas
### Prueba de conectividad
* Desde PC1:

```bash
PC1> ping 192.168.20.10
```
* Desde PC2:

```bash
PC2> traceroute 192.168.10.10
```
### Analizar con Wireshark
* Iniciá una captura en el enlace entre R1 y R2 (10.0.0.1 <-> 10.0.0.2).

* Ejecutá un ping desde PC1 a PC2.

* Observá en Wireshark:

  * El protocolo ICMP.

  * El encabezado IP: campos como versión, TTL, checksum, etc.

  * Verificá cómo cambia el TTL al pasar por cada router.

### Explorar tabla de ruteo
En R1 y R2:

```bash
show ip route
```

## Preguntas para reflexionar
1. ¿Por qué PC1 necesita una puerta de enlace predeterminada?

2. ¿Qué cambios observás en el TTL de los paquetes?

3. ¿Qué sucede si eliminás la ruta estática en uno de los routers?

4. ¿Qué información contiene el encabezado IP que permite el enrutamiento?