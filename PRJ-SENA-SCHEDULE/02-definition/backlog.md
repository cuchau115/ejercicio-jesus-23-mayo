# Backlog Priorizado — PRJ-SENA-SCHEDULE

> **Producido por:** A05 (Product Owner)  
> **Fecha:** 2026-05-20  
> **Fase:** 02-definition  
> **Versión:** 1.0

---

## 1. Resumen Ejecutivo

**Total de Historias:** 30 HU  
**Must Have (MVP):** 24 HU | 85 Story Points  
**Should Have (v1.1):** 5 HU | 16 Story Points  
**Could Have (v1.2):** 1 HU | 3 Story Points  
**Won't Have (futuro):** 0 HU

**Estimación Total MVP:** 85 SP ≈ 6-7 sprints (asumiendo velocidad de 12-15 SP/sprint con 2 devs)

---

## 2. Priorización MoSCoW + RICE Scoring

### Metodología RICE

**RICE Score = (Reach × Impact × Confidence) / Effort**

- **Reach:** # usuarios impactados o % de adopción esperado (escala 1-10)
- **Impact:** Valor de negocio (3=Massive, 2=High, 1=Medium, 0.5=Low)
- **Confidence:** Nivel de certeza del valor (100%=1.0, 80%=0.8, 50%=0.5)
- **Effort:** Story Points estimados

---

## 3. Backlog Completo

### Sprint 1: Fundamentos (Recursos)

| HU ID | Título | Epic | Feature | MoSCoW | SP | Reach | Impact | Conf | RICE | Notas |
|-------|--------|------|---------|--------|----|----|--------|------|------|-------|
| **HU-001** | Crear ambiente | EPC-001 | FEA-001 | Must | 3 | 3 | 2 | 1.0 | 2.0 | Base para asignaciones |
| **HU-002** | Listar ambientes | EPC-001 | FEA-001 | Must | 2 | 3 | 2 | 1.0 | 3.0 | Visibilidad de recursos |
| **HU-003** | Editar ambiente | EPC-001 | FEA-001 | Must | 2 | 3 | 1 | 1.0 | 1.5 | Mantenibilidad |
| **HU-004** | Eliminar ambiente | EPC-001 | FEA-001 | Must | 2 | 3 | 1 | 1.0 | 1.5 | Gestión de ciclo de vida |
| **HU-005** | Registrar instructor | EPC-001 | FEA-002 | Must | 3 | 8 | 2 | 1.0 | 5.3 | Alto reach (30 instructores) |
| **HU-006** | Listar instructores | EPC-001 | FEA-002 | Must | 2 | 8 | 2 | 1.0 | 8.0 | Crítico para asignaciones |
| **HU-007** | Actualizar datos instructor | EPC-001 | FEA-002 | Must | 2 | 8 | 1 | 1.0 | 4.0 | Mantenibilidad |
| **HU-009** | Crear ficha | EPC-002 | FEA-003 | Must | 3 | 10 | 3 | 1.0 | 10.0 | **Max RICE** - Core entity |
| **HU-010** | Listar fichas | EPC-002 | FEA-003 | Must | 2 | 10 | 2 | 1.0 | 10.0 | **Max RICE** - Visibilidad |
| **HU-011** | Editar ficha | EPC-002 | FEA-003 | Must | 2 | 10 | 1 | 1.0 | 5.0 | Flexibilidad operativa |

**Total Sprint 1:** 10 HU | 23 SP

---

### Sprint 2: Programación Core (Horarios + Validaciones)

