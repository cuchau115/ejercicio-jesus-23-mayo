# Idea Refinada — PRJ-SENA-SCHEDULE

> **Producido por:** A05 (Product Owner)  
> **Fecha:** 2026-05-20  
> **Fase:** 01-discovery  
> **Versión:** 1.0

---

## 1. El Problema

**Problema actual:**

El SENA (Servicio Nacional de Aprendizaje) enfrenta complejidad operativa significativa en la gestión manual de horarios académicos. La coordinación de múltiples recursos físicos y humanos (ambientes, instructores de planta y contratistas) con grupos de aprendices (fichas) genera:

- **Conflictos de asignación**: Ambientes reservados simultáneamente o instructores con doble asignación
- **Ineficiencia operativa**: Revisión manual de disponibilidad consume 3-5 horas/semana por coordinador
- **Falta de trazabilidad**: Cambios de último momento sin registro formal generan confusión
- **Dificultad de consulta**: Instructores y aprendices no tienen visibilidad centralizada de horarios

**Cuantificación:**
- Coordinadores académicos dedican ~15-20% de su tiempo semanal a gestión de horarios
- Promedio de 2-3 conflictos de asignación por semana que deben resolverse reactivamente
- Información dispersa en hojas de cálculo y correos electrónicos dificulta auditoría

**Consecuencias:**
- Retrasos en inicio de clases por conflictos no detectados a tiempo
- Frustración de instructores y aprendices por falta de información oportuna
- Riesgo de incumplimiento de programas de formación

---

## 2. Usuarios y Jobs-to-be-Done

### Usuario Primario: Coordinador Académico

**Perfil:**
- Rol: Responsable de programación académica en un centro de formación SENA
- Contexto: Gestiona 10-30 fichas activas simultáneamente, coordina 15-50 instructores
- Competencias: Conocimiento de programas de formación, manejo de Excel, necesidades operativas

**Jobs-to-be-Done:**
1. **Programar horarios semanales** sin conflictos de recursos (ambiente/instructor)
2. **Visualizar disponibilidad** de ambientes e instructores en tiempo real
3. **Resolver conflictos** de asignación con alternativas sugeridas
4. **Registrar cambios y observaciones** con trazabilidad
5. **Consultar historial** de horarios por ficha, instructor o ambiente

### Usuario Secundario: Instructor

**Perfil:**
- Rol: Instructor de planta o contratista
- Contexto: Imparte formación en múltiples fichas, necesita claridad en su agenda
- Competencias: Especialista técnico, usuario de aplicaciones web básicas

**Jobs-to-be-Done:**
1. **Consultar mis horarios asignados** (vista personal)
2. **Ver detalles de fichas** a las que imparto formación
3. **Notificar cambios** o incidencias mediante observaciones

### Usuario Terciario: Aprendiz

**Perfil:**
- Rol: Estudiante inscrito en una ficha
- Contexto: Necesita conocer horarios de su programa de formación

**Jobs-to-be-Done:**
1. **Consultar horarios de mi ficha** (vista pública o restringida)
2. **Ver ambiente asignado** para cada sesión

---

## 3. Propuesta de Valor

### ¿Cómo se resuelve el problema?

Sistema centralizado de gestión de horarios que:

1. **Previene conflictos automáticamente**: Validación en tiempo real de disponibilidad antes de confirmar asignaciones
2. **Centraliza información**: Base de datos única como fuente de verdad, reemplaza hojas de cálculo dispersas
3. **Mejora visibilidad**: Vistas personalizadas por rol (coordinador, instructor, aprendiz)
4. **Garantiza trazabilidad**: Registro de cambios y observaciones con timestamp y usuario
5. **Reduce tiempo de gestión**: Interface intuitiva que reduce de 5 horas/semana a <2 horas

### ¿Por qué ahora?

- SENA está digitalizando procesos operativos (iniciativa institucional)
- Crecimiento de programas de formación aumenta complejidad de coordinación
- Modalidades híbridas (presencial + remoto) requieren mayor control de recursos físicos
- Herramientas actuales (Excel compartido) no escalan con demanda actual

### Diferenciador clave

**Soluciones actuales:**
- Hojas de cálculo compartidas: sin validación automática de conflictos
- Calendarios genéricos (Google Calendar): no diseñados para lógica académica específica (fichas, ambientes, contratistas vs planta)
- Sistemas académicos complejos: curva de aprendizaje alta, sobrecargados de funcionalidad no necesaria

**Nuestro diferenciador:**
- Diseñado específicamente para flujo SENA (fichas, ambientes, instructores por tipo contractual)
- MVP enfocado: solo lo esencial, sin sobrecarga funcional
- Validación automática de reglas de negocio (conflictos de asignación)
- Despliegue rápido con Docker (infraestructura mínima requerida)

