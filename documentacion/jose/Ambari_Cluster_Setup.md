# Documentación: Configuración de Conectividad en Cluster Ambari

## 1. Introducción

Este documento describe el proceso completo para establecer conectividad entre todos los hosts que conforman un cluster de Apache Ambari. El objetivo es garantizar que todos los nodos puedan comunicarse entre sí sin problemas, prerequisito indispensable para la instalación y configuración de los servicios distribuidos.

El cluster está compuesto por 9 hosts con direcciones IP en el rango 172.16.200.x, cada uno con su propia identidad de red y nombre de host único.

---

## 2. Arquitectura de Red

### 2.1 Topología del Cluster

El cluster Ambari se implementó con la siguiente configuración de red:

| Máquina | IP Host | Hostname | Dominio Completo |
|---------|---------|----------|------------------|
| Host 1 | 172.16.200.10 | ambari10 | ambari10.localdomain |
| Host 2 | 172.16.200.11 | ambari11 | ambari11.localdomain |
| Host 3 | 172.16.200.12 | ambari12 | ambari12.localdomain |
| Host 4 | 172.16.200.13 | ambari13 | ambari13.localdomain |
| Host 5 | 172.16.200.15 | ambari15 | ambari15.localdomain |
| Host 6 | 172.16.200.8 | ambari8 | ambari8.localdomain |
| Host 7 | 172.16.200.9 | ambari9 | ambari9.localdomain |
| Host 8 | 172.16.200.5 | ambari5 | ambari5.localdomain |
| Host 9 | 172.16.200.3 | ambari3 | ambari3.localdomain |

### 2.2 Configuración de Adaptadores de Red

Cada máquina física fue configurada con **dos adaptadores de red**:

1. **Adaptador Puente (Bridge)**: Conecta el cluster de hosts entre sí en la red 172.16.200.x
2. **Adaptador NAT**: Proporciona acceso a internet y conectividad externa

Esta configuración dual permite que los nodos se comuniquen entre ellos manteniendo conexión con el exterior.

---

## 3. Pasos de Configuración

### 3.1 Configuración de Direcciones IP

**Objetivo**: Asegurar que todos los hosts tengan direcciones IP en el mismo rango de red.

**Pasos ejecutados**:

1. Se asignó a cada máquina una dirección IP única en el rango 172.16.200.x
2. Se utilizó el patrón: el último octeto de la IP coincide con el número del host
   - Ejemplo: `ambari11` tiene IP `172.16.200.11`
   - Ejemplo: `ambari5` tiene IP `172.16.200.5`

**Ventaja de este patrón**:
- Facilita la identificación rápida de máquinas
- Reduce errores de configuración
- Mejora la documentación y mantenibilidad

### 3.2 Configuración de Hostnames

**Objetivo**: Establecer nombres consistentes y reconocibles para cada host.

**Formato del hostname**:
```
ambariNN
```

Donde NN es el número correlativo del host, matching con el último octeto de la IP.

**Ejemplos**:
- Host con IP 172.16.200.11 → hostname: `ambari11`
- Host con IP 172.16.200.8 → hostname: `ambari8`
- Host con IP 172.16.200.3 → hostname: `ambari3`

**Procedimiento**:
Se modificó el hostname en cada máquina usando el comando correspondiente al sistema operativo (generalmente en `/etc/hostname` en sistemas Linux).

### 3.3 Configuración del Archivo /etc/hosts

**Objetivo**: Permitir que todos los hosts se resuelvan mutuamente por nombre sin depender de DNS.

**Archivo `/etc/hosts` configurado en TODOS los hosts**:

```
172.16.200.11   ambari11  ambari11.localdomain
172.16.200.10   ambari10  ambari10.localdomain
172.16.200.8    ambari8   ambari8.localdomain
172.16.200.5    ambari5   ambari5.localdomain
172.16.200.12   ambari12  ambari12.localdomain
172.16.200.13   ambari13  ambari13.localdomain
172.16.200.15   ambari15  ambari15.localdomain
172.16.200.9    ambari9   ambari9.localdomain
172.16.200.3    ambari3   ambari3.localdomain
```

**Puntos clave**:

- **Mismo contenido en todos los hosts**: Este archivo es idéntico en las 9 máquinas
- **Tres columnas**: IP, hostname corto, nombre de dominio completo (FQDN)
- **FQDN con sufijo .localdomain**: Define un dominio local para el cluster
- **Ventaja**: Los servicios pueden comunicarse usando nombres en lugar de IPs, mejorando la portabilidad

