# Data Model — PRJ-SENA-SCHEDULE

> **Producido por:** A10 (Data Architect)  
> **Fecha:** 2026-05-20  
> **Fase:** 03-design (paso 3a)  
> **Versión:** 1.0  
> **Capacidad:** cap-data-modeling

---

## 1. Visión del Modelo de Datos

### Objetivo

Diseñar un modelo de datos normalizado (3NF) para sistema de gestión de horarios SENA que soporte:
- CRUD de 4 entidades principales (Ambiente, Instructor, Ficha, Horario)
- Validación de conflictos de asignación (solapamiento de tiempo/espacio)
- Trazabilidad con observaciones inmutables
- Consultas optimizadas para calendarios y agendas

### Contexto Arquitectónico

**Patrón:** Hexagonal Architecture (P02) — ver [`pattern-guide.md`](pattern-guide.md)  
**Ubicación del modelo:** Capa `domain/entities` (Backend Go) — lógica de negocio pura  
**Persistencia:** PostgreSQL 15+ con driver pgx (SQL raw) — ver [`architecture.md`](architecture.md) y ADR-002

**Mapeo arquitectónico:**
- **Entities (este modelo)** → `internal/domain/entities/*.go`
- **Repository interfaces** → `internal/ports/out/*_repository.go`
- **Repository impl** → `internal/adapters/out/postgres/repositories/*_repo.go`

---

## 2. Diagrama Entidad-Relación (Conceptual)

```
┌────────────────┐            ┌────────────────┐
│    AMBIENTE    │            │   INSTRUCTOR   │
│                │            │                │
│ PK: id (UUID)  │            │ PK: id (UUID)  │
│ - nombre       │            │ - nombres      │
│ - codigo       │            │ - apellidos    │
│ - capacidad    │            │ - documento    │
│ - tipo         │            │ - tipo_contrato│
│ - estado       │            │ - especialidad │
│ - created_at   │            │ - email        │
│ - updated_at   │            │ - telefono     │
│                │            │ - estado       │
└───────┬────────┘            │ - created_at   │
        │                     │ - updated_at   │
        │                     └────────┬───────┘
        │                              │
        │ 1:N                          │ 1:N
        │                              │
        │        ┌─────────────────────┼────────────────┐
        │        │                     │                │
        │        │   ┌─────────────────▼──────┐         │
        │        │   │      FICHA             │         │
        │        │   │                        │         │
        │        │   │ PK: id (UUID)          │         │
        │        │   │ - codigo               │         │
        │        │   │ - nombre               │         │
        │        │   │ - num_aprendices       │         │
        │        │   │ - fecha_inicio         │         │
        │        │   │ - fecha_fin            │         │
        │        │   │ - estado               │         │
        │        │   │ - created_at           │         │
        │        │   │ - updated_at           │         │
        │        │   └──────────┬─────────────┘         │
        │        │              │                       │
        │        │              │ 1:N                   │
        │        │              │                       │
        │        │   ┌──────────▼─────────────┐         │
        └────────┼───►     HORARIO            │◄────────┘
                 │   │                        │
                 │   │ PK: id (UUID)          │
                 │   │ FK: ambiente_id        │
                 │   │ FK: instructor_id      │
                 │   │ FK: ficha_id           │
                 │   │ - fecha                │
                 │   │ - hora_inicio          │
                 │   │ - hora_fin             │
                 │   │ - tema                 │
                 │   │ - estado               │
                 │   │ - created_at           │
                 │   │ - updated_at           │
                 │   │ - canceled_at          │
                 │   └──────────┬─────────────┘
                 │              │
                 │              │ 1:N
                 │              │
                 │   ┌──────────▼─────────────┐
                 └───►    OBSERVACION         │
                     │                        │
                     │ PK: id (UUID)          │
                     │ FK: horario_id         │
                     │ - contenido            │
                     │ - tipo (enum)          │
                     │ - autor_id             │
                     │ - created_at           │
                     │ (INMUTABLE)            │
                     └────────────────────────┘
```

**Cardinalidades:**
- **Ambiente → Horario:** 1:N (un ambiente tiene muchos horarios)
- **Instructor → Horario:** 1:N (un instructor tiene muchos horarios)
- **Ficha → Horario:** 1:N (una ficha tiene muchos horarios)
- **Horario → Observacion:** 1:N (un horario tiene muchas observaciones)

