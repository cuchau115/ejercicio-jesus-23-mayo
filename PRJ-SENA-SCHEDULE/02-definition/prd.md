# Product Requirements Document (PRD) — PRJ-SENA-SCHEDULE

> **Producido por:** A05 (Product Owner)  
> **Fecha:** 2026-05-20  
> **Fase:** 02-definition  
> **Versión:** 1.0  
> **Estado:** DRAFT → APROBADO

---

## Nomenclatura Base

- **Project key:** `PRJ-SENA-SCHEDULE`
- **Nombre del proyecto:** SENA Schedule Manager
- **Dominio:** Gestión académica / Educación
- **Referencia de estándar:** `lifecycle/governance-naming-hu.md`
- **Ruta del artefacto:** `runs/PRJ-SENA-SCHEDULE/02-definition/prd.md`

---

## 1. Visión del Producto

Sistema centralizado de gestión de horarios académicos que **elimina conflictos de asignación mediante validación automática**, reduciendo el tiempo operativo de coordinadores en 60% y garantizando disponibilidad confiable de recursos (ambientes e instructores) para los programas de formación del SENA.

---

## 2. Objetivos Medibles

| ID | Objetivo | KPI Asociado | Meta 90 días |
|----|----------|--------------|--------------|
| **OBJ-001** | Reducir carga operativa de coordinadores académicos | KPI-001: Tiempo de gestión semanal | ≤2 horas/semana (-60% vs baseline 5h) |
| **OBJ-002** | Eliminar conflictos de asignación reactivos | KPI-002: Conflictos detectados post-programación | 0 conflictos/mes (-100% vs baseline 10/mes) |
| **OBJ-003** | Lograr adopción masiva por usuarios objetivo | KPI-003: Tasa de adopción semanal | ≥95% usuarios activos |
| **OBJ-004** | Acelerar proceso de programación semanal | KPI-004: Tiempo creación horario semanal | ≤20 min/ficha (-60% vs baseline 50 min) |
| **OBJ-005** | Alcanzar satisfacción alta de usuarios | KPI-005: NPS | ≥50 (excelente) |

---

## 3. Usuarios y Jobs-to-be-Done

### Usuario Primario: Coordinador Académico

**Contexto:**
- Gestiona 10-30 fichas activas, coordina 15-50 instructores
- Responsable de cumplimiento de programas de formación
- Herramienta actual: Excel compartido + comunicación manual

**Jobs-to-be-Done:**
1. Programar horarios semanales completos sin conflictos de recursos
2. Visualizar disponibilidad en tiempo real de ambientes e instructores
3. Detectar y resolver conflictos de asignación preventivamente
4. Registrar cambios y observaciones con trazabilidad
5. Consultar historial de programación por múltiples dimensiones

**Pain points:**
- ❌ Validación manual de conflictos consume 30-45 min por semana programada
- ❌ Conflictos descubiertos el día de la clase generan reprogramación urgente
- ❌ Versiones desactualizadas de horarios circulan sin control

---

### Usuario Secundario: Instructor (Planta y Contratista)

**Contexto:**
- Imparte formación en 2-5 fichas simultáneamente
- Necesita conocer agenda personal con anticipación
- Diferenciación: instructores planta (tiempo completo) vs contratistas (horas específicas)

**Jobs-to-be-Done:**
1. Consultar mi agenda personal de horarios asignados
2. Ver detalles de fichas a las que imparto formación (programa, aprendices)
3. Reportar incidencias o cambios mediante observaciones

**Pain points:**
- ❌ Información de horarios dispersa (cartelera física, Excel, WhatsApp)
- ❌ Cambios de último momento sin notificación formal
- ❌ Sin vista unificada de disponibilidad personal

---

### Usuario Terciario: Aprendiz

**Contexto:**
- Inscrito en una ficha específica
- Requiere conocer horarios para asistir a formaciones

**Jobs-to-be-Done:**
1. Consultar horarios de mi ficha (calendario semanal/mensual)
2. Ver ambiente asignado para cada sesión
3. Conocer instructor asignado a cada formación

