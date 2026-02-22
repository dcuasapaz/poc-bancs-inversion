# 🏦 POC: Sistema ETL Bancario de Inversiones

> **Proof of Concept** para un sistema completo de ETL (Extracción, Transformación y Carga) que genera y procesa datos maestros de clientes e inversiones bancarias a través de una arquitectura de **tres zonas: Raw, Curada y Productiva**.

| Propiedad | Valor |
|-----------|-------|
| **Autor** | Diego Cuasapaz |
| **Fecha** | 2026-02-21 |
| **Versión** | 1.0 |
| **Estado** | ✅ Funcional |

---

## 📋 Tabla de Contenidos

- [Descripción General](#descripción-general)
- [Arquitectura](#arquitectura)
- [Requisitos Previos](#requisitos-previos)
- [Instalación](#instalación)
- [Configuración](#configuración)
- [Uso](#uso)
- [Flujo de Datos](#flujo-de-datos)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Notebooks](#notebooks)
- [Bases de Datos](#bases-de-datos)
- [Troubleshooting](#troubleshooting)

---

## 🎯 Descripción General

Este proyecto implementa un **pipeline ETL bancario completo** que:

| # | Funcionalidad | Detalle |
|---|---|---|
| 1️⃣ | **Genera datos sintéticos** | Clientes e inversiones con lógica de negocio realista |
| 2️⃣ | **Valida y transforma** | Aplicando estándares bancarios y nomenclatura |
| 3️⃣ | **Carga en PostgreSQL** | Con particionamiento por períodos YYYYMMDD |
| 4️⃣ | **Exporta reportes** | CSV en cada zona para auditoría y análisis |
| 5️⃣ | **Auditoría completa** | Timestamps y trazabilidad de cambios |

### 🛠️ Componentes Principales

| Componente | Propósito | Salida |
|-----------|----------|--------|
| **01_generation_data_clientes** | Genera 100 clientes sintéticos con Faker | 3 CSV + PostgreSQL |
| **02_generation_data_inversiones** | Genera 550+ inversiones con estados diarios | 3 CSV + PostgreSQL particionado |
| **PostgreSQL** | Almacena datos con esquema de tres zonas | 6 esquemas (zr, zc, zp) |
| **CSV Export** | Exporta datos curados para análisis externo | data/raw, curada, productiva |

---

## 🏗️ Arquitectura

### Modelo de Tres Zonas

```
┌─────────────────────────────────────────────────────────────────┐
│                    DATOS ORIGINALES (Faker)                     │
└────────────────────────────┬────────────────────────────────────┘
                             │
                ┌────────────▼────────────┐
                │   ZONA RAW (ZR)         │
                ├─────────────────────────┤
                │ • Sin transformar       │
                │ • Auditoría completa    │
                │ • CSV respaldo          │
                │ • zr_cli.* / zr_pas.*   │
                └────────────┬────────────┘
                             │
                ┌────────────▼────────────┐
                │  ZONA CURADA (ZC)       │
                ├─────────────────────────┤
                │ • Datos normalizados    │
                │ • Nomenclatura bancaria │
                │ • Validaciones básicas  │
                │ • zc_cli.* / zc_pas.*   │
                └────────────┬────────────┘
                             │
                ┌────────────▼────────────┐
                │  ZONA PRODUCTIVA (ZP)   │
                ├─────────────────────────┤
                │ • Tablas consolidadas   │
                │ • Listos para análisis  │
                │ • Joins cliente+datos   │
                │ • zp.td_* / zp.tn_*     │
                └────────────┬────────────┘
                             │
                        ┌────▼────┐
                        │ CSV/BI  │
                        └─────────┘
```

### Esquemas PostgreSQL

- **`zr_cli` / `zr_pas`**: Raw zone (datos sin procesar)
- **`zc_cli` / `zc_pas`**: Curated zone (datos transformados)
- **`zp`**: Production zone (datos consolidados)

---

## 🔧 Requisitos Previos

### 💻 Sistema Operativo
- ✅ Linux/macOS o WSL2 en Windows
- ✅ Python **3.12+** 
- ✅ PostgreSQL **14+**

### 📦 Librerías Python

```txt
pandas >= 2.0.0          # Manipulación de datos
numpy >= 1.24.0          # Cálculos numéricos
sqlalchemy >= 2.0.0      # ORM y conexiones DB
psycopg2-binary >= 2.9.0 # Driver PostgreSQL
python-dotenv >= 1.0.0   # Variables de entorno
faker >= 40.0.0          # Generación de datos sintéticos
jupyter >= 1.0.0         # Notebooks interactivos
ipykernel >= 6.0.0       # Kernel de Python para Jupyter
```

### 🗄️ Configuración PostgreSQL Requerida

Crear esquemas antes de ejecutar:

```sql
-- Zonas de datos
CREATE SCHEMA zr_cli;  -- Raw clientes
CREATE SCHEMA zr_pas;  -- Raw inversiones
CREATE SCHEMA zc_cli;  -- Curada clientes
CREATE SCHEMA zc_pas;  -- Curada inversiones
CREATE SCHEMA zp;      -- Producción consolidado

-- Crear base de datos
CREATE DATABASE banco_inversiones;
```

---

## 📦 Instalación

### ⏬ 1. Clonar el Repositorio

```bash
git clone <repositorio-url>
cd poc-bancs-inversion
```

### 🐍 2. Crear Entorno Virtual

```bash
# Crear venv
python3.12 -m venv poc-bancs-inversion-venv

# Activar (Linux/macOS)
source poc-bancs-inversion-venv/bin/activate

# Activar (Windows)
poc-bancs-inversion-venv\Scripts\activate
```

### 📥 3. Instalar Dependencias

```bash
# Actualizar pip
pip install --upgrade pip

# Instalar desde requirements.txt (si existe)
pip install -r requirements.txt

# O instalar manualmente
pip install pandas numpy sqlalchemy psycopg2-binary python-dotenv faker jupyter
```

### 🚀 4. Iniciar Jupyter

```bash
jupyter notebook
```

Se abrirá en `http://localhost:8888`

---

## ⚙️ Configuración

### 🔑 Crear Archivo `.env`

Crear `poc-bancs-inversion/.env`:

```env
# 🗄️ Configuración PostgreSQL
DB_USER=usuario_postgres
DB_PASSWORD=tu_contraseña_segura
DB_HOST=localhost
DB_PORT=5432
DB_NAME=banco_inversiones
```

#### ⚠️ IMPORTANTE:

- ❌ **NO** incluir `.env` en git (está en `.gitignore`)
- 🔒 Si contraseña tiene `@`, `#`, `$` → encerrar en comillas: `DB_PASSWORD="p@ss#word"`
- 📝 Crear BD antes de ejecutar:
  ```bash
  createdb -U postgres banco_inversiones
  ```
- 🧪 Verificar conexión:
  ```bash
  psql -U postgres -h localhost -d banco_inversiones
  ```

---

## 🚀 Uso

### 🎬 Ejecución Paso a Paso

#### 📌 Paso 1️⃣: Generar Datos de Clientes

```bash
jupyter notebook notebooks/01_generation_data_clientes.ipynb
```

**¿Qué hace?**
- ✅ Genera **100 clientes sintéticos** con Faker
- ✅ Crea en **tres zonas**: Raw → Curada → Productiva
- ✅ **Exporta CSV** en `data/raw/`, `data/curada/`, `data/productiva/`
- ✅ **Carga en PostgreSQL**:
  - `zr_cli.zr_fake_cli_datos_clientes` (Raw)
  - `zc_cli.zc_cli_datos_clientes` (Curada)
  - `zp.td_datos_clientes` (Productiva)

⏱️ **Tiempo estimado:** 2-5 minutos

---

#### 📌 Paso 2️⃣: Generar Datos de Inversiones

```bash
jupyter notebook notebooks/02_generation_data_inversiones.ipynb
```

**¿Qué hace?**
- ✅ **Carga maestro** de clientes desde `zp.td_datos_clientes`
- ✅ **Genera 550+ inversiones** con lógica de negocio:
  - Estados: `VIGENTE` | `PIGNORADO` | `RENOVADO` | `PRE-CANCELADO` | `VENCIDO`
  - Pignoraciones: ~5% de casos
  - Cancelaciones anticipadas: ~3% de casos
  - Renovaciones: 30% automática, 20% ventanilla, 50% sin renovación
- ✅ **Genera registro diario** para cada inversión (período configurable)
- ✅ **Carga en PostgreSQL con particionamiento**:
  - `zr_pas.zr_fake_pas_inversiones` (Raw particionado)
  - `zc_pas.zc_pas_inversiones` (Curada)
  - `zp.tn_pas_inversiones` (Consolidado + datos clientes)

⏱️ **Tiempo estimado:** 5-15 minutos

---

### ⚙️ Parámetros Configurables

En ambos notebooks, modificar al inicio:

```python
# 👥 Número de registros a generar
N_CLIENTES = 100
N_INVERSIONES_INICIALES = 550

# 📅 Período de análisis (formato: YYYY-MM-DD)
FECHA_INICIO_MES = datetime(2025, 12, 1)
FECHA_FIN_MES = datetime(2025, 12, 31)
```

---

## 📊 Flujo de Datos Detallado

### Pipeline de Clientes

```
100 Faker Records
    ↓ (Generación)
DataFrame Raw (15 columnas)
    ↓ (Validación)
Zona Raw (CSV + PostgreSQL)
    ↓ (Transformación)
- Mapeo de nomenclatura
- Conversión de tipos
- Normalización de strings
    ↓
Zona Curada (CSV + PostgreSQL)
    ↓ (Consolidación)
- Selección de dimensiones
- Reordenamiento de columnas
    ↓
Zona Productiva (CSV + PostgreSQL)
```

### Pipeline de Inversiones

```
Maestro de Clientes
    ↓
550+ Inversiones Sintéticas
    ├─ Monto: $5,000 - $150,000
    ├─ Tasa: 6% - 9% anual
    ├─ Plazo: 90/180/360 días
    └─ Estados: VIGENTE, PIGNORADO, etc.
    ↓
Listado Diario (550 inv × 30 días = 16,500 registros)
    ├─ Cálculos de interés diario
    ├─ Interés acumulado mensual
    └─ Validaciones de monto/tasa
    ↓
Zona Raw → Zona Curada → Zona Productiva
    ↓
Consolidación con Datos de Clientes (JOIN)
```

---

## 📂 Estructura del Proyecto

```
poc-bancs-inversion/
│
├── 📄 README.md                          ← Este archivo
├── 📄 requirements.txt                   ← Dependencias Python
├── 🔐 .env                               ← Configuración local (NO en git)
├── 🔐 .env.example                       ← Plantilla de .env
│
├── 📂 notebooks/                         🎯 Scripts ETL principales
│   ├── 01_generation_data_clientes.ipynb    ✅ Pipeline clientes (100 registros)
│   └── 02_generation_data_inversiones.ipynb ✅ Pipeline inversiones (550+ diarios)
│
├── 📂 data/                              📊 Exportaciones CSV
│   ├── raw/                              ← Zona Raw (sin transformar)
│   │   ├── zr_fake_cli_datos_clientes.csv
│   │   └── zr_fake_pas_inversiones.csv
│   ├── curada/                           ← Zona Curada (normalizados)
│   │   ├── zc_cli_datos_clientes.csv
│   │   └── zc_pas_inversiones.csv
│   └── productiva/                       ← Zona Productiva (análisis)
│       ├── td_datos_clientes.csv
│       └── tn_pas_inversiones.csv
│
├── 📂 logs/                              📝 Logs de ejecución
│   └── etl_inversiones.log               ← Archivo de logs rotativo
│
├── 📂 conf/                              ⚙️ Configuración
│   └── Scripts/                          ← Scripts DBeaver
│
├── 📂 src/                               🔧 Código reutilizable
│   └── [vacío - para funciones comunes]
│
└── 📂 poc-bancs-inversion-venv/          🐍 Entorno virtual Python 3.12
    ├── bin/
    ├── lib/
    ├── include/
    └── share/
```

---

## 🗄️ Bases de Datos

### Tablas de Clientes

#### `zp.td_datos_clientes` (Zona Productiva)

| Columna | Tipo | Descripción |
|---------|------|-------------|
| codigoSecuencialCliente | INT | ID secuencial (1000-1099) |
| codigoIdentificacionCliente | VARCHAR | CUS-XXXXX único |
| tipoIdentificacionCliente | VARCHAR | CÉDULA, RUC, PASAPORTE |
| numeroIdentificacionCliente | VARCHAR | Número de ID único |
| nombreCompletoCliente | VARCHAR | Nombre completo |
| segmentoCliente | VARCHAR | RETAIL, CORPORATIVO, PYME, WEALTH |
| scoreCrediticioCliente | INT | Score 300-1000 |
| provinciaCliente | VARCHAR | Pichincha, Guayas, Azuay, Manabí, Loja |
| ciudadCliente | VARCHAR | Quito, Guayaquil, Cuenca, Manta |
| fechaRegistroCliente | DATE | Fecha de alta |

### Tablas de Inversiones

#### `zp.tn_pas_inversiones` (Zona Productiva - Particionada)

| Columna | Tipo | Descripción |
|---------|------|-------------|
| codigoPeriodo | INT | YYYYMM |
| fechaProceso | DATE | Fecha del registro |
| numeroInversion | VARCHAR | INV-XXXXX |
| codigoIdentificacionCliente | VARCHAR | Vinculación con cliente |
| montoAperturaInversion | NUMERIC | Monto inicial ($) |
| tasaAperturaInversion | NUMERIC | Tasa (%) |
| montoActualInversion | NUMERIC | Monto actual |
| estadoInversion | VARCHAR | VIGENTE, PIGNORADO, RENOVADO, etc. |
| interesDiaInversion | NUMERIC | Interés calculado diario |
| interesAcumuladoMesInversion | NUMERIC | Acumulado mensual |
| fechaProceso | DATE | Timeline |
| fechaIngesta | TIMESTAMP | Auditoría |

**Particionamiento:** Por `periodo` (YYYYMMDD) para optimizar consultas históricas.

---

## 🔍 Troubleshooting

### ❌ Error: "Credenciales incompletas"

**🔍 Causa:** Variables de entorno no cargadas  
**✅ Solución:**

```bash
# Verificar .env existe
cat .env

# Recargar variables (Linux/macOS)
source .env
```

---

### ❌ Error: "Conexión rechazada a PostgreSQL"

**🔍 Causa:** PostgreSQL no está corriendo  
**✅ Solución:**

```bash
# Iniciar PostgreSQL (Linux)
sudo systemctl start postgresql

# O verificar puerto
psql -U postgres -h localhost -p 5432
```

---

### ❌ Error: "Tabla ya existe"

**🔍 Causa:** Ejecución duplicada sin limpiar datos previos  
**✅ Solución:**

```sql
-- Opción 1: Truncar tabla
TRUNCATE TABLE zr_cli.zr_fake_cli_datos_clientes;

-- Opción 2: Eliminar esquema completo
DROP SCHEMA zr_cli CASCADE;
DROP SCHEMA zc_cli CASCADE;
```

---

### ⚠️ Performance lento en Inversiones

**🔍 Causa:** Muchos registros generados (~550 inv × 30 días = 16,500+)  
**✅ Soluciones:**

```python
# 1️⃣ Reducir volumen
N_INVERSIONES_INICIALES = 100  # en lugar de 550

# 2️⃣ Reducir período
FECHA_INICIO_MES = datetime(2025, 12, 1)
FECHA_FIN_MES = datetime(2025, 12, 7)  # Solo 7 días

# 3️⃣ Crear índices
CREATE INDEX idx_periodo ON zp.tn_pas_inversiones(periodo);
CREATE INDEX idx_cliente ON zp.tn_pas_inversiones(codigoIdentificacionCliente);
```

---

### 💾 Memoria insuficiente

**🔍 Causa:** DataFrames grandes en Jupyter  
**✅ Solución:**

```python
# Limpiar memoria manualmente
import gc
del df_clientes
del df_inversiones
gc.collect()
```

---

## 📈 Próximos Pasos

- [ ] 🎯 Crear tabla de auditoría centralizada
- [ ] 🚀 Implementar API REST para consultas
- [ ] 📊 Agregar dashboards con Grafana/Metabase
- [ ] 🤖 Automatizar con Airflow/dbt
- [ ] ✅ Agregar validaciones de calidad de datos
- [ ] 🧪 Implementar tests unitarios e integración
- [ ] 📈 Performance tuning y particionamiento avanzado
- [ ] 🔐 Mejorar seguridad de credenciales (Vault, Secrets Manager)

---

## 📞 Contacto y Soporte

| Aspecto | Detalle |
|--------|--------|
| **Autor** | Diego Cuasapaz |
| **Email** | [tu-email] |
| **Issues** | Reportar en GitHub |
| **Documentación** | Ver secciones de Arquitectura y Bases de Datos |

---

## 📄 Licencia

Especificar licencia (MIT, Apache 2.0, GPL, etc.)

---

## 📚 Referencias Útiles

- 📖 [SQLAlchemy Docs](https://docs.sqlalchemy.org/)
- 🗄️ [PostgreSQL Partitioning](https://www.postgresql.org/docs/current/ddl-partitioning.html)
- 🎭 [Faker Docs](https://faker.readthedocs.io/)
- 🐼 [Pandas ETL Patterns](https://pandas.pydata.org/docs/)
- 📓 [Jupyter Notebook Guide](https://jupyter.org/)