---

## 3. Entidades Detalladas

### 3.1. AMBIENTE

**Propósito:** Representa espacios físicos donde se dictan clases (aulas, laboratorios, talleres).

**Atributos:**

| Nombre | Tipo Lógico | Obligatorio | Descripción | Constraints |
|--------|-------------|-------------|-------------|-------------|
| `id` | UUID | ✅ | Identificador único (PK) | Primary Key |
| `nombre` | String | ✅ | Nombre descriptivo del ambiente | Unique, max 200 caracteres |
| `codigo` | String | ✅ | Código alfanumérico (ej: "LAB-101") | Unique, max 50 caracteres |
| `capacidad` | Integer | ✅ | Número máximo de aprendices | >0 |
| `tipo` | Enum | ✅ | Tipo de ambiente | {AULA, LABORATORIO, TALLER, AUDITORIO} |
| `descripcion` | String | ❌ | Descripción adicional (ubicación, recursos) | max 500 caracteres |
| `estado` | Enum | ✅ | Estado operativo | {ACTIVO, INACTIVO, MANTENIMIENTO} |
| `created_at` | Timestamp | ✅ | Fecha de creación | Audit trail |
| `updated_at` | Timestamp | ✅ | Fecha de última modificación | Audit trail |

**Reglas de Negocio:**
- RN-AMBIENTE-001: `codigo` debe ser único en todo el sistema
- RN-AMBIENTE-002: `capacidad` debe ser >0
- RN-AMBIENTE-003: Ambientes en estado INACTIVO no pueden asignarse a horarios nuevos
- RN-AMBIENTE-004: Eliminar ambiente con horarios activos debe fallar (ON DELETE RESTRICT)

**Patrones de Consulta:**
- Listar ambientes con filtro por `estado` y `tipo` (HU-002)
- Buscar por `codigo` o `nombre` (autocomplete en frontend)
- Consultar disponibilidad en rango de fechas/horas (HU-029)

**PII:** ❌ Ninguno (datos públicos del centro)

---

### 3.2. INSTRUCTOR

**Propósito:** Representa instructores que dictan clases (planta o contratistas).

**Atributos:**

| Nombre | Tipo Lógico | Obligatorio | Descripción | Constraints |
|--------|-------------|-------------|-------------|-------------|
| `id` | UUID | ✅ | Identificador único (PK) | Primary Key |
| `nombres` | String | ✅ | Nombres del instructor | max 200 caracteres |
| `apellidos` | String | ✅ | Apellidos del instructor | max 200 caracteres |
| `documento` | String | ✅ | Número de identificación | Unique, max 50 caracteres |
| `tipo_contrato` | Enum | ✅ | Tipo de vinculación | {PLANTA, CONTRATISTA} |
| `especialidad` | String | ✅ | Área de conocimiento (ej: "Desarrollo Software") | max 200 caracteres |
| `email` | String | ✅ | Correo electrónico | Unique, formato email |
| `telefono` | String | ❌ | Teléfono de contacto | max 20 caracteres |
| `estado` | Enum | ✅ | Estado de vinculación | {ACTIVO, INACTIVO, LICENCIA} |
| `created_at` | Timestamp | ✅ | Fecha de registro | Audit trail |
| `updated_at` | Timestamp | ✅ | Fecha de última modificación | Audit trail |

**Reglas de Negocio:**
- RN-INSTRUCTOR-001: `documento` debe ser único en todo el sistema
- RN-INSTRUCTOR-002: `email` debe ser único
- RN-INSTRUCTOR-003: Instructores en estado INACTIVO no pueden asignarse a horarios nuevos
- RN-INSTRUCTOR-004: Desactivar instructor (soft delete) en lugar de eliminar (HU-008 v1.1)
- RN-INSTRUCTOR-005: Eliminar instructor con horarios activos debe fallar (ON DELETE RESTRICT)

**Patrones de Consulta:**
- Listar instructores con filtro por `estado`, `tipo_contrato`, `especialidad` (HU-006)
- Buscar por `nombres`, `apellidos`, `documento` (autocomplete)
- Consultar agenda personal: horarios asignados en rango de fechas (HU-027)

