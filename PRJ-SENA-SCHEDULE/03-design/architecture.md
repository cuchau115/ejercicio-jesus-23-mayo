# Architecture Design — PRJ-SENA-SCHEDULE

> **Producido por:** A06 (Tech Lead)  
> **Fecha:** 2026-05-20  
> **Fase:** 03-design (paso 2)  
> **Versión:** 1.0  
> **Capacidad:** cap-architecture

---

## 1. Visión Arquitectónica

### Objetivo

Diseñar un sistema web centralizado para gestión de horarios SENA que elimine conflictos de asignación mediante validación automática en tiempo real, reduciendo carga operativa de coordinadores en 60% (de 5h a 2h semanales).

### Principios Arquitectónicos

| Principio | Descripción | Validación |
|-----------|-------------|------------|
| **Separation of Concerns** | Domain lógica independiente de frameworks (Hexagonal) | Domain/ sin imports de Gin/pgx |
| **Testability First** | Cobertura ≥85% en domain, ≥70% en adapters | CI/CD gate |
| **API-First** | Backend expone API REST, frontend consume sin acoplamiento | OpenAPI spec |
| **Performance-Conscious** | Validaciones <500ms, queries <200ms | Load testing A09 |
| **Pragmatism over Perfection** | Soluciones simples antes que arquitecturas complejas | MVP en 14 semanas |

---

## 2. Estilo Arquitectónico

### Patrón: Hexagonal Architecture (Ports & Adapters)

Basado en [`03-design/pattern-guide.md`](pattern-guide.md), la arquitectura sigue el patrón P02 Hexagonal con las siguientes capas:

```
┌──────────────────────────────────────────────────────┐
│              Adapters IN (Entrada)                   │
│         HTTP REST API (Gin framework)                │
└────────────────┬─────────────────────────────────────┘
                 │ implementa
                 ▼
┌──────────────────────────────────────────────────────┐
│              Ports IN (Interfaces)                   │
│    HorarioService, AmbienteService, etc.             │
└────────────────┬─────────────────────────────────────┘
                 │ orquesta
                 ▼
┌──────────────────────────────────────────────────────┐
│              Application Layer                       │
│   Commands (CrearHorario, RegistrarObservacion)     │
│   Queries (ListarHorarios, ObtenerCalendario)       │
└────────────────┬─────────────────────────────────────┘
                 │ usa
                 ▼
┌──────────────────────────────────────────────────────┐
│            Domain Layer (CORE)                       │
│  Entities: Horario, Ambiente, Instructor, Ficha      │
│  Services: ConflictoValidator, CapacidadValidator    │
│              SIN DEPENDENCIAS                        │
└────────────────┬─────────────────────────────────────┘
                 │ define contratos
                 ▼
┌──────────────────────────────────────────────────────┐
│           Ports OUT (Interfaces)                     │
│  HorarioRepository, InstructorRepository, etc.       │
└────────────────┬─────────────────────────────────────┘
                 │ implementa
                 ▼
┌──────────────────────────────────────────────────────┐
│             Adapters OUT (Salida)                    │
│        PostgreSQL (pgx driver)                       │
└──────────────────────────────────────────────────────┘
```

**Referencias a pattern guide:**
- Estructura de carpetas: Sección 3 de `pattern-guide.md`
- Reglas de dependencia: Sección 4 de `pattern-guide.md`
- Naming conventions: Sección 5 de `pattern-guide.md`

---

## 3. Diagrama C4 — Nivel 1: Contexto del Sistema

```
                     ┌──────────────┐
                     │  Coordinador │
                     │   (Usuario)  │
                     └───────┬──────┘
                             │ Gestiona horarios
                             │ Consulta conflictos
                             ▼
                ┌────────────────────────┐
                │  Sistema de Gestión    │
                │  de Horarios SENA      │◄─────── Instructor (consulta agenda)
                │   [Web Application]    │
                └────────────┬───────────┘
                             │ Persiste datos
                             ▼
                    ┌────────────────┐
                    │   PostgreSQL   │
                    │   [Database]   │
                    └────────────────┘
                    
       Aprendiz (consulta horarios de ficha) ─────►│
```

