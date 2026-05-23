# Scope Declaration — PRJ-SENA-SCHEDULE

> **Producido por:** A00 (Orquestador)
> **Fecha:** 2026-05-20
> **Estado:** ACTIVE

---

## 1. Identificación del Proyecto

| Campo | Valor |
|-------|-------|
| **project_key** | `PRJ-SENA-SCHEDULE` |
| **project_name** | SENA Schedule Manager |
| **project_type** | greenfield |
| **workspace_path** | `c:\www\code-dev-projects\schedule-sena` |
| **framework_path** | `C:\www\code-dev-projects\automatization-develop` |
| **framework_mode** | read_only |

---

## 2. Stack Tecnológico Validado

| Componente | Tecnología | Versión mínima |
|------------|------------|----------------|
| **Backend** | Go | 1.21+ |
| **Frontend** | React + TypeScript + Vite | React 18+ |
| **Base de datos** | PostgreSQL | 15+ |
| **Containerización** | Docker + Docker Compose | Docker 24+ |
| **Arquitectura** | Hexagonal (Ports & Adapters) | - |

**Stack de referencia:** `agentic-sdlc-os/stacks/go-react-postgres.md`

---

## 3. Alcance Funcional Base (MVP)

### Entidades principales
1. **Ambientes**: Espacios físicos donde se imparten las formaciones
2. **Horarios**: Programación de actividades formativas
3. **Fichas**: Grupos de aprendices con programa de formación específico
4. **Instructores**: 
   - Planta: Instructores de tiempo completo
   - Contratista: Instructores por contrato
5. **Observaciones**: Notas y comentarios sobre horarios/actividades

### Funcionalidades MVP
- CRUD completo de todas las entidades
- Asignación de instructores a horarios
- Asignación de ambientes a horarios
- Visualización de horarios por ficha
- Gestión de observaciones

---

## 4. Restricciones y Reglas

### Restricciones de workspace
- ✅ **Escritura permitida:** `c:\www\code-dev-projects\schedule-sena`
- ❌ **Solo lectura:** `C:\www\code-dev-projects\automatization-develop`
- ❌ **Prohibido:** Crear artefactos de producto en el framework

### Reglas de trabajo
- Trabajo por fases secuenciales
- Un agente activo a la vez
- Lectura mínima y escalonada del framework
- Handoff mecánico obligatorio entre fases
- Validación de gates antes de avanzar

### Budget de lectura
- Máximo 5 archivos del framework por fase sin justificación
- Si se excede: emitir HALT-CONTEXT y esperar aprobación

---

## 5. Fases SDLC Aplicables

| # | Fase | Agente Lead | Estado |
|---|------|-------------|--------|
| 00 | setup | A00 | ✅ IN_PROGRESS |
| 01 | discovery | A05 | ⏸️ PENDING |
| 02 | definition | A05 | ⏸️ PENDING |
| 03 | design | A06, A10, A03 | ⏸️ PENDING |
| 04 | build | A07, A08 | ⏸️ PENDING |
| 05 | quality-security | A09 | ⏸️ PENDING |
| 06 | release | A11 | ⏸️ PENDING |
| 07 | operate-improve | A11 | ⏸️ PENDING |

---

## 6. Registro de Decisiones Críticas

### 2026-05-20 — Stack PostgreSQL confirmado
- **Contexto:** Stack original solicitado era Go + React + MongoDB
- **Decisión:** Usar PostgreSQL en lugar de MongoDB
- **Razón:** Stack validado disponible en framework (`go-react-postgres.md`)
- **Aprobado por:** Usuario (opción 1)
- **Impacto:** Ninguno en funcionalidad, PostgreSQL cubre todos los requisitos

---

## 7. Validación de Scope

- ✅ Workspace del producto vacío y listo
- ✅ Framework accesible en modo lectura
- ✅ Stack validado disponible
- ✅ project_key definido y válido
- ✅ Agentes aplicables identificados
- ✅ Reglas de trabajo aceptadas

**Estado:** SCOPE_VALID

---

## Notas

Este documento es la fuente única de verdad sobre el scope del proyecto.
Solo A00 puede modificarlo, siempre con entrada en `decision-log.md`.
