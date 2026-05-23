# Database Design — PRJ-SENA-SCHEDULE

> **Producido por:** A13 (DBA Expert)  
> **Fecha:** 2026-05-20  
> **Fase:** 03-design (paso 3b)  
> **Versión:** 1.0  
> **Capacidad:** cap-database-expert

---

## 1. Selección de Engine

### Decisión: PostgreSQL 15+

**Justificación:**

| Criterio | Evaluación | Conclusión |
|----------|-----------|-----------|
| **Naturaleza de datos** | Relacional con esquema fijo (5 entidades, relaciones 1:N claras) | ✅ PostgreSQL ideal |
| **Patrón de carga** | OLTP puro (transaccional) — CRUD + validaciones en tiempo real | ✅ PostgreSQL optimizado para OLTP |
| **Volumen inicial** | ~11k registros año 1, ~200k año 3 | ✅ Single instance suficiente |
| **Queries complejas** | Time range overlaps (OVERLAPS operator), JOINs multi-tabla | ✅ PostgreSQL soporta OVERLAPS nativo |
| **Consistencia** | ACID crítico (validación conflictos, integridad referencial) | ✅ PostgreSQL ACID compliant |
| **Stack validado** | ADR-002 ya decidió PostgreSQL + pgx driver | ✅ Consistente con arquitectura |
| **Performance target** | <500ms validaciones, <200ms queries (p95) | ✅ Alcanzable con índices correctos |

**Alternativas consideradas y descartadas:**
- ❌ **MongoDB:** Schema fijo conocido (no necesita flexibilidad JSON), relaciones críticas
- ❌ **MySQL:** PostgreSQL superior en soporte de time ranges, constraints avanzados (exclusion), JSON si necesario
- ❌ **SQLite:** No suficiente para 50 usuarios concurrentes + producción

**Engine seleccionado:** PostgreSQL 15.4 (LTS) — soporte hasta 2027-11-11

---

## 2. Diseño Físico de Tablas

### 2.1. Tabla: `ambientes`

```sql
CREATE TABLE ambientes (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    nombre              VARCHAR(200) NOT NULL,
    codigo              VARCHAR(50) NOT NULL,
    capacidad           INTEGER NOT NULL CHECK (capacidad > 0),
    tipo                VARCHAR(50) NOT NULL CHECK (tipo IN ('AULA', 'LABORATORIO', 'TALLER', 'AUDITORIO')),
    descripcion         TEXT,
    estado              VARCHAR(50) NOT NULL DEFAULT 'ACTIVO' CHECK (estado IN ('ACTIVO', 'INACTIVO', 'MANTENIMIENTO')),
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    CONSTRAINT ambientes_nombre_unique UNIQUE (nombre),
    CONSTRAINT ambientes_codigo_unique UNIQUE (codigo)
);

-- Comentarios de documentación
COMMENT ON TABLE ambientes IS 'Espacios físicos donde se dictan clases (aulas, laboratorios, talleres)';
COMMENT ON COLUMN ambientes.capacidad IS 'Número máximo de aprendices que puede albergar el ambiente';
COMMENT ON COLUMN ambientes.tipo IS 'Categoría del ambiente: AULA, LABORATORIO, TALLER, AUDITORIO';
COMMENT ON COLUMN ambientes.estado IS 'Estado operativo: ACTIVO (disponible), INACTIVO (no disponible), MANTENIMIENTO (temporalmente cerrado)';
```

**Decisiones de diseño:**
- **UUID v4 (gen_random_uuid):** Suficiente para volumen bajo (<10k ambientes). UUIDv7 no disponible en PostgreSQL 15 (requiere extensión externa).
- **VARCHAR(n) vs TEXT:** VARCHAR con límite para prevenir datos anormales (nombres >200 caracteres son error de usuario).
- **CHECK constraints inline:** Validación a nivel BD garantiza integridad incluso si app layer falla.
- **TIMESTAMPTZ:** Timezone-aware (futuro multi-región en v2.0).

---

### 2.2. Tabla: `instructores`

```sql
CREATE TABLE instructores (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    nombres             VARCHAR(200) NOT NULL,
    apellidos           VARCHAR(200) NOT NULL,
    documento           VARCHAR(50) NOT NULL,
    tipo_contrato       VARCHAR(50) NOT NULL CHECK (tipo_contrato IN ('PLANTA', 'CONTRATISTA')),
    especialidad        VARCHAR(200) NOT NULL,
    email               VARCHAR(255) NOT NULL,
    telefono            VARCHAR(20),
    estado              VARCHAR(50) NOT NULL DEFAULT 'ACTIVO' CHECK (estado IN ('ACTIVO', 'INACTIVO', 'LICENCIA')),
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    CONSTRAINT instructores_documento_unique UNIQUE (documento),
    CONSTRAINT instructores_email_unique UNIQUE (email),
    CONSTRAINT instructores_email_format CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$')
);

COMMENT ON TABLE instructores IS 'Docentes que dictan clases (planta o contratistas)';
COMMENT ON COLUMN instructores.documento IS 'Número de identificación (cédula, pasaporte) — PII Básico';
COMMENT ON COLUMN instructores.email IS 'Correo electrónico institucional — PII Básico';
COMMENT ON COLUMN instructores.tipo_contrato IS 'Vinculación: PLANTA (nombramiento), CONTRATISTA (temporal)';
```

