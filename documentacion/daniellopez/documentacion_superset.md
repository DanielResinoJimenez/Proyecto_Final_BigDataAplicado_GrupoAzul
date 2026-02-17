Aquí está tu documentación actualizada con la explicación del dashboard y la imagen:

```markdown
# Documentación: Apache Superset con PostgreSQL

**Sistema:** CentOS 7 / RHEL 7  
**Servidor:** ambari30 (172.16.200.30)  
**Base de Datos:** PostgreSQL en 172.16.200.43

---

## Índice

1. [Instalación de Apache Superset](#1-instalación-de-apache-superset)
2. [Configuración Inicial](#2-configuración-inicial-de-superset)
3. [Resolución de Problemas de Puerto](#3-resolución-de-problemas-de-puerto)
4. [Instalación del Driver de PostgreSQL](#4-instalación-del-driver-de-postgresql)
5. [Iniciar Apache Superset](#5-iniciar-apache-superset)
6. [Conexión a PostgreSQL](#6-conexión-a-postgresql)
7. [Exploración de Datos con SQL Lab](#7-exploración-de-datos-con-sql-lab)
8. [Creación de Visualizaciones](#8-creación-de-visualizaciones)
9. [Dashboard Creado](#9-dashboard-creado)
10. [Comandos de Referencia Rápida](#10-comandos-de-referencia-rápida)

---

## 1. Instalación de Apache Superset

### 1.1 Instalación de Dependencias

Primero se instalaron las dependencias necesarias del sistema como usuario **root**:

```bash
su -
yum install -y gcc gcc-c++ libffi-devel python-devel python-pip \
python-wheel openssl-devel cyrus-sasl-devel openldap-devel
```

### 1.2 Actualización de pip y setuptools

```bash
pip install --upgrade pip setuptools
```

### 1.3 Instalación de Apache Superset

```bash
pip install apache-superset
```

**Nota:** Durante la instalación pueden aparecer advertencias sobre versiones de dependencias. Estas son normales y no afectan el funcionamiento.

---

## 2. Configuración Inicial de Superset

### 2.1 Crear Base de Datos Interna

```bash
export FLASK_APP=superset
superset db upgrade
```

### 2.2 Crear Usuario Administrador

```bash
superset fab create-admin
```

**Credenciales configuradas:**
- Username: admin
- First name: Admin
- Last name: User
- Email: 15ardagulerrm@gmail.com
- Password: password

### 2.3 Inicializar Superset

```bash
superset init
```

---

## 3. Resolución de Problemas de Puerto

### 3.1 Error: Puerto 9089 en Uso

**Error:** `OSError: [Errno 98] Address already in use`

**Solución:**

```bash
# Identificar proceso
lsof -i :9089

# Detener proceso
kill -9 [PID]
```

---

## 4. Instalación del Driver de PostgreSQL

### 4.1 Error: Invalid Driver

**Causa:** Falta el driver `psycopg2-binary` para conectar con PostgreSQL.

### 4.2 Instalación de psycopg2-binary

```bash
pip3 install psycopg2-binary
```

**Resultado:** `Successfully installed psycopg2-binary-2.9.5`

**Nota:** Las versiones 2.9.6, 2.9.7 y 2.9.8 fallaron al intentar compilar desde el código fuente porque faltaba `pg_config`. La versión 2.9.5 se instaló correctamente usando un paquete wheel precompilado.

---

## 5. Iniciar Apache Superset

### 5.1 Configurar Variable de Entorno

```bash
export FLASK_APP=superset
```

### 5.2 Ejecutar Superset

```bash
superset run -h 0.0.0.0 -p 9089 --with-threads
```

**Salida esperada:**
```
logging was configured successfully
INFO:superset.utils.logging_configurator:logging was configured successfully
 * Serving Flask app "superset"
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
INFO:werkzeug: * Running on http://0.0.0.0:9089/ (Press CTRL+C to quit)
```

### 5.3 Acceso Web

```
http://172.16.200.30:9089
```

Credenciales de acceso: admin / password

---

## 6. Conexión a PostgreSQL

### 6.1 Agregar Base de Datos

Desde la interfaz web de Superset:
1. Click en **Data** → **Databases**
2. Click en **+ DATABASE**
3. Seleccionar **PostgreSQL**

### 6.2 Configuración de la Conexión

**SQLAlchemy URI:**
```
postgresql://ambari:password@172.16.200.43:5432/migasfree
```

**Componentes de la URI:**
- **Protocolo:** `postgresql://`
- **Usuario:** `ambari`
- **Contraseña:** `password` (reemplazar con la contraseña real)
- **Host:** `172.16.200.43`
- **Puerto:** `5432`
- **Base de datos:** `migasfree`