**Pain points:**
- ❌ Horarios impresos en cartelera se desactualizan rápidamente
- ❌ Sin acceso digital a información actualizada

---

## 4. Épicas y Features

### EPC-001: Gestión de Recursos

Administración de recursos físicos (ambientes) y humanos (instructores) disponibles para programación.

#### Features

| Feature ID | Descripción | Prioridad |
|------------|-------------|-----------|
| **FEA-001** | CRUD de Ambientes con atributos (capacidad, tipo, estado) | Must Have |
| **FEA-002** | CRUD de Instructores con diferenciación Planta/Contratista | Must Have |

---

### EPC-002: Gestión de Programación

Creación y administración de programación académica con validación automática de restricciones.

#### Features

| Feature ID | Descripción | Prioridad |
|------------|-------------|-----------|
| **FEA-003** | CRUD de Fichas con programa de formación y cohorte | Must Have |
| **FEA-004** | CRUD de Horarios con asignación de recursos | Must Have |
| **FEA-005** | Validación automática de conflictos de asignación | Must Have |
| **FEA-006** | Gestión de observaciones vinculadas a horarios | Must Have |

---

### EPC-003: Consulta y Visualización

Vistas especializadas por perfil de usuario para consulta de información.

#### Features

| Feature ID | Descripción | Prioridad |
|------------|-------------|-----------|
| **FEA-007** | Vista de horarios por ficha (calendario) | Must Have |
| **FEA-008** | Vista de agenda personal de instructor | Must Have |
| **FEA-009** | Vista de ocupación de ambientes | Should Have |
| **FEA-010** | Dashboard con resumen de conflictos/alertas | Should Have |

---

## 5. Historias de Usuario (MVP)

### EPC-001 / FEA-001: CRUD de Ambientes

#### HU-001: Crear ambiente
**Como** coordinador académico  
**Quiero** registrar un nuevo ambiente en el sistema  
**Para** tenerlo disponible para asignación en horarios

**Prioridad:** Must Have | **Story Points:** 3 | **Sprint:** 1

---

#### HU-002: Listar ambientes
**Como** coordinador académico  
**Quiero** ver todos los ambientes registrados con sus atributos  
**Para** conocer disponibilidad y características de cada uno

**Prioridad:** Must Have | **Story Points:** 2 | **Sprint:** 1

---

#### HU-003: Editar ambiente
**Como** coordinador académico  
**Quiero** modificar información de un ambiente existente  
**Para** mantener datos actualizados (cambio de capacidad, tipo, etc.)

**Prioridad:** Must Have | **Story Points:** 2 | **Sprint:** 1

---

#### HU-004: Eliminar ambiente
**Como** coordinador académico  
**Quiero** eliminar un ambiente que ya no está disponible  
**Para** evitar que sea asignado en nuevos horarios

**Prioridad:** Must Have | **Story Points:** 2 | **Sprint:** 1

---

### EPC-001 / FEA-002: CRUD de Instructores

#### HU-005: Registrar instructor
**Como** coordinador académico  
**Quiero** dar de alta un instructor especificando si es de planta o contratista  
**Para** tenerlo disponible para asignación en horarios

**Prioridad:** Must Have | **Story Points:** 3 | **Sprint:** 1

---

#### HU-006: Listar instructores
**Como** coordinador académico  
**Quiero** ver todos los instructores con filtros por tipo (planta/contratista)  
**Para** identificar disponibilidad según perfil contractual

**Prioridad:** Must Have | **Story Points:** 2 | **Sprint:** 1

---

#### HU-007: Actualizar datos de instructor
**Como** coordinador académico  
**Quiero** modificar información de un instructor (contacto, especialidad, tipo)  
**Para** mantener base de datos actualizada

**Prioridad:** Must Have | **Story Points:** 2 | **Sprint:** 1

---

#### HU-008: Desactivar instructor
**Como** coordinador académico  
**Quiero** marcar un instructor como inactivo sin eliminarlo  
**Para** preservar historial pero evitar nuevas asignaciones

