# Laboratorio práctico: introducción a Apache Kafka desde consola



Nivel inicial – intermedio

Duración estimada: 90 minutos



## 1. Objetivos

Al finalizar el laboratorio, los alumnos deberían poder:

- Comprender los conceptos de broker, topic, producer y consumer.
- Levantar una instancia de Kafka.
- Crear y administrar topics desde la consola.
- Publicar mensajes utilizando productores de consola.
- Consumir mensajes utilizando consumidores de consola.
- Comprender qué son las particiones y los offsets.
- Experimentar con grupos de consumidores.
- Observar qué sucede cuando un consumidor se desconecta y vuelve a conectarse.

La idea es que todo el laboratorio se realice mediante comandos, sin escribir código Java, Python ni ninguna otra aplicación.

## 2. Arquitectura del laboratorio

Vamos a utilizar una única instancia de Kafka en modo KRaft (sin ZooKeeper), para simplificar la infraestructura.

Producer

kafka-console-producer.sh

Publicación de mensajes

Apache Kafka – Broker

Topic pedidos Partición 0

Topic pedidos Partición 1

Consumo de mensajes

Consumer A Grupo de consumo

Consumer B Grupo de consumo

Utilizaremos Docker para no tener que instalar Kafka directamente en las computadoras de los alumnos.

## 3. Preparación del entorno

### Paso 1. Levantar Kafka

Requisitos:

- Docker instalado.
- Docker Compose no es necesario.
- Conexión a Internet para descargar la imagen.

Abrir una terminal y ejecutar:

```
docker run -d \
  --name kafka \
  -p 9092:9092 \
  apache/kafka:4.1.1
```

Esto levanta un broker Kafka utilizando KRaft.

Verificar que esté funcionando:

```
docker ps
```

Y consultar la API del broker:

```
docker exec -it kafka \
  /opt/kafka/bin/kafka-broker-api-versions.sh \
  --bootstrap-server localhost:9092
```

Si devuelve información sobre las versiones de API soportadas, el broker está disponible.

Importante: vamos a ejecutar las herramientas de Kafka dentro del contenedor. No necesitamos instalarlas en el sistema operativo.

### Paso 2. Conocer las herramientas disponibles

Kafka incluye varias herramientas de administración y operación.

| Herramienta                 | Función                               |
| --------------------------- | ------------------------------------- |
| `kafka-topics.sh`           | Crear, eliminar y consultar topics    |
| `kafka-console-producer.sh` | Publicar mensajes                     |
| `kafka-console-consumer.sh` | Consumir mensajes                     |
| `kafka-consumer-groups.sh`  | Administrar grupos de consumidores    |
| `kafka-configs.sh`          | Consultar y modificar configuraciones |

Para simplificar los comandos, podemos abrir una terminal interactiva dentro del contenedor:

```
docker exec -it kafka bash
```

A partir de ahora, los comandos se ejecutan dentro del contenedor y las herramientas están en `/opt/kafka/bin/`.

## 4. Creación y administración de topics

### Paso 3. Crear nuestro primer topic

Vamos a trabajar con un sistema ficticio de gestión de pedidos.

Crear un topic llamado `pedidos`:

```
/opt/kafka/bin/kafka-topics.sh \
  --bootstrap-server localhost:9092 \
  --create \
  --topic pedidos \
  --partitions 2 \
  --replication-factor 1
```

¿Qué significan estos parámetros?

- `bootstrap-server`: dirección del broker.
- `create`: indica que queremos crear un topic.
- `topic`: nombre del topic.
- `partitions`: cantidad de particiones.
- `replication-factor`: cantidad de réplicas de cada partición.

En nuestro caso tenemos dos particiones y una sola réplica, porque utilizamos un único broker.

### Paso 4. Consultar los topics existentes

```
/opt/kafka/bin/kafka-topics.sh \
  --bootstrap-server localhost:9092 \
  --list
```

Deberíamos obtener:

