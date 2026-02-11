# Proyecto Big Data
## Sistema de Control de Accesos NFC - Centros Educativos de Aragón

**Autor:** Rubén Jiménez  
**Proyecto:** Proyecto Integral Big Data 2025-2026  
**Fecha:** 11 de febrero de 2026

---

## 📋 Tabla de Contenidos

- [Introducción](#introducción)
- [Infraestructura](#infraestructura)
- [Productor NFC - Control de Accesos](#productor-nfc---control-de-accesos)
- [Conclusiones Generales](#conclusiones-generales)
- [Anexo: Código Fuente Completo](#anexo-código-fuente-completo)

---

## 🎯 Introducción

Este documento describe la implementación completa del sistema de control de accesos NFC para centros educativos de Aragón, desarrollado como parte del proyecto integral de Big Data 2025-2026.

El sistema simula un entorno real de gestión de accesos mediante tarjetas RFID/NFC, integrando tecnologías de streaming de datos, procesamiento en tiempo real y almacenamiento distribuido.

### Objetivos del Proyecto

- ✅ Implementar un pipeline completo de datos en tiempo real
- ✅ Integrar Apache Kafka como broker de mensajería distribuida
- ✅ Desarrollar productores y consumidores de datos
- ✅ Procesar streams con Apache Spark Streaming y Apache Flink
- ✅ Automatizar workflows con n8n
- ✅ Almacenar datos en PostgreSQL para análisis posterior
- ✅ Generar dashboards de visualización en tiempo real

### 🛠️ Tecnologías Utilizadas

| Componente | Tecnología | Versión |
|------------|------------|---------|
| Broker de mensajería | Apache Kafka | 2.8.1 |
| Coordinación distribuida | Apache Zookeeper | 3.4.6 |
| Procesamiento streaming | Apache Spark / Flink | 3.1.2 / 1.14 |
| Automatización | n8n | 2.6.3 |
| Base de datos | PostgreSQL | 13.x |
| Lenguaje productor | Python | 3.x |
| Containerización | Docker | 20.x |
| Sistema operativo | CentOS / Debian | 7 / 11 |

### 🏗️ Arquitectura General del Sistema

El sistema se compone de múltiples servidores que trabajan de forma distribuida para procesar eventos de acceso NFC en tiempo real.

#### Componentes Principales

- **Productor NFC (nodo1):** Genera eventos de acceso simulados
- **Kafka Cluster:** Dos brokers para alta disponibilidad
  - Broker 10 en nodo1 (172.16.200.28:9092)
  - Broker 1001 en ambari10 (172.16.200.10:9092)
- **n8n (debian-ha):** Workflows de automatización
- **Spark/Flink (nodo2):** Procesamiento en tiempo real
- **PostgreSQL (ambari10):** Almacenamiento persistente

#### Distribución de Servidores

| Servidor | IP | Componentes |
|----------|----|-----------  |
| nodo1 | 172.16.200.28 | Kafka Broker 10, Productor NFC Python |
| ambari10 | 172.16.200.10 | Kafka Broker 1001, Zookeeper, PostgreSQL |
| nodo2 | 172.16.200.29 | Spark Streaming, Apache Flink |
| debian-ha | 172.16.200.32 | n8n (Docker), Consumidores Python |

---

## 🖥️ Infraestructura

### Cluster de Servidores

El proyecto se despliega en un cluster de múltiples nodos virtuales, simulando un entorno distribuido real.

| Servidor | IP | Servicios Desplegados |
|----------|----|-----------------------|
| ambari10 | 172.16.200.10 | Kafka Broker 1001, Zookeeper, PostgreSQL, Ambari |
| nodo1 | 172.16.200.28 | Kafka Broker 10, Productor NFC, Spark |
| nodo2 | 172.16.200.29 | Spark Workers, Flink |
| debian-ha | 172.16.200.32 | n8n (Docker), Consumidores Python |

### 🌐 Configuración de Red

#### Resolución de Nombres

Todos los servidores deben tener configurada la resolución de nombres en `/etc/hosts`:

```bash
127.0.0.1   localhost
127.0.1.1   nombre-servidor

# Cluster Big Data
172.16.200.10   ambari10
172.16.200.28   nodo1
172.16.200.29   nodo2
172.16.200.32   debian-ha

# IPv6
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

## 🎓 Productor NFC - Control de Accesos

### Introducción

El productor NFC simula un sistema de control de accesos mediante tarjetas RFID/NFC en centros educativos de Aragón. Genera eventos realistas de entrada y salida de estudiantes, enviándolos a un topic de Kafka para su procesamiento en tiempo real.

Este componente es fundamental en el pipeline de datos, ya que actúa como fuente de eventos que alimentan todo el sistema de procesamiento distribuido.

### ⚙️ Características Técnicas

#### Especificaciones del Productor

- **Lenguaje:** Python 3.x
- **Broker Kafka:** 172.16.200.28:9092, 172.16.200.10:9092
- **Topic:** `acceso-centros-nfc`
- **Particiones:** 3 particiones para distribución de carga
- **Frecuencia:** 5 eventos por segundo (configurable)
- **Formato de datos:** JSON con codificación UTF-8
- **Serialización:** JSON nativo de Python
- **Garantías de entrega:** `acks='all'` para máxima confiabilidad

#### Librerías Utilizadas

```bash
pip3 install kafka-python faker
```

## Dependencias del productor:

kafka-python 2.0.2: Cliente Kafka oficial para Python

faker 18.x: Generación de datos ficticios realistas en español

json: Serialización de eventos (incluido en Python)

uuid: Generación de identificadores NFC únicos

datetime: Gestión de marcas de tiempo con zona horaria

random: Generación de eventos aleatorios ponderados

🏫 Datos de Centros Educativos
Se han integrado 29 centros educativos reales de Aragón desde el archivo vx-centros.csvproporcionado por el sistema educativo.

Tipo	Descripción	Cantidad
CEIP	Centros de Educación Infantil y Primaria	15
IES	Institutos de Educación Secundaria	10
IPC	Centros Públicos Integrados	3
Agencia de Responsabilidad Civil	Colegios Rurales Agrupados	1
Total		29
Distribución geográfica:

Zaragoza: 22 centros (76%)

Huesca: 4 centros (14%)

Teruel: 3 centros (10%)

# 📄 Estructura del Evento NFC
Cada evento generado por el productor sigue el siguiente esquema JSON:

```JSON
{
  "nfc_id": "NFC-A3F7D9E2C4B1",
  "timestamp": "2026-02-02T08:34:12.456789",
  "estudiante": {
    "nombre": "María García López",
    "curso": "2º ESO"
  },
  "centro": {
    "nombre": "IES Miguel Servet",
    "codigo": "50008174",
    "maintag": "SEC-MIGUELSERVET",
    "provincia": "Zaragoza"
  },
  "tipo_evento": "ENTRADA",
  "franja_horaria": "ENTRADA_MANANA",
  "punto_acceso": "Entrada Principal",
  "estado": "VALIDADO",
  "motivo_rechazo": null,
  "temperatura_corporal": 36.5,
  "sistema_origen": "NFC-Gateway-v2.3",
  "version_schema": "2.0"
}
```
## Descripción de campos:

nfc_id: Identificador único de la tarjeta NFC (formato hexadecimal)

timestamp: Marca temporal en formato ISO 8601

estudiante: Información del estudiante (nombre y curso)

centro: Datos del centro educativo

tipo_evento: ENTRADA o SALIDA

franja_horaria: Periodo horario del acceso

punto_acceso: Ubicación física del lector NFC

estado: VALIDADO o RECHAZADO

motivo_rechazo: Razón del rechazo (si aplica)

temperatura_corporal: Control sanitario (35,8°C - 37,2°C)

sistema_origen: Versión del gateway NFC

version_schema: Versión del esquema de datos

# ⏰ Franjas Horarias Realistas
El sistema simula horarios realistas basados ​​en el calendario escolar de Aragón:

Franja	Horario	Probabilidad	Tipo
ENTRADA_MANANA	07:30 - 09:00	40%	ENTRADA
SALIDA_MEDIODIA	13:30 - 15:00	30%	SALIDA
ENTRADA_TARDE	15:00 - 16:00	15%	ENTRADA
SALIDA_TARDE	17:00 - 19:00	15%	SALIDA
Las probabilidades están ponderadas para reflejar el flujo real de estudiantes en un centro educativo típico.

# 🔧 Configuración de Kafka
Configuración del Broker (nodo1)
Archivo de configuración:/opt/kafka-2.8.1/config/server.properties

texto
# ID único del broker
```bash
broker.id=10
```
# Listeners - Escucha en todas las interfaces
```bash
listeners=PLAINTEXT://0.0.0.0:9092
```
# Advertised listeners - IP anunciada a clientes
```bash
advertised.listeners=PLAINTEXT://172.16.200.28:9092
```
# Conexión a Zookeeper del cluster
```bash
zookeeper.connect=172.16.200.10:2181
```
# Directorio de almacenamiento de logs
```bash
log.dirs=/opt/kafka-2.8.1/kafka-logs
```
# Retención de logs (7 días)
log.retention.hours=168

# Tamaño de segmento de log (1 GB)
log.segment.bytes=1073741824

# Número de hilos de red
num.network.threads=3

# Número de hilos de I/O
num.io.threads=8
Configuración del Broker (ambari10)
Archivo:/etc/kafka/conf/server.properties

texto
# ID único del broker
```bash
broker.id=1001
```
# Listeners
```bash
listeners=PLAINTEXT://0.0.0.0:9092
```
# Advertised listeners
```bash
advertised.listeners=PLAINTEXT://172.16.200.10:9092
```
# Conexión a Zookeeper
```bash
zookeeper.connect=172.16.200.10:2181
```
# Directorio de logs
```bash
log.dirs=/kafka-logs
```
# Retención
log.retention.hours=168
Creación del tema
El tema debe crearse con configuración específica para garantizar alta disponibilidad:

intento
```bash
cd /opt/kafka-2.8.1

bin/kafka-topics.sh --create \
  --topic acceso-centros-nfc \
  --bootstrap-server 172.16.200.28:9092 \
  --partitions 3 \
  --replication-factor 1
```
Justificación de la configuración:

3 particiones: Permite procesamiento paralelo por hasta 3 consumidores simultáneos

Factor de replicación 1: Suficiente para entorno de desarrollo (en producción usar 2 o 3)

Distribución de carga entre los dos brokers del cluster

Verificación del tema:

intento
bin/kafka-topics.sh --describe \
  --topic acceso-centros-nfc \
  --bootstrap-server localhost:9092
Salida esperada:

texto
Topic: acceso-centros-nfc  PartitionCount: 3  ReplicationFactor: 1
    Topic: acceso-centros-nfc  Partition: 0  Leader: 10   Replicas: 10   Isr: 10
    Topic: acceso-centros-nfc  Partition: 1  Leader: 1001 Replicas: 1001 Isr: 1001
    Topic: acceso-centros-nfc  Partition: 2  Leader: 10   Replicas: 10   Isr: 10
💻 Implementación del Productor
Estructura del Proyecto
intento
# Como usuario hadoop en nodo1
mkdir -p ~/kafka-nfc-producer
cd ~/kafka-nfc-producer
Estructura de archivos del proyecto:

texto
/home/hadoop/kafka-nfc-producer/
├── nfc_producer.py      # Script principal del productor
├── vx-centros.csv       # Datos de centros (opcional)
├── nfc_productor.log    # Log de ejecución
└── README.md            # Documentación
Instalación de Dependencias
intento
# Instalar pip si no está disponible
```bash
sudo yum install -y python3-pip
```
# Actualizar pip a la última versión
```bash
pip3 install --upgrade pip
```
# Instalar dependencias del proyecto
```bash
pip3 install kafka-python faker
```
# Verificar instalación
```bash
pip3 list | grep -E "kafka|faker"
```
Salida esperada:

texto
faker              18.13.0
kafka-python       2.0.2
Creación del guión del productor
intento
nano nfc_producer.py
El código completo del productor se encuentra en el Anexo: Código Fuente Completo .

▶️ Ejecución del Productor
Dar Permisos de Ejecución
intento
chmod +x nfc_producer.py

# Verificar permisos
```bash
ls -lh nfc_producer.py
```
Salida esperada:

texto
-rwxr-xr-x 1 hadoop hadoop 8.5K Feb  2 20:30 nfc_producer.py
Ejecución en Primer Plano
Para ver los eventos generados en tiempo real:

intento
python3 nfc_producer.py
Para detener al productor: PresionarCtrl+C

Al detener, se mostrarán las estadísticas finales:

texto
⏹️  Deteniendo productor...

📊 RESUMEN FINAL:
   Total eventos: 582
   ✅ Validados: 573 (98.5%)
   ❌ Rechazados: 9 (1.5%)
   📚 Centros activos: 28
   👨‍🎓 Estudiantes registrados: 258

👋 Productor cerrado correctamente
Ejecución en Background
Para mantener al productor corriendo en el segundo plano:

intento
# Ejecutar con nohup (no hangup)
```bash
nohup python3 nfc_producer.py > nfc_productor.log 2>&1 &
```
# Obtener PID del proceso
```bash
echo $!
```
# Ver logs en tiempo real
```bash
tail -f nfc_productor.log
```
# Salir del tail: Ctrl+C (no detiene el productor)
Para detener el productor en segundo plano:

intento
# Buscar proceso
```bash
ps aux | grep nfc_producer
```
# Detener por nombre
```bash
pkill -f nfc_producer.py
```
# O detener por PID
```bash
kill [PID]
```
Ejecución con pantalla (Recomendado)
Screen permite mantener sesiones persistentes que sobreviven a desconexiones SSH:

intento
# Crear sesión screen con nombre
```bash
screen -S nfc-productor
```
# Dentro de screen, ejecutar el productor
```bash
python3 nfc_producer.py
```
# Salir de screen sin detener (presionar):
# Ctrl+A, luego D (detach)

# Volver a conectar a la sesión
```bash
screen -r nfc-productor
```
# Ver sesiones activas
```bash
screen -ls
```
# Matar sesión desde fuera
```bash
screen -X -S nfc-productor quit
✅ Verificación del Sistema
Verificar Topic en Kafka
intento
cd /opt/kafka-2.8.1
```
# Listar todos los topics
```bash
bin/kafka-topics.sh --list \
  --bootstrap-server localhost:9092
```

# Ver detalles del topic específico
```bash
bin/kafka-topics.sh --describe \
  --topic acceso-centros-nfc \
  --bootstrap-server localhost:9092
```

Consumidor de Consola
Para verificar que los eventos están llegando correctamente:

intento
```bash
cd /opt/kafka-2.8.1

bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic acceso-centros-nfc \
  --from-beginning \
  --max-messages 5
```
Salida esperada (eventos JSON):

```JSON
{"nfc_id":"NFC-A3F7D9E2C4B1","timestamp":"2026-02-02T08:34:12.456789",...}
{"nfc_id":"NFC-B8E3C5A9F1D2","timestamp":"2026-02-02T08:34:13.123456",...}
{"nfc_id":"NFC-C1D4E7F2A5B8","timestamp":"2026-02-02T08:34:14.789012",...}
```

Verificar Offsets (Cantidad de Mensajes)
intento
```bash
cd /opt/kafka-2.8.1

bin/kafka-run-class.sh kafka.tools.GetOffsetShell \
  --broker-list localhost:9092 \
  --topic acceso-centros-nfc
```

Ejemplo de salida:

```
acceso-centros-nfc:0:1234
acceso-centros-nfc:1:1235
acceso-centros-nfc:2:1236
```
Los números después del último :indican la cantidad total de mensajes en cada partición.

# 📊 Estadísticas y Métricas
Estadísticas del Productor
El productor muestra estadísticas en tiempo real cada 20 eventos:

- Métrica	Valor
Total de eventos enviados	582
Eventos validados	573 (98,5%)
Eventos rechazados	9 (1,5%)
Centros activos	28 de 29
Estudiantes únicos registrados	258
Rendimiento promedio	5 eventos/seg
Duración de la prueba	116 segundos
Distribución de Eventos
Configuración de validación:

- Tasa de rechazo: 2% (configurable en el código)

- Motivos de rechazo simulados:

- Tarjeta no autorizada (50%)

- Fuera de horario permitido (30%)

- Tarjeta bloqueada temporalmente (20%)

- Control de temperatura: Rango normal 35,8°C - 37,2°C

- Alertas de temperatura: Se genera alerta si > 37.5°C

- Distribución de eventos por tipo:

Tipo de evento	Cantidad	Porcentaje
ENTRADA (mañana)	233	40%
SALIDA (mediodía)	175	30%
ENTRADA (tarde)	87	15%
SALIDA (tarde)	87	15%
Total	582	100%

# 🔧 Configuración Avanzada
Ajustar Frecuencia de Eventos
Para modificar la velocidad de generación de eventos:

```py
# Frecuencia de eventos (eventos por segundo)
EVENTS_PER_SECOND = 5   # Default: 5 eventos/seg

# Ejemplos de otras configuraciones:
# EVENTS_PER_SECOND = 10  # Alta frecuencia (pruebas de carga)
# EVENTS_PER_SECOND = 2   # Baja frecuencia (debugging)
# EVENTS_PER_SECOND = 1   # Muy baja (demostraciones)
Modificar Tasa de Rechazo
```
---
```py
# En la función generar_evento()
'estado': 'VALIDADO' if random.random() > 0.02 else 'RECHAZADO'

# Explicación:
# 0.02 = 2% de rechazos (configuración actual)
# 0.05 = 5% de rechazos
# 0.01 = 1% de rechazos
# 0.10 = 10% de rechazos
Agregar Más Centros Educativos
Para agregar centros adicionales, edite la lista CENTROSen el código:
```
---

```py
CENTROS = [
    # ... centros existentes ...
    {
        'nombre': 'Nuevo Centro Educativo',
        'codigo': '50099999',
        'maintag': 'TIPO-NOMBRECENTRO',
        'provincia': 'Zaragoza'
    },
]
```
---

# 🐛 Solución de problemas
Errores Comunes y Soluciones
Error: ModuleNotFoundError: No hay ningún módulo llamado 'kafka'

Causa: Librerías Python no instaladas.

Solución:

intento
```bash
pip3 install kafka-python faker
Error: kafka.errors.NoBrokersAvailable
```
Causa: No se puede conectar a los brokers Kafka.

## Solución:

intento
# Verificar que Kafka está corriendo
ps aux | grep kafka

# Verificar que el puerto está abierto
sudo netstat -tulpn | grep 9092

# Probar conectividad
telnet 172.16.200.28 9092

# Reiniciar Kafka si es necesario
```bash
cd /opt/kafka-2.8.1
bin/kafka-server-stop.sh
sleep 5
bin/kafka-server-start.sh -daemon config/server.properties
Error: kafka.errors.UnknownTopicOrPartitionError
```

Causa: El tema acceso-centros-nfcno existe.

## Solución:

intento
# Crear el topic manualmente
cd /opt/kafka-2.8.1
bin/kafka-topics.sh --create \
  --topic acceso-centros-nfc \
  --bootstrap-server localhost:9092 \
  --partitions 3 \
  --replication-factor 1
Error: Permiso denegado

Causa: El script no tiene permisos de ejecución.

## Solución:

intento
```bash
chmod +x nfc_producer.py
```

# 🔗 Integración con Consumidores
El productor NFC está diseñado para integrarse con múltiples tipos de consumidores simultáneamente:

- n8n: Flujos de trabajo de automatización y procesamiento de eventos

- Spark Streaming: Análisis en tiempo real y agregaciones

- Apache Flink: Procesamiento de streams complejos con ventanas temporales

- PostgreSQL: Almacenamiento persistente vía consumidores

- Consumidores personalizados de Python: Lógica de negocio específica

### La arquitectura Kafka permite que cada consumidor mantenga su propio offset, procesando los eventos de forma independiente sin afectar a otros consumidores.

