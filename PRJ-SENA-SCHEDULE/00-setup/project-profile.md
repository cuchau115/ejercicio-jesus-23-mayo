# Project Profile — PRJ-SENA-SCHEDULE

> **Owner:** A00 (Orquestador) | **Producido en:** 00-setup | **Consumido por:** todos
>
> Este archivo determina qué agentes se activan y con qué profundidad.
> Sin este perfil, A00 activa el flujo completo por defecto (conservador).

---

## 1. Identificación

| Campo | Valor |
|-------|-------|
| project_key | `PRJ-SENA-SCHEDULE` |
| project_type | `greenfield` |
| owner_humano | Usuario SENA |
| fecha_kickoff | 2026-05-20 |

## 2. Dimensiones de complejidad

| Dimensión | Valor | Implicancia |
|-----------|-------|-------------|
| **Tiene interfaz de usuario** | si | Activa A03 (UX) en fase 03 |
| **Canal UI** | web | Especificación separada si "ambos" |
| **Persistencia** | sql | Activa A10 y A13 si hay persistencia |
| **Complejidad de dominio** | media | "alta" activa DDD táctico + A01 asesor |
| **Volumen esperado** | < 10k users | Alta exige A13 fase 03 obligatorio |
| **Concurrencia** | media | Alta exige CQRS/event sourcing considerado |
| **PII / compliance** | basico | Regulado activa A04 con nivel OWASP ASVS L2+ |
| **Integraciones externas** | 0 | 4+ activa contratos formales y anti-corruption layer |
| **Tiempo real** | no | Streaming activa consideración de patrón event-driven |
| **Multi-tenant** | no | Hard exige estrategia de BD (schema/DB por tenant) |
| **Disponibilidad requerida** | best-effort | > 99.9% exige A13 fase 03 con RTO/RPO exigentes |

## 3. Matriz de activación de agentes

| Agente | Capacidad | Activación | Estado |
|--------|-----------|------------|--------|
| A00 | orchestration | Siempre | ✅ ACTIVO |
| A01 | asesor arquitecto | Si complejidad_dominio >= media OR volumen > 1M | ⏸️ OPCIONAL (dominio medio pero volumen bajo) |
| A02 | experto agentes | Nunca en runs de proyecto (solo framework) | ❌ N/A |
| A03 | UX design | Si tiene_UI = si | ✅ REQUERIDO |
| A04 | security design | Si PII != ninguno OR integraciones >= 1 | ⚠️ OPCIONAL (PII básico, sin integraciones) |
| A05 | product | Siempre | ✅ REQUERIDO |
| A06 | tech lead (patterns + arch) | Siempre | ✅ REQUERIDO |
| A07 | backend | Si persistencia != ninguna OR tiene lógica servidor | ✅ REQUERIDO |
| A08 | frontend | Si tiene_UI = si | ✅ REQUERIDO |
| A09 | QA | Siempre (con profundidad según perfil) | ✅ REQUERIDO |
| A10 | data modeling | Si persistencia != ninguna | ✅ REQUERIDO |
| A11 | devops release | Siempre | ✅ REQUERIDO |
| A12 | appsec | Si compliance = regulado OR disponibilidad >= 99.9% | ⏸️ OPCIONAL |
| A13 | DBA expert | Si persistencia != ninguna AND (volumen > 10k OR compliance = regulado OR disponibilidad >= 99.9%) | ⏸️ OPCIONAL (volumen bajo) |

## 4. Perfil de patrones sugeridos

Basado en las dimensiones del proyecto:

| Condición del proyecto | Patrón sugerido | Aplicable |
|------------------------|-----------------|-----------|
| Dominio trivial / CRUD | P04 Modular Monolith | ❌ |
| Dominio medio | **P02 Hexagonal (Ports & Adapters)** | ✅ **RECOMENDADO** |
| Dominio alto + equipo >= 3 | P01 DDD táctico + P02 Hexagonal | ⚠️ Considerar si crece |
| Lecturas >> Escrituras, volumen alto | P05 CQRS | ❌ No aplica (MVP) |
| Event-driven / integraciones asíncronas | P05 CQRS + Event Sourcing | ❌ No aplica |
| Equipo pequeño, producto simple | P03 Clean Architecture ligera | ⚠️ Alternativa válida |

