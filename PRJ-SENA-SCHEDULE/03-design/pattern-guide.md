# Pattern Guide — PRJ-SENA-SCHEDULE

> **Producido por:** A06 (Tech Lead)  
> **Fecha:** 2026-05-20  
> **Fase:** 03-design (paso 1)  
> **Versión:** 1.0  
> **Capacidad:** cap-software-patterns

---

## 1. Patrón Arquitectónico Seleccionado

### Decisión: **P02 — Hexagonal Architecture (Ports & Adapters)**

**Justificación:**

Basándome en el análisis del [project-profile.md](../00-setup/project-profile.md), [prd.md](../02-definition/prd.md) y [backlog.md](../02-definition/backlog.md), Hexagonal Architecture es el patrón óptimo para este proyecto por:

1. **Complejidad de dominio media** con reglas de negocio claras:
   - Validación de conflictos de asignación (ambiente, instructor)
   - Restricciones de capacidad
   - Lógica de disponibilidad de recursos
   - Gestión de relaciones entre entidades (Horario → Ficha + Instructor + Ambiente)

2. **Alta testabilidad requerida** (A09 debe ejecutar testing extenso):
   - Ports permiten mocking de infraestructura (BD, validadores)
   - Domain lógica pura sin dependencias externas
   - Testing de reglas de negocio sin levantar PostgreSQL

3. **Independencia de frameworks y tecnologías**:
   - Backend puede evolucionar frameworks Go sin tocar dominio
   - Frontend React consume API REST sin acoplamiento a lógica de negocio
   - PostgreSQL puede ser reemplazado por otra BD con cambio solo en adapters

4. **Alineación con stack validado** (Go + React + PostgreSQL):
   - Go facilita estructura hexagonal con packages claros
   - Separación natural entre API HTTP (adapter in) y lógica (domain)

5. **Vida útil esperada del proyecto** (>2 años según roadmap v2.0):
   - Hexagonal facilita evolución sin grandes refactorings
   - Nuevos adapters (móvil, integraciones) se agregan sin tocar core

---

## 2. Patrones Considerados y Descartados

### P01 — Domain-Driven Design (DDD Táctico)

**Descartado porque:**
- ❌ Complejidad de dominio **media**, no alta
- ❌ No hay múltiples bounded contexts (un solo sistema monolítico)
- ❌ Agregados complejos no son necesarios (entidades simples con relaciones claras)
- ❌ Overhead de conceptos (Value Objects, Domain Events) no justificado en MVP

**Posible reevaluación en v2.0** si:
- Se agregan submódulos complejos (gestión de evaluaciones, certificaciones)
- Multi-tenancy requiere aislamiento de dominios por centro

---

### P03 — Clean Architecture

**Descartado porque:**
- ❌ Overhead de capas adicionales sin beneficio proporcional
- ❌ Hexagonal es más pragmático y suficiente para este alcance
- ❌ Equipo pequeño (2 devs) — simplicidad sobre pureza arquitectónica

**Semejanza con Hexagonal:** Clean Architecture y Hexagonal son primos cercanos; la decisión favorece terminología más directa (Ports/Adapters vs Use Cases/Entities/Gateways)

---

### P04 — Modular Monolith

**Descartado porque:**
- ❌ Dominio tiene reglas de negocio que requieren aislamiento (no es CRUD puro)
- ❌ Sin Ports, testing sería más acoplado a infraestructura
- ❌ Modular Monolith es mejor para CRUDs triviales — este proyecto tiene lógica de validación crítica

---

### P05 — CQRS / Event Sourcing

**Descartado porque:**
- ❌ No hay diferencia significativa entre lecturas y escrituras en MVP
- ❌ Complejidad no justificada para volumen < 10k usuarios
- ❌ Event Sourcing agrega overhead de reconstrucción de estado
- ❌ Posible en v2.0 si analytics requieren historial completo de eventos

---

## 3. Estructura de Carpetas — Backend (Go)