```
pedidos
```

### Paso 5. Consultar los detalles del topic

```
/opt/kafka/bin/kafka-topics.sh \
  --bootstrap-server localhost:9092 \
  --describe \
  --topic pedidos
```

Observar especialmente:

- Número de particiones.
- Líder de cada partición.
- Réplicas asignadas.
- Réplicas sincronizadas (ISR).

Actividad 1: crear un segundo topic llamado `pagos`, con una única partición y factor de replicación 1. Consultar sus características.

## 5. Publicación de mensajes

### Paso 6. Iniciar un productor

Abrir una nueva terminal (sin cerrar la anterior) y ejecutar:

```
docker exec -it kafka \
  /opt/kafka/bin/kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic pedidos
```

La terminal queda esperando que ingresemos mensajes.

Escribir, uno por línea:

```
Pedido 1001 - Cliente Juan
Pedido 1002 - Cliente Maria
Pedido 1003 - Cliente Pedro
Pedido 1004 - Cliente Ana
```

Cada vez que presionamos Enter, Kafka recibe un nuevo mensaje.

Concepto importante

El productor no está enviando mensajes directamente a un consumidor. Está publicándolos en un topic de Kafka.

Kafka almacena los mensajes y los consumidores pueden recuperarlos posteriormente.

### Paso 7. Publicar mensajes con claves

Los mensajes pueden tener una clave (key) y un valor (value).

Esto es importante porque Kafka utiliza la clave para determinar la partición a la que enviará el mensaje (cuando se utiliza el particionador por defecto).

Abrir otro productor:

```
docker exec -it kafka \
  /opt/kafka/bin/kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic pedidos \
  --property parse.key=true \
  --property key.separator=:
```

Ahora ingresar:

```
cliente-1:Pedido 1005
cliente-2:Pedido 1006
cliente-1:Pedido 1007
cliente-3:Pedido 1008
```

La parte anterior a los dos puntos es la clave y la posterior es el valor.

Por ejemplo:

- Key: `cliente-1`
- Value: `Pedido 1005`

Los mensajes con la misma clave se asignan a la misma partición mediante el particionador predeterminado, lo que permite conservar su orden relativo.

## 6. Consumo de mensajes

### Paso 8. Iniciar un consumidor

Abrir una tercera terminal:

```
docker exec -it kafka \
  /opt/kafka/bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic pedidos
```

¿Qué sucede?

El consumidor se queda esperando nuevos mensajes. Si ahora volvemos al productor y publicamos:

```
Pedido 1009 - Cliente Luis
```

El consumidor debería recibirlo.

Pero hay algo interesante: no estamos viendo los mensajes que publicamos antes de iniciar el consumidor.

Por defecto, el consumidor comienza a leer desde el final del log cuando no tiene un offset previamente guardado.

### Paso 9. Leer todos los mensajes desde el principio

Detener el consumidor con `Ctrl+C`.

Ejecutar:

```
docker exec -it kafka \
  /opt/kafka/bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic pedidos \
  --from-beginning
```

Ahora deberían aparecer también los mensajes publicados anteriormente.

Para visualizar las claves, particiones y offsets, utilizar:

```
docker exec -it kafka \
  /opt/kafka/bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic pedidos \
  --from-beginning \
  --property print.key=true \
  --property print.partition=true \
  --property print.offset=true
```

La salida será similar a:

```
null    0    0    Pedido 1001 - Cliente Juan
null    1    0    Pedido 1002 - Cliente Maria
cliente-1    0    4    Pedido 1005
cliente-2    1    3    Pedido 1006
```

Los valores concretos de partición y offset dependerán de la asignación de mensajes y de las publicaciones realizadas.

Conceptos que deben identificar:

- Partition: partición donde se almacenó el mensaje.
- Offset: posición del mensaje dentro de esa partición.
- Key: identificador utilizado para determinar la partición.
- Value: contenido del mensaje.