**Prioridad:** Should Have | **Story Points:** 2 | **Sprint:** 2

---

### EPC-002 / FEA-003: CRUD de Fichas

#### HU-009: Crear ficha
**Como** coordinador académico  
**Quiero** registrar una nueva ficha con programa, cohorte y cantidad de aprendices  
**Para** poder asignarle horarios

**Prioridad:** Must Have | **Story Points:** 3 | **Sprint:** 1

---

#### HU-010: Listar fichas
**Como** coordinador académico  
**Quiero** ver todas las fichas activas con sus atributos  
**Para** gestionar programación académica

**Prioridad:** Must Have | **Story Points:** 2 | **Sprint:** 1

---

#### HU-011: Editar ficha
**Como** coordinador académico  
**Quiero** actualizar información de una ficha (aprendices, fechas, programa)  
**Para** reflejar cambios durante el ciclo de formación

**Prioridad:** Must Have | **Story Points:** 2 | **Sprint:** 1

---

#### HU-012: Finalizar ficha
**Como** coordinador académico  
**Quiero** marcar una ficha como finalizada al completar el programa  
**Para** mantener registro histórico sin afectar programación activa

**Prioridad:** Should Have | **Story Points:** 2 | **Sprint:** 2

---

### EPC-002 / FEA-004: CRUD de Horarios

#### HU-013: Crear horario con asignaciones
**Como** coordinador académico  
**Quiero** programar un horario asignando fecha, hora, ambiente, instructor y ficha  
**Para** establecer la programación académica

**Prioridad:** Must Have | **Story Points:** 5 | **Sprint:** 2

---

#### HU-014: Listar horarios con filtros
**Como** coordinador académico  
**Quiero** consultar horarios con filtros por fecha, ficha, instructor o ambiente  
**Para** gestionar programación de manera eficiente

**Prioridad:** Must Have | **Story Points:** 3 | **Sprint:** 2

---

#### HU-015: Modificar horario existente
**Como** coordinador académico  
**Quiero** cambiar datos de un horario ya programado (hora, ambiente, instructor)  
**Para** ajustar programación según necesidades operativas

**Prioridad:** Must Have | **Story Points:** 3 | **Sprint:** 2

---

#### HU-016: Cancelar horario
**Como** coordinador académico  
**Quiero** eliminar un horario programado  
**Para** liberar recursos cuando una sesión no se realizará

**Prioridad:** Must Have | **Story Points:** 2 | **Sprint:** 2

---

### EPC-002 / FEA-005: Validación de Conflictos

#### HU-017: Validar conflicto de ambiente
**Como** coordinador académico  
**Quiero** que el sistema bloquee asignación si el ambiente ya está ocupado en ese horario  
**Para** evitar dobles reservas

**Prioridad:** Must Have | **Story Points:** 5 | **Sprint:** 2

---

#### HU-018: Validar conflicto de instructor
**Como** coordinador académico  
**Quiero** que el sistema impida asignar un instructor a 2 horarios simultáneos  
**Para** evitar conflictos de disponibilidad humana

**Prioridad:** Must Have | **Story Points:** 5 | **Sprint:** 2

---

#### HU-019: Validar capacidad de ambiente
**Como** coordinador académico  
**Quiero** que el sistema alerte si la capacidad del ambiente es menor que aprendices de la ficha  
**Para** garantizar espacio suficiente

**Prioridad:** Must Have | **Story Points:** 3 | **Sprint:** 2

---

#### HU-020: Mostrar alertas de conflicto en UI
**Como** coordinador académico  
**Quiero** ver mensajes claros cuando intento crear un horario con conflicto  
**Para** entender el problema y tomar decisiones alternativas

**Prioridad:** Must Have | **Story Points:** 3 | **Sprint:** 2

---

### EPC-002 / FEA-006: Observaciones

#### HU-021: Registrar observación en horario
**Como** coordinador o instructor  
**Quiero** agregar una observación a un horario específico  
**Para** documentar cambios, incidencias o notas relevantes