```
backend/
├── cmd/
│   └── api/
│       └── main.go                 # Punto de entrada — inicializa adapters y levanta servidor
│
├── internal/                       # Código privado del proyecto (no exportable)
│   │
│   ├── domain/                     # ⭐ CORE — Lógica de negocio pura
│   │   ├── entities/               # Entidades con identidad
│   │   │   ├── ambiente.go
│   │   │   ├── instructor.go
│   │   │   ├── ficha.go
│   │   │   ├── horario.go
│   │   │   └── observacion.go
│   │   │
│   │   ├── services/               # Domain services — lógica sin estado propio
│   │   │   ├── conflicto_validator.go    # Validación de conflictos (HU-017, HU-018, HU-019)
│   │   │   └── capacidad_validator.go
│   │   │
│   │   └── errors/                 # Errores de dominio (ConflictoAmbienteError, etc.)
│   │       └── domain_errors.go
│   │
│   ├── application/                # Casos de uso — orquesta dominio
│   │   ├── commands/               # Comandos (escritura)
│   │   │   ├── crear_horario.go
│   │   │   ├── crear_ambiente.go
│   │   │   ├── crear_instructor.go
│   │   │   └── registrar_observacion.go
│   │   │
│   │   ├── queries/                # Consultas (lectura)
│   │   │   ├── listar_horarios.go
│   │   │   ├── obtener_calendario_ficha.go
│   │   │   └── obtener_agenda_instructor.go
│   │   │
│   │   └── dtos/                   # Data Transfer Objects — contratos de aplicación
│   │       ├── horario_dto.go
│   │       └── calendario_dto.go
│   │
│   ├── ports/                      # ⭐ CONTRATOS (interfaces)
│   │   ├── in/                     # Puertos de entrada (implementados por adapters/in)
│   │   │   ├── horario_service.go      # Interface que expone HTTP handler
│   │   │   ├── ambiente_service.go
│   │   │   └── instructor_service.go
│   │   │
│   │   └── out/                    # Puertos de salida (implementados por adapters/out)
│   │       ├── horario_repository.go    # Interface que implementa PostgreSQL adapter
│   │       ├── ambiente_repository.go
│   │       ├── instructor_repository.go
│   │       ├── ficha_repository.go
│   │       └── transaction_manager.go   # Transacciones de BD
│   │
│   └── adapters/                   # ⭐ IMPLEMENTACIONES concretas
│       ├── in/                     # Entrada — HTTP REST API
│       │   ├── http/
│       │   │   ├── server.go           # Inicialización Gin (o Chi)
│       │   │   ├── routes.go           # Definición de rutas
│       │   │   ├── middleware/         # Auth, CORS, logging
│       │   │   │   └── logger.go
│       │   │   └── handlers/           # HTTP handlers por entidad
│       │   │       ├── horario_handler.go
│       │   │       ├── ambiente_handler.go
│       │   │       ├── instructor_handler.go
│       │   │       ├── ficha_handler.go
│       │   │       └── observacion_handler.go
│       │   │
│       │   └── requests/               # Request models (validación de entrada)
│       │       ├── crear_horario_request.go
│       │       └── filtro_horarios_request.go
│       │
│       └── out/                    # Salida — PostgreSQL, cache, etc.
│           ├── postgres/
│           │   ├── connection.go       # Pool de conexiones pgx
│           │   ├── migrations/         # SQL migrations (golang-migrate)
│           │   │   ├── 001_create_ambientes.up.sql
│           │   │   ├── 002_create_instructores.up.sql
│           │   │   ├── 003_create_fichas.up.sql
│           │   │   ├── 004_create_horarios.up.sql
│           │   │   └── 005_create_observaciones.up.sql
│           │   │
│           │   └── repositories/       # Implementaciones de ports/out
│           │       ├── horario_repo.go
│           │       ├── ambiente_repo.go
│           │       ├── instructor_repo.go
│           │       ├── ficha_repo.go
│           │       └── observacion_repo.go
│           │
│           └── transaction/
│               └── pg_transaction_manager.go   # Implementa transaction_manager port
│
├── pkg/                            # Código reutilizable (exportable)
│   ├── logger/                     # Logger estructurado (zerolog o slog)
│   ├── validator/                  # Validador de DTOs (go-playground/validator)
│   └── config/                     # Carga de configuración desde env
│
├── go.mod
├── go.sum
├── .env.example
├── Dockerfile
└── README.md
```