Un detalle importante: los offsets son independientes para cada partición.

## 7. Grupos de consumidores (Consumer Groups)

Este es uno de los conceptos más importantes de Kafka.

Un grupo de consumidores permite distribuir el procesamiento de los mensajes entre varias instancias de una aplicación.

### Paso 10. Crear un primer grupo de consumidores

Vamos a abrir dos terminales y ejecutar en ambas el siguiente comando:

Terminal A:

```
docker exec -it kafka \
  /opt/kafka/bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic pedidos \
  --group grupo-pedidos
```

Terminal B:

```
docker exec -it kafka \
  /opt/kafka/bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic pedidos \
  --group grupo-pedidos
```

Ahora publicar desde el productor varios mensajes nuevos:

```
Pedido 1010
Pedido 1011
Pedido 1012
Pedido 1013
Pedido 1014
Pedido 1015
Pedido 1016
Pedido 1017
```

Observar qué sucede:

- Los mensajes se distribuyen entre ambos consumidores.
- Un mensaje es recibido por uno de los consumidores del grupo, no por ambos.
- Kafka asigna las particiones a los consumidores.

Como el topic tiene dos particiones y hay dos consumidores, cada consumidor puede recibir una partición.

Distribución de trabajo

Partición 0

Consumidor A

Partición 1

Consumidor B

Representación posible de la asignación. Kafka puede realizar otra asignación según el estado del grupo.

### Paso 11. Consultar los grupos de consumidores

Ejecutar:

```
docker exec -it kafka \
  /opt/kafka/bin/kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --list
```

Deberíamos encontrar:

```
grupo-pedidos
```

Ahora consultar el estado del grupo:

```
docker exec -it kafka \
  /opt/kafka/bin/kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --describe \
  --group grupo-pedidos
```

Entre otros datos, veremos:

| Campo          | Significado                     |
| -------------- | ------------------------------- |
| GROUP          | Nombre del grupo                |
| TOPIC          | Topic consumido                 |
| PARTITION      | Partición asignada              |
| CURRENT-OFFSET | Último offset confirmado        |
| LOG-END-OFFSET | Último offset disponible        |
| LAG            | Mensajes pendientes de consumir |

El lag representa la diferencia entre los mensajes disponibles y los offsets confirmados por el grupo.

### Paso 12. Experimentar con la cantidad de consumidores

Sin detener los dos consumidores, abrir una tercera terminal y ejecutar:

```
docker exec -it kafka \
  /opt/kafka/bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic pedidos \
  --group grupo-pedidos
```

Publicar nuevos mensajes.

Observar qué sucede.

Como tenemos dos particiones y tres consumidores, uno de los consumidores quedará sin particiones asignadas.

Conclusión: agregar consumidores no siempre incrementa la capacidad de procesamiento. La cantidad de particiones limita el paralelismo dentro de un grupo.

### Paso 13. Comparar dos grupos diferentes

Abrir otra terminal y ejecutar:

```
docker exec -it kafka \
  /opt/kafka/bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic pedidos \
  --group grupo-auditoria \
  --from-beginning
```

Publicar un mensaje nuevo desde el productor.

Observar que:

- Los consumidores del grupo `grupo-pedidos` distribuyen el trabajo.
- El consumidor del grupo `grupo-auditoria` también recibe el mensaje.
- Cada grupo mantiene sus propios offsets.

Esto permite que diferentes aplicaciones consuman los mismos eventos independientemente.

Por ejemplo:

- `grupo-pedidos`: procesa los pedidos.
- `grupo-auditoria`: registra las operaciones.
- `grupo-estadisticas`: genera estadísticas de ventas.

## 8. Experimentación con offsets

### Paso 14. Observar el avance de un grupo

Ejecutar:

```
docker exec -it kafka \
  /opt/kafka/bin/kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --describe \
  --group grupo-pedidos
```

Identificar los offsets de cada partición.

Ahora detener todos los consumidores del grupo `grupo-pedidos` con `Ctrl+C`.

