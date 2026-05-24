# accesorios-dm-database

Base de datos para el sistema de comercio electrónico de accesorios DM, gestionada con Liquibase y PostgreSQL.

## Alcance Actual

Este repositorio administra toda la estructura de la base de datos del proyecto:

- Extensiones: `uuid-ossp`
- Schemas: `security`, `clientes`, `catalogo`, `promociones`, `ventas`, `logistica`, `inventario`
- Tablas: 17 tablas completas (roles, empleados, clientes, categorías, materiales, productos, imágenes, promociones, carritos, pedidos, inventario)
- Datos iniciales: roles, tipos de movimiento, estados de pedido, categorías, materiales, productos demo
- Funciones y triggers: actualización automática de stock
- Índices: 35 índices de rendimiento
- Vistas: 10 vistas para reportes
- Políticas RLS: seguridad a nivel de fila

## Estructura del Proyecto

```text
accesorios-dm-database/
|-- changelog-master.yaml
|-- docker-compose.yml
|-- .env.example
|-- .gitignore
|-- README.md
|-- verify-all.ps1
|-- 01_ddl/
|   |-- changelog.yaml
|   |-- 00_extensions/
|   |   |-- changelog.yaml
|   |   `-- 001_enable_uuid_extension.sql
|   |-- 01_schemas/
|   |   |-- changelog.yaml
|   |   `-- 001_create_schemas.sql
|   |-- 02_types/
|   |   `-- changelog.yaml
|   |-- 03_tables/
|   |   |-- changelog.yaml
|   |   |-- 001_create_security_tables.sql
|   |   |-- 002_create_clientes_tables.sql
|   |   |-- 003_create_catalogo_tables.sql
|   |   |-- 004_create_imagen_producto_table.sql
|   |   |-- 005_create_promociones_tables.sql
|   |   |-- 006_create_carrito_tables.sql
|   |   |-- 007_create_pedidos_tables.sql
|   |   `-- 008_create_inventario_movimiento_table.sql
|   |-- 04_views/
|   |   |-- changelog.yaml
|   |   `-- 001_report_views.sql
|   |-- 05_materialized_views/
|   |   `-- changelog.yaml
|   |-- 06_functions/
|   |   |-- changelog.yaml
|   |   `-- 001_update_stock_functions.sql
|   |-- 07_procedures/
|   |   `-- changelog.yaml
|   |-- 08_triggers/
|   |   |-- changelog.yaml
|   |   `-- 001_inventario_triggers.sql
|   `-- 09_indexes/
|       |-- changelog.yaml
|       `-- 001_performance_indexes.sql
|-- 02_dml/
|   |-- changelog.yaml
|   `-- 00_inserts/
|       |-- changelog.yaml
|       `-- 001_initial_data.sql
|-- 03_dcl/
|   |-- changelog.yaml
|   |-- 00_roles/
|   |   |-- changelog.yaml
|   |   `-- 001_create_roles.sql
|   |-- 01_grants/
|   |   |-- changelog.yaml
|   |   `-- 001_grants.sql
|   `-- 02_policies/
|       |-- changelog.yaml
|       `-- 001_rls_policies.sql
|-- 04_tcl/
|   `-- changelog.yaml
|-- 05_rollbacks/
|   |-- 01_ddl/
|   |   |-- 00_extensions/
|   |   |-- 01_schemas/
|   |   |-- 02_types/
|   |   |-- 03_tables/
|   |   |-- 04_views/
|   |   |-- 05_materialized_views/
|   |   |-- 06_functions/
|   |   |-- 07_procedures/
|   |   |-- 08_triggers/
|   |   `-- 09_indexes/
|   |-- 02_dml/
|   |   `-- 00_inserts/
|   |-- 03_dcl/
|   |   |-- 00_roles/
|   |   |-- 01_grants/
|   |   `-- 02_policies/
|   `-- 04_tcl/
|-- docker/
|   `-- liquibase/
|       `-- Dockerfile
`-- docs/
    `-- scripts-verificacion/
        |-- verify-develop.ps1
        |-- verify-qa.ps1
        `-- verify-main.ps1
```

## Arquitectura de Capas

La arquitectura está organizada por responsabilidad SQL:

|Capa	|Propósito|
|-------|---------|
|01_ddl	|Cambios estructurales (extensiones, schemas, tablas, vistas, funciones, triggers, índices)|
|02_dml	|Cambios de datos (inserts, updates, deletes)|
|03_dcl	|Seguridad, permisos y control de acceso (roles, grants, políticas RLS)|
|04_tcl	|Operaciones transaccionales o de recuperación excepcionales|
|05_rollbacks	|Scripts de reversión (estructura espejo de las capas activas)|


## Ambientes y Puertos

|Rama	|Puerto	|Ambiente	|Contenedor|
|-------|-------|-----------|----------|
|develop	|5432	|Desarrollo	|accesorios-dm-postgres-dev|
|qa	|5433	|Calidad	|accesorios-dm-postgres-qa|
|main	|5434	|Producción	|accesorios-dm-postgres-prod|

## Requisitos

- Docker Desktop

- Docker Compose

- Git

- PowerShell (para scripts de verificación en Windows)

## Configuración Inicial

### 1. Clonar el repositorio

```bash
git clone https://github.com/SergioLosadaDev/accesorios-dm-database.git
cd accesorios-dm-database
```

### 2. Configurar variables de entorno (opcional)

```bash
cp .env.example .env
```

### 3. Levantar el ambiente de desarrollo

```bash
git checkout develop
docker-compose -f docker-compose.yml up -d
```

### 4. Verificar que todo funciona

```bash
.\verify-all.ps1
```

## Comandos Útiles

### Levantar ambientes

```bash
# Desarrollo (puerto 5432)
git checkout develop
docker-compose -f docker-compose.yml up -d

