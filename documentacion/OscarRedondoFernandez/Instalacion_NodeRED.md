# 🚀 Instalación y Configuración de Node-RED en CentOS 7 (VM9 - ambari9)

![status](https://img.shields.io/badge/status-operational-brightgreen)
![centos](https://img.shields.io/badge/CentOS-7-blue)
![node](https://img.shields.io/badge/Node.js-16.x-green)
![nodered](https://img.shields.io/badge/Node--RED-3.1.3-red)

---

## 📑 Tabla de Contenidos
- [1. Introducción](#1-introducción)
- [2. Problema de Compatibilidad: glibc-217](#2-problema-de-compatibilidad-glibc-217)
- [3. Instalación Manual de Nodejs-16](#3-instalación-manual-de-nodejs-16)
  - [3.1 Proceso de Instalación](#31-proceso-de-instalación)
- [4. Configuración de npm](#4-configuración-de-npm)
- [5. Instalación de Node-RED](#5-instalación-de-node-red)
  - [5.1 Nodos y Módulos Adicionales](#51-nodos-y-módulos-adicionales)
- [6. Validación con Kafka](#6-validación-con-kafka)
  - [6.1 Conversión Buffer → Texto](#61-conversión-buffer--texto)
  - [6.2 Envío Manual de Datos](#62-envío-manual-de-datos)
- [7. APIs REST y Dashboard](#7-apis-rest-y-dashboard)
- [8. Servicio Systemd](#8-servicio-systemd)

---

## 1. Introducción
En la máquina virtual **ambari9 (VM9)** se instaló **Node.js**, **npm** y **Node-RED** para habilitar un entorno de integración ligera y dashboards complementarios al ecosistema Big Data.  
CentOS 7 requiere versiones antiguas de Node.js debido a la dependencia con **glibc 2.17**.

---

## 2. Problema de Compatibilidad: glibc 2.17
Las versiones modernas de Node.js requieren glibc ≥ 2.28, lo que genera errores como:

glibc >= 2.28 required
libstdc++.so.6(GLIBCXX_3.4.21) required
libm.so.6(GLIBC_2.27) required

Código

---

## 3. Instalación Manual de Node.js 16
Node.js **16** es la última versión compatible con glibc 2.17.

### 3.1 Proceso de Instalación

# Acceso al directorio y descarga
cd /opt
sudo curl -O https://nodejs.org/dist/latest-v16.x/node-v16.20.2-linux-x64.tar.xz

# Descompresión y organización
sudo tar -xf node-v16.20.2-linux-x64.tar.xz
sudo mv node-v16.20.2-linux-x64 node16

# Configuración del PATH en ~/.bashrc
echo 'export PATH=/opt/node16/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
4. Configuración de npm
Para evitar errores EACCES al instalar paquetes globales:

mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=$HOME/.npm-global/bin:$PATH' >> ~/.bashrc
5. Instalación de Node-RED
npm install -g --unsafe-perm node-red@3.1.3
5.1 Nodos y Módulos Adicionales
Instalados en ~/.node-red:

node-red-contrib-kafkajs

node-red-contrib-redis

node-red-contrib-postgres

node-red-dashboard

6. Validación con Kafka
6.1 Conversión Buffer → Texto
if (msg.payload && msg.payload.value) {
    msg.payload = msg.payload.value.toString();
    return msg;
}
6.2 Envío Manual de Datos
echo "Hola desde Ambari9, probando Kafka" | /usr/bin/kafka-console-producer.sh \
--broker-list 172.16.200.28:9092 --topic sensores_data
Resultado: El mensaje se recibió correctamente en Node-RED.

7. APIs REST y Dashboard
Servicios implementados:

/api/estado

/api/eventos

/api/health

8. Servicio Systemd
[Unit]
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
