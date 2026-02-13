# 🚀 Instalación y Configuración de Node-RED en CentOS 7 (VM9)

## 1. Introducción
[cite_start]En la máquina virtual **ambari9 (VM9)** se ha realizado la instalación de Node.js, npm y Node-RED para habilitar un entorno de integración ligera y dashboards complementarios al ecosistema Big Data. [cite_start]Debido a que el sistema operativo es CentOS 7, fue necesario adaptar el proceso debido a las limitaciones de `glibc`.

## 2. Problema de Compatibilidad: glibc 2.17
[cite_start]CentOS 7 utiliza `glibc 2.17`, lo que impide instalar versiones modernas de Node.js (18, 20+). [cite_start]Los intentos de instalación devolvieron errores de dependencias requeridas como `glibc >= 2.28`.

## 3. Solución: Instalación Manual de Node.js 16
[cite_start]Node.js 16 es la última versión compatible con `glibc 2.17`.

### 3.1 Proceso de Instalación
```bash
# Descarga del binario oficial en /opt
cd /opt
sudo curl -O [https://nodejs.org/dist/latest-v16.x/node-v16.20.2-linux-x64.tar.xz](https://nodejs.org/dist/latest-v16.x/node-v16.20.2-linux-x64.tar.xz)

# Descompresión y configuración
sudo tar -xf node-v16.20.2-linux-x64.tar.xz
sudo mv node-v16.20.2-linux-x64 node16

# Añadir al PATH
echo 'export PATH=/opt/node16/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