Consultar nuevamente el grupo.

Los offsets confirmados deberían permanecer almacenados.

Concepto: Kafka no elimina automáticamente los mensajes cuando un consumidor los lee. El offset indica hasta dónde avanzó el grupo.

### Paso 15. Leer nuevamente los mensajes desde el principio

Podemos modificar los offsets de un grupo para volver a procesar mensajes.

Primero, verificar que todos los consumidores de `grupo-pedidos` estén detenidos.

Ejecutar:

```
docker exec -it kafka \
  /opt/kafka/bin/kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group grupo-pedidos \
  --topic pedidos \
  --reset-offsets \
  --to-earliest \
  --execute
```

Esto posiciona los offsets del grupo al comienzo de cada partición.

Volver a iniciar un consumidor:

```
docker exec -it kafka \
  /opt/kafka/bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic pedidos \
  --group grupo-pedidos
```

Deberían volver a aparecer los mensajes históricos.

## 9. Actividades de integración


| N.º | Consigna                                                             | Conceptos involucrados     |
| --- | -------------------------------------------------------------------- | -------------------------- |
| 1   | Crear un topic `clientes` con 3 particiones.                         | Topics y particiones       |
| 2   | Publicar 10 mensajes en `clientes`.                                  | Productores                |
| 3   | Consumir los mensajes desde el principio.                            | Consumidores y offsets     |
| 4   | Publicar mensajes utilizando como clave el DNI del cliente.          | Keys y particionamiento    |
| 5   | Crear dos consumidores pertenecientes al mismo grupo.                | Consumer Groups            |
| 6   | Crear un consumidor de otro grupo y observar los mensajes recibidos. | Independencia entre grupos |
| 7   | Agregar un tercer consumidor al primer grupo.                        | Asignación de particiones  |
| 8   | Detener un consumidor y observar qué ocurre con las particiones.     | Rebalanceo                 |
| 9   | Consultar el lag de un grupo.                                        | Monitoreo                  |
| 10  | Reiniciar los offsets para reprocesar los mensajes.                  | Gestión de offsets         |

## 10. Cierre del laboratorio

### Paso 16. Eliminar el entorno

Cuando terminen, detener y eliminar el contenedor:

```
docker stop kafka
docker rm kafka
```

Como no configuramos un volumen persistente, los datos se perderán al eliminar el contenedor.

## 11. Preguntas de comprensión

1. ¿Cuál es la diferencia entre un topic y una partición?
2. ¿Qué sucede si dos consumidores pertenecen al mismo grupo?
3. ¿Qué sucede si pertenecen a grupos diferentes?
4. ¿Puede haber más consumidores que particiones? ¿Qué ocurre?
5. ¿Qué función cumple el offset?
6. ¿Kafka elimina un mensaje cuando un consumidor lo lee?
7. ¿Qué sucede si un consumidor se desconecta inesperadamente?
8. ¿Por qué utilizaríamos una clave al publicar un mensaje?
9. ¿Para qué podría servir que tres aplicaciones diferentes consuman el mismo topic?
10. ¿Qué ventajas ofrece Kafka frente a una comunicación directa entre dos aplicaciones?

## 12. Conceptos que deberían quedar claros

Resumen conceptual

| Concepto       | Idea fundamental                                    |
| -------------- | --------------------------------------------------- |
| Broker         | Servidor que almacena y distribuye mensajes         |
| Topic          | Categoría o canal lógico de mensajes                |
| Partition      | División de un topic que permite paralelismo        |
| Producer       | Aplicación que publica mensajes                     |
| Consumer       | Aplicación que recibe mensajes                      |
| Consumer Group | Conjunto de consumidores que distribuyen el trabajo |
| Offset         | Posición de un mensaje dentro de una partición      |
| Consumer Lag   | Diferencia entre lo disponible y lo confirmado      |
| Key            | Clave utilizada para determinar la partición        |
