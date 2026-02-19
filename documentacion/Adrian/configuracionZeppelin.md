# Configuración de Zeppelin

## Spark, PostgreSQL y Stack de Visualización en Clúster Hadoop (Ambari)

---

## 1\. Resumen Ejecutivo

El presente documento detalla las intervenciones técnicas realizadas para habilitar un entorno de ciencia de datos sobre un clúster legacy (**CentOS 7**). Se aborda la resolución de problemas de conectividad con bases de datos externas, la migración crítica del entorno de ejecución a **Python 3** y la sincronización de dependencias en un entorno distribuido de 9 nodos gestionado por **Ambari**.

---

## 2\. Conectividad JDBC: Spark a PostgreSQL Externo

**Desafío:** Inexistencia de drivers nativos en el ecosistema Spark para la extracción de datos desde fuentes relacionales externas, resultando en errores de tipo `ClassNotFoundException`.

**Solución Aplicada:**

* **Implementación de Driver:** Despliegue del conector `postgresql-42.5.0.jar` en el nodo maestro (`ambari10`) bajo la ruta `/usr/lib/zeppelin/lib/`.  
* **Configuración del Classpath:** Modificación del template `spark2-env` en la consola de administración de Ambari para incluir el JAR en el flujo de ejecución global:  
    
  export SPARK\_DIST\_CLASSPATH=$(hadoop classpath):/usr/lib/zeppelin/lib/postgresql-jdbc.jar  
    
* **Persistencia en Intérprete:** Registro de la dependencia en la propiedad `common.jars` del motor de ejecución de Apache Zeppelin para asegurar la carga del driver en tiempo de ejecución.

---

## 3\. Estandarización del Entorno de Ejecución (Python 3.x)

**Desafío:** Incompatibilidad del código analítico moderno y librerías de Machine Learning con el intérprete nativo del sistema (Python 2.7).

**Solución Aplicada:**

* **Redirección de Binarios:** Forzado del sistema para utilizar `/usr/bin/python3` en todas las capas del servicio.  
* **Configuración de Zeppelin:** Actualización de las propiedades del intérprete para mitigar el uso de la versión legacy:  
  * `zeppelin.pyspark.python` → `/usr/bin/python3`  
  * `spark.pyspark.python` → `/usr/bin/python3`  
* **Hardening en Ambari:** Configuración de la variable de entorno `PYSPARK_PYTHON` en el servicio **Spark2** para garantizar la persistencia de la versión 3.x tras reinicios del clúster.

---

## 4\. Gestión de Repositorios y Dependencias (CentOS 7 EOL)

**Desafío:** Fallo crítico en el gestor de paquetes `yum` debido al fin de vida (EOL) de CentOS 7 y la inoperatividad de los repositorios SCL (Software Collections).

**Solución Aplicada:**

* **Saneamiento de Repositorios:** Desactivación de mirrors obsoletos para normalizar el flujo de trabajo de `yum`:  
    
  sudo yum-config-manager \--disable centos-sclo-rh centos-sclo-sclo  
    
* **Estrategia de Instalación:** Implementación de instalación mediante binarios pre-compilados (**Wheels**) para omitir la compilación local y resolver conflictos de dependencias:  
    
  sudo pip3 install \--only-binary=:all: pandas matplotlib seaborn Pillow

---

## 5\. Sincronización Distribuida y Visualización

**Desafío:** Errores de importación (`ModuleNotFoundError`) en los nodos esclavos durante la ejecución de tareas distribuidas en YARN, debido a la asimetría de librerías entre nodos.

**Solución Aplicada:**

* **Despliegue Multi-Nodo:** Ejecución de scripts de automatización vía SSH para replicar el stack tecnológico de forma idéntica en la totalidad de los nodos del clúster (**ambari3 a ambari15**).  
* **Gestión de Privilegios:** Ajuste de permisos recursivos en las rutas de librerías para permitir la ejecución bajo el usuario de servicio `zeppelin`:  
    
  sudo chmod \-R 755 /usr/lib64/python3.6/site-packages  
    
* **Configuración de Renderizado:** Desactivación de `zeppelin.pyspark.useIPython` para estabilizar la salida gráfica y reinicio mandatorio del intérprete para refrescar el `sys.path`.

---

## 6\. Arquitectura Final de la Solución

Tras las intervenciones, el sistema queda validado con las siguientes capacidades operativas:

1. **Ingesta:** Pipeline funcional Spark-JDBC para extracción eficiente de datos relacionales.  
2. **Procesamiento:** Motor PySpark 2.x operando sobre un entorno distribuido de Python 3.6 en 9 nodos.  
3. **Análisis y Visualización:** Disponibilidad total de las librerías `NumPy`, `Pandas`, `Scipy`, `Matplotlib` y `Seaborn` para la generación de reportes e insights directamente en la interfaz web de Zeppelin.

**Certificación de Estado:** El entorno se encuentra **estable y operativo**, con las dependencias sincronizadas y los permisos de ejecución validados para el usuario final.  
