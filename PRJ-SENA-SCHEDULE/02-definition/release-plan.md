# Release Plan — PRJ-SENA-SCHEDULE

> **Producido por:** A05 (Product Owner)  
> **Fecha:** 2026-05-20  
> **Fase:** 02-definition  
> **Versión:** 1.0

---

## 1. Estrategia de Releases

**Enfoque:** Entrega incremental con MVP funcional completo en Release 1.0, seguido de mejoras iterativas basadas en feedback de usuarios.

**Filosofía:** Priorizar valor de negocio (eliminación de conflictos) sobre features avanzadas. Cada release debe ser desplegable y usable en producción.

---

## 2. Roadmap Ejecutivo

| Release | Fecha Objetivo | Duración | HU | SP | Objetivo de Negocio |
|---------|---------------|----------|----|----|---------------------|
| **MVP (1.0)** | 2026-09-30 | 14 semanas | 24 | 78 | Sistema funcional completo con validación automática de conflictos |
| **v1.1** | 2026-11-15 | 6 semanas | 5 | 16 | Mejoras operativas y gestión de ciclo de vida |
| **v1.2** | 2026-12-31 | 6 semanas | 1 | 3 | Features de conveniencia (exportación) |
| **v2.0** | 2027-Q2 | TBD | TBD | TBD | Autenticación robusta, integraciones, multi-tenancy |

**Cadencia:** Releases cada 6-8 semanas post-MVP, con hotfixes según necesidad

---

## 3. Release 1.0 — MVP (Producto Mínimo Viable)

### Objetivo

Sistema centralizado de gestión de horarios SENA que elimina conflictos de asignación mediante validación automática, reduciendo tiempo operativo de coordinadores en 60%.

### Fecha Objetivo

**2026-09-30** (14 semanas desde kickoff 2026-05-20)

### Alcance

**Must Have:** 24 HU | 78 SP

#### Epic EPC-001: Gestión de Recursos
- ✅ HU-001: Crear ambiente
- ✅ HU-002: Listar ambientes
- ✅ HU-003: Editar ambiente
- ✅ HU-004: Eliminar ambiente
- ✅ HU-005: Registrar instructor (Planta/Contratista)
- ✅ HU-006: Listar instructores con filtro
- ✅ HU-007: Actualizar datos instructor

#### Epic EPC-002: Gestión de Programación
- ✅ HU-009: Crear ficha
- ✅ HU-010: Listar fichas
- ✅ HU-011: Editar ficha
- ✅ HU-013: Crear horario con asignaciones
- ✅ HU-014: Listar horarios con filtros
- ✅ HU-015: Modificar horario existente
- ✅ HU-016: Cancelar horario
- ✅ HU-017: Validar conflicto de ambiente (automático)
- ✅ HU-018: Validar conflicto de instructor (automático)
- ✅ HU-019: Validar capacidad de ambiente (warning)
- ✅ HU-020: Mostrar alertas de conflicto en UI
- ✅ HU-021: Registrar observación en horario
- ✅ HU-022: Ver observaciones de horario

#### Epic EPC-003: Consulta y Visualización
- ✅ HU-024: Ver calendario semanal de ficha
- ✅ HU-025: Ver detalles de horario en calendario
- ✅ HU-027: Ver agenda personal de instructor

### Criterios de Aceptación del Release

| Criterio | Métrica | Meta |
|----------|---------|------|
| **Funcionalidad completa** | % HU Must Have implementadas | 100% (24/24) |
| **Calidad** | Bugs P0/P1 abiertos | 0 |
| **Performance** | Tiempo validación conflictos | <500ms |
| **Usabilidad** | Usuarios completan flujo sin ayuda | ≥80% |
| **Estabilidad** | Uptime en staging última semana | ≥98% |

### Hitos del MVP

| Hito | Fecha | Entregables | Responsable |
|------|-------|-------------|-------------|
| **M1: Fundamentos** | 2026-06-30 | CRUD Ambientes + Instructores + Fichas (Sprint 1) | A07, A08 |
| **M2: Core Programación** | 2026-07-31 | CRUD Horarios + Validaciones (Sprint 2) | A07, A08 |
| **M3: Vistas y Observaciones** | 2026-08-31 | Calendarios + Agenda + Observaciones (Sprint 3) | A08 |
| **M4: QA y Staging** | 2026-09-15 | Testing completo + Corrección bugs | A09 |
| **M5: Go-Live MVP** | 2026-09-30 | Despliegue producción + Capacitación | A11 |

### Sprints Detallados

