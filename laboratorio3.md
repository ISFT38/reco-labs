# Laboratorio 3: Spanning Tree Protocol (STP) con GNS3 y Wireshark

## Objetivos

1. Comprender el problema de los loops en redes Ethernet.
2. Analizar el funcionamiento del protocolo Spanning Tree Protocol (STP).
3. Identificar Root Bridge, Root Ports, Designated Ports y puertos bloqueados.
4. Capturar y analizar BPDUs utilizando Wireshark.
5. Observar el proceso de convergencia de STP y RSTP.
6. Experimentar fallas y recuperación de enlaces redundantes.

---

# Requisitos

1. GNS3 instalado con soporte para switches Cisco IOSvL2, IOU/IOL o similar.
> https://gns3.com/cisco-iosvl2
2. Wireshark instalado.
3. Conocimiento básico de switching Ethernet y VLANs.
4. Conocimientos básicos sobre comandos Cisco IOS.

---

# Topología del laboratorio

```bash
                +---------+
                |   SW1   |
                +---------+
                 /       \
                /         \
               /           \
      +---------+---------+---------+
      |                   |         |
+---------+         +---------+     |
|   SW2   |---------|   SW3   |-----+
+---------+         +---------+

PC1 ---------------- SW2
PC2 ---------------- SW3
```

---

# Direccionamiento

| Dispositivo | Dirección       |
| ----------- | --------------- |
| PC1         | 192.168.1.10/24 |
| PC2         | 192.168.1.20/24 |

No es necesario configurar gateway.

---

# Parte 1: Crear la topología en GNS3

## Paso 1: Crear un nuevo proyecto

1. Abrí GNS3.
2. Creá un proyecto llamado:

```bash
STP-LAB
```

---

## Paso 2: Agregar dispositivos

Desde la barra lateral agregá:

* 3 switches Cisco IOSvL2 (o equivalente).
* 2 PCs VPCS.

---

## Paso 3: Conectar dispositivos

### Conexiones entre switches

| Dispositivo | Interfaz | Dispositivo | Interfaz |
| ----------- | -------- | ----------- | -------- |
| SW1         | Gi0/0    | SW2         | Gi0/0    |
| SW1         | Gi0/1    | SW3         | Gi0/0    |
| SW2         | Gi0/1    | SW3         | Gi0/1    |

### Conexiones de PCs

| PC  | Switch |
| --- | ------ |
| PC1 | SW2    |
| PC2 | SW3    |

---

# Parte 2: Configuración inicial

## Configuración básica de los switches

En cada switch:

```bash
enable
configure terminal

hostname SW1
no ip domain-lookup

spanning-tree mode pvst

end
write memory
```

En SW2 y SW3 cambiar el hostname correspondiente.

---

## Configuración de las PCs

### PC1

```bash
ip 192.168.1.10/24
```

### PC2

```bash
ip 192.168.1.20/24
```

---

# Parte 3: Verificación de conectividad

## Prueba de conectividad

Desde PC1:

```bash
ping 192.168.1.20
```

La conectividad debería funcionar correctamente.

---

# Parte 4: Analizar STP

## Verificar el estado del spanning tree

En cada switch ejecutar:

```bash
show spanning-tree
```

---

# Actividades de análisis

Identificar:

1. Qué switch fue elegido como Root Bridge.
2. Qué puertos son Root Ports.
3. Qué puertos son Designated Ports.
4. Qué puerto quedó bloqueado.

---

# Observaciones esperadas

Uno de los enlaces redundantes debe quedar bloqueado para evitar loops.

Por ejemplo:

```bash
Role: Alternate
State: Blocking
```

---

# Preguntas para reflexionar

1. ¿Por qué STP bloquea uno de los enlaces?
2. ¿Qué problema ocurriría si ambos enlaces permanecieran activos?
3. ¿Cómo decide STP qué puerto bloquear?

---

# Parte 5: Captura de BPDUs con Wireshark

## Paso 1: Iniciar captura

1. Hacer clic derecho sobre el enlace entre SW1 y SW2.
2. Seleccionar:

   * “Start Capture”.

Wireshark se abrirá automáticamente.

---

## Paso 2: Filtrar tráfico STP

En Wireshark utilizar el filtro:

