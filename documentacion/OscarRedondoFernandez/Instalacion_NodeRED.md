Instalación de Node.js, npm y Node-RED en CentOS 7 (VM9 - ambari9)

Autor: Óscar Redondo Fernández 




Módulo: Big Data Aplicado (IABD) 


1. Introducción
En la máquina virtual ambari9 (VM9) se ha realizado la instalación de Node.js, npm y Node-RED para habilitar un entorno de integración ligera y dashboards complementarios al ecosistema Big Data. Debido a que el sistema operativo es CentOS 7, se adaptó el proceso para superar limitaciones de compatibilidad con versiones modernas.


2. Problema inicial: Incompatibilidad con CentOS 7
CentOS 7 utiliza glibc 2.17, lo que impide la instalación de Node.js 18, 20 o superiores. Intentar usar versiones modernas devuelve errores de dependencias como:


glibc >= 2.28 required 


libstdc++.so.6(GLIBCXX_3.4.21) required 

3. Solución adoptada: Instalación manual de Node.js 16
Node.js 16 es la última versión compatible con glibc 2.17. Se procedió con la instalación mediante binarios oficiales:

3.1 Descarga y descompresión
Bash

cd /opt
sudo curl -O https://nodejs.org/dist/latest-v16.x/node-v16.20.2-linux-x64.tar.xz
sudo tar -xf node-v16.20.2-linux-x64.tar.xz
sudo mv node-v16.20.2-linux-x64 node16



3.2 Configuración del PATH
Bash

echo 'export PATH=/opt/node16/bin:$PATH' >> ~/.bashrc
source ~/.bashrc


3.3 Verificación

Node: node -v (Resultado esperado: v16.20.2) 



npm: npm -v (Resultado esperado: 8.19.4) 


4. Configuración de npm (Permisos)
Para evitar errores EACCES al instalar paquetes globales en /opt/node16, se configuró un directorio propio para el usuario:

Bash

mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=$HOME/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc



5. Instalación de Node-RED
Dado que Node-RED 4.x requiere Node ≥ 18.5, se instaló la versión 3.1.3, compatible con Node 16.

Bash

npm install -g --unsafe-perm node-red@3.1.3


6. Instalación de nodos adicionales
Se instalaron los siguientes módulos dentro de ~/.node-red para permitir integraciones:


Kafka: npm install node-red-contrib-kafka-node 


Redis: npm install node-red-contrib-redis 


PostgreSQL: npm install node-red-contrib-postgres 


Dashboard: npm install node-red-dashboard 

7. Ejecución y Acceso
Node-RED se puede iniciar manualmente con el comando node-red.


Editor: http://ambari9.localdomain:1880 


Dashboard: http://ambari9.localdomain:1880/ui 

8. Configuración de APIs REST Internas
Se implementaron varios endpoints siguiendo la estructura [http in] → [function] → [http response]:

8.1 Endpoint /api/estado
Devuelve el estado general del servicio:

JavaScript

msg.payload = {
    estado: "activo",
    host: "ambari9",
    timestamp: Date.now(),
    mensaje: "Node-RED operativo y escuchando"
};
return msg;



8.2 Endpoint /api/health (Cálculo de Uptime)
Utiliza una variable de flujo para calcular el tiempo de actividad:


JavaScript

const inicio = flow.get("inicio") || Date.now();
const uptime = Math.floor((Date.now() - inicio) / 1000);
msg.payload = {
    status: "ok",
    host: "ambari9",
    uptime_segundos: uptime,
    timestamp: Date.now()
};
return msg;



9. Conclusión
La VM9 dispone ahora de un entorno estable de Node-RED 3.1.3, con dashboards funcionales y APIs REST operativas, listo para la integración final con el clúster (Kafka, Redis y PostgreSQL).