**PII:** ✅ **Sí** (nombres, apellidos, documento, email, teléfono)  
**Clasificación:** PII Básico — no sensible, pero sujeto a protección de datos personales RGPD.

---

### 3.3. FICHA

**Propósito:** Representa grupos de aprendices inscritos en un programa formativo.

**Atributos:**

| Nombre | Tipo Lógico | Obligatorio | Descripción | Constraints |
|--------|-------------|-------------|-------------|-------------|
| `id` | UUID | ✅ | Identificador único (PK) | Primary Key |
| `codigo` | String | ✅ | Código de ficha (ej: "2491234") | Unique, max 50 caracteres |
| `nombre` | String | ✅ | Nombre del programa formativo | max 300 caracteres |
| `num_aprendices` | Integer | ✅ | Número total de aprendices matriculados | >0 |
| `fecha_inicio` | Date | ✅ | Fecha de inicio del programa | |
| `fecha_fin` | Date | ✅ | Fecha de finalización del programa | >fecha_inicio |
| `estado` | Enum | ✅ | Estado del ciclo formativo | {ACTIVA, FINALIZADA, SUSPENDIDA} |
| `created_at` | Timestamp | ✅ | Fecha de creación | Audit trail |
| `updated_at` | Timestamp | ✅ | Fecha de última modificación | Audit trail |

**Reglas de Negocio:**
- RN-FICHA-001: `codigo` debe ser único en todo el sistema
- RN-FICHA-002: `fecha_fin` debe ser posterior a `fecha_inicio`
- RN-FICHA-003: `num_aprendices` debe ser >0
- RN-FICHA-004: Fichas en estado FINALIZADA no pueden tener horarios nuevos creados
- RN-FICHA-005: Finalizar ficha (cambio a estado FINALIZADA) es operación explícita (HU-012 v1.1)
- RN-FICHA-006: Eliminar ficha con horarios debe fallar (ON DELETE RESTRICT)

**Patrones de Consulta:**
- Listar fichas con filtro por `estado`, rango de `fecha_inicio` (HU-010)
- Buscar por `codigo` o `nombre` (autocomplete)
- Consultar calendario semanal: horarios de la ficha en semana específica (HU-024)

**PII:** ❌ Ninguno (datos del grupo, no individuales)

---

### 3.4. HORARIO

**Propósito:** Representa asignaciones de clases con validación de conflictos de tiempo/espacio.

**Atributos:**

| Nombre | Tipo Lógico | Obligatorio | Descripción | Constraints |
|--------|-------------|-------------|-------------|-------------|
| `id` | UUID | ✅ | Identificador único (PK) | Primary Key |
| `ambiente_id` | UUID | ✅ | FK a AMBIENTE | Foreign Key |
| `instructor_id` | UUID | ✅ | FK a INSTRUCTOR | Foreign Key |
| `ficha_id` | UUID | ✅ | FK a FICHA | Foreign Key |
| `fecha` | Date | ✅ | Fecha de la clase | |
| `hora_inicio` | Time | ✅ | Hora de inicio (ej: 08:00) | |
| `hora_fin` | Time | ✅ | Hora de finalización (ej: 10:00) | >hora_inicio |
| `tema` | String | ❌ | Tema o contenido de la clase | max 500 caracteres |
| `estado` | Enum | ✅ | Estado del horario | {PROGRAMADO, REALIZADO, CANCELADO} |
| `created_at` | Timestamp | ✅ | Fecha de creación | Audit trail |
| `updated_at` | Timestamp | ✅ | Fecha de última modificación | Audit trail |
| `canceled_at` | Timestamp | ❌ | Fecha de cancelación (si estado=CANCELADO) | Nullable |

**Reglas de Negocio:**
- **RN-HORARIO-001:** `hora_fin` debe ser posterior a `hora_inicio`
- **RN-HORARIO-002:** No puede haber dos horarios con el mismo `ambiente_id` en rangos de tiempo solapados (HU-017 — core del MVP)
- **RN-HORARIO-003:** No puede haber dos horarios con el mismo `instructor_id` en rangos de tiempo solapados (HU-018)
- **RN-HORARIO-004:** Si `num_aprendices` de ficha > `capacidad` de ambiente, generar warning (HU-019 — no bloqueante)
- **RN-HORARIO-005:** Cancelar horario actualiza `estado` a CANCELADO y `canceled_at` (HU-016)
- **RN-HORARIO-006:** Horarios cancelados no participan en validación de conflictos
- **RN-HORARIO-007:** Modificar horario debe revalidar conflictos (HU-015)
- **RN-HORARIO-008:** Eliminar horario debe ser soft delete (cambiar estado) o ON DELETE RESTRICT con observaciones

