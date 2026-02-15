# Resumen Ejecutivo

Este documento detalla la configuración del JMX Prometheus Exporter en el broker Kafka ubicado en `nodo1`, permitiendo la exposición de métricas operativas en formato Prometheus para su posterior consumo por sistemas de monitorización externos.

### Especificaciones Técnicas

| Parámetro | Valor |
|-----------|-------|
| **Versión** | 1.0 |
| **Fecha** | 13 de Febrero de 2026 |
| **Servidor** | nodo1 (172.16.200.28) |
| **Puerto Métricas** | 7071 |
| **JMX Exporter** | 0.20.0 |
| **Kafka Version** | 2.8.1 |
| **Topic Principal** | acceso-centros-nfc |

---

## Tabla de Contenidos

1. [Introducción](#introducción)
2. [Arquitectura de la Solución](#arquitectura-de-la-solución)
3. [Procedimiento de Instalación](#procedimiento-de-instalación)
4. [Verificación de la Configuración](#verificación-de-la-configuración)
5. [Información para Sistemas Externos](#información-para-sistemas-externos)
6. [Conclusiones](#conclusiones)
7. [Referencias](#referencias)
8. [Apéndices](#apéndices)

---

## Introducción

### Objetivo

Configurar el servidor Kafka en `nodo1` para exponer métricas operativas mediante JMX Prometheus Exporter, permitiendo que sistemas externos de monitorización (Prometheus, Grafana) puedan consumir información sobre el estado y rendimiento del broker.

### Contexto del Proyecto

El sistema de control de accesos NFC utiliza Apache Kafka como plataforma central de streaming de eventos. El broker Kafka en `nodo1` procesa eventos generados por el productor `nfc_producer.py` y los distribuye a múltiples consumidores (n8n, Flink, etc.).

Para garantizar la observabilidad del sistema, se implementa JMX Exporter que:

- Extrae métricas JMX nativas de Kafka
- Las transforma a formato Prometheus (text-based)
- Las expone vía HTTP en el puerto 7071
- Permite monitorización sin impacto en el rendimiento

### Alcance de este Documento

Este documento cubre **únicamente** la configuración realizada en `nodo1`:

-  Descarga e instalación del JMX Exporter
-  Configuración del archivo YAML de reglas
-  Modificación del script de inicio de Kafka
-  Configuración del firewall
-  Verificación local de métricas

**Fuera de alcance:** Configuración de Prometheus, Grafana o sistemas de monitorización externos.

### Infraestructura

| Componente | Detalle |
|------------|---------|
| Servidor | nodo1 |
| IP | 172.16.200.28 |
| Sistema Operativo | CentOS 7 |
| Kafka Broker ID | 10 |
| Kafka Puerto | 9092 |
| JMX Puerto | 9999 |
| Prometheus Exporter Puerto | 7071 |
| Kafka Home | /opt/kafka-2.8.1 |

---

## Arquitectura de la Solución

### Flujo de Datos

```
┌──────────────────────────────────────────┐
│  Kafka Broker (JVM)                      │
│  - Expone métricas vía JMX               │
│  - Puerto JMX: 9999                      │
└────────────────┬─────────────────────────┘
                 │
                 ↓
┌──────────────────────────────────────────┐
│  JMX Prometheus Exporter (JavaAgent)     │
│  - Lee métricas JMX                      │
│  - Transforma a formato Prometheus       │
│  - Expone en HTTP puerto 7071            │
└────────────────┬─────────────────────────┘
                 │
                 ↓
        http://172.16.200.28:7071/metrics
                 │
                 ↓
    ┌────────────┴─────────────┐
    │                          │
    ↓                          ↓
Prometheus                  Grafana
(Sistema Externo)          (Sistema Externo)
```

### Componentes

| Componente | Función |
|------------|---------|
| **Kafka JMX** | Sistema nativo de Java que expone métricas internas del broker (particiones, mensajes, latencia, etc.) |
| **JMX Exporter** | Agente Java (JAR) que se adjunta al proceso Kafka mediante `-javaagent`, lee métricas JMX y las transforma |
| **kafka-jmx-config.yml** | Archivo de configuración que define qué métricas se exportan y cómo se etiquetan |
| **Puerto 7071** | Endpoint HTTP donde se exponen las métricas en formato texto compatible con Prometheus |
| **Firewall** | Reglas de firewalld que permiten acceso externo al puerto 7071 |

---

## Procedimiento de Instalación

### Descarga del JMX Exporter

Debido a que `nodo1` no tiene acceso directo a Internet, el archivo se descarga desde `ambari11` (que sí tiene conectividad) y se transfiere posteriormente.

#### Paso 1: Descarga en ambari11

```bash
# Conectar a ambari11 desde nodo1
ssh hadoop@172.16.200.11

# Navegar a directorio temporal
cd /tmp

# Descargar JMX Prometheus Exporter version 0.20.0
wget https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.20.0/jmx_prometheus_javaagent-0.20.0.jar

# Verificar descarga
ls -lh jmx_prometheus_javaagent-0.20.0.jar
```

**Salida esperada:**
```
-rw-rw-r-- 1 hadoop hadoop 564K ago 12 2023 jmx_prometheus_javaagent-0.20.0.jar
```

#### Paso 2: Transferencia a nodo1

```bash
# Salir de ambari11 (volver a nodo1)
exit

# Desde nodo1, traer el archivo de ambari11 usando SCP
scp hadoop@172.16.200.11:/tmp/jmx_prometheus_javaagent-0.20.0.jar /tmp/

# Cambiar a usuario root
su -

# Mover archivo a ubicacion definitiva
mv /tmp/jmx_prometheus_javaagent-0.20.0.jar /opt/

# Establecer permisos correctos
chmod 644 /opt/jmx_prometheus_javaagent-0.20.0.jar
chown root:root /opt/jmx_prometheus_javaagent-0.20.0.jar

# Verificar instalacion
ls -lh /opt/jmx_prometheus_javaagent-0.20.0.jar
file /opt/jmx_prometheus_javaagent-0.20.0.jar
```

**Verificación exitosa:**
```
-rw-r--r--. 1 root root 564K feb 13 19:17 jmx_prometheus_javaagent-0.20.0.jar
jmx_prometheus_javaagent-0.20.0.jar: Java archive data (JAR)
```

---

### Configuración del Exporter

#### Creación del Archivo de Configuración

El archivo `kafka-jmx-config.yml` define las reglas de transformación de métricas JMX a formato Prometheus.

```bash
# Como root en nodo1
cat > /opt/kafka-jmx-config.yml << 'EOF'
lowercaseOutputName: true
lowercaseOutputLabelNames: true
rules:
  # Reglas para metricas de Kafka Broker
  - pattern: kafka.server<type=(.+), name=(.+), clientId=(.+), topic=(.+), partition=(.*)><>Value
    name: kafka_server_$1_$2
    type: GAUGE
    labels:
      clientId: "$3"
      topic: "$4"
      partition: "$5"
  
  - pattern: kafka.server<type=(.+), name=(.+), clientId=(.+), brokerHost=(.+), brokerPort=(.+)><>Value
    name: kafka_server_$1_$2
    type: GAUGE
    labels:
      clientId: "$3"
      broker: "$4:$5"
  
  - pattern: kafka.server<type=(.+), name=(.+)><>Value
    name: kafka_server_$1_$2
    type: GAUGE
  
  # Metricas de red
  - pattern: kafka.network<type=(.+), name=(.+), request=(.+)><>Value
    name: kafka_network_$1_$2
    type: GAUGE
    labels:
      request: "$3"
  
  # Metricas de log
  - pattern: kafka.log<type=(.+), name=(.+), topic=(.+), partition=(.+)><>Value
    name: kafka_log_$1_$2
    type: GAUGE
    labels:
      topic: "$3"
      partition: "$4"
  
  # Metricas de controller
  - pattern: kafka.controller<type=(.+), name=(.+)><>Value
    name: kafka_controller_$1_$2
    type: GAUGE
EOF

# Verificar que se creo correctamente
cat /opt/kafka-jmx-config.yml
```

#### Explicación de las Reglas

| Parámetro | Descripción |
|-----------|-------------|
| `lowercaseOutputName` | Convierte nombres de métricas a minúsculas para consistencia |
| `lowercaseOutputLabelNames` | Convierte nombres de etiquetas a minúsculas |
| `pattern` | Expresión regular que coincide con el nombre JMX de la métrica |
| `name` | Nombre resultante de la métrica en formato Prometheus |
| `type` | Tipo de métrica (GAUGE, COUNTER, HISTOGRAM) |
| `labels` | Etiquetas adicionales extraídas del patrón JMX |

---

### Modificación del Script de Kafka

Para que Kafka inicie con el JMX Exporter, es necesario modificar el script de arranque agregando el JavaAgent.

```bash
# Editar el script de inicio de Kafka
nano /opt/kafka-2.8.1/bin/kafka-server-start.sh
```

**Buscar la sección de configuración JMX (aproximadamente líneas 30-50) y añadir/modificar:**

```bash
# JMX settings
if [ -z "$KAFKA_JMX_OPTS" ]; then
  KAFKA_JMX_OPTS="-javaagent:/opt/jmx_prometheus_javaagent-0.20.0.jar=7071:/opt/kafka-jmx-config.yml -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.port=9999 -Djava.rmi.server.hostname=172.16.200.28"
fi
export KAFKA_JMX_OPTS
```

#### Explicación de los Parámetros JMX

| Parámetro | Descripción |
|-----------|-------------|
| `-javaagent:/opt/jmx_...jar=7071:/opt/...yml` | Carga el agente JMX Exporter, expone métricas en puerto 7071 usando reglas del archivo YAML |
| `-Dcom.sun.management.jmxremote` | Habilita el servidor JMX remoto |
| `-Dcom.sun.management.jmxremote.authenticate=false` | Desactiva autenticación JMX (desarrollo/pruebas) |
| `-Dcom.sun.management.jmxremote.ssl=false` | Desactiva SSL para conexiones JMX |
| `-Dcom.sun.management.jmxremote.port=9999` | Puerto para conexiones JMX nativas |
| `-Djava.rmi.server.hostname=172.16.200.28` | IP del servidor para conexiones RMI |

> ** Nota de Seguridad:**  
> En entornos de producción, se recomienda activar autenticación JMX y SSL. Para este proyecto educativo se desactivan para simplificar la configuración.

---

### Reinicio de Kafka

Una vez realizadas las modificaciones, es necesario reiniciar Kafka para aplicar los cambios.

```bash
# Detener Kafka
/opt/kafka-2.8.1/bin/kafka-server-stop.sh

# Esperar cierre completo del proceso
sleep 10

# Verificar que no quedan procesos activos
ps aux | grep kafka

# Iniciar Kafka con la nueva configuracion
/opt/kafka-2.8.1/bin/kafka-server-start.sh -daemon /opt/kafka-2.8.1/config/server.properties

# Esperar a que el servicio arranque completamente
sleep 15

# Verificar que Kafka esta corriendo
ps aux | grep kafka
```

**Salida esperada:**
```
root  7138  ... java -Xmx1G -Xms1G ... -javaagent:/opt/jmx_prometheus... 
                kafka.Kafka /opt/kafka-2.8.1/config/server.properties
```

El parámetro `-javaagent` debe aparecer en la línea del proceso, confirmando que el JMX Exporter está activo.

---

### Configuración del Firewall

Para permitir que sistemas externos (Prometheus) puedan acceder al endpoint de métricas, es necesario abrir el puerto 7071 en el firewall.

```bash
# Verificar estado del firewall
systemctl status firewalld

# Abrir puerto 7071 de forma permanente
firewall-cmd --permanent --add-port=7071/tcp

# Recargar configuracion del firewall
firewall-cmd --reload

# Verificar que el puerto se añadio correctamente
firewall-cmd --list-ports
```

**Salida esperada:**
```
9092/tcp 7071/tcp
```

> ** CRÍTICO:**  
> **Sin este paso, la monitorización externa fallará.** Prometheus mostrará el error: `"connect: no route to host"`. Este es el problema más común al configurar JMX Exporter.

#### Verificación de Reglas de Firewall

```bash
# Ver todas las reglas activas
firewall-cmd --list-all

# Ver puertos específicamente
firewall-cmd --list-ports

# Verificar zona activa
firewall-cmd --get-active-zones

# Si usas iptables en lugar de firewalld
iptables -L -n | grep 7071
```

---

## Verificación de la Configuración

### Verificación del Puerto

```bash
# Verificar puerto en escucha
netstat -tulpn | grep 7071

# Alternativa con ss
ss -tulpn | grep 7071
```

**Salida esperada:**
```
tcp6  0  0 :::7071  :::*  LISTEN  7138/java
```

Esto confirma que:
- El puerto 7071 está en estado LISTEN
- Escucha en todas las interfaces (`:::` indica IPv6 wildcard)
- El proceso es Java con PID 7138 (Kafka)

---

### Verificación del Endpoint de Métricas

```bash
# Ver primeras 30 lineas de metricas
curl http://localhost:7071/metrics | head -30

# Contar total de metricas disponibles
curl -s http://localhost:7071/metrics | grep "^kafka_" | wc -l

# Ver metricas especificas del broker
curl -s http://localhost:7071/metrics | grep "kafka_server"

# Verificar metricas de topics
curl -s http://localhost:7071/metrics | grep "kafka_log"
```

---

### Ejemplo de Métricas Expuestas

**Formato de salida Prometheus:**
```
# HELP kafka_server_replicamanager_leadercount Attribute exposed
# TYPE kafka_server_replicamanager_leadercount gauge
kafka_server_replicamanager_leadercount 3.0

# HELP kafka_server_replicamanager_partitioncount Attribute exposed
# TYPE kafka_server_replicamanager_partitioncount gauge
kafka_server_replicamanager_partitioncount 3.0

# HELP kafka_server_brokertopicmetrics_messagesinpersec Attribute exposed
# TYPE kafka_server_brokertopicmetrics_messagesinpersec gauge
kafka_server_brokertopicmetrics_messagesinpersec{topic="acceso-centros-nfc"} 5.0

# HELP kafka_log_logsize Attribute exposed
# TYPE kafka_log_logsize gauge
kafka_log_logsize{topic="acceso-centros-nfc",partition="0"} 2048576.0
```

---

### Verificación desde Otra Máquina

Para confirmar que el endpoint es accesible externamente:

```bash
# Desde ambari10 u otra maquina
curl http://172.16.200.28:7071/metrics | head -10

# Probar conectividad TCP
telnet 172.16.200.28 7071

# Verificar con timeout
timeout 5 curl http://172.16.200.28:7071/metrics
```

Si estas pruebas funcionan, el endpoint está correctamente configurado y accesible para Prometheus.

---

### Métricas Disponibles

#### Categorías de Métricas

| Categoría | Descripción |
|-----------|-------------|
| `kafka_server_*` | Métricas del broker: particiones, réplicas, estado |
| `kafka_network_*` | Métricas de red: latencia, throughput, conexiones |
| `kafka_log_*` | Métricas de logs: tamaño, offset, segmentos |
| `kafka_controller_*` | Métricas del controller: elecciones, particiones offline |
| `jvm_*` | Métricas de JVM: memoria, garbage collection, threads |

#### Métricas Clave del Broker

| Métrica | Descripción |
|---------|-------------|
| `kafka_server_replicamanager_leadercount` | Número de particiones líderes en este broker |
| `kafka_server_replicamanager_partitioncount` | Total de particiones gestionadas |
| `kafka_server_brokertopicmetrics_messagesinpersec` | Tasa de mensajes entrantes por segundo |
| `kafka_server_brokertopicmetrics_bytesinpersec` | Tasa de bytes entrantes por segundo |
| `kafka_server_replicamanager_underreplicatedpartitions` | Particiones sin replicación completa (debe ser 0) |
| `kafka_server_kafkaserver_brokerstate` | Estado del broker (3 = Running) |

---

### Troubleshooting

#### Problema: Puerto 7071 no responde

**Síntoma:**
```
curl: (7) Failed to connect to localhost port 7071: Connection refused
```

**Diagnóstico:**
```bash
# 1. Verificar que Kafka esta corriendo
ps aux | grep kafka

# 2. Revisar logs de Kafka
tail -100 /opt/kafka-2.8.1/logs/server.log

# 3. Buscar errores de JMX
grep -i "jmx\|prometheus\|error" /opt/kafka-2.8.1/logs/server.log

# 4. Verificar que el JAR existe
ls -lh /opt/jmx_prometheus_javaagent-0.20.0.jar

# 5. Verificar configuracion YAML
cat /opt/kafka-jmx-config.yml
```

**Soluciones:**
- Verificar ruta correcta del JAR en `kafka-server-start.sh`
- Comprobar sintaxis del archivo YAML (espacios, no tabs)
- Revisar permisos del archivo JAR (debe ser readable)
- Reiniciar Kafka limpiamente

---

#### Problema: Acceso denegado desde exterior

**Síntoma:**
```
curl: (7) Failed to connect to 172.16.200.28 port 7071: No route to host
```

**Diagnóstico:**
```bash
# 1. Verificar firewall
firewall-cmd --list-ports

# 2. Verificar que el puerto escucha en todas las interfaces
netstat -tulpn | grep 7071

# 3. Probar localmente primero
curl http://localhost:7071/metrics

# 4. Verificar iptables
iptables -L -n | grep 7071
```

**Solución:**
```bash
# Abrir puerto en firewall
firewall-cmd --permanent --add-port=7071/tcp
firewall-cmd --reload
firewall-cmd --list-ports
```

---

### Script de Verificación Completo

```bash
#!/bin/bash

echo "======================================================"
echo "  VERIFICACION JMX EXPORTER - KAFKA NODO1"
echo "======================================================"

# 1. Verificar puerto
echo -e "\n[1/5] Verificando puerto 7071..."
if netstat -tulpn | grep -q ":7071"; then
    echo "  ✓ Puerto 7071 activo"
else
    echo "  ✗ ERROR: Puerto 7071 no responde"
    exit 1
fi

# 2. Verificar endpoint
echo -e "\n[2/5] Verificando endpoint HTTP..."
if curl -s http://localhost:7071/metrics > /dev/null; then
    echo "  ✓ Endpoint accesible"
else
    echo "  ✗ ERROR: Endpoint no responde"
    exit 1
fi

# 3. Contar metricas
echo -e "\n[3/5] Contando metricas disponibles..."
METRICS=$(curl -s http://localhost:7071/metrics | grep -c "^kafka_")
if [ $METRICS -gt 50 ]; then
    echo "  ✓ $METRICS metricas disponibles"
else
    echo "  ⚠ Solo $METRICS metricas (esperado >50)"
fi

# 4. Verificar firewall
echo -e "\n[4/5] Verificando firewall..."
if firewall-cmd --list-ports | grep -q "7071/tcp"; then
    echo "  ✓ Puerto abierto en firewall"
else
    echo "  ⚠ Puerto no configurado en firewall"
fi

# 5. Verificar acceso externo
echo -e "\n[5/5] Verificando acceso desde red interna..."
echo "  URL: http://172.16.200.28:7071/metrics"

echo -e "\n======================================================"
echo "  VERIFICACION COMPLETADA"
echo "======================================================"
```

---

## Información para Sistemas Externos

### Endpoint de Métricas

** Endpoint Configurado:**

```
URL: http://172.16.200.28:7071/metrics
Formato: Prometheus text format
Intervalo recomendado: 15 segundos
Timeout recomendado: 10 segundos
Protocolo: HTTP (no HTTPS)
```

---

### Configuración Recomendada para Prometheus

```yaml
scrape_configs:
  - job_name: 'kafka-nodo1'
    scrape_interval: 15s
    scrape_timeout: 10s
    static_configs:
      - targets: ['172.16.200.28:7071']
        labels:
          cluster: 'kafka-accesos-nfc'
          broker_id: '10'
          environment: 'produccion'
```

> **Nota:** Esta configuración debe ser implementada por el equipo responsable de Prometheus/Grafana.

---

## Conclusiones

### Resumen de la Configuración

Se ha configurado exitosamente el JMX Prometheus Exporter en el broker Kafka de `nodo1`, permitiendo:

-  Exposición de métricas JMX en formato Prometheus
-  Acceso HTTP en puerto 7071
-  Más de 50 métricas operativas disponibles
-  Acceso externo habilitado mediante firewall
-  Sistema listo para integración con Prometheus/Grafana

---


4. ✅ **Simplificado formato** para compatibilidad con GitHub

¡Ahora debería visualizarse perfectamente en GitHub! 🚀
