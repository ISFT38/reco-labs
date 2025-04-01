### **Laboratorio 1: Introducción a Redes y Herramientas**

#### **Objetivos**
1. Familiarizarse con GNS3 y Wireshark.
2. Comprender la estructura de una red básica.
3. Identificar las capas del modelo OSI en los paquetes capturados.

#### **Parte 1: Configuración del Entorno**

##### **Instalación de GNS3 y Wireshark**

###### **Para Windows:**
1. Descarga GNS3 desde su [sitio oficial](https://gns3.com/).
2. Sigue el asistente de instalación y asegúrate de incluir las opciones para instalar Wireshark.
3. Una vez instalado, abre GNS3 y verifica que funcione correctamente.

###### **Para Linux (Ubuntu/Debian):**
1. Sigue las instrucciones del [sitio oficial](https://docs.gns3.com/docs/getting-started/installation/linux/)
   Si da error en paquete dynamips, agregar non-free a sources.list
   Si no hay pcs virtuales descargar la última versión desde [github](https://github.com/GNS3/vpcs/releases) e instalar con
   ```bash
   tar zxf vpcs-x.x.x.tar.gz
   cd vpcs-x.x.x/src
   ./mk.sh
   sudo cp vpcs /usr/bin/
   ```
   Si un error dice que no existe uBridge descargar el código fuente desde [github](https://github.com/GNS3/ubridge) e instalar con
   ```bash
   make
   sudo make install
   ```

2. Durante la instalación de Wireshark, selecciona "Yes" cuando pregunte si los usuarios no root pueden capturar paquetes.
3. Agrega tu usuario al grupo `wireshark` (requiere reiniciar la sesión):
   ```bash
   sudo usermod -aG wireshark $USER
   ```
4. Verifica que GNS3 y Wireshark estén instalados correctamente.

######  **Imágenes de routers:**
Desde [este sitio](https://www.telectronika.com/descargas/cisco-imagenes-ios-para-gns3-dynamips-y-vm/) se pueden descargar las imágenes del sistema operativo de algunos routers cisco para simularlos dentro de GNS3.

##### **Prueba de las Herramientas**
1. Abre GNS3 y crea un nuevo proyecto.
2. Abre Wireshark y selecciona una interfaz de red para comenzar a capturar paquetes (asegúrate de detener la captura antes de cerrar).

---

#### **Parte 2: Crear una Red Básica en GNS3**

##### **Paso 1: Diseñar la Red**
1. Abre GNS3 y crea un nuevo proyecto llamado "Laboratorio1".
2. Desde la barra lateral, agrega los siguientes dispositivos:
   - Un router (p. ej., `Cisco c7200`).
   - Una PC (usa una máquina virtual o el nodo `VPCS`).
3. Conecta la PC al router usando un cable Ethernet:
   - Haz clic en el ícono de "Cable".
   - Selecciona "Ethernet" y conecta la PC al router.

##### **Paso 2: Configurar Direcciones IP**
1. Haz clic derecho sobre la PC y selecciona "Configuración" o abre la consola de `VPCS`.
2. Asigna una dirección IP:
   ```bash
   ip 192.168.1.2 255.255.255.0 192.168.1.1
   ```
   Donde:
   - `192.168.1.2` es la IP de la PC.
   - `255.255.255.0` es la máscara de subred.
   - `192.168.1.1` es la puerta de enlace (el router).
3. Haz clic derecho sobre el router y abre la consola para configurarlo:
   ```bash
   enable
   configure terminal
   interface FastEthernet0/0
   ip address 192.168.1.1 255.255.255.0
   no shutdown
   exit
   ```
4. Verifica la conectividad desde la PC:
   ```bash
   ping 192.168.1.1
   ```

---

#### **Parte 3: Captura de Tráfico con Wireshark**

##### **Paso 1: Iniciar la Captura**
1. Abre Wireshark y selecciona la interfaz de red correspondiente (en GNS3, el adaptador virtual asociado a tu proyecto).
2. Haz clic en el botón de "Iniciar Captura".

##### **Paso 2: Generar Tráfico**
1. Desde la consola de la PC, realiza un ping al router:
   ```bash
   ping 192.168.1.1
   ```
2. Observa los paquetes ICMP en Wireshark.

##### **Paso 3: Analizar los Paquetes**
1. Detén la captura en Wireshark.
2. Filtra los paquetes ICMP escribiendo `icmp` en la barra de búsqueda.
3. Selecciona un paquete y analiza:
   - La capa de red (IP).
   - La capa de enlace de datos (Ethernet).
   - Los datos encapsulados en el paquete.

---

#### **Conclusión**
- Este laboratorio introduce la estructura básica de una red y permite observar cómo las capas del modelo OSI interactúan.
- Las herramientas GNS3 y Wireshark son fundamentales para diseñar y analizar redes, y serán usadas en los laboratorios siguientes.

#### **Preguntas para Reflexión**
1. ¿Qué capas del modelo OSI se pueden observar en un paquete ICMP?
2. ¿Cuáles son las ventajas de usar herramientas como Wireshark para analizar redes?
3. ¿Qué sucedería si configuraras una dirección IP en una subred diferente?