#### Sprint 1 (2026-06-03 → 2026-06-16) — Fundamentos
**Objetivo:** Establecer entidades base y CRUD completo de recursos

| HU | Título | SP | Dev Asignado |
|----|--------|----|--------------|
| HU-001 | Crear ambiente | 3 | Backend |
| HU-002 | Listar ambientes | 2 | Backend + Frontend |
| HU-003 | Editar ambiente | 2 | Backend + Frontend |
| HU-004 | Eliminar ambiente | 2 | Backend + Frontend |
| HU-005 | Registrar instructor | 3 | Backend |
| HU-006 | Listar instructores | 2 | Backend + Frontend |
| HU-007 | Actualizar instructor | 2 | Backend + Frontend |
| HU-009 | Crear ficha | 3 | Backend |
| HU-010 | Listar fichas | 2 | Backend + Frontend |
| HU-011 | Editar ficha | 2 | Backend + Frontend |

**Total:** 10 HU | 23 SP

**Entregables:**
- API REST endpoints: `/ambientes`, `/instructores`, `/fichas` (CRUD completo)
- Componentes React: formularios + listas para cada entidad
- Esquema PostgreSQL: tablas `ambientes`, `instructores`, `fichas`
- Tests unitarios backend + tests de integración

---

#### Sprint 2 (2026-06-17 → 2026-06-30) — Programación Core
**Objetivo:** Implementar horarios con validación automática de conflictos

| HU | Título | SP | Dev Asignado |
|----|--------|----|--------------|
| HU-013 | Crear horario con asignaciones | 5 | Backend + Frontend |
| HU-014 | Listar horarios con filtros | 3 | Backend + Frontend |
| HU-015 | Modificar horario existente | 3 | Backend + Frontend |
| HU-016 | Cancelar horario | 2 | Backend + Frontend |
| HU-017 | Validar conflicto ambiente | 5 | Backend |
| HU-018 | Validar conflicto instructor | 5 | Backend |
| HU-019 | Validar capacidad ambiente | 3 | Backend |
| HU-020 | Mostrar alertas conflicto UI | 3 | Frontend |

**Total:** 8 HU | 29 SP

**Entregables:**
- API REST endpoints: `/horarios` (CRUD + validaciones)
- Lógica de validación de conflictos en backend (Go)
- Componente React: formulario horarios con feedback en tiempo real
- Esquema PostgreSQL: tabla `horarios` con FKs
- Tests de validación de conflictos (casos edge)

---

#### Sprint 3 (2026-07-01 → 2026-07-14) — Observaciones + Vistas
**Objetivo:** Implementar vistas especializadas por usuario y observaciones

| HU | Título | SP | Dev Asignado |
|----|--------|----|--------------|
| HU-021 | Registrar observación | 3 | Backend + Frontend |
| HU-022 | Ver observaciones horario | 2 | Backend + Frontend |
| HU-024 | Calendario semanal ficha | 5 | Frontend |
| HU-025 | Detalles horario en calendario | 3 | Frontend |
| HU-027 | Agenda personal instructor | 5 | Frontend |

**Total:** 5 HU | 18 SP

**Entregables:**
- API REST endpoints: `/observaciones`, `/fichas/:id/calendario`, `/instructores/:id/agenda`
- Componentes React: calendario semanal (librería FullCalendar.js o similar)
- Vista agenda personal instructor
- Esquema PostgreSQL: tabla `observaciones`

---

#### Sprint 4-5 (2026-07-15 → 2026-08-11) — Buffer + Refinamiento
**Objetivo:** Completar features pendientes, refactoring, deuda técnica

**Actividades:**
- Completar HU retrasadas de sprints anteriores
- Refactoring de código con code smells
- Optimización de performance (índices DB, caching)
- Mejoras de UX basadas en feedback interno
- Documentación técnica (API, arquitectura, deployment)

---

#### Sprint 6 (2026-08-12 → 2026-08-25) — QA Integral
**Objetivo:** Testing exhaustivo, corrección de bugs, preparación para staging

**Actividades (A09 - QA Lead):**
- Testing funcional completo (24 HU)
- Testing de integración end-to-end
- Testing de regresión automatizado
- Performance testing (10,000 horarios)
- Testing de usabilidad con usuarios piloto (3 coordinadores)
- Corrección de bugs P0, P1, P2

---

#### Sprint 7 (2026-08-26 → 2026-09-08) — Staging y UAT
**Objetivo:** User Acceptance Testing en ambiente staging

**Actividades:**
- Despliegue a staging (A11 - DevOps)
- UAT con coordinadores reales (2 semanas)
- Capacitación de usuarios (sesiones de 2 horas)
- Corrección de issues identificados en UAT
- Preparación de plan de rollback