---

## 4. Reglas de Dependencia (Hexagonal — No Negociables)

### Diagrama de Dependencias

```
┌─────────────────────────────────────────────────┐
│                  adapters/in                    │
│          (HTTP Handlers, CLI, gRPC)             │
└──────────────────┬──────────────────────────────┘
                   │ implementa
                   ▼
┌─────────────────────────────────────────────────┐
│                   ports/in                      │
│           (Interfaces de servicios)             │
└──────────────────┬──────────────────────────────┘
                   │ llama
                   ▼
┌─────────────────────────────────────────────────┐
│                application                      │
│      (Commands, Queries, Use Cases)             │
└──────────────────┬──────────────────────────────┘
                   │ orquesta
                   ▼
┌─────────────────────────────────────────────────┐
│                  domain                         │
│    (Entities, Services, Business Logic)         │
│              ⭐ SIN DEPENDENCIAS ⭐              │
└──────────────────┬──────────────────────────────┘
                   │ define interfaces
                   ▼
┌─────────────────────────────────────────────────┐
│                  ports/out                      │
│      (Repository interfaces, External APIs)     │
└──────────────────┬──────────────────────────────┘
                   │ implementa
                   ▼
┌─────────────────────────────────────────────────┐
│                 adapters/out                    │
│         (PostgreSQL, Redis, APIs externas)      │
└─────────────────────────────────────────────────┘
```

### Reglas Obligatorias

| # | Regla | Violación = HALT |
|---|-------|------------------|
| **R1** | `domain/` NO puede importar `application/`, `ports/`, `adapters/` | ✅ Domain es puro |
| **R2** | `application/` solo importa `domain/` y `ports/out` (interfaces) | ✅ Use cases orquestan |
| **R3** | `ports/` solo define interfaces, cero implementación | ✅ Contratos puros |
| **R4** | `adapters/in` implementa `ports/in` | ✅ HTTP handler implementa service interface |
| **R5** | `adapters/out` implementa `ports/out` | ✅ PostgreSQL repo implementa repository interface |
| **R6** | `adapters/` puede importar frameworks externos (Gin, pgx) | ✅ Acoplamiento controlado |
| **R7** | Entities en `domain/` no tienen tags de JSON/DB | ✅ Domain agnóstico de serialización |

### Validación en Code Review (A06 responsable)

**Antes de aprobar PR:**
- ✅ Grep `"github.com/gin-gonic"` en `internal/domain/` → debe retornar 0 resultados
- ✅ Grep `"github.com/jackc/pgx"` en `internal/domain/` → debe retornar 0 resultados
- ✅ Verificar que entities no tienen tags `json:` ni `db:`
- ✅ Confirmar que `ports/out/*.go` solo contiene interfaces (keyword `type`)

---

## 5. Naming Conventions — Backend (Go)

### Packages

| Tipo | Naming | Ejemplo |
|------|--------|---------|
| Entities | Singular, lowercase | `domain/entities/horario.go` |
| Services | Sufijo `_service` | `domain/services/conflicto_validator.go` |
| Repositories (interface) | Sufijo `_repository` | `ports/out/horario_repository.go` |
| Repositories (impl) | Sufijo `_repo` | `adapters/out/postgres/repositories/horario_repo.go` |
| Handlers | Sufijo `_handler` | `adapters/in/http/handlers/horario_handler.go` |
| Commands | Verbo + sustantivo | `application/commands/crear_horario.go` |
| Queries | Verbo + sustantivo | `application/queries/listar_horarios.go` |

### Structs y Funciones