```bash
stp
```

También puede utilizarse:

```bash
llc
```

---

# Análisis de las BPDUs

Seleccionar un paquete BPDU y observar:

* Root ID.
* Bridge ID.
* Root Path Cost.
* Timers STP.

---

# Actividades

Completar la siguiente tabla:

| Campo          | Valor observado |
| -------------- | --------------- |
| Root Bridge    |                 |
| Sender Bridge  |                 |
| Root Path Cost |                 |
| Hello Time     |                 |

---

# Parte 6: Forzar la elección del Root Bridge

## Configurar SW1 como Root Bridge

En SW1:

```bash
configure terminal
spanning-tree vlan 1 priority 4096
end
```

---

## Verificar cambios

En todos los switches:

```bash
show spanning-tree root
```

---

# Actividades

1. Verificar si cambió el puerto bloqueado.
2. Analizar cómo cambia la topología.
3. Comparar con el estado anterior.

---

# Parte 7: Simulación de falla y convergencia

## Paso 1: Identificar el puerto bloqueado

Ejecutar:

```bash
show spanning-tree
```

---

## Paso 2: Generar una falla

Desconectar uno de los enlaces activos entre switches.

Por ejemplo:

```bash
SW1 <-> SW2
```

---

# Observaciones esperadas

STP debería:

* recalcular la topología.
* desbloquear el enlace redundante.
* restaurar conectividad.

---

# Medición de convergencia

Desde PC1 ejecutar un ping continuo:

```bash
ping 192.168.1.20
```

Observar:

* cuántos paquetes se pierden.
* cuánto tarda la recuperación.

---

# Explicación

En STP clásico la convergencia puede tardar:

```bash
30 a 50 segundos
```

---

# Parte 8: Migrar a RSTP

## Configurar Rapid Spanning Tree

En todos los switches:

```bash
configure terminal
spanning-tree mode rapid-pvst
end
```

---

## Verificar el protocolo activo

```bash
show spanning-tree summary
```

---

# Repetir prueba de falla

1. Desconectar nuevamente un enlace.
2. Observar el tiempo de recuperación.

---

# Comparación esperada

| Protocolo | Tiempo de convergencia |
| --------- | ---------------------- |
| STP       | 30-50 segundos         |
| RSTP      | 1-5 segundos           |

---

# Parte 9: Analizar cambios de topología

## Comando

```bash
show spanning-tree detail
```

---

# Buscar

```bash
Number of topology changes
```

---

# Actividad

1. Desconectar y reconectar enlaces.
2. Verificar cómo aumenta el contador.
3. Relacionar los cambios físicos con los eventos STP.

---

# Parte 10: PortFast y BPDU Guard

## Configurar PortFast

En los puertos conectados a PCs:

```bash
interface range gi0/2 - 3
spanning-tree portfast
```

---

## Configurar BPDU Guard

```bash
spanning-tree bpduguard default
```

---

# Experimento

1. Conectar un switch adicional en un puerto PortFast.
2. Observar el comportamiento del puerto.

---

# Resultado esperado

El puerto debería pasar a estado err-disabled para proteger la red.

---

# Parte 11: Análisis avanzado con Wireshark

## Filtros útiles

### Filtrar BPDUs

```bash
stp
```

### Filtrar MAC multicast STP

```bash
eth.dst == 01:80:c2:00:00:00
```

---

# Preguntas para reflexionar

1. ¿Cada cuánto tiempo se envían las BPDUs?
2. ¿Qué información contienen?
3. ¿Qué diferencias observás entre STP y RSTP?
4. ¿Por qué Ethernet necesita mecanismos de prevención de loops?

---

# Conclusión

* STP permite mantener redundancia física evitando loops de capa 2.
* Los switches intercambian BPDUs para calcular una topología libre de loops.
* RSTP mejora significativamente los tiempos de convergencia.
* Wireshark permite observar el funcionamiento interno del protocolo en tiempo real.

---

# Preguntas finales

1. ¿Qué sucedería si desactivaras STP en una red redundante?
2. ¿Qué ventajas aporta RSTP respecto a STP clásico?
3. ¿Por qué es importante controlar qué switch será el Root Bridge?
4. ¿Qué función cumple un puerto bloqueado?