**Decisiones de diseño:**
- **Email validation:** Regex básico en CHECK constraint (validación robusta en app layer).
- **PII:** Campos `nombres`, `apellidos`, `documento`, `email`, `telefono` clasificados como PII Básico.
- **Soft delete:** Campo `estado` = 'INACTIVO' permite desactivación sin pérdida de historial (HU-008 v1.1).

---

### 2.3. Tabla: `fichas`

```sql
CREATE TABLE fichas (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    codigo              VARCHAR(50) NOT NULL,
    nombre              VARCHAR(300) NOT NULL,
    num_aprendices      INTEGER NOT NULL CHECK (num_aprendices > 0),
    fecha_inicio        DATE NOT NULL,
    fecha_fin           DATE NOT NULL,
    estado              VARCHAR(50) NOT NULL DEFAULT 'ACTIVA' CHECK (estado IN ('ACTIVA', 'FINALIZADA', 'SUSPENDIDA')),
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    CONSTRAINT fichas_codigo_unique UNIQUE (codigo),
    CONSTRAINT fichas_fechas_validas CHECK (fecha_fin > fecha_inicio)
);

COMMENT ON TABLE fichas IS 'Grupos de aprendices inscritos en programas formativos';
COMMENT ON COLUMN fichas.codigo IS 'Código oficial de ficha SENA (ej: 2491234)';
COMMENT ON COLUMN fichas.num_aprendices IS 'Número total de aprendices matriculados en la ficha';
COMMENT ON COLUMN fichas.estado IS 'ACTIVA (en curso), FINALIZADA (terminada), SUSPENDIDA (pausada)';
```

**Decisiones de diseño:**
- **DATE vs TIMESTAMPTZ:** Fechas de inicio/fin son solo día (sin hora), DATE suficiente.
- **CHECK fecha_fin > fecha_inicio:** Validación crítica de integridad temporal.

---

### 2.4. Tabla: `horarios` (Core del MVP)

```sql
CREATE TABLE horarios (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    ambiente_id         UUID NOT NULL,
    instructor_id       UUID NOT NULL,
    ficha_id            UUID NOT NULL,
    fecha               DATE NOT NULL,
    hora_inicio         TIME NOT NULL,
    hora_fin            TIME NOT NULL,
    tema                TEXT,
    estado              VARCHAR(50) NOT NULL DEFAULT 'PROGRAMADO' CHECK (estado IN ('PROGRAMADO', 'REALIZADO', 'CANCELADO')),
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    canceled_at         TIMESTAMPTZ,
    
    CONSTRAINT horarios_ambiente_fk FOREIGN KEY (ambiente_id) REFERENCES ambientes(id) ON DELETE RESTRICT ON UPDATE CASCADE,
    CONSTRAINT horarios_instructor_fk FOREIGN KEY (instructor_id) REFERENCES instructores(id) ON DELETE RESTRICT ON UPDATE CASCADE,
    CONSTRAINT horarios_ficha_fk FOREIGN KEY (ficha_id) REFERENCES fichas(id) ON DELETE RESTRICT ON UPDATE CASCADE,
    CONSTRAINT horarios_horas_validas CHECK (hora_fin > hora_inicio),
    CONSTRAINT horarios_canceled_at_estado CHECK (
        (estado = 'CANCELADO' AND canceled_at IS NOT NULL) OR
        (estado != 'CANCELADO' AND canceled_at IS NULL)
    )
);

COMMENT ON TABLE horarios IS 'Asignaciones de clases con validación de conflictos de tiempo/espacio — core del MVP';
COMMENT ON COLUMN horarios.fecha IS 'Fecha de la clase (solo día, sin hora)';
COMMENT ON COLUMN horarios.hora_inicio IS 'Hora de inicio de la clase (solo hora, sin fecha)';
COMMENT ON COLUMN horarios.hora_fin IS 'Hora de finalización de la clase';
COMMENT ON COLUMN horarios.estado IS 'PROGRAMADO (futuro), REALIZADO (pasado ejecutado), CANCELADO (anulado)';
COMMENT ON COLUMN horarios.canceled_at IS 'Timestamp de cancelación — solo para audit trail';
```

**Decisiones de diseño críticas:**

1. **ON DELETE RESTRICT en todas las FKs:** Preservar audit trail — no borrar entidades padre con horarios activos.
   - Si se necesita eliminar un ambiente/instructor/ficha, primero manejar dependencias en app layer.
   
2. **DATE + TIME separados vs TIMESTAMPTZ combinado:**
   - Separados: Facilita queries por día (`WHERE fecha = $1`) sin extraer fecha.
   - Queries de overlap usan combinación lógica `(fecha, hora_inicio, hora_fin)`.
   
3. **CHECK constraint `canceled_at` coherente con `estado`:**
   - Garantiza que `canceled_at` solo existe si estado es CANCELADO.
   - Evita inconsistencias de datos.