**Prioridad:** Must Have | **Story Points:** 3 | **Sprint:** 3

---

#### HU-022: Ver observaciones de un horario
**Como** coordinador o instructor  
**Quiero** consultar todas las observaciones asociadas a un horario  
**Para** revisar historial de cambios y notas

**Prioridad:** Must Have | **Story Points:** 2 | **Sprint:** 3

---

#### HU-023: Categorizar observación por tipo
**Como** coordinador  
**Quiero** clasificar observaciones por tipo (cambio/incidencia/nota)  
**Para** filtrar información relevante rápidamente

**Prioridad:** Should Have | **Story Points:** 2 | **Sprint:** 3

---

### EPC-003 / FEA-007: Vista de Horarios por Ficha

#### HU-024: Ver calendario semanal de ficha
**Como** coordinador o aprendiz  
**Quiero** visualizar los horarios de una ficha en formato calendario semanal  
**Para** conocer la programación completa de manera visual

**Prioridad:** Must Have | **Story Points:** 5 | **Sprint:** 3

---

#### HU-025: Ver detalles de horario en calendario
**Como** coordinador o aprendiz  
**Quiero** hacer clic en un horario y ver detalles (instructor, ambiente, tema, observaciones)  
**Para** obtener información completa de la sesión

**Prioridad:** Must Have | **Story Points:** 3 | **Sprint:** 3

---

#### HU-026: Exportar horarios de ficha
**Como** coordinador  
**Quiero** exportar los horarios de una ficha a PDF o Excel  
**Para** compartir con aprendices o imprimir en cartelera

**Prioridad:** Could Have | **Story Points:** 3 | **Sprint:** 4

---

### EPC-003 / FEA-008: Agenda Personal de Instructor

#### HU-027: Ver mi agenda como instructor
**Como** instructor  
**Quiero** consultar todos mis horarios asignados en un calendario personal  
**Para** planificar mi trabajo semanal

**Prioridad:** Must Have | **Story Points:** 5 | **Sprint:** 3

---

#### HU-028: Filtrar mi agenda por rango de fechas
**Como** instructor  
**Quiero** ver mi agenda de la semana actual, próxima semana o mes completo  
**Para** tener visibilidad de mi disponibilidad

**Prioridad:** Should Have | **Story Points:** 2 | **Sprint:** 3

---

### EPC-003 / FEA-009: Ocupación de Ambientes

#### HU-029: Ver disponibilidad de ambientes por fecha
**Como** coordinador  
**Quiero** consultar qué ambientes están disponibles en un horario específico  
**Para** tomar decisiones de asignación rápidas

**Prioridad:** Should Have | **Story Points:** 5 | **Sprint:** 4

---

### EPC-003 / FEA-010: Dashboard de Alertas

#### HU-030: Ver resumen de conflictos pendientes
**Como** coordinador  
**Quiero** tener un dashboard con alertas de conflictos o horarios sin asignar  
**Para** priorizar acciones operativas

**Prioridad:** Should Have | **Story Points:** 5 | **Sprint:** 4

---

## 6. Criterios de Aceptación (Sampling)

### HU-001: Crear ambiente

| AC ID | Criterio (Given/When/Then) |
|-------|----------------------------|
| **AC-001** | **Given** estoy en el formulario de ambiente **When** ingreso nombre, capacidad, tipo y estado válidos **Then** el ambiente se guarda en BD y aparece en lista de ambientes |
| **AC-002** | **Given** estoy en el formulario de ambiente **When** intento guardar sin completar campo obligatorio **Then** veo mensaje de error indicando campo faltante |
| **AC-003** | **Given** estoy en el formulario de ambiente **When** ingreso capacidad negativa o no numérica **Then** veo mensaje de validación "Capacidad debe ser número positivo" |

---

### HU-013: Crear horario con asignaciones