**Actores externos:**
- **Coordinador Académico** (primario): Crea/edita horarios, gestiona recursos, resuelve conflictos
- **Instructor** (secundario): Consulta agenda personal, visualiza asignaciones
- **Aprendiz** (terciario): Consulta calendario de su ficha

**Sistemas externos:**
- Ninguno en MVP (v2.0 integrará Sofia Plus — sistema SENA legacy)

---

## 4. Diagrama C4 — Nivel 2: Contenedores

```
┌─────────────────────────────────────────────────────────────────┐
│                        Browser (Usuario)                        │
└────────────────────┬────────────────────────────────────────────┘
                     │ HTTPS
                     ▼
┌─────────────────────────────────────────────────────────────────┐
│               Frontend SPA (React + TypeScript)                 │
│  • Componentes: HorarioForm, CalendarioSemanal, AgendaInstructor│
│  • Estado: Context API (global) + React Query (server state)   │
│  • Build: Vite 5+                                               │
│  • Puerto: 3000 (dev), 80/443 (prod)                            │
└────────────────────┬────────────────────────────────────────────┘
                     │ HTTP REST (JSON)
                     │ /api/v1/...
                     ▼
┌─────────────────────────────────────────────────────────────────┐
│                 Backend API (Go 1.21+)                          │
│  • Framework: Gin (HTTP router)                                 │
│  • Arquitectura: Hexagonal (domain + application + adapters)    │
│  • Validaciones: ConflictoValidator, CapacidadValidator         │
│  • Puerto: 8080                                                 │
│  • Endpoints: /ambientes, /instructores, /fichas, /horarios     │
└────────────────────┬────────────────────────────────────────────┘
                     │ pgx driver (SQL)
                     │ Connection pool
                     ▼
┌─────────────────────────────────────────────────────────────────┐
│                   PostgreSQL 15+                                │
│  • Schema: 5 tablas (ambientes, instructores, fichas,           │
│    horarios, observaciones)                                     │
│  • Constraints: FKs, checks, unique indexes                     │
│  • Puerto: 5432                                                 │
│  • Backup: diario automático (A11 responsable)                  │
└─────────────────────────────────────────────────────────────────┘

┌────────────────────┐
│  Docker Compose    │  ← Orquesta todos los contenedores (dev/staging)
│  [Orquestador]     │
└────────────────────┘
```

**Flujo de comunicación:**
1. Usuario interactúa con Frontend (React SPA en browser)
2. Frontend hace HTTP requests a Backend API (REST JSON)
3. Backend valida, ejecuta lógica de negocio, persiste en PostgreSQL
4. Backend responde con JSON (datos o errores)
5. Frontend actualiza UI con respuesta

---

## 5. Componentes — Backend (Go)

### 5.1. Domain Layer (Core)

**Responsabilidad:** Lógica de negocio pura, independiente de frameworks.

| Componente | Descripción | Artefactos Clave |
|------------|-------------|------------------|
| **Entities** | Modelos de dominio con identidad | `Horario`, `Ambiente`, `Instructor`, `Ficha`, `Observacion` |
| **ConflictoValidator** | Valida solapamientos de asignación | `ValidarConflictoAmbiente()`, `ValidarConflictoInstructor()` |
| **CapacidadValidator** | Valida capacidad de ambiente vs ficha | `ValidarCapacidad()` (warning, no bloqueante) |
| **Domain Errors** | Errores tipados de negocio | `ConflictoAmbienteError`, `CapacidadExcedidaWarning` |