**Impacto**: Cuando un nodo necesita conectar con otro durante la instalación de servicios, puede usar `ambari11.localdomain` en lugar de memorizar IPs.

### 3.4 Deshabilitación del Firewall

**Objetivo**: Eliminar barreras de red entre los hosts durante la configuración inicial.

**Acción realizada**:
Se deshabilitó el firewall en todas las máquinas para permitir comunicación sin restricciones.

**Nota de seguridad**: 
En un entorno de producción, esto debería ser reemplazado por reglas de firewall específicas que permitan solo los puertos necesarios para Ambari y los servicios Hadoop.

---

### 3.5 Modificación de Archivo de Configuración Crítico

**Objetivo**: Establecer el host principal (master) del cluster.

**Archivo modificado**: `/etc/ambari-agent/conf/ambari-agent.ini` (en todos los hosts)

**Cambio realizado**:

En la sección `[server]`, se modificó el parámetro `hostname`:

```ini
[server]
hostname=localhost                    # ANTES
hostname=ambari11.localdomain         # DESPUÉS
url_port=8440
secured_url_port=8441
```

**Razón**: Esto permite que todos los agentes del cluster se conecten al servidor Ambari maestro (ambari11) en lugar de intentar conectarse a `localhost`. Sin este cambio, los agentes no pueden comunicarse con el servidor para recibir instrucciones e instalar servicios.

**Reinicio necesario**:
```bash
sudo ambari-agent restart
```

---

## 4. Instalación de Servicios

### 4.1 Orden de Instalación

Una vez establecida la conectividad, se procedió a instalar los siguientes servicios distribuidos en el cluster:

| Servicio          | Descripción                            |
| ----------------- | -------------------------------------- |
| HDFS              | Sistema de ficheros distribuido        |
| YARN              | Gestor de recursos y scheduler         |
| MapReduce2        | Framework de procesamiento distribuido |
| Tez               | Motor de ejecución para Hive y Pig     |
| Hive              | SQL distribuido sobre Hadoop           |
| HBase             | Base de datos NoSQL distribuida        |
| ZooKeeper         | Coordinación y sincronización          |
| Ambari Metrics    | Recopilación de métricas               |
| Kafka             | Streaming de eventos                   |
| Spark             | Procesamiento en memoria               |
| Zeppelin Notebook | Interfaz interactiva                   |
| Flink             | Procesamiento streaming avanzado       |
| Solr              | Motor de búsqueda distribuido          |

### 4.2 Distribución Automática

La instalación y distribución de estos servicios fue realizada por **Ambari siguiendo sus recomendaciones por defecto**. Ambari automáticamente:

1. Analiza los recursos disponibles en cada nodo
2. Distribuye componentes maestros (Master) en nodos apropiados
3. Distribuye componentes esclavos (Slave) en todos los nodos
4. Gestiona dependencias entre servicios

---

### 4.3 Distribución de Servicios por Nodo

Una vez completada la instalación, los servicios quedaron distribuidos de la siguiente manera:

| Nodo | IP | Roles principales |
|------|-----|-------------------|
| **ambari10** | 172.16.200.10 | NameNode HDFS, Kafka Broker, Grafana, WebHCat, Spark History Server, Zeppelin Notebook, Metrics Monitor |
| **ambari11** | 172.16.200.11 | ResourceManager YARN, MapReduce2 History Server, ZooKeeper Server, Metrics Monitor |
| **ambari12** | 172.16.200.12 | HiveServer2, Hive Metastore, MySQL (Hive), ZooKeeper Server, HBase Master (Active), Metrics Monitor |
| **ambari13** | 172.16.200.13 | Ambari Metrics Collector, DataNode, NodeManager, ZooKeeper Client, Metrics Monitor |
| **ambari15** | 172.16.200.15 | DataNode, NodeManager, Metrics Monitor |
| **ambari9** | 172.16.200.9 | DataNode, NodeManager, Spark Thrift Server, HBase RegionServer, Metrics Monitor |
| **ambari8** | 172.16.200.8 | DataNode, NodeManager, Metrics Monitor |
| **ambari5** | 172.16.200.5 | DataNode, NodeManager, Metrics Monitor |
| **ambari3** | 172.16.200.3 | DataNode, NodeManager, ZooKeeper Client, Metrics Monitor |