**Decisión sugerida:** Arquitectura Hexagonal (Ports & Adapters) por:
- Dominio de complejidad media bien definido
- Facilita testing y mantenibilidad
- Alineado con stack Go + React validado
- Escalabilidad futura sin reescritura

> A06 tiene la decisión final. Este perfil es sugerencia, no imposición.

## 5. Profundidad de QA

Calibración para A09:

| Condición | Profundidad QA | Aplica |
|-----------|----------------|--------|
| trivial + sin PII | Funcional + smoke regresión | ❌ |
| media | **Funcional + integración + regresión + accesibilidad** | ✅ **APLICAR** |
| alta O compliance = regulado | Todo lo anterior + performance + seguridad + chaos | ❌ |

**Estrategia QA:**
- Testing funcional de todas las entidades CRUD
- Testing de integración Backend ↔ PostgreSQL
- Testing de integración Frontend ↔ Backend API
- Regresión automatizada en CI/CD
- Accesibilidad básica (WCAG 2.1 Level A mínimo)

## 6. Gates obligatorios según perfil

| Gate | Activación | Estado |
|------|------------|--------|
| HALT-PATTERN | Siempre (pattern-guide.md obligatorio antes de build) | ⏸️ PENDING |
| HALT-DBDESIGN | Si persistencia != ninguna (db-design.md antes de build) | ⏸️ PENDING |
| HALT-SECURITY | Si compliance = regulado (security-threat-model.md antes de build) | ⏸️ OPCIONAL |
| HALT-UX | Si tiene_UI = si (ui-spec.md antes de A08) | ⏸️ PENDING |

## 7. Consideraciones específicas del dominio

### Dominio: Gestión de Horarios Académicos SENA

**Entidades principales:**
1. **Ambiente**: Espacio físico con capacidad y características
2. **Horario**: Evento programado con fecha, hora, ambiente, instructor, ficha
3. **Ficha**: Grupo de formación con programa y cohorte específicos
4. **Instructor**: Recursos humanos (planta o contratista)
5. **Observación**: Anotaciones vinculadas a horarios

**Reglas de negocio esperadas:**
- Un ambiente no puede tener dos horarios simultáneos
- Un instructor no puede estar en dos lugares simultáneamente
- Una ficha tiene múltiples horarios a lo largo del programa
- Validación de capacidad de ambiente vs tamaño de ficha
- Diferenciación contractual: planta vs contratista

**Complejidad justificada:**
- Gestión de conflictos de recursos (ambientes, instructores)
- Múltiples dimensiones de consulta (por ficha, por instructor, por ambiente, por fecha)
- Reglas de asignación con restricciones

## 8. Stack y arquitectura

| Componente | Tecnología | Justificación |
|------------|------------|---------------|
| Backend | Go 1.21+ | Performance, concurrencia, tipado fuerte |
| Frontend | React 18 + TypeScript | Ecosistema maduro, componentes reutilizables |
| Base de datos | PostgreSQL 15+ | Relacional, ACID, constraints nativos, JSON support |
| Containerización | Docker Compose | Despliegue consistente, ambiente de desarrollo unificado |
| Build tool (FE) | Vite | HMR rápido, optimización de assets |

**Arquitectura:** Hexagonal (Ports & Adapters)
- `domain/`: Entidades, reglas de negocio, interfaces
- `application/`: Casos de uso, servicios de aplicación
- `ports/`: Contratos (HTTP handlers, DB repositories)
- `adapters/`: Implementaciones concretas (Gin, PostgreSQL driver)

## 9. Registro

- **Producido por:** A00 en `00-setup`
- **Ruta:** `runs/PRJ-SENA-SCHEDULE/00-setup/project-profile.md`
- **Entrada obligatoria de:** `run-state.json` (campo `profile_ref`)
- **Modificación:** Solo A00 puede actualizar, siempre con entrada en `decision-log.md`
- **Versión:** 1.0
- **Última actualización:** 2026-05-20

---

## Resumen ejecutivo

**Proyecto:** MVP de gestión de horarios para SENA con complejidad media, stack validado Go+React+PostgreSQL, arquitectura hexagonal sugerida.

**Agentes críticos:** A05 (Product), A06 (Tech Lead), A07 (Backend), A08 (Frontend), A10 (Data), A09 (QA), A11 (DevOps), A03 (UX).

**Gates obligatorios:** HALT-PATTERN, HALT-DBDESIGN, HALT-UX.

**Próximo paso:** Activar A05 para fase 01-discovery con `cap-product-discovery`.
