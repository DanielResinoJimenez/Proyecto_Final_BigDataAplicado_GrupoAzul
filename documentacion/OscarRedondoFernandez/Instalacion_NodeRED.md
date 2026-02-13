Entiendo tu frustración. Al ser un formato largo, a veces el sistema corta el renderizado. Aquí tienes el documento absolutamente completo, de principio a fin, sin cortes y con todo el detalle técnico de tu PDF original más la prueba de Kafka.

Copia desde el primer # hasta el último código de systemd:

🚀 Instalación y Configuración de Node-RED en CentOS 7 (VM9 - ambari9)
1. Introducción
En la máquina virtual ambari9 (VM9) se ha realizado la instalación de Node.js, npm y Node-RED para habilitar un entorno de integración ligera y dashboards complementarios al ecosistema Big Data. Debido a que el sistema operativo es CentOS 7, fue necesario adaptar el proceso debido a las limitaciones de glibc y la incompatibilidad con versiones modernas de Node.js.
+1

2. Problema de Compatibilidad: glibc 2.17
CentOS 7 utiliza glibc 2.17, lo que impide instalar Node.js 18, 20 o superiores. Los intentos de instalación devolvieron errores de dependencias como:


glibc >= 2.28 required.


libstdc++.so.6(GLIBCXX_3.4.21) required.


libm.so.6(GLIBC_2.27) required.

3. Solución: Instalación Manual de Node.js 16
Node.js 16 es la última versión compatible con glibc 2.17, por lo que se instaló mediante binarios oficiales.

3.1 Proceso de Instalación
Bash

# Acceso al directorio y descarga
cd /opt [cite: 21]
sudo curl -O https://nodejs.org/dist/latest-v16.x/node-v16.20.2-linux-x64.tar.xz [cite: 22]

# Descompresión y organización
sudo tar -xf node-v16.20.2-linux-x64.tar.xz [cite: 24]
sudo mv node-v16.20.2-linux-x64 node16 [cite: 24]

# Configuración del PATH en ~/.bashrc
echo 'export PATH=/opt/node16/bin:$PATH' >> ~/.bashrc [cite: 26]
source ~/.bashrc [cite: 26]
4. Configuración de npm (Evitar errores de permisos)
Para evitar errores de tipo EACCES al instalar paquetes globales, se configuró un directorio propio del usuario:


Crear directorio: mkdir ~/.npm-global.


Configurar prefijo: npm config set prefix '~/.npm-global'.


Actualizar PATH: echo 'export PATH=$HOME/.npm-global/bin:$PATH' >> ~/.bashrc.

5. Instalación de Node-RED
Se instaló la versión 3.1.3, seleccionada por ser totalmente compatible con Node 16.

Bash

npm install -g --unsafe-perm node-red@3.1.3 [cite: 49]
5.1 Nodos y Módulos Adicionales
Se instalaron los siguientes módulos dentro de ~/.node-red:


Kafka: node-red-contrib-kafkajs (validado con el clúster en 172.16.200.28).


Redis: node-red-contrib-redis.


PostgreSQL: node-red-contrib-postgres.


Dashboard: node-red-dashboard.

6. Validación de Integración con Kafka
Se realizó una prueba de flujo de datos para validar la recepción de mensajes desde el clúster en la IP 172.16.200.28:9092.

6.1 Lógica de Procesamiento (Buffer a Texto)
Se empleó un nodo Function para convertir los datos binarios de Kafka en texto legible:

JavaScript

if (msg.payload && msg.payload.value) {
    msg.payload = msg.payload.value.toString();
    return msg;
}
6.2 Inyección Manual de Datos
Se validó el circuito enviando un mensaje desde la terminal de la VM9 (ambari9):

Bash

echo "Hola desde Ambari9, probando Kafka" | /usr/bin/kafka-console-producer.sh --broker-list 172.16.200.28:9092 --topic sensores_data
Resultado: El mensaje se visualizó correctamente en el panel de debug de Node-RED.

7. APIs REST y Dashboard
Se implementaron servicios internos para interoperabilidad:
+1


/api/estado: Reporta el host, timestamp y estado del servicio.
+1


/api/eventos: Simula una lista de eventos NFC recientes (equipos PC-01, PC-02, PC-03).
+1


/api/health: Monitorización de salud mediante el cálculo de uptime.
+1

8. Automatización con Systemd
Se creó un servicio para garantizar el arranque automático tras reinicios:

Ini, TOML

[Unit]
Description=Node-RED [cite: 85]
After=network.target [cite: 86]

[Service]
Type=simple [cite: 88]
User=hadoop [cite: 89]
Group=hadoop [cite: 90]
ExecStart=/home/hadoop/.npm-global/bin/node-red [cite: 92]
Restart=on-failure [cite: 92]

[Install]
WantedBy=multi-user.target [cite: 94]