#### Servicios Externos al Cluster

Adicionalmente, se configuraron dos máquinas virtuales externas con servicios auxiliares:

| Máquina | IP | Servicios |
|---------|-----|-----------|
| **VM Redis** | 172.16.200.23 | Servidor Redis (VM sobre PC físico de ambari3) |
| **VM HA Debian** | 172.30.101.216 | Home Assistant en Docker (VM sobre PC físico de ambari5) |

#### Arquitectura de Comunicación

**HDFS**:
- NameNode (`ambari10`) coordina los DataNodes en `ambari3`, `5`, `8`, `9`, `13`, `15`
- Los clientes HDFS contactan al NameNode para metadata y a los DataNodes para bloques

**YARN / MapReduce2 / Spark**:
- ResourceManager (`ambari11`) gestiona los NodeManagers en `ambari3`, `5`, `8`, `9`, `13`, `15`
- History Server (`ambari11`) recopila logs de jobs
- Spark utiliza YARN para ejecutar jobs distribuidos

**Ambari Metrics**:
- Metrics Collector (`ambari13`) recibe métricas de todos los Metrics Monitor
- Grafana (`ambari10`) visualiza las métricas desde el Collector

**HBase**:
- HBase Master (`ambari12`) coordina los RegionServers
- RegionServer en `ambari9` almacena datos en HDFS

**Hive**:
- HiveServer2 y Metastore (`ambari12`) con MySQL como backend
- Los clientes Hive se conectan a HiveServer2 para consultas SQL

**ZooKeeper**:
- Ensemble formado por servidores en `ambari10`, `11`, `12`
- Clientes en otros nodos usan el ensemble para coordinación

**Kafka**:
- Broker en `ambari10` conectado al ensemble de ZooKeeper

---

## 5. Diagrama Conceptual

```
┌─────────────────────────────────────────────────────────────┐
│                    CLUSTER AMBARI                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  ambari10    │  │  ambari11    │  │  ambari12    │      │
│  │ 172.16.200.10│  │ 172.16.200.11│  │ 172.16.200.12│      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │
│         │                 │                 │              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  ambari8     │  │  ambari9     │  │  ambari13    │      │
│  │ 172.16.200.8 │  │ 172.16.200.9 │  │ 172.16.200.13│      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │
│         │                 │                 │              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  ambari5     │  │  ambari3     │  │  ambari15    │      │
│  │ 172.16.200.5 │  │ 172.16.200.3 │  │ 172.16.200.15│      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │
│         │                 │                 │              │
│  └──────────────────────┬──────────────────┘              │
│          Red: 172.16.200.0/24 (Bridge Network)             │
│                                                              │
│  + Adaptador NAT en cada máquina para Internet              │
└─────────────────────────────────────────────────────────────┘
```

---

## 6. Problemas Comunes y Soluciones

| Problema | Causa | Solución |
|----------|-------|----------|
| No se resuelven nombres entre hosts | /etc/hosts incorrecto o incompleto | Verificar que TODOS los hosts tengan el mismo /etc/hosts |
| Conexión rechazada a servicios | Firewall bloqueando puertos | Deshabilitar firewall o añadir reglas específicas |
| Ambari Agent no conecta con Server | Hostname o IP mal configurado | Verificar que el hostname resoluble apunta a la IP correcta |
| Servicios no se instalan correctamente | Falta de conectividad entre nodos | Ejecutar ping y tests SSH antes de instalar servicios |

---

## 7. Resumen de Configuración

**Pasos clave ejecutados**:

1. ✅ Asignación de IPs fijas en rango 172.16.200.x
2. ✅ Configuración de hostnames (ambari3-ambari15)
3. ✅ Creación de /etc/hosts idéntico en todos los hosts
4. ✅ Deshabilitación del firewall
5. ✅ Configuración de servidor Ambari principal
6. ✅ Instalación de 12 servicios diferentes
7. ✅ Distribución automática según recomendaciones de Ambari

**Resultado final**: 
Un cluster totalmente funcional de 9 nodos con conectividad completa, capaz de ejecutar workloads distribuidos de Hadoop y servicios complementarios.

---

## 8. Referencias Técnicas

- **Documentación Ambari**: https://ambari.apache.org/
- **HDFS Architecture**: https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html
- **YARN Overview**: https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html
- **ZooKeeper Ensemble**: https://zookeeper.apache.org/doc/current/zookeeperInstall.html