### 6.3 Test de Conexión

Se probó la conexión con el botón **TEST CONNECTION**.

**Resultado:** ✅ Conexión exitosa

---

## 7. Exploración de Datos con SQL Lab

### 7.1 Acceso a SQL Lab

Desde el menú superior: **SQL Lab** → **SQL Editor**

### 7.2 Listar Tablas Disponibles

```sql
SELECT table_name 
FROM information_schema.tables 
WHERE table_schema = 'public' 
ORDER BY table_name;
```

### 7.3 Explorar Estructura de Tabla

```sql
SELECT column_name, data_type 
FROM information_schema.columns 
WHERE table_name = 'catalog_application'
ORDER BY ordinal_position;
```

---

## 8. Creación de Visualizaciones

### 8.1 Primer Gráfico: Total de Aplicaciones

**Consulta SQL:**
```sql
SELECT COUNT(*) as total_aplicaciones 
FROM catalog_application;
```

**Resultado:** 414 aplicaciones

**Pasos:**
1. Ejecutar la consulta en SQL Lab
2. Click en **EXPLORE**
3. **Visualization Type:** Big Number
4. Click en **RUN** y **SAVE**

### 8.2 Segundo Gráfico: Aplicaciones por Categoría

**Consulta SQL:**
```sql
SELECT category, COUNT(*) as total 
FROM catalog_application 
GROUP BY category 
ORDER BY total DESC;
```

**Tipo:** Pie Chart

---

## 9. Dashboard Creado: Panel Migasfree

### 9.1 Vista General del Dashboard

![Dashboard Migasfree](imagenes/imgsuuperset.png)

**URL de acceso:** `http://172.16.200.30:9089/superset/dashboard/1/`

El dashboard **"Panel migasfree"** integra múltiples visualizaciones que ofrecen una visión completa del inventario de centros educativos gestionados con Migasfree.

### 9.2 Componentes del Dashboard

#### **1. Aplicaciones por categoría** (Superior izquierda)
- **Tipo:** Gráfico de sectores (Pie Chart)
- **Datos:** Distribución de las 414 aplicaciones clasificadas en 14 categorías
- **Categorías principales:** 
  - Categoría 1 (naranja): mayor volumen
  - Categoría 2, 3, 4 (rojo, gris, rosa): distribución media
  - Resto de categorías: menor representación
- **Utilidad:** Identificar qué tipo de software predomina en el inventario

#### **2. Centros por provincia** (Superior centro)
- **Tipo:** Gráfico de barras vertical
- **Datos:** Total de centros distribuidos por provincias
- **Destacados:**
  - Provincia con más centros: ~100 centros
  - Otras provincias: entre 20-30 centros
- **Utilidad:** Visualizar la concentración geográfica de centros educativos

#### **3. Distribución de Centros** (Superior derecha)
- **Tipo:** Mapa de calor (TreeMap)
- **Datos:** 30 centros distribuidos en diferentes localizaciones
- **Provincias visibles:** Huesca (27), Teruel, CEI Zaragoza, CEI Huesca
- **Centros destacados:** CEI Huesca Puente Sardas, CEIP Cerbén de Campo, etc.
- **Utilidad:** Identificar densidad de centros por zona geográfica