**Integridad Referencial:**
- `ambiente_id` → AMBIENTE.id (ON DELETE RESTRICT — no borrar ambientes con horarios)
- `instructor_id` → INSTRUCTOR.id (ON DELETE RESTRICT)
- `ficha_id` → FICHA.id (ON DELETE RESTRICT)

**Patrones de Consulta:**
- **Validación de conflictos ambiente:** Query con OVERLAPS time range (crítico — <500ms)
  ```sql
  SELECT * FROM horarios
  WHERE ambiente_id = $1
    AND fecha = $2
    AND estado != 'CANCELADO'
    AND (hora_inicio, hora_fin) OVERLAPS ($3, $4)
  ```
- **Validación de conflictos instructor:** Similar a conflicto ambiente
- **Calendario ficha:** Horarios de una ficha en rango de fechas (HU-024)
- **Agenda instructor:** Horarios de un instructor en rango de fechas (HU-027)
- **Listado con filtros:** Por fecha, ambiente, instructor, ficha, estado (HU-014)

**Índices Críticos (sugerencia para A13):**
- Composite index: `(ambiente_id, fecha, hora_inicio, hora_fin)` — para conflicto ambiente
- Composite index: `(instructor_id, fecha, hora_inicio, hora_fin)` — para conflicto instructor
- Index: `ficha_id, fecha` — para calendario ficha
- Index: `estado` — para filtros

**PII:** ❌ Ninguno (datos de asignación académica)

---

### 3.5. OBSERVACION

**Propósito:** Registro inmutable de notas/comentarios sobre horarios (auditoría).

**Atributos:**

| Nombre | Tipo Lógico | Obligatorio | Descripción | Constraints |
|--------|-------------|-------------|-------------|-------------|
| `id` | UUID | ✅ | Identificador único (PK) | Primary Key |
| `horario_id` | UUID | ✅ | FK a HORARIO | Foreign Key |
| `contenido` | String | ✅ | Texto de la observación | max 2000 caracteres |
| `tipo` | Enum | ✅ | Categoría de observación (v1.1) | {CAMBIO, INCIDENCIA, NOTA} |
| `autor_id` | String | ✅ | ID del usuario que creó la observación | max 100 caracteres (placeholder MVP) |
| `created_at` | Timestamp | ✅ | Fecha de creación | Audit trail |

**Reglas de Negocio:**
- **RN-OBSERVACION-001:** Observaciones son **INMUTABLES** tras creación (HU-021, HU-022)
- **RN-OBSERVACION-002:** No hay UPDATE ni DELETE — solo INSERT y SELECT
- **RN-OBSERVACION-003:** `tipo` es opcional en MVP, obligatorio en v1.1 (HU-023)
- **RN-OBSERVACION-004:** Eliminar horario debe preservar observaciones o CASCADE (decisión de A13)

**Integridad Referencial:**
- `horario_id` → HORARIO.id (ON DELETE CASCADE o RESTRICT — decisión de A13 con justificación)

**Patrones de Consulta:**
- Listar observaciones de un horario (HU-022)
- Filtrar por `tipo` (v1.1 — HU-023)
- Ordenar por `created_at DESC` (más reciente primero)

**PII:** ⚠️ **Potencial** (contenido puede incluir nombres de personas)  
**Clasificación:** Depende del contenido — asumir PII Básico para cumplimiento.

---

## 4. Normalización y Forma Normal

### Análisis de Normalización

**Forma Normal Objetivo:** **3NF** (Tercera Forma Normal)

**Verificación por entidad:**

| Entidad | 1NF | 2NF | 3NF | Justificación |
|---------|-----|-----|-----|---------------|
| AMBIENTE | ✅ | ✅ | ✅ | Sin dependencias transitivas, atributos atómicos |
| INSTRUCTOR | ✅ | ✅ | ✅ | Sin dependencias transitivas, atributos atómicos |
| FICHA | ✅ | ✅ | ✅ | Sin dependencias transitivas, atributos atómicos |
| HORARIO | ✅ | ✅ | ✅ | FKs apuntan a PKs de entidades relacionadas, sin transitivas |
| OBSERVACION | ✅ | ✅ | ✅ | Sin dependencias transitivas, inmutable |