**Reglas de negocio críticas:**
- **RN-001:** Un ambiente no puede tener horarios solapados en mismo día/hora (HU-017)
- **RN-002:** Un instructor no puede tener asignaciones solapadas (HU-018)
- **RN-003:** Capacidad de ambiente < aprendices de ficha genera warning (HU-019)
- **RN-004:** Horarios cancelados no validan conflictos (HU-016)
- **RN-005:** Observaciones son inmutables tras creación (HU-021, HU-022)

**Testing:** ≥85% cobertura — Unit tests puros sin BD.

---

### 5.2. Application Layer

**Responsabilidad:** Orquestar casos de uso, coordinar domain + ports.

| Componente | Descripción | Dependencias |
|------------|-------------|--------------|
| **Commands** | Escrituras (create, update, delete) | Domain entities + Ports OUT (repositories) |
| **Queries** | Lecturas (listar, filtrar, obtener) | Ports OUT (repositories) |
| **DTOs** | Data Transfer Objects — entrada/salida | Ninguna (structs Go planos) |

**Commands principales:**
- `CrearHorario` (HU-013): Valida conflictos → guarda en repo → retorna ID
- `ModificarHorario` (HU-015): Carga existente → aplica cambios → revalida → guarda
- `CancelarHorario` (HU-016): Marca como cancelado (soft delete)
- `RegistrarObservacion` (HU-021): Asocia observación a horario

**Queries principales:**
- `ListarHorarios` (HU-014): Filtra por fecha/ambiente/instructor/ficha
- `ObtenerCalendarioFicha` (HU-024): Vista semanal de horarios de una ficha
- `ObtenerAgendaInstructor` (HU-027): Horarios asignados a un instructor

**Testing:** ≥80% cobertura — Unit tests con mocks de repositories.

---

### 5.3. Ports (Interfaces)

**Responsabilidad:** Contratos entre capas — desacoplamiento total.

#### Ports IN (Entrada)

| Interface | Métodos | Implementado por |
|-----------|---------|------------------|
| `HorarioService` | `CrearHorario()`, `ListarHorarios()`, `ModificarHorario()` | HTTP handlers (adapters/in) |
| `AmbienteService` | `CrearAmbiente()`, `ListarAmbientes()` | HTTP handlers |
| `InstructorService` | `RegistrarInstructor()`, `ListarInstructores()` | HTTP handlers |

#### Ports OUT (Salida)

| Interface | Métodos | Implementado por |
|-----------|---------|------------------|
| `HorarioRepository` | `Save()`, `FindByID()`, `FindByFilters()`, `Delete()` | PostgreSQL adapter |
| `AmbienteRepository` | `Save()`, `FindByID()`, `FindAll()` | PostgreSQL adapter |
| `InstructorRepository` | `Save()`, `FindByID()`, `FindAll()`, `Deactivate()` | PostgreSQL adapter |
| `FichaRepository` | `Save()`, `FindByID()`, `FindAll()`, `Finalize()` | PostgreSQL adapter |
| `ObservacionRepository` | `Save()`, `FindByHorarioID()` | PostgreSQL adapter |
| `TransactionManager` | `BeginTx()`, `Commit()`, `Rollback()` | PostgreSQL adapter |

**Testing:** Interfaces no se testean — se mockean en tests de application layer.

---

### 5.4. Adapters IN (HTTP REST API)

**Responsabilidad:** Exponer endpoints HTTP, validar requests, mapear DTOs.

| Componente | Descripción | Framework |
|------------|-------------|-----------|
| **Gin Server** | Router HTTP con middleware (CORS, logging, recovery) | github.com/gin-gonic/gin |
| **Handlers** | Controllers por entidad (HorarioHandler, AmbienteHandler) | Gin Context |
| **Middleware** | Logger (structured logs), CORS, auth placeholder (v1.0 básico) | Gin middleware |
| **Request Models** | Structs para parsear JSON con validación | go-playground/validator |

**Endpoints principales:**

