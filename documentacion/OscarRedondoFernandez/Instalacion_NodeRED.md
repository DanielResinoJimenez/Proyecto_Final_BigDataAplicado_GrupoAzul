# Memoria Técnica: Instalación y Configuración de Node-RED (VM9)

## 1. Introducción
[cite_start]En la máquina virtual **ambari9** se instaló Node-RED para habilitar dashboards y servicios REST dentro del ecosistema Big Data[cite: 9]. [cite_start]Debido al uso de CentOS 7, se adaptó el proceso por limitaciones de la librería `glibc`[cite: 10].

## 2. Instalación de Node.js y npm
* [cite_start]**Problema**: CentOS 7 usa `glibc 2.17`, incompatible con Node.js moderno[cite: 12].
* [cite_start]**Solución**: Instalación manual de **Node.js 16.20.2** mediante binarios oficiales[cite: 18, 19].
* [cite_start]**Configuración**: Se redirigió el prefijo de npm a `~/.npm-global` para evitar errores de permisos[cite: 40, 43].

## 3. Configuración de Node-RED
[cite_start]Se instaló la versión **3.1.3**[cite: 47]. Se añadieron nodos para:
* [cite_start]**Kafka**: `node-red-contrib-kafkajs` (vía npm install)[cite: 59, 61].
* [cite_start]**Persistencia/Cache**: Redis y PostgreSQL[cite: 63, 65].
* [cite_start]**Interfaz**: Node-RED Dashboard[cite: 67].

## 4. Validación de Integración con Kafka
Se realizó una prueba de flujo de datos con el clúster en la IP **172.16.200.28**.

### 4.1 Configuración del Flujo
1. **Consumer**: Escuchando el tópico `sensores_data`.
2. **Función (Buffer a Texto)**: Convierte el binario de Kafka a string legible:
   ```javascript
   if (msg.payload && msg.payload.value) {
       msg.payload = msg.payload.value.toString();
       return msg;
   }