**Desnormalización Considerada y Rechazada:**
- ❌ **Guardar `nombre_ambiente` en HORARIO:** Rechazado — violación 3NF, `nombre` puede cambiar, JOIN es suficiente.
- ❌ **Cache de `num_conflictos` en AMBIENTE:** Rechazado — complejidad de mantenimiento no justifica ganancia.
- ⏸️ **Materializar vista de calendario ficha:** Evaluable en v1.1 si queries >300ms con volumen alto.

---

## 5. Políticas de Integridad Referencial

### Foreign Keys y Cascadas

| Entidad Hija | FK | Entidad Padre | ON DELETE | ON UPDATE | Justificación |
|--------------|----|--------------|-----------|-----------| --------------|
| HORARIO | `ambiente_id` | AMBIENTE | **RESTRICT** | CASCADE | No borrar ambientes con horarios activos |
| HORARIO | `instructor_id` | INSTRUCTOR | **RESTRICT** | CASCADE | No borrar instructores con horarios activos |
| HORARIO | `ficha_id` | FICHA | **RESTRICT** | CASCADE | No borrar fichas con horarios activos |
| OBSERVACION | `horario_id` | HORARIO | **RESTRICT** | CASCADE | Preservar audit trail (decisión final de A13) |

**Alternativa para OBSERVACION:** ON DELETE CASCADE si se decide que observaciones son efímeras.  
**Recomendación A10:** RESTRICT para audit trail completo.

---

## 6. Estrategia de Soft Delete vs Hard Delete

| Entidad | Estrategia | Implementación | Razón |
|---------|-----------|----------------|-------|
| AMBIENTE | **Hard Delete** | DELETE SQL directo (con RESTRICT si hay FKs) | Ambientes eliminados no reaparecen |
| INSTRUCTOR | **Soft Delete** (v1.1) | Campo `estado` = INACTIVO (HU-008) | Preservar historial de asignaciones |
| FICHA | **Estado FINALIZADA** | Campo `estado` (HU-012 v1.1) | Ciclo de vida explícito |
| HORARIO | **Soft Delete** | Campo `estado` = CANCELADO + `canceled_at` (HU-016) | Audit trail de cancelaciones |
| OBSERVACION | **Inmutable** | No DELETE | Auditoría completa |

---

## 7. Clasificación de Datos (PII / Sensibilidad)

| Entidad | Atributos con PII | Clasificación | Protección Requerida | Retención |
|---------|-------------------|---------------|----------------------|-----------|
| AMBIENTE | ❌ Ninguno | Público | Ninguna especial | Indefinida |
| INSTRUCTOR | `nombres`, `apellidos`, `documento`, `email`, `telefono` | **PII Básico** | RGPD aplicable, no anonimizar en logs | 5 años post-inactivo (regulación SENA) |
| FICHA | ❌ Ninguno | Público | Ninguna especial | Indefinida |
| HORARIO | ❌ Ninguno | Público | Ninguna especial | Indefinida |
| OBSERVACION | `contenido` (potencial) | **PII Potencial** | Evitar nombres propios en `contenido`, sanitizar logs | Indefinida (audit trail) |

**Recomendaciones para A04 (Security Designer):**
- INSTRUCTOR: Hash de `documento` en logs (no plaintext en logs de aplicación)
- OBSERVACION: Policy de no incluir datos sensibles en `contenido` (entrenamiento de usuarios)

---

## 8. Patrones de Consulta Priorizados (Input para A13)

### 8.1. Consultas Críticas (Performance <500ms)

**Q1: Validación de Conflicto de Ambiente** (HU-017 — core del MVP)
```sql
-- Input: ambiente_id, fecha, hora_inicio, hora_fin
SELECT id, fecha, hora_inicio, hora_fin, instructor_id, ficha_id
FROM horarios
WHERE ambiente_id = $1
  AND fecha = $2
  AND estado != 'CANCELADO'
  AND (hora_inicio, hora_fin) OVERLAPS ($3, $4)
LIMIT 1; -- Solo necesitamos saber si existe conflicto
```
**Frecuencia:** Alta (cada creación/modificación de horario)  
**Índice sugerido:** `(ambiente_id, fecha, estado, hora_inicio, hora_fin)`