---

## 4. Definición del MVP

### Alcance del Primer Release (Capacidades Core)

**MVP incluye exactamente:**

1. **Gestión de Ambientes**
   - CRUD completo (crear, leer, actualizar, eliminar)
   - Atributos: nombre, capacidad, tipo (aula, taller, laboratorio), estado (disponible/mantenimiento)

2. **Gestión de Instructores**
   - CRUD completo con diferenciación: Planta vs Contratista
   - Atributos: nombre, identificación, tipo_contrato, especialidad, email, teléfono

3. **Gestión de Fichas**
   - CRUD completo
   - Atributos: código_ficha, programa_formación, cohorte, cantidad_aprendices, fecha_inicio, fecha_fin

4. **Gestión de Horarios**
   - CRUD completo con asignaciones
   - Atributos: fecha, hora_inicio, hora_fin, ambiente_id, instructor_id, ficha_id, tema
   - **Validación automática:**
     - Un ambiente no puede tener 2 horarios simultáneos
     - Un instructor no puede estar en 2 lugares simultáneamente
     - Capacidad del ambiente >= cantidad de aprendices de la ficha

5. **Gestión de Observaciones**
   - CRUD completo vinculado a horarios
   - Atributos: horario_id, autor, fecha_registro, texto_observacion, tipo (cambio/incidencia/nota)

6. **Consultas Principales**
   - Vista de horarios por ficha (calendario semanal/mensual)
   - Vista de horarios por instructor (agenda personal)
   - Vista de ocupación de ambientes (disponibilidad)

7. **Interfaz Web Responsiva**
   - Formularios de gestión para coordinador
   - Vistas de consulta para instructores y aprendices
   - Dashboard con resumen de conflictos/alertas

### Explícitamente FUERA del MVP

❌ **No incluido en esta versión:**

- Autenticación y autorización (todos los usuarios tienen acceso completo en MVP)
- Notificaciones por email/SMS sobre cambios de horario
- Integración con sistemas externos del SENA (Sofia Plus, etc.)
- Reportes avanzados y analítica
- Aplicación móvil nativa
- Gestión de recursos adicionales (equipamiento, materiales)
- Reservas de ambientes para otros propósitos (reuniones, eventos)
- Multi-tenancy (MVP es para un solo centro de formación)
- Sincronización con calendarios externos (Google Calendar, Outlook)
- Historial completo de cambios (solo observaciones manuales)
- Sugerencias automáticas de horarios óptimos (IA/ML)

**Justificación del recorte:**
- Autenticación: complejidad de implementación vs valor en ambiente controlado inicial
- Integraciones: dependencias externas que pueden retrasar MVP
- Reportes avanzados: priorizar flujo operativo básico primero

---

## 5. Supuestos Críticos que el MVP Debe Validar

### Supuestos de Valor

| # | Supuesto | Cómo se valida |
|---|----------|----------------|
| S1 | Coordinadores reducirán tiempo de gestión de horarios de 5h/semana a <2h/semana | Medir tiempo promedio de creación de horario semanal antes/después |
| S2 | Reducción de conflictos de asignación de 2-3/semana a <1/mes | Conteo de conflictos detectados reactivamente |
| S3 | Instructores consultarán sus horarios al menos 2x/semana | Métrica de uso: vistas de agenda personal |
| S4 | Sistema será adoptado en 2 semanas de capacitación mínima | Encuesta de usabilidad post-capacitación |

### Supuestos Técnicos

| # | Supuesto | Riesgo si es falso |
|---|----------|-------------------|
| T1 | PostgreSQL escala para 10,000+ horarios/año | Degradación de performance |
| T2 | Interface React web es suficiente (no se requiere app móvil para MVP) | Baja adopción por instructores |
| T3 | Validación de conflictos en backend es suficientemente rápida (<500ms) | Experiencia de usuario frustrada |
| T4 | Despliegue Docker en infraestructura SENA es posible sin burocracia excesiva | Retraso en puesta en producción |

### Supuestos de Usabilidad

| # | Supuesto | Validación |
|---|----------|------------|
| U1 | Coordinadores pueden crear un horario semanal completo en <30 minutos | Pruebas de usuario con prototipo |
| U2 | Interfaz no requiere manual de usuario extenso | Test de usabilidad con usuarios reales |
| U3 | Móvil no es requerido para consulta de horarios (basta web responsive) | Encuesta a instructores |

---

## 6. Alternativas Actuales (Sin Este Producto)

### ¿Qué hace hoy el usuario?

**Solución actual del Coordinador:**

