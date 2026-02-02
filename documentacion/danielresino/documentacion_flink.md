# Documentación Técnica: Instalación, Validación y Despliegue de Apache Flink

**Autor:** Daniel Resino Jiménez  
**Fecha:** Febrero 2026

---

## 📑 Índice
1. [Arquitectura del Sistema](#-arquitectura-del-sistema)
2. [Proceso de Instalación y Configuración](#-proceso-de-instalación-y-configuración)
3. [Resolución de Problemas (Troubleshooting)](#-resolución-de-problemas-troubleshooting)
4. [Implementación del Pipeline de Persistencia](#-implementación-del-pipeline-de-persistencia)
5. [Despliegue y Resultados en Vivo](#-despliegue-y-resultados-en-vivo)
6. [Conclusiones](#-conclusiones)

---

## 🏗️ Arquitectura del Sistema
La infraestructura configurada se basa en los siguientes componentes técnicos:

* **Motor de Procesamiento:** Apache Flink 1.15.4 (Scala 2.12)
* **Entorno de Scripting:** Python 3.6
* **Nodo Maestro:** `ambari13` (172.16.200.13)
* **Base de Datos de Persistencia:** Redis (172.16.200.23)
* **Interfaz de Gestión:** Flink Web UI operativa en el puerto `8081`



---

## ⚙️ Proceso de Instalación y Configuración
Se actualizaron las herramientas de gestión de paquetes e instalamos las dependencias necesarias para la comunicación entre Python y Flink.

### Preparación del Entorno Python
* **Actualización de Pip:** Se ejecutó `pip3 install --user --upgrade pip setuptools wheel`.
* **Instalación de PyFlink:** Versión específica mediante `pip3 install --user apache-flink==1.15.4`.
* **Librerías de Cliente:** Se instaló el módulo `redis` para Python para permitir la comunicación con el sumidero externo.

### Configuración Crítica del Motor (`flink-conf.yaml`)
Se modificó el archivo maestro para permitir el acceso remoto y la ejecución de Python:
* `rest.address: 0.0.0.0` (Acceso externo a la Web UI).
* `python.executable: /usr/bin/python3` (Ruta del intérprete en TaskManagers).

---

## 🛠️ Resolución de Problemas (Troubleshooting)
Durante el proceso se solventaron incidentes críticos que afectaban la estabilidad del clúster:

| Incidente | Solución Técnica |
| :--- | :--- |
| **Error de Inicialización** | Uso del constructor `EnvironmentSettings.new_instance().in_streaming_mode().build()` para evitar el *AttributeError* en Flink 1.15. |
| **Gestión de Slots** | Limpieza de procesos *zombie* de Python y reinicio del clúster para liberar Task Slots bloqueados. |
| **Dependencias Distribuidas** | Inclusión del `import redis` dentro de la función UDF para carga dinámica en TaskManagers. |

---

## 🐍 Implementación del Pipeline de Persistencia
Se desarrolló el script `final_match.py` orientado a un flujo de streaming continuo mediante la Table API.

**Fragmento de la UDF y Sink:**

```python
@udf(result_type=DataTypes.STRING())
def send_to_redis(id_val):
    import redis
    try:
        r = redis.StrictRedis(host='172.16.200.23', port=6379, db=0, password='password')
        r.set(f"sensor_id_{id_val}", "CLUSTER_ACTIVE")
        return f"ID {id_val}: OK"
    except Exception as e:
        return str(e)