# QA (puerto 5433)
git checkout qa
docker-compose -f docker-compose.yml up -d

# Produccion(puerto 5434)
git checkout main
docker-compose -f docker-compose.yml up -d
```

### Ver logs de Liquibase

```bash
docker-compose -f docker-compose.yml logs liquibase
```

### Conectar a PostgreSQL

```bash
# Desarrollo
docker exec -it accesorios-dm-postgres-dev psql -U admin -d accesorios_dm_db

# QA
docker exec -it accesorios-dm-postgres-qa psql -U admin -d accesorios_dm_db

# Producción
docker exec -it accesorios-dm-postgres-main psql -U admin -d accesorios_dm_db
```

### Detener y limpiar

```bash
docker-compose -f docker-compose.yml down -v
```

## Scripts de Verificación

|Script|	Propósito|
|------|-------------|
|verify-all.ps1	|Verifica los 3 ambientes (develop, qa, main)|
|docs/scripts-verificacion/verify-develop.ps1	|Verifica solo develop|
|docs/scripts-verificacion/verify-qa.ps1	|Verifica solo QA|
|docs/scripts-verificacion/verify-main.ps1	|Verifica solo main|

## Estructura de la Base de Datos

### Schemas (8)

|Schema	|Propósito|
|-------|---------|
|security	|Roles y empleados|
|clientes	|Clientes|
|catalogo	|Categorías, materiales, productos, imágenes|
|promociones	|Promociones y relación con productos|
|ventas	|Carritos, pedidos y detalles|
|logistica	|Estados de pedido y historial|
|inventario	|Tipos de movimiento y movimientos de inventario|
|public	|Schema por defecto|

### Tablas (17)

|Tabla	|Schema|
|-------|------|
|rol	|security|
|empleado	|security|
|cliente	|clientes|
|categoria	|catalogo|
|material	|catalogo|
|producto	|catalogo|
|imagen_producto	|catalogo|
|promocion	|promociones|
|promocion_producto	|promociones|
|carrito	|ventas|
|item_carrito	|ventas|
|pedido	|ventas|
|detalle_pedido	|ventas|
|estado_pedido	|logistica|
|historial_estado_pedido	|logistica|
|tipo_movimiento	|inventario|
|inventario_movimiento	|inventario|

### Historias de Usuario Completadas

| HU       | Nombre                                                                    | Descripción                                                                                                                          |
| -------- | ------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| HU-01 | Habilitar extensión UUID en la base de datos                              | Activación de la extensión UUID para soportar generación de identificadores únicos dentro del sistema.                               |
| HU-02 | Creación de esquemas de base de datos                                     | Definición de esquemas lógicos para segmentar y organizar los dominios de información de la base de datos.                           |
| HU-03 | Definición de tipos de datos personalizados                               | Creación de la capa de tipos reutilizables y estructura inicial para tipos personalizados del sistema.                               |
| HU-04 | Creación completa de tablas base del sistema                              | Implementación de las tablas principales relacionadas con seguridad, clientes, catálogo, promociones, carrito, pedidos e inventario. |
| HU-05 | Creación de vistas de reporte del sistema                                 | Implementación de vistas SQL para consultas consolidadas y generación de reportes operativos.                                        |
| HU-06 | Implementación de funciones de actualización de inventario                | Desarrollo de funciones almacenadas encargadas de gestionar la lógica de actualización y control de stock.                           |
| HU-07 | Implementación de triggers de control de inventario                       | Automatización de procesos de inventario mediante triggers asociados a movimientos y operaciones del sistema.                        |
| HU-08 | Creación de índices de optimización y rendimiento                         | Implementación de índices para mejorar el rendimiento y optimizar tiempos de respuesta en consultas críticas.                        |
| HU-09 | Configuración de roles, permisos y políticas de seguridad                 | Configuración de roles, grants y políticas RLS para controlar el acceso y la seguridad de los datos.                                 |
| HU-10 | Inserción de datos iniciales del sistema                                  | Registro de datos semilla y configuraciones base necesarias para el funcionamiento inicial del sistema.                              |
| HU-11 | Implementación de scripts de rollback de base de datos                    | Incorporación de scripts de reversión para garantizar recuperación y control ante cambios en la base de datos.                       |
| HU-12 | Configuración de infraestructura y documentación técnica de base de datos | Integración de contenedorización, scripts operativos, configuración de entorno y documentación técnica del proyecto.                 |


## Flujo de Trabajo con Git

### Crear una nueva HU 

**Ejemplo con develop**

```bash
git checkout develop
git pull origin develop
git checkout -b HU-XX-develop-nombre
# Realizar cambios...
git add .
git commit -m "feat: HU-XX-develop-nombre descripcion"
git push origin HU-XX-develop-nombre
# Crear Pull Request a develop
```


## Convención de Commits

|Tipo	|Uso	|Ejemplo|
|-------|-------|-------|
|feat	|Nueva funcionalidad / HU	|feat: HU-DEV-JSA-01 crear estructura|
|fix	|Corrección de errores	|fix: corregir sintaxis|
|docs	|Documentación	|docs: actualizar README|
|chore	|Mantenimiento	|chore: actualizar .gitignore|

## Licencia

Proyecto interno de accesorios DM.