1. **Excel compartido en red local o OneDrive**
   - Hojas separadas por semana/mes
   - Validación manual de conflictos (búsqueda visual en tablas)
   - Compartir por email o carpetas compartidas

2. **Comunicación por WhatsApp/Email**
   - Notificación de cambios de último momento
   - Confirmación manual de disponibilidad con instructores

3. **Pizarra física o impresiones**
   - Horarios semanales impresos en cartelera
   - Actualizaciones con borrador/marcador

**Dolores con la alternativa actual:**
- ❌ Conflictos no detectados hasta que se presentan físicamente
- ❌ Versiones desactualizadas circulando simultáneamente
- ❌ Sin historial de cambios confiable
- ❌ Tiempo perdido en validación manual
- ❌ Dependencia de persona específica que "conoce" el Excel

**Por qué la alternativa persiste:**
- Bajo costo (herramientas ya disponibles)
- Familiaridad del equipo con Excel
- Sin presupuesto previo aprobado para solución especializada
- Percepción de que "funciona, aunque sea engorroso"

---

## 7. Riesgos de Producto

### Top 5 Riesgos

| # | Riesgo | Probabilidad | Impacto | Mitigación |
|---|--------|--------------|---------|------------|
| R1 | Resistencia al cambio: coordinadores prefieren Excel "conocido" | Media | Alto | Capacitación + demostración de ahorro de tiempo tangible |
| R2 | Infraestructura SENA rechaza Docker o requiere aprobaciones extensas | Baja | Crítico | Validar requisitos de infra en Discovery, plan B con VM tradicional |
| R3 | MVP no cubre caso de uso crítico no identificado | Media | Medio | Sesiones de validación con coordinadores antes de PRD |
| R4 | Performance insuficiente con volumen real de datos | Baja | Medio | Testing de carga con dataset realista (1 año de horarios) |
| R5 | Alcance crece durante desarrollo por "solo falta esto" | Alta | Medio | Backlog estricto con MoSCoW, scope freeze post-PRD aprobado |

---

## 8. Open Questions

### Preguntas Abiertas (Bloqueantes)

| # | Pregunta | Impacto en MVP | Responsable resolver |
|---|----------|----------------|---------------------|
| Q1 | ¿Qué nivel de autenticación mínimo es aceptable para producción? | Bloqueante si se requiere integración con AD/LDAP institucional | Stakeholder SENA + A04 (Security) |
| Q2 | ¿Existe infraestructura Docker disponible o hay que aprovisionar? | Bloqueante para deployment | A11 (DevOps) + TI SENA |
| Q3 | ¿Cuántos centros de formación pilotarán el MVP simultáneamente? | Afecta estrategia de datos (multi-tenant o single-tenant) | Stakeholder SENA |

### Preguntas Diferibles (Post-MVP)

| # | Pregunta | Impacto |
|---|----------|---------|
| Q4 | ¿Se requiere integración con sistema Sofia Plus a corto plazo? | Diferible, no bloquea MVP funcional |
| Q5 | ¿Hay presupuesto para infraestructura cloud o debe ser on-premise? | Afecta estrategia de release, no development |
| Q6 | ¿Aprendices requieren autenticación individual o consulta pública? | Diferible, se puede agregar post-MVP |

---

## 9. Próximos Pasos (Post-Discovery)

### Entregables Siguientes

1. **Fase 02-definition (A05):**
   - PRD completo con épicas, features y HU
   - Backlog priorizado MoSCoW/RICE
   - Release plan (MVP + v1.1)

2. **Fase 03-design (A06, A10, A03):**
   - Pattern guide (arquitectura hexagonal confirmada)
   - Data model (esquema PostgreSQL con constraints)
   - UX flows y UI spec (coordinador, instructor, aprendiz)

3. **Validación requerida antes de PRD:**
   - Sesión con coordinador real (validar jobs-to-be-done)
   - Respuesta a Q1, Q2, Q3 (bloqueantes)
   - Confirmación de stakeholder SENA sobre alcance MVP

---

## Validación y Aprobación

**Criterios de aprobación de Discovery:**
- ✅ Problema cuantificado y usuario identificado
- ✅ Propuesta de valor clara vs alternativas actuales
- ✅ Alcance MVP definido con fuera-de-scope explícito
- ✅ KPIs con baseline definidos (ver kpi-baseline.md)
- ✅ Riesgos top 5 identificados con mitigación
- ⚠️ Open questions bloqueantes pendientes de respuesta

**Estado:** ✅ **DISCOVERY COMPLETO** (pendiente resolver Q1-Q3 antes de iniciar Development)

---

**Registro:**
- Producido: 2026-05-20
- Última revisión: 2026-05-20
- Próxima revisión: Al aprobar PRD (fase 02-definition)