```go
// Entity (domain)
type Horario struct {
    ID           uuid.UUID
    Fecha        time.Time
    HoraInicio   time.Time
    HoraFin      time.Time
    AmbienteID   uuid.UUID
    InstructorID uuid.UUID
    FichaID      uuid.UUID
    Tema         string
}

// Domain service
type ConflictoValidator struct{}

func (v *ConflictoValidator) ValidarConflictoAmbiente(
    horario *Horario,
    horariosExistentes []*Horario,
) error {
    // Lógica de validación pura
}

// Port (interface)
type HorarioRepository interface {
    Save(horario *Horario) error
    FindByID(id uuid.UUID) (*Horario, error)
    FindByFilters(filters HorarioFilters) ([]*Horario, error)
}

// Adapter implementation
type PostgresHorarioRepo struct {
    db *pgxpool.Pool
}

func (r *PostgresHorarioRepo) Save(horario *Horario) error {
    // Implementación PostgreSQL
}

// HTTP Handler
type HorarioHandler struct {
    service ports.HorarioService  // Inyección de dependencia vía port
}

func (h *HorarioHandler) CrearHorario(c *gin.Context) {
    // Parsear request, llamar service, responder
}
```

---

## 6. Estructura de Carpetas — Frontend (React + TypeScript)

```
frontend/
├── src/
│   ├── modules/                    # Organización por feature (vertical slices)
│   │   │
│   │   ├── ambientes/
│   │   │   ├── components/         # Componentes React específicos
│   │   │   │   ├── AmbienteForm.tsx
│   │   │   │   ├── AmbienteList.tsx
│   │   │   │   └── AmbienteCard.tsx
│   │   │   ├── hooks/              # Custom hooks del módulo
│   │   │   │   └── useAmbientes.ts
│   │   │   ├── services/           # API clients (HTTP)
│   │   │   │   └── ambienteService.ts
│   │   │   ├── types/              # TypeScript types
│   │   │   │   └── ambiente.types.ts
│   │   │   └── pages/              # Páginas del módulo
│   │   │       ├── AmbientesPage.tsx
│   │   │       └── AmbienteDetailPage.tsx
│   │   │
│   │   ├── instructores/
│   │   │   ├── components/
│   │   │   ├── hooks/
│   │   │   ├── services/
│   │   │   ├── types/
│   │   │   └── pages/
│   │   │
│   │   ├── fichas/
│   │   │   └── ...
│   │   │
│   │   ├── horarios/               # ⭐ Módulo central
│   │   │   ├── components/
│   │   │   │   ├── HorarioForm.tsx
│   │   │   │   ├── HorarioList.tsx
│   │   │   │   ├── CalendarioSemanal.tsx      # HU-024
│   │   │   │   └── ConflictoAlert.tsx         # HU-020
│   │   │   ├── hooks/
│   │   │   │   ├── useHorarios.ts
│   │   │   │   └── useValidacionConflictos.ts
│   │   │   ├── services/
│   │   │   │   └── horarioService.ts
│   │   │   ├── types/
│   │   │   │   └── horario.types.ts
│   │   │   └── pages/
│   │   │       ├── HorariosPage.tsx
│   │   │       └── CalendarioFichaPage.tsx
│   │   │
│   │   ├── observaciones/
│   │   │   └── ...
│   │   │
│   │   └── dashboard/              # Vista ejecutiva (HU-030)
│   │       └── ...
│   │
│   ├── shared/                     # Código compartido cross-module
│   │   ├── components/             # Componentes reutilizables
│   │   │   ├── ui/                 # Componentes UI básicos
│   │   │   │   ├── Button.tsx
│   │   │   │   ├── Input.tsx
│   │   │   │   ├── Modal.tsx
│   │   │   │   └── Card.tsx
│   │   │   └── layout/             # Layouts
│   │   │       ├── Header.tsx
│   │   │       ├── Sidebar.tsx
│   │   │       └── MainLayout.tsx
│   │   │
│   │   ├── hooks/                  # Hooks genéricos
│   │   │   ├── useApi.ts
│   │   │   ├── useDebounce.ts
│   │   │   └── useLocalStorage.ts
│   │   │
│   │   ├── services/               # Servicios cross-cutting
│   │   │   ├── httpClient.ts       # Axios config centralizada
│   │   │   └── errorHandler.ts
│   │   │
│   │   ├── types/                  # Types compartidos
│   │   │   ├── api.types.ts
│   │   │   └── common.types.ts
│   │   │
│   │   └── utils/                  # Utilidades
│   │       ├── formatters.ts
│   │       ├── validators.ts
│   │       └── dateHelpers.ts
│   │
│   ├── router/                     # React Router setup
│   │   ├── AppRouter.tsx
│   │   └── routes.ts
│   │
│   ├── App.tsx                     # Componente raíz
│   ├── main.tsx                    # Punto de entrada
│   └── vite-env.d.ts
│
├── public/
├── index.html
├── vite.config.ts
├── tsconfig.json
├── .env.example
├── Dockerfile
└── README.md
```