---

#### Sprint 8 (2026-09-09 → 2026-09-22) — Pre-Producción
**Objetivo:** Preparación final y despliegue a producción

**Actividades:**
- Migración de datos de Excel a PostgreSQL (si aplica)
- Configuración de ambiente productivo (A11)
- Smoke testing en producción
- Documentación de usuario final
- Plan de soporte post-lanzamiento

---

#### Lanzamiento MVP (2026-09-30)
**Go-Live en producción**

**Actividades:**
- Comunicación oficial a usuarios
- Sesión de onboarding presencial
- Monitoreo intensivo primera semana
- Soporte dedicado 8am-6pm
- Recolección de feedback (encuesta NPS)

---

## 4. Release 1.1 — Mejoras Operativas

### Objetivo

Agregar features de gestión de ciclo de vida y mejoras de UX solicitadas por usuarios en MVP.

### Fecha Objetivo

**2026-11-15** (6 semanas post-MVP)

### Alcance

**Should Have:** 5 HU | 16 SP

| HU | Título | Justificación |
|----|--------|---------------|
| HU-008 | Desactivar instructor | Mantener historial sin eliminar registros |
| HU-012 | Finalizar ficha | Gestión de ciclo de vida de fichas |
| HU-023 | Categorizar observación | Filtrado mejorado (cambio/incidencia/nota) |
| HU-028 | Filtrar agenda por fechas | UX instructor: vista semanal/mensual |
| HU-029 | Disponibilidad ambientes | Dashboard de disponibilidad para coordinador |
| HU-030 | Dashboard alertas | Vista ejecutiva de conflictos pendientes |

### Hitos

| Hito | Fecha | Entregables |
|------|-------|-------------|
| **M6: Sprint 9** | 2026-10-20 | HU-008, HU-012, HU-023, HU-028 completadas |
| **M7: Sprint 10** | 2026-11-03 | HU-029, HU-030 completadas + testing |
| **M8: Release 1.1** | 2026-11-15 | Despliegue a producción |

---

## 5. Release 1.2 — Conveniencia

### Objetivo

Features nice-to-have basadas en feedback de usuarios.

### Fecha Objetivo

**2026-12-31** (6 semanas post-v1.1)

### Alcance

**Could Have:** 1 HU | 3 SP

| HU | Título | Justificación |
|----|--------|---------------|
| HU-026 | Exportar horarios PDF/Excel | Facilitar compartir con aprendices y para impresión |

---

## 6. Release 2.0 — Escalabilidad y Seguridad (Futuro)

### Objetivo

Preparar sistema para múltiples centros SENA con autenticación robusta e integraciones.

### Fecha Objetivo

**2027-Q2** (6 meses post-MVP)

### Alcance Tentativo

**Features principales:**
- Autenticación y autorización (LDAP/AD integración)
- Multi-tenancy (múltiples centros con aislamiento de datos)
- Notificaciones automáticas (email/SMS)
- Integración con Sofia Plus (API externa SENA)
- Reportes avanzados y analytics
- Aplicación móvil nativa (React Native)
- Historial completo de auditoría
- Workflow de aprobación de cambios

**Condiciones para iniciar v2.0:**
- ✅ MVP con ≥90% adopción en centro piloto
- ✅ NPS ≥50 sostenido por 3 meses
- ✅ Aprobación presupuestaria para equipo ampliado
- ✅ Validación de necesidad de multi-tenancy con SENA

---

## 7. Dependencias Externas

| Dependencia | Impacto en Release | Responsable | Estado |
|-------------|-------------------|-------------|--------|
| **Infraestructura Docker** | MVP deployment | TI SENA + A11 | ⚠️ Pendiente validar |
| **Acceso a ambiente staging** | QA Sprint 6 | TI SENA | ⚠️ Pendiente provisionar |
| **Usuarios piloto para UAT** | Validación Sprint 7 | Stakeholder SENA | ⚠️ Pendiente confirmar |
| **Datos de horarios históricos** | Opcional para testing | Coordinador SENA | ⏸️ Nice-to-have |

---

## 8. Riesgos del Release Plan

