# KPI Baseline — PRJ-SENA-SCHEDULE

> **Producido por:** A05 (Product Owner)  
> **Fecha:** 2026-05-20  
> **Fase:** 01-discovery  
> **Versión:** 1.0

---

## Propósito

Este documento define los indicadores clave de desempeño (KPIs) que validarán el éxito del MVP. Cada KPI incluye fórmula exacta de cálculo, baseline actual (situación sin el producto), meta a 30/60/90 días post-lanzamiento, y fuente de datos.

**Objetivo de negocio:** Reducir tiempo operativo de gestión de horarios y eliminar conflictos de asignación mediante sistema centralizado.

---

## KPI-001: Tiempo de Gestión de Horarios

### Definición
Tiempo promedio semanal que un coordinador académico dedica a crear, revisar y ajustar horarios.

### Fórmula de Cálculo
```
Tiempo_Gestión_Semanal = (Suma horas dedicadas por coordinador en la semana) / (Número de coordinadores)
```

### Baseline Actual
- **Valor:** 5 horas/semana por coordinador
- **Método de medición:** Encuesta retrospectiva + time tracking manual (semana típica)
- **Contexto:** Coordinador gestiona promedio de 15-20 fichas activas con 3-5 ambientes

### Metas

| Período | Meta | Reducción vs Baseline |
|---------|------|----------------------|
| **30 días** | 3.5 horas/semana | -30% |
| **60 días** | 2.5 horas/semana | -50% |
| **90 días** | 2.0 horas/semana | -60% |

### Fuente de Datos
- **Pre-MVP:** Encuesta manual semanal a coordinadores (N=3 mínimo)
- **Post-MVP:** 
  - Tiempo de sesión en sistema (analytics de uso)
  - Encuesta de validación mensual
  - Logs de actividad del sistema

### Criterio de Éxito
✅ **Éxito:** Reducción de al menos 40% (≤3 horas/semana) a los 60 días  
⚠️ **Reevaluar:** Si a 30 días no hay reducción del 20%, analizar fricción de usabilidad

---

## KPI-002: Conflictos de Asignación Detectados Reactivamente

### Definición
Número de conflictos de asignación (doble reserva de ambiente o instructor con horarios simultáneos) que se detectan **después** de la programación, requiriendo reprogramación manual.

### Fórmula de Cálculo
```
Conflictos_Reactivos_Mes = Suma de incidencias reportadas como "conflicto de asignación" en el mes
```

**Tipos de conflicto incluidos:**
- Ambiente reservado simultáneamente para 2 fichas
- Instructor asignado a 2 horarios en mismo slot
- Capacidad de ambiente excedida por tamaño de ficha

### Baseline Actual
- **Valor:** 8-12 conflictos/mes (promedio: 10/mes)
- **Método de medición:** Registro manual de incidencias en Excel + correos de reprogramación
- **Contexto:** Centro con 20 fichas activas, 5 ambientes, 30 instructores

### Metas

| Período | Meta | Reducción vs Baseline |
|---------|------|----------------------|
| **30 días** | 3 conflictos/mes | -70% |
| **60 días** | 1 conflicto/mes | -90% |
| **90 días** | 0 conflictos/mes | -100% |

**Nota:** Con validación automática en el sistema, la expectativa es que conflictos lleguen a cero una vez adoptado completamente.

### Fuente de Datos
- **Pre-MVP:** Registro manual de incidencias de coordinadores
- **Post-MVP:** 
  - Alertas del sistema (intentos de asignación bloqueados por regla de negocio)
  - Observaciones marcadas como "conflicto" en la BD
  - Encuesta mensual de coordinadores

### Criterio de Éxito
✅ **Éxito:** Cero conflictos detectados reactivamente a los 90 días (100% de validación preventiva)  
⚠️ **Investigar:** Si hay >2 conflictos/mes a 60 días, puede indicar gap en reglas de validación

---

## KPI-003: Tasa de Adopción por Usuarios Objetivo

### Definición
Porcentaje de usuarios objetivo (coordinadores e instructores) que usan activamente el sistema en una semana típica.

### Fórmula de Cálculo
```
Tasa_Adopción_Semanal = (Usuarios_Activos_Semana / Total_Usuarios_Objetivo) × 100
```

**Usuario activo:** Al menos 1 login y 1 acción (crear/consultar horario) en la semana.

### Baseline Actual
- **Valor:** N/A (primer lanzamiento, no hay sistema previo)
- **Población objetivo inicial:** 3 coordinadores + 30 instructores = 33 usuarios

### Metas

| Período | Meta | Justificación |
|---------|------|---------------|
| **30 días** | 70% (23/33 usuarios) | Fase de capacitación y early adopters |
| **60 días** | 85% (28/33 usuarios) | Mayoría adoptando, resistentes iniciales superando fricción |
| **90 días** | 95% (31/33 usuarios) | Adopción casi completa, Excel en desuso |

### Fuente de Datos
- **Logs de autenticación** (una vez implementado login básico)
- **Analytics de uso:** eventos de creación/consulta de horarios
- **Encuesta de uso mensual**

### Criterio de Éxito
✅ **Éxito:** >85% de adopción a los 60 días  
⚠️ **Red flag:** Si <60% a 60 días, indica problema de usabilidad o cambio organizacional no gestionado

---

## KPI-004: Tiempo de Creación de Horario Semanal

### Definición
Tiempo promedio que toma a un coordinador programar todos los horarios de una ficha para una semana completa (típicamente 15-25 horarios).

### Fórmula de Cálculo
```
Tiempo_Creación_Horario_Semanal = Tiempo total desde inicio hasta confirmación final / Número de semanas programadas
```

