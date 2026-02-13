Memoria Técnica: Ecosistema Node-RED en CentOS 7 (VM9)1. IntroducciónEn la máquina virtual ambari9 se ha desplegado un entorno basado en Node-RED para la integración de servicios Big Data y la creación de dashboards operativos. Debido a las limitaciones de glibc 2.17 en CentOS 7, se implementó una arquitectura basada en Node.js 16 para garantizar la compatibilidad.+42. Preparación del Entorno (Node.js y npm)2.1 Solución a IncompatibilidadesLas versiones modernas de Node.js (v18+) requieren glibc >= 2.28, incompatible con CentOS 7. Se instaló manualmente la versión 16.20.2.+3Comandos de instalación:Bash# Descarga y descompresión en /opt
cd /opt
sudo curl -O https://nodejs.org/dist/latest-v16.x/node-v16.20.2-linux-x64.tar.xz
sudo tar -xf node-v16.20.2-linux-x64.tar.xz
sudo mv node-v16.20.2-linux-x64 node16

# Configuración de variables de entorno
echo 'export PATH=/opt/node16/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
2.2 Gestión de Permisos de npmPara evitar errores de acceso (EACCES), se redirigió el prefijo global al directorio del usuario hadoop.Directorio: ~/.npm-global.PATH: Se incluyó $HOME/.npm-global/bin en el archivo .bashrc.3. Despliegue de Node-REDSe instaló la versión 3.1.3, optimizada para el motor de Node 16.+13.1 Módulos Adicionales InstaladosServicioMódulo npmPropósitoKafkanode-red-contrib-kafkajsIngesta de mensajes del clúster.Redisnode-red-contrib-redisAlmacenamiento en caché rápida.PostgreSQLnode-red-contrib-postgresPersistencia de datos relacionales.Dashboardnode-red-dashboardInterfaz gráfica de usuario (UI).4. Integración Crítica: Kafka (Caso de Éxito)Se validó la conectividad con el broker externo en la IP 172.16.200.28.4.1 Flujo de Recepción de DatosEl flujo se diseñó para transformar mensajes binarios en información legible:Consumidor: Conectado al tópico sensores_data.Transformación: Nodo Function con el siguiente script:JavaScriptif (msg.payload && msg.payload.value) {
    msg.payload = msg.payload.value.toString();
    return msg;
}
Salida: Visualización en el panel de Debug.4.2 Prueba de Estrés y ValidaciónSe utilizó un productor de consola para inyectar datos manualmente:Bashecho "Hola desde Ambari9, probando Kafka" | /usr/bin/kafka-console-producer.sh --broker-list 172.16.200.28:9092 --topic sensores_data
Resultado: Recepción inmediata del string en el Dashboard de Node-RED.5. Desarrollo de APIs y Dashboard5.1 Endpoints REST Implementados/api/estado: Reporta el host (ambari9) y timestamp actual.+1/api/eventos: Simula una lista de actividad de equipos (ej. PC-01, PC-02).+1/api/health: Calcula el uptime del servicio mediante variables de flujo (flow.get("inicio")).+15.2 Elementos Visuales (Dashboard)Gauge: Indicador de "Equipos dentro" mediante valores aleatorios (Math.random).+1Chart: Gráfico de actividad histórica de sensores.Text: Reloj en tiempo real sincronizado con el sistema.+16. Configuración de Autoconsumo (Systemd)Para garantizar la disponibilidad, Node-RED se configuró como servicio del sistema:Ini, TOML[Unit]
Description=Node-RED
After=network.target

[Service]
Type=simple
User=hadoop
Group=hadoop
ExecStart=/home/hadoop/.npm-global/bin/node-red
Restart=on-failure

[Install]
WantedBy=multi-user.target