| ID | Riesgo | Probabilidad | Impacto | Mitigación |
|----|--------|--------------|---------|------------|
| **RR-001** | Velocidad real <12 SP/sprint retrasa MVP | Media | Alto | Buffer de 2 semanas, priorizar despiadadamente |
| **RR-002** | Infraestructura no lista para Go-Live | Media | Crítico | Validar con TI en semana 1, plan B con VM tradicional |
| **RR-003** | UAT descubre caso de uso crítico no cubierto | Media | Alto | Sesiones de validación en Sprint 3-4, early UAT |
| **RR-004** | Usuarios piloto no disponibles para UAT | Baja | Medio | Confirmar usuarios en Sprint 1, backup de otros centros |
| **RR-005** | Bugs P0 en producción post-lanzamiento | Media | Alto | Testing exhaustivo Sprint 6-7, plan de rollback |

---

## 9. Métricas de Éxito por Release

### MVP (1.0)

| KPI | Meta 30 días | Meta 60 días | Meta 90 días |
|-----|--------------|--------------|--------------|
| Tiempo gestión coordinador | 3.5h/sem (-30%) | 2.5h/sem (-50%) | 2h/sem (-60%) |
| Conflictos reactivos | 3/mes (-70%) | 1/mes (-90%) | 0/mes (-100%) |
| Tasa de adopción | 70% | 85% | 95% |
| NPS | ≥0 (neutral) | ≥30 (positivo) | ≥50 (excelente) |

### v1.1

| KPI | Meta |
|-----|------|
| Uso de dashboard alertas | ≥60% coordinadores lo consultan diariamente |
| Observaciones categorizadas | ≥80% tienen categoría asignada |

### v2.0

| KPI | Meta |
|-----|------|
| Centros SENA activos | ≥5 centros usando sistema |
| Integraciones exitosas | Sofia Plus sincronización ≥99% accuracy |

---

## 10. Estrategia de Comunicación

### Stakeholders

| Stakeholder | Frecuencia | Canal | Responsable |
|-------------|-----------|-------|-------------|
| Coordinadores SENA | Semanal | Email + reunión | A05 |
| TI SENA | Bisemanal | Reunión técnica | A11 |
| Dirección SENA | Mensual | Reporte ejecutivo | A05 |
| Equipo desarrollo | Diario | Standup + Slack | A06 |

### Hitos de Comunicación

| Hito | Fecha | Audiencia | Mensaje |
|------|-------|-----------|---------|
| Kickoff oficial | 2026-05-20 | Todos | Inicio proyecto, alcance MVP |
| Demo M1 | 2026-06-30 | Stakeholders | CRUD completo, primera versión funcional |
| Demo M2 | 2026-07-31 | Stakeholders | Validaciones automáticas funcionando |
| UAT inicio | 2026-08-26 | Usuarios piloto | Invitación a testing, capacitación |
| Go-Live MVP | 2026-09-30 | Todos | Lanzamiento oficial, celebración |
| Retrospectiva 30 días | 2026-10-30 | Equipo + Stakeholders | Lecciones aprendidas, ajustes |

---

## 11. Plan de Rollback

**Trigger para rollback:** Bugs P0 que bloquean uso del sistema por >2 horas

**Procedimiento:**
1. Notificar a usuarios (5 minutos)
2. Revertir a última versión estable en producción (15 minutos)
3. Restaurar backup de BD si es necesario (30 minutos)
4. Validar funcionalidad básica (15 minutos)
5. Comunicar resolución a usuarios
6. Post-mortem dentro de 24 horas

**Responsable:** A11 (DevOps) + A06 (Tech Lead)

---

## 12. Criterios de Go/No-Go para Producción

| Criterio | Umbral | Verificador | Estado |
|----------|--------|-------------|--------|
| Todas HU Must Have completadas | 100% | A05 | ⏸️ |
| Bugs P0/P1 cerrados | 0 | A09 | ⏸️ |
| UAT aprobado por usuarios piloto | ≥2 coordinadores | A05 | ⏸️ |
| Performance testing pasado | <500ms validaciones | A09 | ⏸️ |
| Ambiente productivo configurado | 100% | A11 | ⏸️ |
| Backup automático funcionando | 100% | A11 | ⏸️ |
| Plan de capacitación ejecutado | ≥80% asistencia | A05 | ⏸️ |
| Documentación de usuario lista | 100% | A05 | ⏸️ |

**Decisión final Go/No-Go:** A05 (Product Owner) + Stakeholder SENA

---

## Referencias

- **PRD:** [`02-definition/prd.md`](prd.md)
- **Backlog:** [`02-definition/backlog.md`](backlog.md)
- **KPI Baseline:** [`01-discovery/kpi-baseline.md`](../01-discovery/kpi-baseline.md)

---

**Producido:** 2026-05-20  
**Última actualización:** 2026-05-20  
**Próxima revisión:** Al finalizar Sprint 1 (ajustar fechas según velocidad real)