---

**Q2: Validación de Conflicto de Instructor** (HU-018)
```sql
SELECT id, fecha, hora_inicio, hora_fin, ambiente_id, ficha_id
FROM horarios
WHERE instructor_id = $1
  AND fecha = $2
  AND estado != 'CANCELADO'
  AND (hora_inicio, hora_fin) OVERLAPS ($3, $4)
LIMIT 1;
```
**Frecuencia:** Alta  
**Índice sugerido:** `(instructor_id, fecha, estado, hora_inicio, hora_fin)`

---

### 8.2. Consultas de Alta Frecuencia (Performance <200ms)

**Q3: Calendario Semanal de Ficha** (HU-024)
```sql
SELECT h.id, h.fecha, h.hora_inicio, h.hora_fin, h.tema,
       a.nombre AS ambiente_nombre, a.codigo AS ambiente_codigo,
       i.nombres AS instructor_nombres, i.apellidos AS instructor_apellidos
FROM horarios h
  JOIN ambientes a ON h.ambiente_id = a.id
  JOIN instructores i ON h.instructor_id = i.id
WHERE h.ficha_id = $1
  AND h.fecha BETWEEN $2 AND $3  -- Rango de semana (7 días)
  AND h.estado != 'CANCELADO'
ORDER BY h.fecha, h.hora_inicio;
```
**Frecuencia:** Alta (vista principal de coordinadores y aprendices)  
**Índice sugerido:** `(ficha_id, fecha, estado)`

---

**Q4: Agenda Personal de Instructor** (HU-027)
```sql
SELECT h.id, h.fecha, h.hora_inicio, h.hora_fin, h.tema,
       a.nombre AS ambiente_nombre, a.codigo AS ambiente_codigo,
       f.codigo AS ficha_codigo, f.nombre AS ficha_nombre
FROM horarios h
  JOIN ambientes a ON h.ambiente_id = a.id
  JOIN fichas f ON h.ficha_id = f.id
WHERE h.instructor_id = $1
  AND h.fecha BETWEEN $2 AND $3  -- Rango de fechas (ej: mes)
  AND h.estado != 'CANCELADO'
ORDER BY h.fecha, h.hora_inicio;
```
**Frecuencia:** Alta (vista principal de instructores)  
**Índice sugerido:** `(instructor_id, fecha, estado)`

---

**Q5: Listar Horarios con Filtros** (HU-014)
```sql
SELECT h.id, h.fecha, h.hora_inicio, h.hora_fin, h.estado,
       a.nombre AS ambiente, i.nombres || ' ' || i.apellidos AS instructor,
       f.codigo AS ficha
FROM horarios h
  JOIN ambientes a ON h.ambiente_id = a.id
  JOIN instructores i ON h.instructor_id = i.id
  JOIN fichas f ON h.ficha_id = f.id
WHERE ($1::uuid IS NULL OR h.ambiente_id = $1)
  AND ($2::uuid IS NULL OR h.instructor_id = $2)
  AND ($3::uuid IS NULL OR h.ficha_id = $3)
  AND ($4::date IS NULL OR h.fecha = $4)
  AND ($5::text IS NULL OR h.estado = $5)
ORDER BY h.fecha DESC, h.hora_inicio DESC
LIMIT $6 OFFSET $7;  -- Paginación
```
**Frecuencia:** Media  
**Índices sugeridos:** Composite por cada filtro frecuente

---

### 8.3. Consultas de Baja Frecuencia

**Q6: Dashboard de Disponibilidad de Ambientes** (HU-029 v1.1)
```sql
-- Ambientes sin horarios en rango de tiempo específico
SELECT a.id, a.codigo, a.nombre, a.capacidad
FROM ambientes a
WHERE a.estado = 'ACTIVO'
  AND NOT EXISTS (
    SELECT 1 FROM horarios h
    WHERE h.ambiente_id = a.id
      AND h.fecha = $1
      AND h.estado != 'CANCELADO'
      AND (h.hora_inicio, h.hora_fin) OVERLAPS ($2, $3)
  )
ORDER BY a.nombre;
```
**Frecuencia:** Baja (solo coordinadores, planificación)  
**Performance:** <500ms aceptable

