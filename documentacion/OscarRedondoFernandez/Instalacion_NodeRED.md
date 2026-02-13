🚀 Documentación Técnica: Ecosistema Node-RED en Ambari9
1. Introducción
En la máquina virtual ambari9 (VM9) se ha desplegado Node-RED para habilitar un entorno de integración ligera y dashboards complementarios al ecosistema Big Data. Debido a que CentOS 7 utiliza glibc 2.17, se realizó una instalación manual adaptada para garantizar compatibilidad.
+2

2. Instalación de Node.js 16 (Solución a Incompatibilidades)
CentOS 7 impide instalar Node.js 18+ debido a requisitos de glibc. Por ello, se optó por la versión 16.20.2.
+2

2.1 Proceso de Instalación Manual
Bash

# Descarga del binario oficial
cd /opt
sudo curl -O https://nodejs.org/dist/latest-v16.x/node-v16.20.2-linux-x64.tar.xz

# Descompresión y renombrado
sudo tar -xf node-v16.20.2-linux-x64.tar.xz
sudo mv node-v16.20.2-linux-x64 node16

# Configuración del PATH en ~/.bashrc
echo 'export PATH=/opt/node16/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
3. Configuración de npm y Node-RED
Para evitar errores de permisos (EACCES), se configuró un directorio global propio del usuario.


Directorio Global: Se creó ~/.npm-global y se añadió al PATH.
+2


Instalación de Node-RED: Se forzó la versión 3.1.3 para asegurar estabilidad con Node 16.
+1

Bash

npm install -g --unsafe-perm node-red@3.1.3
4. Integración de Módulos (Nodos Adicionales)
Se instalaron complementos específicos dentro de ~/.node-red para la arquitectura del proyecto:


Kafka: node-red-contrib-kafka-node.


Redis: node-red-contrib-redis.


PostgreSQL: node-red-contrib-postgres.


Dashboard: node-red-dashboard.

5. Validación de Datos: Caso Kafka
Se documenta el éxito en la recepción de datos desde el clúster (IP 172.16.200.28).

5.1 Flujo de Procesamiento
El flujo utiliza un nodo Function para la conversión de datos binarios:

JavaScript

// Conversión de Buffer a String
if (msg.payload && msg.payload.value) {
    msg.payload = msg.payload.value.toString();
    return msg;
}
5.2 Prueba de Inyección Manual
Validación mediante productor de consola en la VM9:

Bash

echo "Hola desde Ambari9, probando Kafka" | /usr/bin/kafka-console-producer.sh --broker-list 172.16.200.28:9092 --topic sensores_data
6. Endpoints REST y Dashboard
Se implementaron servicios para interoperabilidad con otras herramientas (como n8n):


/api/estado: Estado general del host.


/api/eventos: Simulación de actividad NFC.


/api/health: Monitorización de salud y cálculo de uptime.

7. Automatización con Systemd
Para garantizar que Node-RED sea un servicio resiliente, se configuró el archivo /etc/systemd/system/node-red.service con inicio automático tras fallos.