| AC ID | Criterio (Given/When/Then) |
|-------|----------------------------|
| **AC-013** | **Given** estoy en el formulario de horario **When** selecciono fecha, hora, ambiente, instructor y ficha válidos **Then** el horario se guarda exitosamente |
| **AC-014** | **Given** estoy creando un horario **When** la hora de fin es anterior a hora de inicio **Then** veo error "Hora de fin debe ser posterior a hora de inicio" |
| **AC-015** | **Given** estoy guardando un horario **When** existe validación de conflicto activa **Then** el sistema ejecuta validaciones antes de confirmar guardado |

---

### HU-017: Validar conflicto de ambiente

| AC ID | Criterio (Given/When/Then) |
|-------|----------------------------|
| **AC-017** | **Given** existe un horario en Ambiente A de 08:00-10:00 **When** intento crear otro horario en Ambiente A de 09:00-11:00 **Then** veo error "Ambiente no disponible en ese horario" y no se guarda |
| **AC-018** | **Given** existe un horario en Ambiente A de 08:00-10:00 **When** intento crear horario en Ambiente A de 10:00-12:00 **Then** se guarda exitosamente (sin solapamiento) |
| **AC-019** | **Given** existe un horario en Ambiente A de 08:00-10:00 **When** intento crear horario en Ambiente B de 09:00-11:00 **Then** se guarda exitosamente (ambientes distintos) |

---

### HU-024: Ver calendario semanal de ficha

| AC ID | Criterio (Given/When/Then) |
|-------|----------------------------|
| **AC-024** | **Given** selecciono una ficha con horarios **When** accedo a vista de calendario **Then** veo todos los horarios de esa ficha en layout de calendario semanal |
| **AC-025** | **Given** una ficha tiene horarios en semana actual **When** visualizo el calendario **Then** veo cada horario con hora, ambiente e instructor |
| **AC-026** | **Given** una ficha no tiene horarios en la semana seleccionada **When** visualizo el calendario **Then** veo mensaje "Sin horarios programados para esta semana" |

---

## 7. Requisitos No Funcionales

### Performance

| ID | Requisito | Criterio de éxito |
|----|-----------|-------------------|
| **NFR-001** | Validación de conflictos | < 500ms respuesta en backend |
| **NFR-002** | Carga de calendario semanal | < 1 segundo para ficha con 25 horarios |
| **NFR-003** | Lista de ambientes/instructores | < 300ms para dataset de 100 registros |

### Usabilidad

| ID | Requisito | Criterio de éxito |
|----|-----------|-------------------|
| **NFR-004** | Interface responsiva | Funcional en desktop (1920x1080) y tablet (768px) |
| **NFR-005** | Mensajes de error claros | Usuario entiende acción correctiva sin manual |
| **NFR-006** | Accesibilidad básica | Cumplir WCAG 2.1 Level A mínimo |

### Seguridad

| ID | Requisito | Criterio de éxito |
|----|-----------|-------------------|
| **NFR-007** | Inyección SQL | Backend usa prepared statements en 100% de queries |
| **NFR-008** | Validación de entrada | Sanitización de inputs en frontend y backend |
| **NFR-009** | HTTPS | Comunicación encriptada en producción |

### Escalabilidad

| ID | Requisito | Criterio de éxito |
|----|-----------|-------------------|
| **NFR-010** | Volumen de datos | Soportar 10,000 horarios/año sin degradación |
| **NFR-011** | Usuarios concurrentes | Mínimo 20 usuarios simultáneos sin lag |

### Disponibilidad

| ID | Requisito | Criterio de éxito |
|----|-----------|-------------------|
| **NFR-012** | Uptime | 95% disponibilidad en horario laboral (8am-6pm) |
| **NFR-013** | Backup | Backup diario automático de BD |

---

## 8. Supuestos y Riesgos

### Supuestos Críticos

| ID | Supuesto | Impacto si es falso |
|----|----------|---------------------|
| **SUP-001** | Infraestructura SENA permite despliegue Docker | Sin esto, retraso en deployment |
| **SUP-002** | Coordinadores tienen acceso a navegador web moderno (Chrome/Edge) | Problemas de compatibilidad |
| **SUP-003** | MVP no requiere autenticación con LDAP/AD institucional | Complejidad técnica adicional |
| **SUP-004** | Un solo centro piloto para MVP (no multi-tenant) | Arquitectura más simple |