| HU ID | Título | Epic | Feature | MoSCoW | SP | Reach | Impact | Conf | RICE | Notas |
|-------|--------|------|---------|--------|----|----|--------|------|------|-------|
| **HU-013** | Crear horario con asignaciones | EPC-002 | FEA-004 | Must | 5 | 10 | 3 | 1.0 | 6.0 | **Corazón del MVP** |
| **HU-014** | Listar horarios con filtros | EPC-002 | FEA-004 | Must | 3 | 10 | 2 | 1.0 | 6.7 | Consulta operativa |
| **HU-015** | Modificar horario existente | EPC-002 | FEA-004 | Must | 3 | 10 | 2 | 1.0 | 6.7 | Flexibilidad crítica |
| **HU-016** | Cancelar horario | EPC-002 | FEA-004 | Must | 2 | 10 | 1 | 1.0 | 5.0 | Gestión de cambios |
| **HU-017** | Validar conflicto ambiente | EPC-002 | FEA-005 | Must | 5 | 10 | 3 | 1.0 | 6.0 | **KPI-002 crítico** |
| **HU-018** | Validar conflicto instructor | EPC-002 | FEA-005 | Must | 5 | 10 | 3 | 1.0 | 6.0 | **KPI-002 crítico** |
| **HU-019** | Validar capacidad ambiente | EPC-002 | FEA-005 | Must | 3 | 10 | 2 | 1.0 | 6.7 | Prevención de errores |
| **HU-020** | Mostrar alertas conflicto UI | EPC-002 | FEA-005 | Must | 3 | 10 | 2 | 1.0 | 6.7 | UX de validaciones |

**Total Sprint 2:** 8 HU | 29 SP

---

### Sprint 3: Observaciones + Vistas

| HU ID | Título | Epic | Feature | MoSCoW | SP | Reach | Impact | Conf | RICE | Notas |
|-------|--------|------|---------|--------|----|----|--------|------|------|-------|
| **HU-021** | Registrar observación | EPC-002 | FEA-006 | Must | 3 | 10 | 2 | 1.0 | 6.7 | Trazabilidad |
| **HU-022** | Ver observaciones horario | EPC-002 | FEA-006 | Must | 2 | 10 | 1 | 1.0 | 5.0 | Consulta de historial |
| **HU-024** | Calendario semanal ficha | EPC-003 | FEA-007 | Must | 5 | 10 | 3 | 0.8 | 4.8 | **KPI-006** - Vista principal |
| **HU-025** | Detalles horario en calendario | EPC-003 | FEA-007 | Must | 3 | 10 | 2 | 1.0 | 6.7 | Info completa |
| **HU-027** | Agenda personal instructor | EPC-003 | FEA-008 | Must | 5 | 8 | 3 | 0.8 | 3.8 | **KPI-006** - Usuario secundario |
| **HU-008** | Desactivar instructor | EPC-001 | FEA-002 | Should | 2 | 8 | 1 | 1.0 | 4.0 | Historial sin eliminación |
| **HU-012** | Finalizar ficha | EPC-002 | FEA-003 | Should | 2 | 10 | 1 | 1.0 | 5.0 | Gestión de ciclo |
| **HU-023** | Categorizar observación | EPC-002 | FEA-006 | Should | 2 | 10 | 1 | 0.8 | 4.0 | Filtrado mejorado |
| **HU-028** | Filtrar agenda por fechas | EPC-003 | FEA-008 | Should | 2 | 8 | 1 | 1.0 | 4.0 | UX instructor |

**Total Sprint 3:** 9 HU | 26 SP

---

### Sprint 4: Funcionalidades Complementarias

| HU ID | Título | Epic | Feature | MoSCoW | SP | Reach | Impact | Conf | RICE | Notas |
|-------|--------|------|---------|--------|----|----|--------|------|------|-------|
| **HU-029** | Disponibilidad ambientes | EPC-003 | FEA-009 | Should | 5 | 3 | 2 | 0.8 | 1.0 | Ayuda a coordinador |
| **HU-030** | Dashboard alertas | EPC-003 | FEA-010 | Should | 5 | 3 | 2 | 0.8 | 1.0 | Vista ejecutiva |
| **HU-026** | Exportar horarios PDF/Excel | EPC-003 | FEA-007 | Could | 3 | 10 | 0.5 | 0.5 | 0.8 | Nice-to-have |

**Total Sprint 4:** 3 HU | 13 SP

---

## 4. Backlog por Prioridad MoSCoW

### Must Have (MVP - Release 1.0)

**Total: 24 HU | 85 SP**