#### **4. Listado de centros** (Inferior izquierda)
- **Tipo:** Tabla detallada
- **Columnas:**
  - `centro`: Nombre del centro educativo
  - `codcentro`: Código identificador único
  - `maintag`: Etiqueta de mantenimiento
  - `provincia`: Ubicación provincial
- **Registros visibles:** 5 primeros centros de Fraga y Huesca
- **Utilidad:** Consulta rápida de información específica de cada centro

#### **5. Entradas y salida ordenadores** (Inferior centro)
- **Tipo:** Gráfico de barras horizontal (comparativo)
- **Datos:** Volumen de ENTRADA vs SALIDA de equipos
- **Valores:** ~1.5k entradas y similar cantidad de salidas
- **Colores:** Rosa (entrada), Rosa oscuro (salida)
- **Utilidad:** Monitorear flujo de equipamiento informático

#### **6. 10 centros más activos** (Inferior derecha)
- **Tipo:** Gráfico de barras horizontal
- **Métrica:** Total de accesos por centro
- **Rango:** Entre 80-120 accesos por centro
- **Centros listados:** IES, CEIP y otros centros educativos
- **Utilidad:** Identificar los centros con mayor actividad en el sistema

### 9.3 Métricas Clave del Dashboard

| Métrica | Valor | Descripción |
|---------|-------|-------------|
| **Total Aplicaciones** | 414 | Software disponible en inventario |
| **Total Centros** | 30 | Centros educativos registrados |
| **Provincias** | 4 | Huesca (27), Teruel, Zaragoza CEI |
| **Flujo Equipos** | ~1.5k entrada/salida | Movimiento de ordenadores |
| **Categorías Software** | 14 | Clasificación de aplicaciones |

### 9.4 Casos de Uso del Dashboard

1. **Gestión de Inventario:** Supervisar la distribución de software y hardware por centro
2. **Planificación Geográfica:** Identificar zonas con mayor/menor densidad de centros
3. **Monitoreo de Actividad:** Detectar centros con alto uso del sistema
4. **Auditoría de Equipos:** Rastrear entradas y salidas de ordenadores
5. **Análisis de Software:** Evaluar qué categorías de aplicaciones son más utilizadas

### 9.5 Actualización de Datos

Los datos del dashboard se alimentan directamente de la base de datos PostgreSQL en **172.16.200.43:5432** y se actualizan en tiempo real al refrescar la página.

---

## 10. Comandos de Referencia Rápida

### Iniciar Superset
```bash
export FLASK_APP=superset
superset run -h 0.0.0.0 -p 9089 --with-threads
```

### Verificar Puerto en Uso
```bash
lsof -i :9089
```

### Matar Proceso en Puerto
```bash
kill -9 [PID]
```

### URI de Conexión PostgreSQL
```
postgresql://usuario:contraseña@host:puerto/base_datos
```

---

## 11. Problemas Comunes y Soluciones

| Problema | Error | Solución |
|----------|-------|----------|
| Puerto en Uso | `OSError: [Errno 98] Address already in use` | `lsof -i :9089` y `kill -9 [PID]` |
| Driver PostgreSQL | `Invalid driver` | `pip3 install psycopg2-binary` |
| Error Compilación | `pg_config executable not found` | Usar versión 2.9.5 de psycopg2-binary |

---

## 12. Notas de Seguridad

**ADVERTENCIA:** El comando `superset run` inicia un servidor de desarrollo. Para producción, se recomienda usar:

- **Gunicorn** (servidor WSGI)
- **Nginx** (proxy reverso)
- **Supervisor** o **systemd** (gestión de procesos)

**Configuración Recomendada para Producción:**

```bash
gunicorn -w 10 \
  -k gevent \
  --timeout 120 \
  -b 0.0.0.0:9089 \
  --limit-request-line 0 \
  --limit-request-field_size 0 \
  "superset.app:create_app()"
```