### Riesgos

| ID | Riesgo | Probabilidad | Impacto | Mitigación |
|----|--------|--------------|---------|------------|
| **RISK-001** | Resistencia al cambio por parte de coordinadores | Media | Alto | Capacitación temprana + demos |
| **RISK-002** | Performance insuficiente con datos reales | Baja | Medio | Testing de carga en Sprint 3 |
| **RISK-003** | Caso de uso crítico no identificado aparece en UAT | Media | Medio | Sesiones de validación con usuarios reales |
| **RISK-004** | Scope creep durante desarrollo | Alta | Medio | Backlog congelado post-aprobación PRD |

---

## 9. Alcance MVP (Release 1.0)

**Incluido en MVP:**
- ✅ CRUD completo de 4 entidades: Ambientes, Instructores, Fichas, Horarios
- ✅ Validación automática de conflictos (ambiente, instructor, capacidad)
- ✅ Observaciones vinculadas a horarios
- ✅ Vista de calendario semanal por ficha
- ✅ Vista de agenda personal de instructor
- ✅ Interface web responsiva (desktop + tablet)
- ✅ Dashboard básico con alertas de conflictos

**Historias Must Have:** HU-001 a HU-025 + HU-027 (27 HU totales)

---

## 10. Explícitamente Fuera de Alcance

**No incluido en MVP (diferido a releases futuras):**

❌ Autenticación y autorización robusta (LDAP/AD)  
❌ Notificaciones automáticas (email/SMS)  
❌ Integración con Sofia Plus u otros sistemas SENA  
❌ Reportes avanzados y analytics  
❌ Aplicación móvil nativa  
❌ Gestión de equipamiento/materiales  
❌ Multi-tenancy (múltiples centros)  
❌ Sincronización con calendarios externos  
❌ Historial completo de auditoría (solo observaciones manuales)  
❌ Sugerencias automáticas de horarios con IA  
❌ Gestión de cambios con workflow de aprobación  
❌ Reservas de ambientes para eventos no académicos  

---

## 11. Criterios de Éxito del Release

El MVP será considerado exitoso si cumple:

1. ✅ **Funcional:** 27 HU Must Have completadas y probadas
2. ✅ **Performance:** NFR-001 a NFR-003 cumplidos (< 500ms validación)
3. ✅ **Adopción:** ≥70% de usuarios activos a 30 días post-lanzamiento
4. ✅ **Valor:** Reducción ≥30% tiempo gestión a 30 días (KPI-001)
5. ✅ **Calidad:** Cero conflictos reactivos a 60 días (KPI-002)
6. ✅ **Satisfacción:** NPS ≥0 a 30 días (neutral o positivo)

---

## 12. Referencias

- **Idea refinada:** [`01-discovery/idea-refined.md`](../01-discovery/idea-refined.md)
- **KPI Baseline:** [`01-discovery/kpi-baseline.md`](../01-discovery/kpi-baseline.md)
- **Project Profile:** [`00-setup/project-profile.md`](../00-setup/project-profile.md)
- **Backlog detallado:** [`02-definition/backlog.md`](backlog.md)
- **Release Plan:** [`02-definition/release-plan.md`](release-plan.md)
- **Gobernanza:** Framework `lifecycle/governance-naming-hu.md`

---

## Aprobaciones

| Rol | Nombre | Fecha | Firma |
|-----|--------|-------|-------|
| Product Owner | A05 | 2026-05-20 | ✅ |
| Tech Lead | A06 | Pendiente | ⏸️ |
| Stakeholder SENA | TBD | Pendiente | ⏸️ |

**Estado:** ✅ **LISTO PARA REVISIÓN**

---

**Producido:** 2026-05-20  
**Última revisión:** 2026-05-20  
**Próxima revisión:** Al iniciar Sprint 1 (fase 04-build)