| Orden | HU ID | Título Corto | SP | RICE | Sprint | Dependencias |
|-------|-------|--------------|----|----|--------|--------------|
| 1 | HU-009 | Crear ficha | 3 | 10.0 | 1 | - |
| 2 | HU-010 | Listar fichas | 2 | 10.0 | 1 | HU-009 |
| 3 | HU-006 | Listar instructores | 2 | 8.0 | 1 | HU-005 |
| 4 | HU-013 | Crear horario | 5 | 6.0 | 2 | HU-009, HU-005, HU-001 |
| 5 | HU-017 | Validar conflicto ambiente | 5 | 6.0 | 2 | HU-013 |
| 6 | HU-018 | Validar conflicto instructor | 5 | 6.0 | 2 | HU-013 |
| 7 | HU-014 | Listar horarios filtros | 3 | 6.7 | 2 | HU-013 |
| 8 | HU-019 | Validar capacidad | 3 | 6.7 | 2 | HU-013 |
| 9 | HU-020 | Alertas conflicto UI | 3 | 6.7 | 2 | HU-017, HU-018 |
| 10 | HU-021 | Registrar observación | 3 | 6.7 | 3 | HU-013 |
| 11 | HU-025 | Detalles horario calendario | 3 | 6.7 | 3 | HU-024 |
| 12 | HU-015 | Modificar horario | 3 | 6.7 | 2 | HU-013 |
| 13 | HU-005 | Registrar instructor | 3 | 5.3 | 1 | - |
| 14 | HU-011 | Editar ficha | 2 | 5.0 | 1 | HU-009 |
| 15 | HU-016 | Cancelar horario | 2 | 5.0 | 2 | HU-013 |
| 16 | HU-022 | Ver observaciones | 2 | 5.0 | 3 | HU-021 |
| 17 | HU-024 | Calendario semanal ficha | 5 | 4.8 | 3 | HU-013 |
| 18 | HU-007 | Actualizar instructor | 2 | 4.0 | 1 | HU-005 |
| 19 | HU-027 | Agenda instructor | 5 | 3.8 | 3 | HU-013 |
| 20 | HU-002 | Listar ambientes | 2 | 3.0 | 1 | HU-001 |
| 21 | HU-001 | Crear ambiente | 3 | 2.0 | 1 | - |
| 22 | HU-003 | Editar ambiente | 2 | 1.5 | 1 | HU-001 |
| 23 | HU-004 | Eliminar ambiente | 2 | 1.5 | 1 | HU-001 |

---

### Should Have (Release 1.1)

**Total: 5 HU | 16 SP**

| Orden | HU ID | Título Corto | SP | RICE | Justificación Diferido |
|-------|-------|--------------|----|----|----------------------|
| 24 | HU-012 | Finalizar ficha | 2 | 5.0 | No bloquea MVP, mejora gestión |
| 25 | HU-008 | Desactivar instructor | 2 | 4.0 | Workaround: eliminar físicamente |
| 26 | HU-023 | Categorizar observación | 2 | 4.0 | Filtrado es nice-to-have |
| 27 | HU-028 | Filtrar agenda fechas | 2 | 4.0 | Vista básica suficiente en MVP |
| 28 | HU-029 | Disponibilidad ambientes | 5 | 1.0 | Coordinador puede revisar lista |
| 29 | HU-030 | Dashboard alertas | 5 | 1.0 | Alertas inline suficientes en MVP |

---

### Could Have (Release 1.2 o posterior)

**Total: 1 HU | 3 SP**

| Orden | HU ID | Título Corto | SP | RICE | Justificación Diferido |
|-------|-------|--------------|----|----|----------------------|
| 30 | HU-026 | Exportar PDF/Excel | 3 | 0.8 | Usuarios pueden hacer screenshot o copiar |

---

### Won't Have (Fuera de roadmap actual)

- Autenticación LDAP/AD (complejidad alta, valor en MVP bajo)
- Notificaciones push/email (infraestructura externa requerida)
- Integración Sofia Plus (dependencia de API externa)
- Aplicación móvil nativa (MVP web suficiente)
- IA para sugerencias de horarios (complejidad sin validación de valor)

---

## 5. Criterios de Aceptación (AC) por Historia

### HU-001: Crear ambiente

| AC ID | Criterio |
|-------|----------|
| AC-001 | **Given** formulario ambiente **When** ingreso datos válidos **Then** se guarda y lista |
| AC-002 | **Given** formulario ambiente **When** falta campo obligatorio **Then** error de validación |
| AC-003 | **Given** formulario ambiente **When** capacidad no numérica **Then** error "Debe ser número positivo" |