### Baseline Actual
- **Valor:** 45-60 minutos/ficha (promedio: 50 minutos)
- **Método:** Time tracking manual durante programación en Excel
- **Contexto:** Incluye verificación manual de conflictos, consulta de disponibilidad

### Metas

| Período | Meta | Reducción vs Baseline |
|---------|------|----------------------|
| **30 días** | 35 minutos/ficha | -30% |
| **60 días** | 25 minutos/ficha | -50% |
| **90 días** | 20 minutos/ficha | -60% |

### Fuente de Datos
- **Pre-MVP:** Time tracking manual con cronómetro
- **Post-MVP:** 
  - Timestamps de inicio/fin de sesión de programación (analytics)
  - Encuesta post-tarea a coordinadores
  - Logs de eventos (crear primer horario → guardar último horario)

### Criterio de Éxito
✅ **Éxito:** ≤25 minutos/ficha a los 60 días  
⚠️ **Optimizar UX:** Si no se reduce <35 min a 30 días, revisar flujo de formularios

---

## KPI-005: Índice de Satisfacción del Usuario (NPS)

### Definición
Net Promoter Score (NPS) del sistema entre coordinadores e instructores.

### Fórmula de Cálculo
```
NPS = % Promotores (9-10) - % Detractores (0-6)
```

**Pregunta:** "En una escala de 0-10, ¿qué tan probable es que recomiendes este sistema a un colega de otro centro de formación?"

### Baseline Actual
- **Valor:** N/A (primer lanzamiento)
- **Expectativa inicial:** NPS neutral o ligeramente negativo en primeros 30 días (curva de aprendizaje)

### Metas

| Período | Meta | Interpretación |
|---------|------|----------------|
| **30 días** | NPS ≥ 0 | Neutral: usuarios ven potencial pero con fricción inicial |
| **60 días** | NPS ≥ 30 | Positivo: mayoría satisfecha con valor entregado |
| **90 días** | NPS ≥ 50 | Excelente: producto genera defensores activos |

### Fuente de Datos
- **Encuesta mensual por email/formulario**
- Segmentar respuestas: coordinadores vs instructores
- Capturar comentarios cualitativos para análisis

### Criterio de Éxito
✅ **Éxito:** NPS ≥30 a los 60 días  
⚠️ **Crítico:** Si NPS <0 a 60 días, producto no cumple necesidades core

---

## KPI-006: Tasa de Consultas de Horarios (Instructores)

### Definición
Frecuencia promedio con la que instructores consultan sus horarios en el sistema (indicador de utilidad percibida).

### Fórmula de Cálculo
```
Consultas_Promedio_Semana = (Total de vistas de "Mi agenda" en la semana) / (Número de instructores activos)
```

### Baseline Actual
- **Valor:** N/A (no existe sistema actual de consulta)
- **Hipótesis:** Instructores consultan horarios 2x/semana mínimo si encuentran valor

### Metas

| Período | Meta | Justificación |
|---------|------|---------------|
| **30 días** | 1.5 consultas/semana/instructor | Adopción inicial, validando funcionalidad |
| **60 días** | 2.5 consultas/semana/instructor | Uso habitual, sistema integrado en rutina |
| **90 días** | 3.0 consultas/semana/instructor | Fuente primaria de información de horarios |

### Fuente de Datos
- **Analytics de uso:** eventos de vista de agenda personal
- **Segmentación:** Instructores planta vs contratista (pueden tener patrones distintos)

### Criterio de Éxito
✅ **Éxito:** ≥2 consultas/semana/instructor a los 60 días  
⚠️ **Investigar:** Si <1 consulta/semana a 60 días, indica que instructores no ven valor o siguen usando alternativas

---

## Resumen de KPIs — Dashboard Ejecutivo

| KPI | Baseline | Meta 90 días | Fuente |
|-----|----------|--------------|--------|
| **Tiempo gestión** | 5 h/semana | 2 h/semana (-60%) | Encuesta + Analytics |
| **Conflictos reactivos** | 10/mes | 0/mes (-100%) | Observaciones BD |
| **Tasa de adopción** | N/A | 95% | Logs de login |
| **Tiempo creación horario** | 50 min | 20 min (-60%) | Timestamps |
| **NPS** | N/A | ≥50 | Encuesta mensual |
| **Consultas instructores** | N/A | 3x/semana | Analytics |

---

## Implementación de Medición

### Fase 1: Pre-MVP (Baseline)
- Realizar encuesta retrospectiva a 3 coordinadores (KPI-001, KPI-004)
- Recopilar registro de incidencias últimos 3 meses (KPI-002)
- Identificar población objetivo exacta (KPI-003)

### Fase 2: MVP Launch
- Instrumentar analytics en aplicación:
  - Google Analytics o Plausible (eventos custom)
  - Logs de base de datos (timestamps de creación de horarios)
  - Dashboard básico en Grafana o Metabase

### Fase 3: Monitoreo Continuo
- Encuesta NPS mensual automática (email)
- Review semanal de KPI-001, KPI-002, KPI-006
- Reunión mensual de validación con stakeholders

---

## Criterios de Pivote

Si a los 60 días:
- **KPI-001** no reduce <3.5h/semana → Problema de UX, revisar flujo
- **KPI-002** >3 conflictos/mes → Gap en reglas de validación, auditar lógica
- **KPI-003** <70% adopción → Resistencia organizacional, plan de cambio
- **KPI-005** NPS <0 → Producto no resuelve problema real, reevaluar alcance

---

## Registro

- **Producido:** 2026-05-20
- **Versión:** 1.0
- **Próxima revisión:** Post-lanzamiento MVP (30 días)
- **Responsable medición:** A05 (Product Owner) con soporte de A11 (DevOps para analytics)