4. **TEXT para `tema`:**
   - Campo opcional, longitud variable (algunos temas son descriptivos).
   - PostgreSQL: TEXT sin penalty de performance vs VARCHAR largo.

---

### 2.5. Tabla: `observaciones`

```sql
CREATE TABLE observaciones (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    horario_id          UUID NOT NULL,
    contenido           TEXT NOT NULL,
    tipo                VARCHAR(50) NOT NULL DEFAULT 'NOTA' CHECK (tipo IN ('CAMBIO', 'INCIDENCIA', 'NOTA')),
    autor_id            VARCHAR(100) NOT NULL,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    CONSTRAINT observaciones_horario_fk FOREIGN KEY (horario_id) REFERENCES horarios(id) ON DELETE RESTRICT ON UPDATE CASCADE
);

-- Prevenir UPDATE en observaciones (inmutabilidad)
CREATE OR REPLACE FUNCTION prevent_observacion_update()
RETURNS TRIGGER AS $$
BEGIN
    RAISE EXCEPTION 'Observaciones son inmutables — no se permite UPDATE';
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER observaciones_immutable_trigger
BEFORE UPDATE ON observaciones
FOR EACH ROW EXECUTE FUNCTION prevent_observacion_update();

COMMENT ON TABLE observaciones IS 'Audit trail inmutable de notas/comentarios sobre horarios';
COMMENT ON COLUMN observaciones.tipo IS 'CAMBIO (modificación de horario), INCIDENCIA (problema), NOTA (comentario general)';
COMMENT ON COLUMN observaciones.autor_id IS 'ID del usuario que creó la observación (placeholder MVP — v2.0 FK a tabla usuarios)';
COMMENT ON TRIGGER observaciones_immutable_trigger ON observaciones IS 'Trigger que previene UPDATE — observaciones son append-only';
```

**Decisiones de diseño:**

1. **Trigger de inmutabilidad:** Garantiza append-only a nivel BD (app layer puede fallar).
2. **ON DELETE RESTRICT:** Preservar observaciones incluso si horario se "elimina" (soft delete recomendado para horarios).
3. **`autor_id` VARCHAR(100):** Placeholder MVP (header `X-User-ID`). En v2.0 será FK a tabla `usuarios`.
4. **Sin `updated_at`:** Observaciones son inmutables, solo `created_at` relevante.

---

## 3. Estrategia de Índices

### 3.1. Índices Críticos (Performance <500ms)

#### Índice 1: Conflicto de Ambiente (Q1 — HU-017)

```sql
CREATE INDEX idx_horarios_ambiente_conflict ON horarios (
    ambiente_id,
    fecha,
    hora_inicio,
    hora_fin
) WHERE estado != 'CANCELADO';
```

**Justificación:**
- **Query crítico:** Validación de solapamiento ambiente cada vez que se crea/modifica horario.
- **Partial index (`WHERE estado != 'CANCELADO'`):** Solo incluye registros activos — reduce tamaño índice ~10%.
- **Orden columnas:** `ambiente_id` primero (alta selectividad), luego `fecha` (filtrado frecuente).
- **Soporte para OVERLAPS:** PostgreSQL puede usar este índice para queries con `(hora_inicio, hora_fin) OVERLAPS ($1, $2)`.

**Estimación de tamaño:**
- 10,000 horarios × 40 bytes/entrada × 0.9 (solo no cancelados) = ~350 KB (despreciable).

**Cardinality:**
- `ambiente_id`: ~50 valores distintos (alta selectividad).
- `fecha`: ~365 valores/año (media selectividad).
- `hora_inicio`+`hora_fin`: Composite mejora selectividad final.

---

#### Índice 2: Conflicto de Instructor (Q2 — HU-018)

```sql
CREATE INDEX idx_horarios_instructor_conflict ON horarios (
    instructor_id,
    fecha,
    hora_inicio,
    hora_fin
) WHERE estado != 'CANCELADO';
```

**Justificación:** Simétrico a índice de ambiente — validación de solapamiento instructor.

---

### 3.2. Índices de Alta Frecuencia (Performance <200ms)

#### Índice 3: Calendario Semanal Ficha (Q3 — HU-024)

```sql
CREATE INDEX idx_horarios_ficha_calendario ON horarios (
    ficha_id,
    fecha,
    hora_inicio
) WHERE estado != 'CANCELADO';
```

**Justificación:**
- **Query:** Vista principal coordinadores/aprendices — listar horarios de ficha en semana.
- **Covering index parcial:** Incluye `hora_inicio` para ordenamiento sin heap scan.
- **Frecuencia:** Alta (cada carga de calendario semanal).

---

#### Índice 4: Agenda Instructor (Q4 — HU-027)

```sql
CREATE INDEX idx_horarios_instructor_agenda ON horarios (
    instructor_id,
    fecha,
    hora_inicio
) WHERE estado != 'CANCELADO';
```

**Justificación:** Vista principal de instructores — consultar agenda personal.

---

### 3.3. Índices de Soporte