| Método | Path | Handler | HU |
|--------|------|---------|-----|
| POST | `/api/v1/ambientes` | `CrearAmbiente` | HU-001 |
| GET | `/api/v1/ambientes` | `ListarAmbientes` | HU-002 |
| PUT | `/api/v1/ambientes/:id` | `ActualizarAmbiente` | HU-003 |
| DELETE | `/api/v1/ambientes/:id` | `EliminarAmbiente` | HU-004 |
| POST | `/api/v1/instructores` | `RegistrarInstructor` | HU-005 |
| GET | `/api/v1/instructores` | `ListarInstructores` | HU-006 |
| PUT | `/api/v1/instructores/:id` | `ActualizarInstructor` | HU-007 |
| POST | `/api/v1/fichas` | `CrearFicha` | HU-009 |
| GET | `/api/v1/fichas` | `ListarFichas` | HU-010 |
| PUT | `/api/v1/fichas/:id` | `EditarFicha` | HU-011 |
| POST | `/api/v1/horarios` | `CrearHorario` | HU-013 |
| GET | `/api/v1/horarios` | `ListarHorarios` | HU-014 |
| PUT | `/api/v1/horarios/:id` | `ModificarHorario` | HU-015 |
| DELETE | `/api/v1/horarios/:id` | `CancelarHorario` | HU-016 |
| POST | `/api/v1/horarios/:id/observaciones` | `RegistrarObservacion` | HU-021 |
| GET | `/api/v1/horarios/:id/observaciones` | `ListarObservaciones` | HU-022 |
| GET | `/api/v1/fichas/:id/calendario` | `ObtenerCalendarioFicha` | HU-024 |
| GET | `/api/v1/instructores/:id/agenda` | `ObtenerAgendaInstructor` | HU-027 |

**Error handling:**
- 200/201: Éxito
- 400: Bad Request (validación fallida)
- 404: Not Found (recurso no existe)
- 409: Conflict (conflicto de asignación — RN-001, RN-002)
- 500: Internal Server Error (error inesperado)

**Testing:** ≥60% cobertura — E2E tests con `httptest`.

---

### 5.5. Adapters OUT (PostgreSQL)

**Responsabilidad:** Persistencia de datos, implementación de repositories.

| Componente | Descripción | Tecnología |
|------------|-------------|------------|
| **Connection Pool** | Pool de conexiones a PostgreSQL | github.com/jackc/pgx/v5 |
| **Repositories** | Implementaciones concretas de ports/out | SQL raw (no ORM — ver ADR-002) |
| **Migrations** | Scripts SQL versionados | golang-migrate/migrate |
| **Transaction Manager** | Manejo de transacciones ACID | pgx transactions |

**Decisiones técnicas:**
- **SQL raw sobre ORM:** Mayor control, performance, simplicidad para este alcance (ver ADR-002)
- **pgx sobre pq:** Mejor performance, soporte nativo de PostgreSQL 15+ features
- **Migrations como código:** golang-migrate con archivos `.up.sql` y `.down.sql`

**Constraints de BD (responsabilidad de A13/A10):**
- Foreign Keys con `ON DELETE CASCADE` o `RESTRICT` según regla de negocio
- Unique indexes: `ambiente + fecha + hora_rango` (para validación de conflictos)
- Check constraints: `hora_fin > hora_inicio`, `capacidad > 0`

**Testing:** ≥70% cobertura — Integration tests con `testcontainers-go`.

---

## 6. Componentes — Frontend (React + TypeScript)

### 6.1. Módulos Feature-First

**Responsabilidad:** Encapsular lógica por dominio funcional.