---

## 9. Estrategia de Auditoría

### Campos de Auditoría Estándar

Todas las entidades principales (excepto OBSERVACION) incluyen:
- `created_at` — Timestamp de creación (inmutable)
- `updated_at` — Timestamp de última modificación (actualizado en cada UPDATE)

**Implementación:**
- Backend: Aplicación layer establece valores antes de llamar repository
- BD: Triggers de PostgreSQL como fallback (decisión de A13)

### Inmutabilidad de OBSERVACION

- OBSERVACION no tiene `updated_at` porque es **append-only**
- Audit trail completo: historial de cambios en horarios vía observaciones
- Ejemplo: "Cambio de instructor de Juan Pérez a María Gómez — Solicitado por coordinador"

---

## 10. Volumen de Datos Esperado (Input para A13)

| Entidad | MVP (año 1) | v1.1 (año 2) | v2.0 (año 3+) | Growth Rate |
|---------|-------------|--------------|---------------|-------------|
| AMBIENTE | 50 | 100 | 500 (multi-centro) | 2x anual |
| INSTRUCTOR | 100 | 200 | 1,000 | 2x anual |
| FICHA | 50 | 100 | 500 | 2x anual |
| HORARIO | 10,000 | 30,000 | 150,000 | 3x anual |
| OBSERVACION | 1,000 | 5,000 | 50,000 | 5x anual |

**Total registros año 1:** ~11,200 (manejable con BD single instance)  
**Total registros año 3:** ~201,500 (requiere índices optimizados, posible particionamiento)

**Implicaciones para A13:**
- MVP: Índices estándar suficientes
- v1.1: Monitorear performance de queries Q1-Q5
- v2.0: Considerar particionamiento de HORARIO por año (PARTITION BY RANGE fecha)

---

## 11. Migraciones de Datos

### Escenario: Migración desde Excel (Actual)

**Estado actual:** Coordinadores usan Excel para gestión manual de horarios.

**Estrategia de migración:**
1. **Fase 1 — Importación inicial (pre-lanzamiento):**
   - Cargar AMBIENTE desde hoja "Ambientes" (mapeo directo)
   - Cargar INSTRUCTOR desde hoja "Instructores" (mapeo directo)
   - Cargar FICHA desde hoja "Fichas" (mapeo directo)
   - **NO migrar HORARIO históricos** (solo últimas 2 semanas) — evitar conflictos legacy

2. **Fase 2 — Convivencia (semana 1-2 post-lanzamiento):**
   - Sistema nuevo es source of truth desde Go-Live
   - Excel se descontinúa (no sincronización bidireccional)

3. **Fase 3 — Históricos (opcional v1.1):**
   - Importar horarios históricos (>2 semanas antiguos) para analytics
   - Estado = REALIZADO (no participan en validaciones)

**ETL Script (responsabilidad de A07 en Sprint 8):**
- Lenguaje: Go (consistente con stack)
- Validación: Ejecutar RN-001 a RN-008 en cada fila
- Rollback: Transacción completa o ninguna (ACID)

---

## 12. Extensiones Futuras (v1.1+)

### Posibles Evoluciones del Modelo

| Feature | Versión | Impacto en Modelo |
|---------|---------|-------------------|
| **Desactivar instructor** (HU-008) | v1.1 | Ya contemplado con campo `estado` |
| **Finalizar ficha** (HU-012) | v1.1 | Ya contemplado con campo `estado` |
| **Categorizar observación** (HU-023) | v1.1 | Ya contemplado con campo `tipo` |
| **Exportar PDF/Excel** (HU-026) | v1.2 | Sin cambios en modelo (solo queries) |
| **Multi-tenancy** (v2.0) | v2.0 | Agregar `centro_id` a todas las entidades (FK a nueva entidad CENTRO) |
| **Autenticación robusta** (v2.0) | v2.0 | Nueva entidad USUARIO con roles (reemplaza placeholder `autor_id`) |
| **Integración Sofia Plus** (v2.0) | v2.0 | Nueva entidad APRENDIZ (1:N con FICHA), sync externo |

---

