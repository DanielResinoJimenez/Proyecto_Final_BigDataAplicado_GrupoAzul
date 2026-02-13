📄 Instalación de Node.js, npm y Node-RED en CentOS 7 (VM9 – ambari9)

Autor: Óscar Redondo Fernández
Módulo: Big Data Aplicado
Entorno: VM9 – ambari9 (CentOS 7)

1. Introducción

En la máquina virtual ambari9 (VM9) se realizó la instalación de Node.js, npm y Node-RED para habilitar un entorno de integración ligera y dashboards complementarios al ecosistema Big Data.

Debido a que el sistema operativo es CentOS 7, fue necesario adaptar el proceso por limitaciones de glibc y la incompatibilidad con versiones modernas de Node.js.

2. Problema inicial: incompatibilidad de Node.js moderno con CentOS 7

CentOS 7 utiliza:

glibc 2.17


Esto impide instalar Node.js 18, 20 o superiores desde NodeSource.

Errores obtenidos:

glibc >= 2.28 required
libstdc++.so.6(GLIBCXX_3.4.21) required
libm.so.6(GLIBC_2.27) required


Por tanto, se descartó instalar versiones modernas desde repositorios.

3. Solución adoptada: instalación manual de Node.js 16

Node.js 16 es la última versión compatible con glibc 2.17.

3.1 Descarga del binario
cd /opt
sudo curl -O https://nodejs.org/dist/latest-v16.x/node-v16.20.2-linux-x64.tar.xz

3.2 Descompresión
sudo tar -xf node-v16.20.2-linux-x64.tar.xz
sudo mv node-v16.20.2-linux-x64 node16

3.3 Añadir Node.js al PATH
echo 'export PATH=/opt/node16/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

3.4 Verificación
node -v
npm -v


Resultado esperado:

v16.20.2
8.19.4

4. Configuración de npm para evitar errores de permisos

Por defecto, npm intentaba instalar paquetes globales en /opt/node16, generando errores EACCES.

4.1 Crear directorio npm-global
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'

4.2 Añadirlo al PATH
echo 'export PATH=$HOME/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

5. Instalación de Node-RED compatible con Node.js 16

Node-RED 4.x requiere Node ≥ 18.5.
Se instaló Node-RED 3.1.3, compatible con Node 16.

5.1 Instalación
npm install -g --unsafe-perm node-red@3.1.3

5.2 Verificación
which node-red


Resultado esperado:

/home/hadoop/.npm-global/bin/node-red

6. Instalación de nodos adicionales para integraciones

Instalados dentro de ~/.node-red:

6.1 Kafka
npm install node-red-contrib-kafka-node

6.2 Redis
npm install node-red-contrib-redis

6.3 PostgreSQL
npm install node-red-contrib-postgres

6.4 Dashboard
npm install node-red-dashboard


Los avisos "deprecated" son normales en CentOS 7 y no afectan al funcionamiento.

7. Arranque de Node-RED
node-red


Acceso desde navegador:

http://ambari9.localdomain:1880
http://172.16.200.9:1880

8. (Opcional) Creación de servicio systemd

Crear el archivo:

sudo nano /etc/systemd/system/node-red.service


Contenido:

[Unit]
Description=Node-RED
After=network.target

[Service]
Type=simple
User=hadoop
Group=hadoop
Environment="PATH=/home/hadoop/.npm-global/bin:/opt/node16/bin:/usr/bin:/bin"
ExecStart=/home/hadoop/.npm-global/bin/node-red
Restart=on-failure

[Install]
WantedBy=multi-user.target


Activación:

sudo systemctl daemon-reload
sudo systemctl enable node-red
sudo systemctl start node-red

9. Conclusión

La instalación se completó con éxito mediante:

Instalación manual de Node.js 16

Configuración personalizada de npm

Instalación de Node-RED 3.1.3

Integración con Kafka, Redis, PostgreSQL y Dashboard

10. Configuración de Dashboard, simuladores y APIs REST en Node-RED

Tras la instalación, se configuró:

Dashboard interactivo

Simulación de datos

Endpoints REST internos

10.1 Creación del Dashboard en Node-RED

Acceso:

http://ambari9.localdomain:1880/ui
http://172.16.200.9:1880/ui

Estructura creada

Tab: Monitorización NFC

Group: Estado general

10.1.1 Indicador “Equipos dentro” (ui_gauge)
msg.payload = Math.floor(Math.random() * 30) + 1;
return msg;

10.1.2 Gráfico de actividad NFC (ui_chart)
msg.payload = Math.floor(Math.random() * 10);
return msg;

10.1.3 Reloj en tiempo real (ui_text)
msg.payload = new Date().toLocaleTimeString();
return msg;

10.2 Creación de APIs REST internas

Estructura común:

[http in] → [function] → [http response]

10.2.1 Endpoint /api/estado
msg.payload = {
    estado: "activo",
    host: "ambari9",
    timestamp: Date.now(),
    mensaje: "Node-RED operativo y escuchando"
};
return msg;

10.2.2 Endpoint /api/eventos
msg.payload = [
    { equipo: "PC-01", accion: "entrada", timestamp: Date.now() - 5000 },
    { equipo: "PC-02", accion: "salida", timestamp: Date.now() - 12000 },
    { equipo: "PC-03", accion: "entrada", timestamp: Date.now() - 30000 }
];
return msg;

10.2.3 Endpoint /api/health

Nodo Inject (al inicio):

flow.set("inicio", Date.now());
return msg;


Endpoint:

const inicio = flow.get("inicio") || Date.now();
const uptime = Math.floor((Date.now() - inicio) / 1000);

msg.payload = {
    status: "ok",
    host: "ambari9",
    uptime_segundos: uptime,
    timestamp: Date.now()
};

return msg;

10.3 Resultado final

La VM9 dispone de:

Dashboard operativo con datos en tiempo real

3 endpoints REST funcionales

Entorno estable listo para producción

10.4 Verificación de integración con Kafka
10.4.1 Configuración del flujo

Nodo kafkajs-consumer

Broker: 172.16.200.28:9092

Topic: sensores_data

Conversión binario → texto (Function):

msg.payload = msg.payload.toString();
return msg;

10.4.2 Inyección manual de datos
echo "Hola desde Ambari9, probando Kafka" | \
/usr/bin/kafka-console-producer.sh \
--broker-list 172.16.200.28:9092 \
--topic sensores_data


Resultado correcto en debug:

Hola desde Ambari9, probando Kafka

10.4.3 Diagnóstico de red

Advertencias detectadas:

Fallo de conexión a 172.16.200.10:9092

Conclusión:

La comunicación básica funciona.
Para producción, la VM9 debe tener visibilidad completa sobre todos los nodos del clúster.