---

### HU-002: Listar ambientes

| AC ID | Criterio |
|-------|----------|
| AC-004 | **Given** existen ambientes en BD **When** accedo a lista **Then** veo todos con atributos |
| AC-005 | **Given** no hay ambientes **When** accedo a lista **Then** mensaje "Sin ambientes registrados" |
| AC-006 | **Given** lista de ambientes **When** selecciono uno **Then** veo botones editar/eliminar |

---

### HU-005: Registrar instructor

| AC ID | Criterio |
|-------|----------|
| AC-007 | **Given** formulario instructor **When** selecciono tipo "Planta" o "Contratista" **Then** se guarda con tipo correcto |
| AC-008 | **Given** formulario instructor **When** email inválido **Then** error "Email no válido" |
| AC-009 | **Given** formulario instructor **When** todos los datos válidos **Then** instructor disponible para asignación |

---

### HU-013: Crear horario con asignaciones

| AC ID | Criterio |
|-------|----------|
| AC-013 | **Given** formulario horario **When** selecciono todos los recursos **Then** se guarda exitosamente |
| AC-014 | **Given** formulario horario **When** hora_fin < hora_inicio **Then** error "Hora fin debe ser posterior" |
| AC-015 | **Given** creando horario **When** validaciones activas **Then** se ejecutan antes de guardar |
| AC-016 | **Given** horario guardado **When** consulto lista **Then** aparece con todos los detalles |

---

### HU-017: Validar conflicto de ambiente

| AC ID | Criterio |
|-------|----------|
| AC-017 | **Given** ambiente ocupado 08:00-10:00 **When** intento 09:00-11:00 mismo ambiente **Then** error "Ambiente no disponible" |
| AC-018 | **Given** ambiente ocupado 08:00-10:00 **When** intento 10:00-12:00 mismo ambiente **Then** se guarda (sin solapamiento) |
| AC-019 | **Given** ambiente A ocupado 08:00-10:00 **When** intento ambiente B mismo horario **Then** se guarda |
| AC-020 | **Given** intento crear conflicto **When** backend detecta **Then** status 400 con mensaje descriptivo |

---

### HU-018: Validar conflicto de instructor

| AC ID | Criterio |
|-------|----------|
| AC-021 | **Given** instructor asignado 08:00-10:00 **When** intento 09:00-11:00 mismo instructor **Then** error "Instructor no disponible" |
| AC-022 | **Given** instructor A ocupado **When** intento instructor B mismo horario **Then** se guarda |
| AC-023 | **Given** instructor ocupado 08:00-10:00 **When** intento 10:00-12:00 **Then** se guarda |

---

### HU-019: Validar capacidad de ambiente

| AC ID | Criterio |
|-------|----------|
| AC-024 | **Given** ambiente cap=20, ficha con 25 aprendices **When** intento asignar **Then** warning "Capacidad insuficiente" |
| AC-025 | **Given** ambiente cap=30, ficha con 25 aprendices **When** intento asignar **Then** se guarda sin warning |
| AC-026 | **Given** warning capacidad **When** usuario confirma **Then** permite guardar (es warning, no bloqueo) |

---

### HU-024: Ver calendario semanal de ficha

| AC ID | Criterio |
|-------|----------|
| AC-027 | **Given** ficha con horarios **When** accedo a calendario **Then** veo layout semanal con horarios |
| AC-028 | **Given** calendario semanal **When** cada horario **Then** muestra hora, ambiente, instructor |
| AC-029 | **Given** ficha sin horarios en semana **When** calendario **Then** "Sin horarios programados" |
| AC-030 | **Given** calendario **When** cambio semana (anterior/siguiente) **Then** se actualiza contenido |

---

### HU-027: Ver agenda personal de instructor

| AC ID | Criterio |
|-------|----------|
| AC-031 | **Given** instructor con horarios **When** accedo a mi agenda **Then** veo solo mis asignaciones |
| AC-032 | **Given** mi agenda **When** cada horario **Then** muestra ficha, ambiente, tema |
| AC-033 | **Given** instructor sin horarios **When** mi agenda **Then** "Sin horarios asignados" |

---

## 6. Dependencias Críticas entre HU

### Diagrama de Dependencias

