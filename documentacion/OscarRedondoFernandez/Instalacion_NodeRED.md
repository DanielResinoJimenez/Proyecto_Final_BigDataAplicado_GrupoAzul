Instalación de Node.js, npm y Node-RED en CentOS 7 (VM9 - ambari9)
1. Introducción
En la máquina virtual ambari9 (VM9) se ha realizado la instalación para habilitar un entorno de integración ligera y dashboards complementarios. Debido a que CentOS 7 utiliza glibc 2.17, se optó por la instalación manual de Node.js 16, que es la última versión compatible con esta librería.
+2

2. Instalación de Node.js 16

Descarga y descompresión: Se descargó el binario oficial en /opt y se descomprimió.
+1


Configuración del PATH: Se añadió el binario al perfil del usuario:

Bash

echo 'export PATH=/opt/node16/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
3. Configuración de npm y Node-RED

npm sin root: Se creó un directorio ~/.npm-global para evitar errores de permisos.


Node-RED: Se instaló la versión 3.1.3 para asegurar compatibilidad con Node 16.
+1


Nodos adicionales: Se instalaron módulos para Kafka, Redis, PostgreSQL y Dashboard.
+3

4. Verificación de integración con Kafka
Se realizó una prueba de extremo a extremo para validar la recepción de mensajes.

4.1 Configuración del Flujo
Nodo kafkajs-consumer: Escucha el tópico sensores_data en el broker 172.16.200.28:9092.

Nodo Function: Convierte el Buffer binario a texto legible:

JavaScript

if (msg.payload && msg.payload.value) {
    msg.payload = msg.payload.value.toString();
    return msg;
}
4.2 Inyección de datos
Desde la terminal de ambari9, se envió un mensaje manual:

Bash

echo "Hola desde Ambari9, probando Kafka" | /usr/bin/kafka-console-producer.sh --broker-list 172.16.200.28:9092 --topic sensores_data
5. Endpoints de la API REST
Se implementaron los siguientes servicios internos:


/api/estado: Devuelve el estado general del servicio.


/api/eventos: Simula una lista de eventos recientes.


/api/health: Calcula el uptime del servicio de forma profesional.