---

## 7. Reglas de Dependencia — Frontend (React)

### Principios de Organización

| Principio | Descripción |
|-----------|-------------|
| **Feature-first** | Organizar por módulo funcional (ambientes, horarios), no por tipo técnico (components/, services/) |
| **Colocation** | Mantener archivos relacionados cerca (componente + hook + service en mismo módulo) |
| **Shared last** | Código va a `shared/` solo si es usado por ≥3 módulos |

### Reglas de Importación

| # | Regla | Ejemplo |
|---|-------|---------|
| **F1** | Módulos NO pueden importar de otros módulos directamente | ❌ `ambientes/` NO importa de `horarios/` |
| **F2** | Módulos solo importan de `shared/` | ✅ `horarios/` importa `shared/components/ui/Button` |
| **F3** | `shared/` NO importa de módulos | ✅ Shared es agnóstico de features |
| **F4** | Services solo llaman API backend, cero lógica de negocio | ✅ `horarioService.ts` solo hace `fetch()` |
| **F5** | Hooks contienen lógica de estado y side effects | ✅ `useHorarios.ts` maneja loading, error, data |

### Validación en Code Review (A08 responsable)

**Antes de aprobar PR:**
- ✅ Grep `import.*modules/ambientes` en `modules/horarios/` → debe retornar 0 resultados
- ✅ Verificar que services no tienen `if`, `for`, validaciones complejas (solo HTTP calls)
- ✅ Confirmar que componentes UI en `shared/` no tienen `useState` con lógica de dominio

---

## 8. Naming Conventions — Frontend (TypeScript)

### Archivos y Componentes

| Tipo | Naming | Ejemplo |
|------|--------|---------|
| Componentes | PascalCase | `AmbienteForm.tsx`, `HorarioList.tsx` |
| Hooks | camelCase con `use` prefix | `useHorarios.ts`, `useValidacionConflictos.ts` |
| Services | camelCase con sufijo `Service` | `horarioService.ts`, `ambienteService.ts` |
| Types | PascalCase con sufijo `.types.ts` | `horario.types.ts`, `ambiente.types.ts` |
| Utils | camelCase | `formatters.ts`, `dateHelpers.ts` |
| Pages | PascalCase con sufijo `Page` | `HorariosPage.tsx`, `CalendarioFichaPage.tsx` |

### TypeScript Types

```typescript
// Entity type (espejo de backend)
export interface Horario {
  id: string;
  fecha: string;  // ISO 8601
  horaInicio: string;
  horaFin: string;
  ambienteId: string;
  instructorId: string;
  fichaId: string;
  tema: string;
}

// DTO para creación
export interface CrearHorarioRequest {
  fecha: string;
  horaInicio: string;
  horaFin: string;
  ambienteId: string;
  instructorId: string;
  fichaId: string;
  tema?: string;
}

// Response de API con metadata
export interface HorariosResponse {
  data: Horario[];
  total: number;
  page: number;
  pageSize: number;
}

// Error de validación de conflicto
export interface ConflictoError {
  tipo: 'ambiente' | 'instructor' | 'capacidad';
  mensaje: string;
  horarioConflictivo?: Horario;
}
```

---

## 9. Flujo de Datos End-to-End (Ejemplo: Crear Horario)

### Request Flow