```
Sprint 1 (Fundamentos)
├─ HU-001 (Crear ambiente) → HU-002, HU-003, HU-004
├─ HU-005 (Crear instructor) → HU-006, HU-007, HU-008
└─ HU-009 (Crear ficha) → HU-010, HU-011, HU-012

Sprint 2 (Programación Core)
├─ HU-013 (Crear horario) ← [HU-001, HU-005, HU-009]
│   ├─ HU-014 (Listar horarios)
│   ├─ HU-015 (Modificar horario)
│   ├─ HU-016 (Cancelar horario)
│   ├─ HU-017 (Validar ambiente) → HU-020
│   ├─ HU-018 (Validar instructor) → HU-020
│   └─ HU-019 (Validar capacidad) → HU-020

Sprint 3 (Vistas)
├─ HU-021 (Observación) ← [HU-013] → HU-022, HU-023
├─ HU-024 (Calendario ficha) ← [HU-013] → HU-025
└─ HU-027 (Agenda instructor) ← [HU-013] → HU-028

Sprint 4 (Complementarios)
├─ HU-029 (Disponibilidad) ← [HU-013, HU-024]
└─ HU-030 (Dashboard) ← [HU-017, HU-018]
```

---

## 7. Estimación y Capacidad

### Velocidad Estimada

**Equipo:** 1 Backend Dev (Go) + 1 Frontend Dev (React) = 2 devs

**Velocidad esperada:** 12-15 SP/sprint (sprint de 2 semanas)

### Planificación de Releases

| Release | Sprints | SP Total | HU Count | Objetivo |
|---------|---------|----------|----------|----------|
| **MVP (1.0)** | 1-3 | 78 SP | 24 HU | Must Have + críticos |
| **v1.1** | 4 | 16 SP | 5 HU | Should Have |
| **v1.2** | 5 | 3 SP | 1 HU | Could Have |

**Duración MVP:** 6-7 sprints (12-14 semanas) con velocidad conservadora de 12 SP/sprint

---

## 8. Riesgos del Backlog

| ID | Riesgo | Probabilidad | Mitigación |
|----|--------|--------------|------------|
| BR-001 | HU-017/HU-018 validaciones más complejas de lo estimado | Media | Spike técnico en Sprint 1, prototipar lógica |
| BR-002 | HU-024 calendario UI requiere librería externa con curva aprendizaje | Media | Research en Sprint 1 (FullCalendar.js o similar) |
| BR-003 | Velocidad real <12 SP/sprint | Media | Buffer: planificar 6-7 sprints en lugar de 5 |
| BR-004 | Nueva HU crítica aparece en UAT | Media | Reservar 10% capacidad para hotfixes |

---

## 9. Definición de Done (DoD)

Una HU se considera **Done** cuando:

1. ✅ Código implementado y revisado (code review aprobado)
2. ✅ Todos los AC de la HU pasando (testing manual o automatizado)
3. ✅ Tests unitarios con cobertura ≥70% (backend)
4. ✅ Tests de integración para HU críticas (HU-013, HU-017, HU-018)
5. ✅ UI responsive en desktop y tablet (si aplica)
6. ✅ Sin bugs P0 o P1 abiertos relacionados con la HU
7. ✅ Documentación técnica actualizada (API endpoints, componentes)
8. ✅ Merge a rama `main` y desplegado en ambiente de staging

---

## 10. Métricas de Seguimiento

### Por Sprint

- **Velocidad real** (SP completados vs planificados)
- **Burndown chart** (SP restantes por día)
- **Bugs introducidos** por HU (tracking de calidad)
- **Tiempo de ciclo** por HU (inicio dev → done)

### Por Release

- **Feature completion rate** (% HU Must Have completadas)
- **Scope creep** (HU no planificadas agregadas)
- **Deuda técnica** (HU con shortcuts que requieren refactor)

---

## Referencias

- **PRD:** [`02-definition/prd.md`](prd.md)
- **Release Plan:** [`02-definition/release-plan.md`](release-plan.md)
- **Gobernanza:** Framework `lifecycle/governance-naming-hu.md`

---

**Producido:** 2026-05-20  
**Última actualización:** 2026-05-20  
**Próxima revisión:** Al finalizar Sprint 1