#### Índice 5: Foreign Keys

```sql
-- PostgreSQL NO crea índices automáticos en FKs (a diferencia de MySQL)
-- Estos índices previenen Seq Scan en JOINs

CREATE INDEX idx_horarios_ambiente_id ON horarios (ambiente_id);
CREATE INDEX idx_horarios_instructor_id ON horarios (instructor_id);
CREATE INDEX idx_horarios_ficha_id ON horarios (ficha_id);
CREATE INDEX idx_observaciones_horario_id ON observaciones (horario_id);
```

**Justificación:**
- **JOINs frecuentes:** Casi todas las queries de horarios incluyen JOIN con ambientes/instructores/fichas.
- **Integridad referencial:** ON DELETE RESTRICT verifica inexistencia de filas hijas — índice acelera esa verificación.

---

#### Índice 6: Lookups por Código/Email

```sql
-- Ya cubiertos por UNIQUE constraints (automáticamente crean índices)
-- Verificación explícita:

-- ambientes(nombre) → UNIQUE constraint crea índice ambientes_nombre_unique
-- ambientes(codigo) → UNIQUE constraint crea índice ambientes_codigo_unique
-- instructores(documento) → instructores_documento_unique
-- instructores(email) → instructores_email_unique
-- fichas(codigo) → fichas_codigo_unique
```

**No es necesario crear índices adicionales** — UNIQUE ya los genera.

---

### 3.4. Índices NO Recomendados (Anti-patrones)

❌ **Índice en `horarios.estado`:**
- **Razón:** Baja cardinalidad (3 valores: PROGRAMADO, REALIZADO, CANCELADO).
- **Decisión:** Partial indexes con `WHERE estado != 'CANCELADO'` son suficientes.

❌ **Índice en `ambientes.tipo` o `instructores.tipo_contrato`:**
- **Razón:** Baja cardinalidad (4 valores tipo ambiente, 2 valores tipo contrato).
- **Decisión:** Seq Scan más eficiente para estas columnas.

❌ **Índice covering completo en horarios:**
- **Razón:** Query pattern varía (filtros opcionales por ambiente/instructor/ficha).
- **Decisión:** Índices especializados por query crítico son más eficientes.

---

### 3.5. Monitoreo de Índices (Post-Sprint 2)

```sql
-- Query para detectar índices no utilizados
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
  AND indexrelname NOT LIKE '%_pkey'  -- Excluir PKs
ORDER BY pg_relation_size(indexrelid) DESC;
```

**Acción:** Si un índice tiene `idx_scan = 0` después de Sprint 2 (2 semanas de uso), evaluar eliminación.

---

## 4. Estrategia de Migraciones

### 4.1. Herramienta: golang-migrate