```
1. Usuario completa formulario
   └─> HorarioForm.tsx (React Component)

2. Submit llama hook
   └─> useHorarios.ts (Custom Hook)
       └─> crearHorario({ fecha, horaInicio, ... })

3. Hook llama service
   └─> horarioService.ts (API Client)
       └─> POST /api/v1/horarios
           Headers: Content-Type: application/json
           Body: { fecha, horaInicio, horaFin, ambienteId, instructorId, fichaId }

4. Request llega a backend
   └─> adapters/in/http/handlers/horario_handler.go
       └─> CrearHorario(c *gin.Context)
           ├─> Parsear request (validación básica)
           ├─> Mapear a DTO
           └─> Llamar port: h.service.CrearHorario(dto)

5. Port delega a application layer
   └─> application/commands/crear_horario.go
       └─> Execute(cmd CrearHorarioCommand) error
           ├─> Validar con domain service: conflictoValidator.Validar(horario)
           ├─> Si no hay conflicto: horarioRepo.Save(horario)
           └─> Si hay conflicto: return ConflictoAmbienteError

6. Domain service ejecuta regla de negocio
   └─> domain/services/conflicto_validator.go
       └─> ValidarConflictoAmbiente(horario, existentes)
           ├─> Iterar existentes
           ├─> Comparar rangos de tiempo
           └─> Return error si hay solapamiento

7. Repository persiste (si pasa validación)
   └─> adapters/out/postgres/repositories/horario_repo.go
       └─> Save(horario *Horario) error
           └─> INSERT INTO horarios (...) VALUES (...) RETURNING id

8. Response vuelve al frontend
   └─> 201 Created: { id, fecha, horaInicio, ... }
   └─> 400 Bad Request: { error: "Ambiente no disponible", tipo: "conflicto_ambiente" }

9. Hook actualiza estado
   └─> useHorarios.ts
       ├─> Si success: invalidar query, mostrar toast éxito
       └─> Si error: setError(error.mensaje), mostrar ConflictoAlert component

10. UI refleja cambio
    └─> HorarioList.tsx se actualiza con nuevo horario
    └─> O ConflictoAlert.tsx muestra mensaje de error
```

---

## 10. Testing Strategy por Capa

### Backend