| Módulo | Descripción | Componentes Clave |
|--------|-------------|-------------------|
| **ambientes/** | CRUD de ambientes | `AmbienteForm`, `AmbienteList`, `AmbienteCard` |
| **instructores/** | CRUD de instructores | `InstructorForm`, `InstructorList`, `InstructorDetail` |
| **fichas/** | CRUD de fichas | `FichaForm`, `FichaList`, `FichaDetail` |
| **horarios/** | ⭐ Gestión de horarios | `HorarioForm`, `HorarioList`, `CalendarioSemanal`, `ConflictoAlert` |
| **observaciones/** | Registro y consulta | `ObservacionForm`, `ObservacionList` |
| **dashboard/** | Vista ejecutiva coordinador | `DashboardAlertas`, `DisponibilidadAmbientes` |

**Estructura por módulo:**
```
horarios/
  ├── components/     # Componentes React
  ├── hooks/          # useHorarios, useValidacionConflictos
  ├── services/       # horarioService (API client)
  ├── types/          # horario.types.ts
  └── pages/          # HorariosPage, CalendarioFichaPage
```

---

### 6.2. Shared Layer

**Responsabilidad:** Código reutilizable cross-module.

| Categoría | Componentes | Descripción |
|-----------|-------------|-------------|
| **UI Components** | `Button`, `Input`, `Modal`, `Card`, `Select`, `DatePicker` | Componentes básicos reutilizables |
| **Layout** | `Header`, `Sidebar`, `MainLayout` | Estructura de páginas |
| **Hooks** | `useApi`, `useDebounce`, `useLocalStorage` | Hooks genéricos |
| **Services** | `httpClient` (Axios config), `errorHandler` | HTTP client centralizado |
| **Utils** | `formatters`, `validators`, `dateHelpers` | Funciones utilidad |

---

### 6.3. Estado (State Management)

**Decisión:** Combinación Context API + React Query (ver ADR-004)

| Tipo de Estado | Solución | Uso |
|----------------|----------|-----|
| **UI State** (sidebar abierto, modal visible) | `useState` local | Componente individual |
| **Shared UI State** (usuario logueado, tema) | Context API | Global app |
| **Server State** (horarios, ambientes, fichas) | React Query | Cache + sync con backend |

**React Query ventajas:**
- Cache automático (reduce requests)
- Refetch inteligente (stale-while-revalidate)
- Optimistic updates (UX mejorada)
- Invalidación de queries (post-mutation)

---

### 6.4. Routing

**Framework:** React Router v6

**Rutas principales:**

| Path | Componente | Rol |
|------|-----------|------|
| `/` | `DashboardPage` | Coordinador |
| `/ambientes` | `AmbientesPage` | Coordinador |
| `/instructores` | `InstructoresPage` | Coordinador |
| `/fichas` | `FichasPage` | Coordinador |
| `/horarios` | `HorariosPage` | Coordinador |
| `/horarios/nuevo` | `HorarioFormPage` | Coordinador |
| `/fichas/:id/calendario` | `CalendarioFichaPage` | Todos |
| `/instructores/:id/agenda` | `AgendaInstructorPage` | Instructor |

**Protección de rutas:** MVP sin auth robusta — placeholder con localStorage (v2.0 con LDAP/AD).

---

## 7. Integraciones

### 7.1. Frontend ↔ Backend

**Protocolo:** HTTP REST (JSON)  
**Base URL:** `http://localhost:8080/api/v1` (dev), `https://horarios.sena.edu.co/api/v1` (prod)  
**Content-Type:** `application/json`  
**CORS:** Habilitado en backend (origen permitido: frontend domain)

**Autenticación (MVP):** Header `X-User-ID` (placeholder — v2.0 con JWT/OAuth)

**Error handling:**
- Frontend: Interceptor Axios para manejar errores globales
- 409 Conflict: Mostrar `ConflictoAlert` con detalles del conflicto
- 500: Mostrar error genérico + enviar telemetría (v1.1)

---

### 7.2. Backend ↔ PostgreSQL

**Driver:** pgx v5 (github.com/jackc/pgx/v5)  
**Connection String:** `postgres://user:pass@localhost:5432/sena_horarios?sslmode=disable` (dev)  
**Pool size:** 10 conexiones (ajustable según load testing)  
**Timeout:** Query 5s, Connection 10s

**Transactions:**
- Crear horario: transacción (insert horario + validar conflictos + commit)
- Modificar horario: transacción (update + revalidar + commit)
- Rollback automático en caso de conflicto

---

### 7.3. Sistemas Externos (v2.0)

**Fuera de scope MVP:**
- Sofia Plus (sistema SENA legacy): Sincronización de fichas y aprendices
- LDAP/Active Directory: Autenticación de usuarios
- Email/SMS: Notificaciones de cambios en horarios

**Preparación futura:** Adapters OUT diseñados para ser extensibles (ports/out puede agregar `SofiaPlusClient` interface).

---

## 8. Atributos de Calidad

### 8.1. Performance

| Métrica | Objetivo | Validación |
|---------|----------|------------|
| **Validación de conflictos** | <500ms (p95) | Load testing A09 con 100 horarios simultáneos |
| **Queries de listado** | <200ms (p95) | JMeter con 1000 registros |
| **Carga de calendario semanal** | <300ms (p95) | Network throttling 3G |
| **API Response Time** | <1s (p99) | Prometheus + Grafana (v1.1) |

**Optimizaciones:**
- Índices PostgreSQL en columnas filtradas (fecha, ambiente_id, instructor_id)
- Paginación por defecto (20 items por página)
- Cache de queries repetitivas con React Query (frontend)

---

### 8.2. Escalabilidad

| Dimensión | Capacidad Inicial | Escalabilidad Futura |
|-----------|-------------------|----------------------|
| **Usuarios concurrentes** | 50 | Horizontal scaling backend (v1.1) |
| **Horarios activos** | 10,000 | Particionamiento PostgreSQL por año (v2.0) |
| **Requests/segundo** | 100 | Load balancer + múltiples instancias backend (v2.0) |

**Estrategia:**
- MVP: Single instance backend + PostgreSQL
- v1.1: Docker Swarm o Kubernetes con 2+ replicas backend
- v2.0: Microservicios si supera 100k horarios/año

---

### 8.3. Disponibilidad

| Objetivo | Implementación | Responsable |
|----------|----------------|-------------|
| **Uptime 98%** (MVP) | Health check endpoint `/health` | A11 |
| **Backup diario** | pg_dump automatizado 3am | A11 |
| **Recovery Time <1h** | Restore script + runbook | A11 |
| **Uptime 99.5%** (v1.1) | Réplica PostgreSQL read-only | A11 |

---

### 8.4. Seguridad

| Amenaza | Mitigación | Implementación |
|---------|------------|----------------|
| **SQL Injection** | Parametrized queries (pgx) | 100% queries usan placeholders `$1`, `$2` |
| **XSS** | React sanitiza por defecto | `dangerouslySetInnerHTML` prohibido (code review) |
| **CORS** | Whitelist de orígenes | Gin middleware con allowed origins |
| **Auth bypass** | Validación de user ID (MVP básico) | Middleware en todas las rutas protegidas |
| **Data leak** | Sin PII sensible (nombres públicos) | RGPD no crítico en MVP |

**Pentesting:** A04 (Security Designer) ejecutará threat modeling en paralelo (opcional pero recomendado).

---

### 8.5. Observabilidad

| Componente | Herramienta | Métricas |
|------------|-------------|----------|
| **Logs estructurados** | zerolog (Go) | JSON logs con trace_id, user_id, timestamp |
| **Métricas** | Prometheus (v1.1) | Request rate, latency, error rate |
| **Tracing** | OpenTelemetry (v2.0) | Distributed tracing backend → DB |
| **Healthcheck** | `/health` endpoint | Status: UP/DOWN, DB connection OK |

**MVP:** Logs a stdout (capturados por Docker), sin métricas avanzadas.  
**v1.1:** Prometheus + Grafana dashboards.  
**v2.0:** ELK stack o Datadog.

---

## 9. Diagrama de Flujo — Crear Horario (HU-013)

```
┌────────┐                    ┌──────────┐                  ┌─────────────┐
│Usuario │                    │ Frontend │                  │   Backend   │
└───┬────┘                    └────┬─────┘                  └──────┬──────┘
    │                              │                               │
    │ 1. Completa formulario       │                               │
    │ (fecha, hora, ambiente, etc) │                               │
    │─────────────────────────────>│                               │
    │                              │                               │
    │                              │ 2. POST /api/v1/horarios      │
    │                              │   Body: { fecha, horaInicio, ...}
    │                              │──────────────────────────────>│
    │                              │                               │
    │                              │                               │ 3. Parse & validate request
    │                              │                               │
    │                              │                               │ 4. CrearHorarioCommand.Execute()
    │                              │                               │    ├─ ConflictoValidator.Validar()
    │                              │                               │    │  └─ Query horarios existentes
    │                              │                               │    │     (mismo ambiente + rango fecha)
    │                              │                               │    │
    │                              │                               │    ├─ Si conflicto:
    │                              │                               │    │  └─ Return ConflictoAmbienteError
    │                              │                               │    │
    │                              │                               │    └─ Si OK:
    │                              │                               │       └─ HorarioRepo.Save(horario)
    │                              │                               │          └─ INSERT INTO horarios...
    │                              │                               │
    │                              │ 5a. Si conflicto: 409 Conflict│
    │                              │   { error: "Ambiente no disponible",
    │                              │     tipo: "conflicto_ambiente" }
    │                              │<──────────────────────────────│
    │                              │                               │
    │                              │ 5b. Si éxito: 201 Created    │
    │                              │   { id, fecha, horaInicio, ... }
    │                              │<──────────────────────────────│
    │                              │                               │
    │ 6a. Si conflicto:            │                               │
    │    Mostrar ConflictoAlert    │                               │
    │<─────────────────────────────│                               │
    │                              │                               │
    │ 6b. Si éxito:                │                               │
    │    Mostrar toast éxito       │                               │
    │    Redirigir a lista horarios│                               │
    │<─────────────────────────────│                               │
    │                              │                               │
```

---

## 10. Decisiones Técnicas (ADRs)

Las siguientes decisiones arquitectónicas críticas están documentadas en ADRs separados:

| ADR | Título | Decisión |
|-----|--------|----------|
| [ADR-001](adr/adr-001.md) | Framework HTTP Backend | **Gin** sobre Chi y net/http |
| [ADR-002](adr/adr-002.md) | ORM vs SQL Raw | **pgx SQL raw** sobre GORM |
| [ADR-003](adr/adr-003.md) | Build Tool Frontend | **Vite** sobre Create React App |
| [ADR-004](adr/adr-004.md) | State Management Frontend | **Context API + React Query** sobre Redux/Zustand |

Ver [`03-design/adr/`](adr/) para detalles completos de cada ADR.

---

## 11. Deployment Architecture

### Dev Environment

```
developer-machine:
  ├── docker-compose.yml
  │   ├── backend:8080 (Go con hot-reload)
  │   ├── frontend:3000 (Vite dev server)
  │   └── postgres:5432 (PostgreSQL)
  └── volumes:
      └── pgdata/ (persistencia local)
```

**Comandos:**
- `docker-compose up -d` — Levanta todo el stack
- `docker-compose logs -f backend` — Ver logs backend
- `docker-compose exec postgres psql -U user -d sena_horarios` — Acceder a DB

---

### Staging Environment

```
staging-server:
  ├── Nginx (reverse proxy + static files)
  │   ├── / → frontend build (HTML/CSS/JS estáticos)
  │   └── /api → backend:8080 (proxy_pass)
  ├── Backend container (Docker)
  └── PostgreSQL container (Docker)
```

**Acceso:** `https://staging.horarios.sena.edu.co`

---

### Production Environment (v1.0)

```
production-server:
  ├── Nginx (SSL termination + reverse proxy)
  │   ├── HTTPS (Let's Encrypt certificate)
  │   ├── / → frontend build
  │   └── /api → backend:8080
  ├── Backend container (Docker, 2 replicas v1.1+)
  ├── PostgreSQL (Docker + volume persistence)
  └── Backup cron job (pg_dump diario 3am)
```

**Acceso:** `https://horarios.sena.edu.co`

**CI/CD (v1.1):** GitHub Actions → Build → Test → Deploy staging → Manual approval → Deploy prod

---

## 12. Restricciones y Asunciones

### Restricciones

| ID | Restricción | Impacto |
|----|------------|---------|
| **C-001** | Infraestructura Docker disponible | ⚠️ Pendiente validación con TI SENA (open question Q2) |
| **C-002** | Sin presupuesto para cloud en MVP | Deployment on-premises obligatorio |
| **C-003** | Equipo de 2 devs (backend + frontend) | Arquitectura debe ser simple, no microservicios |
| **C-004** | Timeline 14 semanas para MVP | Features avanzadas diferidas a v1.1+ |
| **C-005** | Sin autenticación robusta en MVP | Placeholder básico — LDAP/AD en v2.0 |

---

### Asunciones

| ID | Asunción | Riesgo si Falso |
|----|----------|-----------------|
| **A-001** | Velocidad 12-15 SP/sprint | Timeline desfasado — activar buffer sprints 4-5 |
| **A-002** | Coordinadores tienen habilidades digitales básicas | Capacitación extendida requerida |
| **A-003** | PostgreSQL cubre todos los requisitos | N/A — validado con perfil de proyecto |
| **A-004** | <50 usuarios concurrentes en MVP | Performance suficiente con single instance |
| **A-005** | TI SENA proveerá ambiente staging semana 6 | Retraso en UAT — escalar a stakeholder |

---

## 13. Roadmap Técnico

### Sprint 1-2 (Fundamentos)
- ✅ Estructura de carpetas Hexagonal
- ✅ Entities de dominio (Horario, Ambiente, Instructor, Ficha)
- ✅ PostgreSQL schema inicial
- ✅ CRUD endpoints backend
- ✅ Componentes React básicos (formularios + listas)

### Sprint 3 (Core Lógica)
- ✅ ConflictoValidator implementado
- ✅ Validaciones automáticas (HU-017, HU-018, HU-019)
- ✅ Frontend: ConflictoAlert component
- ✅ Integration tests con testcontainers

### Sprint 4-5 (Vistas + Refinamiento)
- ✅ CalendarioSemanal component (HU-024)
- ✅ AgendaInstructor component (HU-027)
- ✅ Observaciones (HU-021, HU-022)
- ✅ Refactoring + performance tuning

### Sprint 6-8 (QA + Deployment)
- ✅ Testing exhaustivo (A09)
- ✅ Deployment a staging
- ✅ UAT con usuarios reales
- ✅ Go-Live producción

---

## 14. Referencias

- **Pattern Guide:** [`03-design/pattern-guide.md`](pattern-guide.md)
- **PRD:** [`02-definition/prd.md`](../02-definition/prd.md)
- **Backlog:** [`02-definition/backlog.md`](../02-definition/backlog.md)
- **Release Plan:** [`02-definition/release-plan.md`](../02-definition/release-plan.md)
- **ADRs:** [`03-design/adr/`](adr/)

---

## Aprobaciones

| Rol | Nombre | Fecha | Firma |
|-----|--------|-------|-------|
| Tech Lead | A06 | 2026-05-20 | ✅ |
| Data Architect | A10 | Pendiente | ⏸️ Debe producir data-model.md |
| UX Designer | A03 | Pendiente | ⏸️ Debe producir ux-flows.md |
| Security Designer | A04 | Opcional | ⏸️ Threat model si PII crítico |

**Estado:** ✅ **APROBADO — BASE PARA DESIGN FASE 3**

---

**Producido:** 2026-05-20  
**Última revisión:** 2026-05-20  
**Próxima revisión:** Post-Sprint 3 (validar decisiones técnicas con realidad)