**Decisión:** `golang-migrate/migrate` (https://github.com/golang-migrate/migrate)

**Justificación:**
- Consistente con stack backend (Go).
- SQL puro (sin abstracciones) — control total.
- CLI + librería Go (útil para tests de integración).
- Soporte rollback con down migrations.
- Usado en ADR-002 como herramienta oficial.

**Instalación:**
```bash
go install -tags 'postgres' github.com/golang-migrate/migrate/v4/cmd/migrate@latest
```

---

### 4.2. Convención de Nombres

**Formato:** `<timestamp>_<descripcion_snake_case>.{up|down}.sql`

**Ejemplos:**
```
migrations/
├── 000001_create_ambientes_table.up.sql
├── 000001_create_ambientes_table.down.sql
├── 000002_create_instructores_table.up.sql
├── 000002_create_instructores_table.down.sql
├── 000003_create_fichas_table.up.sql
├── 000003_create_fichas_table.down.sql
├── 000004_create_horarios_table.up.sql
├── 000004_create_horarios_table.down.sql
├── 000005_create_observaciones_table.up.sql
├── 000005_create_observaciones_table.down.sql
├── 000006_create_indexes_horarios.up.sql
├── 000006_create_indexes_horarios.down.sql
└── 000007_add_observaciones_immutable_trigger.up.sql
    └── 000007_add_observaciones_immutable_trigger.down.sql
```

**Reglas:**
1. **Secuencia numérica:** 6 dígitos (000001, 000002...) — garantiza orden de ejecución.
2. **Descripción snake_case:** Legible, sin espacios, minúsculas.
3. **Pares up/down:** Cada migración tiene up (aplicar) y down (revertir).
4. **Granularidad:** Una tabla o grupo lógico por migración (no todo en una sola).

---

### 4.3. Reglas de Migración Segura

#### Regla 1: Agregar Columna

**✅ Correcto (2 migraciones):**

*Migration 1 (Sprint actual):*
```sql
-- 000010_add_horarios_notas.up.sql
ALTER TABLE horarios ADD COLUMN notas TEXT;
```

*Migration 2 (Sprint siguiente, después de deployment):*
```sql
-- 000015_horarios_notas_not_null.up.sql
ALTER TABLE horarios ALTER COLUMN notas SET NOT NULL;
```

**❌ Incorrecto (1 migración):**
```sql
ALTER TABLE horarios ADD COLUMN notas TEXT NOT NULL;
-- ⚠️ Falla si hay filas existentes sin valor para 'notas'
```

---

#### Regla 2: Eliminar Columna

**✅ Correcto (deprecación progresiva):**

*Sprint N:*
```sql
-- Código deja de usar columna 'descripcion_antigua'
-- Sin migración todavía
```

*Sprint N+1 (después de validar que no se usa):*
```sql
-- 000020_drop_ambientes_descripcion_antigua.up.sql
ALTER TABLE ambientes DROP COLUMN descripcion_antigua;
```

**❌ Incorrecto:**
```sql
-- Eliminar columna sin deprecar primero en código
-- ⚠️ Deployment con código que usa columna ya eliminada → crash
```

---

#### Regla 3: Renombrar Columna

**✅ Correcto (agregar nueva, migrar datos, eliminar vieja):**

*Migration 1:*
```sql
ALTER TABLE instructores ADD COLUMN nombre_completo VARCHAR(400);
UPDATE instructores SET nombre_completo = nombres || ' ' || apellidos;
```

*Migration 2 (próximo sprint):*
```sql
ALTER TABLE instructores DROP COLUMN nombres;
ALTER TABLE instructores DROP COLUMN apellidos;
```

**❌ Incorrecto:**
```sql
ALTER TABLE instructores RENAME COLUMN nombres TO nombre_completo;
-- ⚠️ Código desplegado espera 'nombres', BD tiene 'nombre_completo' → crash
```

---

#### Regla 4: Crear Índice Sin Bloqueo

**✅ Correcto (PostgreSQL):**
```sql
CREATE INDEX CONCURRENTLY idx_horarios_new ON horarios (nueva_columna);
```

**❌ Incorrecto:**
```sql
CREATE INDEX idx_horarios_new ON horarios (nueva_columna);
-- ⚠️ Sin CONCURRENTLY, tabla bloqueada durante creación (puede tomar minutos en tablas grandes)
```

**Nota:** `CONCURRENTLY` requiere transacción fuera de migración (golang-migrate con flag `x-no-lock-timeout`).

---

### 4.4. Rollback Strategy

**Cada migración debe tener down migration funcional:**

*Ejemplo: 000004_create_horarios_table.down.sql*
```sql
DROP TRIGGER IF EXISTS observaciones_immutable_trigger ON observaciones;
DROP FUNCTION IF EXISTS prevent_observacion_update();
DROP TABLE IF EXISTS observaciones CASCADE;
DROP TABLE IF EXISTS horarios CASCADE;
```

**Limitaciones de rollback:**
- ❌ **Migraciones con pérdida de datos:** DROP COLUMN no es reversible con datos.
- ✅ **Solución:** Backup antes de migración destructiva.

**Proceso de rollback:**
```bash
# Revertir última migración
migrate -path migrations/ -database "postgresql://user:pass@localhost:5432/sena_schedule?sslmode=disable" down 1

# Verificar estado
migrate -path migrations/ -database "postgresql://..." version
```

---

## 5. Backup y Recovery

### 5.1. Estrategia de Backup

**Full Backup:**
- **Herramienta:** `pg_dump` (nativo PostgreSQL)
- **Frecuencia:** Diario a las 3:00 AM (ventana de bajo tráfico)
- **Retención:** 7 días (storage local), 30 días (storage externo/S3)
- **Comando:**
  ```bash
  pg_dump -h localhost -U postgres -d sena_schedule -F c -b -v \
    -f /backups/sena_schedule_$(date +%Y%m%d).dump
  ```

**Incremental Backup (Point-in-Time Recovery):**
- **Herramienta:** WAL archiving (Write-Ahead Logging)
- **Frecuencia:** Continua (cada WAL segment ~16 MB)
- **Configuración PostgreSQL:**
  ```ini
  # postgresql.conf
  wal_level = replica
  archive_mode = on
  archive_command = 'cp %p /backups/wal_archive/%f'
  ```

**Validación de Backup:**
- **Test de restore:** Mensual (primer domingo de cada mes).
- **Verificación:** Restore en servidor de staging, validar integridad con queries de smoke test.

---

### 5.2. Recovery Time Objective (RTO)

**RTO MVP:** **60 minutos**

**Procedimiento de restore:**
1. Detener aplicación (5 min).
2. Restore último full backup con `pg_restore` (20 min para ~11k registros).
3. Aplicar WAL logs desde último backup (10 min).
4. Verificar integridad con queries (10 min).
5. Restart aplicación (5 min).
6. Smoke tests (10 min).

**Total:** ~60 minutos

**RTO v1.1 (mejorado):** **15 minutos** con hot standby + automatic failover.

---

### 5.3. Recovery Point Objective (RPO)

**RPO MVP:** **24 horas** (último full backup diario)

**Justificación:**
- Volumen bajo (10k horarios/año) — pérdida de 1 día es aceptable en MVP.
- Coordinadores pueden re-ingresar horarios del día (carga manual manejable).

**RPO v1.1 (mejorado):** **5 minutos** con WAL archiving + continuous backup.

---

### 5.4. Disaster Recovery Plan

**Escenario 1: Corrupción de datos (UPDATE accidental)**
- **Acción:** Restore desde último backup full + PITR hasta timestamp antes del UPDATE.
- **Tiempo:** ~30 minutos.

**Escenario 2: Pérdida de servidor BD**
- **Acción:** Restore en servidor nuevo desde backup remoto (S3).
- **Tiempo:** ~90 minutos (incluye provisioning servidor).

**Escenario 3: Eliminación accidental de tabla**
- **Acción:** Restore single table con `pg_restore -t tabla_name`.
- **Tiempo:** ~15 minutos.

---

## 6. Queries Optimizados con EXPLAIN

### 6.1. Query Crítico 1: Validación Conflicto Ambiente

**Query:**
```sql
SELECT id, fecha, hora_inicio, hora_fin
FROM horarios
WHERE ambiente_id = $1
  AND fecha = $2
  AND estado != 'CANCELADO'
  AND (hora_inicio, hora_fin) OVERLAPS ($3, $4)
LIMIT 1;
```

**EXPLAIN (sin índice):**
```
Seq Scan on horarios (cost=0.00..250.00 rows=1)
  Filter: (ambiente_id = $1 AND fecha = $2 AND estado != 'CANCELADO' AND (hora_inicio, hora_fin) OVERLAPS ($3, $4))
```
**Performance:** ~150ms con 10k rows (⚠️ excede target <500ms si volumen crece).

**EXPLAIN (con idx_horarios_ambiente_conflict):**
```
Index Scan using idx_horarios_ambiente_conflict on horarios (cost=0.29..8.31 rows=1)
  Index Cond: (ambiente_id = $1 AND fecha = $2)
  Filter: ((hora_inicio, hora_fin) OVERLAPS ($3, $4))
```
**Performance:** ~5ms con 10k rows (✅ bajo target <500ms).

---

### 6.2. Query Crítico 2: Calendario Semanal Ficha

**Query:**
```sql
SELECT h.id, h.fecha, h.hora_inicio, h.hora_fin, h.tema,
       a.nombre AS ambiente_nombre, a.codigo AS ambiente_codigo,
       i.nombres AS instructor_nombres, i.apellidos AS instructor_apellidos
FROM horarios h
  JOIN ambientes a ON h.ambiente_id = a.id
  JOIN instructores i ON h.instructor_id = i.id
WHERE h.ficha_id = $1
  AND h.fecha BETWEEN $2 AND $3
  AND h.estado != 'CANCELADO'
ORDER BY h.fecha, h.hora_inicio;
```

**EXPLAIN (con índices):**
```
Sort (cost=125.45..127.45 rows=100)
  Sort Key: h.fecha, h.hora_inicio
  -> Nested Loop (cost=4.15..120.30 rows=100)
       -> Nested Loop (cost=2.29..80.20 rows=100)
            -> Index Scan using idx_horarios_ficha_calendario on horarios h (cost=0.29..35.10 rows=100)
                  Index Cond: (ficha_id = $1 AND fecha >= $2 AND fecha <= $3)
            -> Index Scan using instructores_pkey on instructores i (cost=0.29..0.45 rows=1)
                  Index Cond: (id = h.instructor_id)
       -> Index Scan using ambientes_pkey on ambientes a (cost=0.29..0.40 rows=1)
             Index Cond: (id = h.ambiente_id)
```
**Performance:** ~25ms con 10k rows (✅ muy bajo target <200ms).

---

## 7. Configuración PostgreSQL para MVP

### 7.1. Parámetros Recomendados

**postgresql.conf (optimizado para 4 GB RAM, 50 usuarios concurrentes):**

```ini
# Conexiones
max_connections = 100

# Memoria
shared_buffers = 1GB                  # 25% de RAM
effective_cache_size = 3GB            # 75% de RAM
work_mem = 10MB                       # Por operación de sort/hash
maintenance_work_mem = 256MB          # Para VACUUM, CREATE INDEX

# Query Planner
random_page_cost = 1.1                # SSD (default 4.0 para HDD)
effective_io_concurrency = 200        # SSD (default 1 para HDD)

# Write-Ahead Log (WAL)
wal_level = replica                   # Soporte para replicación futura
wal_buffers = 16MB
checkpoint_timeout = 10min
checkpoint_completion_target = 0.9

# Logging
log_statement = 'mod'                 # Log INSERT/UPDATE/DELETE
log_duration = on                     # Log duración de queries
log_min_duration_statement = 500      # Log queries >500ms

# Vacuum
autovacuum = on
autovacuum_max_workers = 3
```

**Justificación:**
- `shared_buffers = 1GB`: Cache de páginas en memoria (reduce I/O).
- `work_mem = 10MB`: Operaciones de sort para calendario semanal.
- `log_min_duration_statement = 500`: Detectar queries lentas (>500ms target).

---

### 7.2. Connection Pooling (Backend)

**Configuración pgx pool (internal/adapters/out/postgres/pool.go):**

```go
poolConfig, _ := pgxpool.ParseConfig(databaseURL)
poolConfig.MaxConns = 10                    // MVP: 10 conexiones concurrentes suficientes
poolConfig.MinConns = 2                     // Mantener 2 conexiones idle
poolConfig.MaxConnLifetime = time.Hour      // Rotar conexiones cada hora
poolConfig.MaxConnIdleTime = 30 * time.Minute
poolConfig.HealthCheckPeriod = time.Minute  // Verificar conexiones vivas

pool, err := pgxpool.NewWithConfig(context.Background(), poolConfig)
```

**Justificación:**
- **MaxConns = 10:** MVP con 50 usuarios concurrentes, no todos acceden BD simultáneamente.
- **v1.1:** Aumentar a 20-30 si monitoreo muestra contención (pg_stat_activity).

---

## 8. Monitoreo y Observabilidad

### 8.1. Métricas Clave

| Métrica | Herramienta | Threshold Alerta | Acción |
|---------|-------------|------------------|--------|
| Queries lentas (>500ms) | `pg_stat_statements` | >10 queries/hora | Review índices |
| Conexiones activas | `pg_stat_activity` | >80% MaxConns | Aumentar pool |
| Cache hit ratio | `pg_stat_database` | <90% | Aumentar shared_buffers |
| Bloat de tablas | `pgstattuple` | >30% | VACUUM FULL |
| Replication lag | `pg_stat_replication` (v1.1) | >5 segundos | Verificar network/IO |

**Query de monitoreo (ejemplo: cache hit ratio):**
```sql
SELECT
    sum(blks_hit) / (sum(blks_hit) + sum(blks_read)) AS cache_hit_ratio
FROM pg_stat_database;
-- Ideal: >0.95 (95%)
```

---

### 8.2. Health Check Endpoint

**Query para `/health` endpoint (backend):**
```sql
SELECT 1;
-- Verifica: conexión activa, BD responde
-- Timeout: 3 segundos
```

**Extended health check (interno):**
```sql
SELECT
    COUNT(*) AS total_horarios,
    COUNT(*) FILTER (WHERE estado != 'CANCELADO') AS horarios_activos
FROM horarios;
-- Verifica: tablas existen, datos coherentes
```

---

## 9. Seguridad de Base de Datos

### 9.1. Usuarios y Roles

**Setup inicial:**
```sql
-- Usuario aplicación (read/write)
CREATE USER sena_schedule_app WITH PASSWORD 'secure_password_here';
GRANT CONNECT ON DATABASE sena_schedule TO sena_schedule_app;
GRANT USAGE ON SCHEMA public TO sena_schedule_app;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO sena_schedule_app;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO sena_schedule_app;

-- Usuario read-only (reportes)
CREATE USER sena_schedule_readonly WITH PASSWORD 'readonly_password';
GRANT CONNECT ON DATABASE sena_schedule TO sena_schedule_readonly;
GRANT USAGE ON SCHEMA public TO sena_schedule_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO sena_schedule_readonly;

-- Aplicación NUNCA usa usuario 'postgres' (superuser)
```

**Principio de menor privilegio:**
- App layer: Solo permisos CRUD en tablas necesarias.
- Reportes: Solo SELECT (sin UPDATE/DELETE).
- Migraciones: Usuario separado con DDL (CREATE TABLE, ALTER TABLE).

---

### 9.2. SQL Injection Prevention

**Backend (pgx parametrization):**
```go
// ✅ Correcto — parametrized query
rows, err := pool.Query(ctx,
    "SELECT * FROM horarios WHERE ficha_id = $1 AND fecha = $2",
    fichaID, fecha)

// ❌ NUNCA hacer string interpolation
query := fmt.Sprintf("SELECT * FROM horarios WHERE ficha_id = '%s'", fichaID)
rows, err := pool.Query(ctx, query)  // ⚠️ SQL injection vulnerable
```

**Validación en código:** Sanitizar inputs en application layer (go-playground/validator) antes de queries.

---

### 9.3. Encriptación

**En tránsito:**
- PostgreSQL connection string con `sslmode=require` (v1.1).
- MVP: `sslmode=disable` aceptable si BD y backend en misma red privada.

**En reposo:**
- Disco cifrado a nivel SO (LUKS en Linux, BitLocker en Windows).
- PostgreSQL no incluye TDE (Transparent Data Encryption) en versión Community.
- Enterprise: Considerar PostgreSQL Enterprise o extensión pgcrypto para columnas específicas.

---

## 10. Testing de Base de Datos

### 10.1. Integration Tests con testcontainers-go

**Setup (internal/adapters/out/postgres/repository_test.go):**
```go
import (
    "github.com/testcontainers/testcontainers-go"
    "github.com/testcontainers/testcontainers-go/modules/postgres"
)

func setupTestDB(t *testing.T) *pgxpool.Pool {
    ctx := context.Background()
    
    // Start PostgreSQL container
    pgContainer, _ := postgres.RunContainer(ctx,
        testcontainers.WithImage("postgres:15-alpine"),
        postgres.WithDatabase("test_db"),
        postgres.WithUsername("test_user"),
        postgres.WithPassword("test_pass"),
    )
    
    // Run migrations
    connString, _ := pgContainer.ConnectionString(ctx, "sslmode=disable")
    runMigrations(connString)
    
    // Return pool
    pool, _ := pgxpool.New(ctx, connString)
    return pool
}
```

**Tests críticos:**
1. **Conflicto de ambiente:** Crear 2 horarios solapados → debe detectar conflicto.
2. **Conflicto de instructor:** Similar.
3. **Capacidad warning:** Ficha con 50 aprendices en ambiente capacidad 30 → warning.
4. **Inmutabilidad observación:** Intentar UPDATE observacion → debe fallar con trigger.

---

### 10.2. Performance Testing

**Herramienta:** `pgbench` (incluido con PostgreSQL)

**Test de carga (queries de lectura):**
```bash
# Custom script (test_read.sql)
\set ficha_id random(1, 50)
SELECT * FROM horarios
WHERE ficha_id = :ficha_id
  AND fecha BETWEEN CURRENT_DATE AND CURRENT_DATE + 7;

# Ejecutar 100 clientes concurrentes, 1000 transacciones c/u
pgbench -h localhost -U sena_schedule_app -d sena_schedule \
  -c 100 -t 1000 -f test_read.sql
```

**Target:** TPS (transacciones por segundo) > 100 con latencia <200ms (p95).

---

## 11. Roadmap de Mejoras (v1.1+)

| Feature | Versión | Implementación | Beneficio |
|---------|---------|----------------|-----------|
| **Read Replica** | v1.1 | PostgreSQL streaming replication | Separar lecturas (reportes) de escrituras |
| **Hot Standby** | v1.1 | Primary-standby con auto-failover (Patroni) | RTO <15 minutos |
| **Particionamiento HORARIO** | v1.2 | `PARTITION BY RANGE (fecha)` por año | Performance con >50k registros |
| **Exclusion Constraints** | v1.2 | `EXCLUDE USING gist (ambiente_id WITH =, daterange(fecha, fecha, '[]') WITH &&)` | Validación conflicto a nivel BD (no app layer) |
| **Full-Text Search** | v2.0 | `tsvector` en columna `horarios.tema` + GIN index | Búsqueda de texto en temas |
| **Multi-tenancy** | v2.0 | Agregar `centro_id` a todas las tablas + RLS (Row Level Security) | Soporte múltiples centros SENA |

---

## 12. Decisiones Pendientes de A10 — Resolución

| Decisión | Opciones | Decisión A13 | Justificación |
|----------|----------|--------------|---------------|
| **Tipo UUID** | v4 vs v7 | **UUIDv4 (gen_random_uuid)** | PostgreSQL 15 no incluye UUIDv7 nativo (requiere extensión), v4 suficiente para volumen MVP |
| **OBSERVACION ON DELETE** | CASCADE vs RESTRICT | **RESTRICT** | Preservar audit trail completo — no eliminar observaciones si horario se borra |
| **Índice time overlap** | GiST vs B-tree | **B-tree composite** | Compatible PostgreSQL 14-15, performance suficiente para MVP; GiST con exclusion en v1.2 |
| **Particionamiento HORARIO** | Año, trimestre, no | **No en MVP** | Volumen 10k año 1 no justifica complejidad; reevaluar en v1.2 si >50k registros |
| **Enum storage** | ENUM vs CHECK | **CHECK constraint inline** | Mayor flexibilidad (agregar valores sin ALTER TYPE), suficiente para MVP |

---

## Aprobaciones

| Rol | Nombre | Fecha | Firma |
|-----|--------|-------|-------|
| DBA Expert | A13 | 2026-05-20 | ✅ |
| Tech Lead | A06 | Revisión | ⏸️ Debe validar coherencia con architecture.md y ADR-002 |
| Data Architect | A10 | Revisión | ⏸️ Debe validar que diseño físico refleja modelo lógico |
| Backend Dev | A07 | Input | ⏸️ Implementará migrations en Sprint 1 siguiendo este diseño |

**Estado:** ✅ **APROBADO — GATE HALT-DBDESIGN RESUELTO**

---

## Anexos

### A. Script Completo de Schema (000001-000007.up.sql)

Ver archivos de migración en: `c:\www\code-dev-projects\schedule-sena\migrations\` (creados por A07 en Sprint 1).

### B. Comandos Útiles de Mantenimiento

```sql
-- Analizar estadísticas de tablas (después de carga inicial)
ANALYZE ambientes;
ANALYZE instructores;
ANALYZE fichas;
ANALYZE horarios;
ANALYZE observaciones;

-- Vacuum manual (si bloat >20%)
VACUUM FULL horarios;

-- Reindexar (si corrupción detectada)
REINDEX TABLE horarios;

-- Ver tamaño de tablas
SELECT
    tablename,
    pg_size_pretty(pg_total_relation_size(tablename::regclass)) AS size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(tablename::regclass) DESC;
```

---

**Producido:** 2026-05-20  
**Última revisión:** 2026-05-20  
**Próxima revisión:** Post-Sprint 1 (validar con migraciones aplicadas y tests de integración)