| Capa | Tipo de Test | Herramientas Go | Cobertura Meta |
|------|--------------|-----------------|----------------|
| **domain/** | Unit tests | `testing`, `testify/assert` | ≥85% |
| **application/** | Unit tests con mocks | `testify/mock`, `gomock` | ≥80% |
| **adapters/out/** | Integration tests | `testcontainers-go` (PostgreSQL) | ≥70% |
| **adapters/in/** | E2E tests | `httptest`, `testify/suite` | ≥60% |

**Ejemplo de test (domain service):**
```go
func TestConflictoValidator_ValidarConflictoAmbiente(t *testing.T) {
    validator := &ConflictoValidator{}
    
    // Given: Horario existente 08:00-10:00
    existente := &Horario{
        AmbienteID: uuid.New(),
        HoraInicio: parseTime("08:00"),
        HoraFin:    parseTime("10:00"),
    }
    
    // When: Intento crear horario 09:00-11:00 mismo ambiente
    nuevo := &Horario{
        AmbienteID: existente.AmbienteID,
        HoraInicio: parseTime("09:00"),
        HoraFin:    parseTime("11:00"),
    }
    
    // Then: Debe retornar error de conflicto
    err := validator.ValidarConflictoAmbiente(nuevo, []*Horario{existente})
    assert.Error(t, err)
    assert.IsType(t, &ConflictoAmbienteError{}, err)
}
```

### Frontend

| Tipo de Test | Herramientas | Cobertura Meta |
|--------------|--------------|----------------|
| **Unit tests** (hooks, utils) | Vitest, React Testing Library | ≥75% |
| **Component tests** | Vitest, React Testing Library | ≥70% |
| **Integration tests** (API mocking) | MSW (Mock Service Worker) | ≥60% |
| **E2E tests** | Playwright (opcional v1.1) | ≥50% |

**Ejemplo de test (componente):**
```typescript
import { render, screen, fireEvent } from '@testing-library/react';
import { HorarioForm } from './HorarioForm';

test('muestra error de conflicto cuando API retorna 400', async () => {
  // Mock API response
  mockHorarioService.crearHorario.mockRejectedValue({
    tipo: 'conflicto_ambiente',
    mensaje: 'Ambiente no disponible'
  });
  
  render(<HorarioForm />);
  
  // Completar formulario
  fireEvent.change(screen.getByLabelText('Fecha'), { target: { value: '2026-06-01' } });
  fireEvent.click(screen.getByText('Guardar'));
  
  // Verificar que se muestra alerta
  expect(await screen.findByText('Ambiente no disponible')).toBeInTheDocument();
});
```

---

## 11. Criterios de Validación del Patrón (Gates)

### HALT-PATTERN (Bloqueante para Build)

A07 y A08 **NO PUEDEN INICIAR** fase 04-build hasta que:

| Criterio | Verificación | Responsable |
|----------|-------------|-------------|
| ✅ Este `pattern-guide.md` existe y está aprobado | A00 valida handoff | A06 |
| ✅ Estructura de carpetas creada según este documento | `ls -R backend/ frontend/` | A07, A08 |
| ✅ README técnico con diagrama de capas | Exists `backend/README.md` | A07 |
| ✅ Ejemplo de entity + port + adapter (hello world) | Smoke test: `go run cmd/api/main.go` funciona | A07 |
| ✅ Frontend con módulo ejemplo (ambientes) | `npm run dev` levanta sin errores | A08 |

### Code Review Checklist (Durante Build)

**A06 rechazará PR si:**
- ❌ `domain/` importa `gin` o `pgx`
- ❌ Entities tienen tags `json:` o `db:`
- ❌ Handlers contienen lógica de negocio (más de mapear request/response)
- ❌ Repositories (interface) tienen implementación
- ❌ Frontend: módulo importa de otro módulo directamente
- ❌ Frontend: services tienen lógica compleja (solo HTTP calls permitidos)

---

## 12. Evolución del Patrón (Post-MVP)

### Posibles Ajustes en v1.1+

| Escenario | Patrón Alternativo | Trigger |
|-----------|-------------------|---------|
| Analytics requiere lecturas optimizadas | **CQRS ligero** | Queries lentas con joins complejos |
| Multi-tenancy requerido | **DDD con bounded contexts** | v2.0 — múltiples centros SENA |
| Microservicios necesarios | **Hexagonal se mantiene por servicio** | v2.0 — equipo >6 devs |

**Hexagonal facilita estas evoluciones** porque:
- CQRS: agregar ports/out separados para read models
- DDD: refactorizar domain/ en bounded contexts manteniendo ports
- Microservicios: cada servicio replica estructura hexagonal independiente

---

## 13. Referencias

- **Hexagonal Architecture (Alistair Cockburn):** https://alistair.cockburn.us/hexagonal-architecture/
- **Go Project Layout:** https://github.com/golang-standards/project-layout
- **Stack Validado:** [`agentic-sdlc-os/stacks/go-react-postgres.md`](C:\www\code-dev-projects\automatization-develop\agentic-sdlc-os\stacks\go-react-postgres.md)
- **Project Profile:** [`00-setup/project-profile.md`](../00-setup/project-profile.md)
- **PRD:** [`02-definition/prd.md`](../02-definition/prd.md)
- **Backlog:** [`02-definition/backlog.md`](../02-definition/backlog.md)

---

## Aprobaciones

| Rol | Nombre | Fecha | Firma |
|-----|--------|-------|-------|
| Tech Lead | A06 | 2026-05-20 | ✅ |
| Arquitecto IA (asesor) | A01 | N/A | ⏸️ Opcional |
| Backend Dev | A07 | Pendiente | ⏸️ Debe leer antes de iniciar |
| Frontend Dev | A08 | Pendiente | ⏸️ Debe leer antes de iniciar |

**Estado:** ✅ **APROBADO — GATE PATTERN DESBLOQUEADO**

---

**Producido:** 2026-05-20  
**Última revisión:** 2026-05-20  
**Próxima revisión:** Post-Sprint 1 (validar adherencia al patrón)