## 13. Decisiones Pendientes para A13 (DBA Expert)

| Decisión | Opciones | Recomendación A10 | Justificación |
|----------|----------|-------------------|---------------|
| **Tipo UUID:** UUID v4 vs UUIDv7 (timestamp-ordered) | v4, v7 | **UUIDv7** si PostgreSQL 16+ | Mejor performance en índices B-tree |
| **OBSERVACION ON DELETE:** CASCADE vs RESTRICT | CASCADE, RESTRICT | **RESTRICT** | Preservar audit trail completo |
| **Índice para time overlap:** GiST vs B-tree composite | GiST, B-tree | **B-tree composite** si PostgreSQL <14 | GiST mejor con exclusion constraints (PostgreSQL 14+) |
| **Particionamiento HORARIO:** Por año, por trimestre | Año, trimestre | **No en MVP**, evaluar v1.1 si >50k registros | Complejidad no justificada en MVP |
| **Enum storage:** PostgreSQL ENUM vs CHECK constraint | ENUM, CHECK | **ENUM** | Type-safe a nivel BD, mejor performance |

---

## 14. Validación del Modelo

### Checklist de Cumplimiento

- ✅ **Normalización 3NF:** Sin dependencias transitivas
- ✅ **Sin atributos multi-valor:** Cada atributo es atómico
- ✅ **PKs definidas:** Todas las entidades tienen UUID como PK
- ✅ **FKs con política ON DELETE:** RESTRICT en todas (audit trail)
- ✅ **Inmutabilidad de Value Objects:** OBSERVACION es append-only
- ✅ **Auditoría base:** `created_at`, `updated_at` en entidades mutables
- ✅ **PII etiquetada:** INSTRUCTOR con clasificación PII Básico
- ✅ **Separación transaccional/analítico:** Un solo modelo transaccional (analytics en v2.0)
- ✅ **Naming consistente:** `snake_case` en todos los atributos (estándar PostgreSQL)

---

## 15. Mapeo a Capa Domain (Go)

### Ejemplo: Entity HORARIO en Go

```go
// internal/domain/entities/horario.go
package entities

import (
    "time"
    "github.com/google/uuid"
)

type Horario struct {
    ID           uuid.UUID
    AmbienteID   uuid.UUID
    InstructorID uuid.UUID
    FichaID      uuid.UUID
    Fecha        time.Time
    HoraInicio   time.Time
    HoraFin      time.Time
    Tema         string
    Estado       EstadoHorario
    CreatedAt    time.Time
    UpdatedAt    time.Time
    CanceledAt   *time.Time  // Nullable
}

type EstadoHorario string

const (
    EstadoProgramado EstadoHorario = "PROGRAMADO"
    EstadoRealizado  EstadoHorario = "REALIZADO"
    EstadoCancelado  EstadoHorario = "CANCELADO"
)

// Domain method — validación básica
func (h *Horario) Validar() error {
    if h.HoraFin.Before(h.HoraInicio) {
        return errors.New("hora_fin debe ser posterior a hora_inicio")
    }
    return nil
}
```

**Nota:** Este modelo Go NO tiene tags JSON ni DB (arquitectura Hexagonal — domain puro).  
Tags se agregan en adapters layer (DTOs, request models, repository mappers).

---

## Referencias

- **PRD:** [`02-definition/prd.md`](../02-definition/prd.md)
- **Architecture:** [`03-design/architecture.md`](architecture.md)
- **Pattern Guide:** [`03-design/pattern-guide.md`](pattern-guide.md)
- **ADR-002:** [`03-design/adr/adr-002.md`](adr/adr-002.md) — Persistencia con pgx SQL raw

---

## Aprobaciones

| Rol | Nombre | Fecha | Firma |
|-----|--------|-------|-------|
| Data Architect | A10 | 2026-05-20 | ✅ |
| DBA Expert | A13 | Pendiente | ⏸️ Debe producir db-design.md con schema físico |
| Tech Lead | A06 | Revisión | ⏸️ Debe validar consistencia con architecture.md |

**Estado:** ✅ **APROBADO — INPUT PARA A13 (DB DESIGN)**

---

**Producido:** 2026-05-20  
**Última revisión:** 2026-05-20  
**Próxima revisión:** Post-Sprint 2 (validar con esquema PostgreSQL real)
