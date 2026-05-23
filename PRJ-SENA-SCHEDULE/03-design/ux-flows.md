# UX Flows — PRJ-SENA-SCHEDULE

> **Producido por:** A03 (UX Designer)  
> **Fecha:** 2026-05-20  
> **Fase:** 03-design (paso 3c)  
> **Versión:** 1.0  
> **Capacidad:** cap-ux-design

---

## 1. Perfiles de Usuario y Jobs-to-be-Done

### Perfil 1: Coordinador Académico (Usuario Primario)

**Contexto:** Gestiona horarios de múltiples fichas, ambientes e instructores. Necesita evitar conflictos de asignación y optimizar uso de recursos.

**Jobs-to-be-Done:**
1. **Crear horario sin conflictos** — Asignar clase validando disponibilidad de ambiente e instructor en tiempo real.
2. **Visualizar dashboard de alertas** — Ver conflictos detectados y capacidad excedida en un vistazo.
3. **Consultar calendario semanal** — Ver programación completa de una ficha en semana específica.
4. **Modificar horario existente** — Cambiar ambiente/instructor/hora revalidando conflictos.
5. **Cancelar horario** — Marcar clase como cancelada con observación justificativa.
6. **Gestionar recursos** — CRUD de ambientes, instructores, fichas.

---

### Perfil 2: Instructor (Usuario Secundario)

**Contexto:** Consulta su agenda personal para conocer asignaciones. No modifica horarios.

**Jobs-to-be-Done:**
1. **Consultar agenda personal** — Ver mis horarios asignados en rango de fechas (semana, mes).
2. **Ver detalle de horario** — Conocer ambiente, ficha, tema, observaciones de una clase específica.

---

### Perfil 3: Aprendiz (Usuario Terciario)

**Contexto:** Consulta el calendario de su ficha para conocer clases programadas. Solo lectura.

**Jobs-to-be-Done:**
1. **Consultar calendario de ficha** — Ver horarios de mi ficha en semana/mes.
2. **Ver detalle de horario** — Conocer instructor, ambiente, tema de una clase.

---

## 2. Flujos de Usuario

### F-001: Crear Horario (Coordinador) — CRÍTICO MVP

**Trigger:** Coordinador necesita programar una nueva clase.

**Precondiciones:**
- Al menos 1 ambiente activo
- Al menos 1 instructor activo
- Al menos 1 ficha activa
- Usuario autenticado como coordinador

**Happy Path:**

1. **Dashboard** → Usuario hace clic en "Crear Horario" (botón primario).
2. **Formulario Crear Horario** (pantalla modal o página):
   - Campos visibles:
     - Ficha (dropdown con autocomplete)
     - Ambiente (dropdown con autocomplete)
     - Instructor (dropdown con autocomplete)
     - Fecha (date picker)
     - Hora inicio (time picker)
     - Hora fin (time picker)
     - Tema (textarea opcional)
   - Usuario completa todos los campos obligatorios.
   - Usuario hace clic en "Validar" (botón secundario).
3. **Validación en tiempo real:**
   - Sistema ejecuta validación de conflictos (HU-017, HU-018).
   - **Caso 3a — Sin conflictos:**
     - Mensaje verde: "✓ Horario válido. No se detectaron conflictos."
     - Si capacidad ambiente < aprendices ficha: Warning amarillo "⚠️ Capacidad del ambiente (X) es menor que aprendices de la ficha (Y)".
     - Botón "Guardar" habilitado.
   - **Caso 3b — Con conflictos:**
     - Alerta roja: "✕ Conflicto detectado: El ambiente LAB-101 ya está asignado el 2026-06-15 de 08:00 a 10:00."
     - O: "✕ Conflicto detectado: El instructor Juan Pérez ya está asignado el 2026-06-15 de 08:00 a 10:00."
     - Botón "Guardar" deshabilitado.
4. **Usuario corrige** (si conflicto):
   - Cambia ambiente, instructor o rango de horas.
   - Hace clic en "Validar" nuevamente.
   - Repite hasta resolver conflicto.
5. **Usuario guarda:**
   - Hace clic en "Guardar".
   - Loading spinner (2-3 seg).
   - Mensaje toast verde: "✓ Horario creado exitosamente."
   - Redirect a Dashboard o Calendario.

**Flujo Alternativo 1 — Error de validación de formulario:**
- Usuario intenta guardar con campos vacíos.
- Mensajes de error inline bajo cada campo: "Este campo es obligatorio."
- Focus automático en primer campo con error.

**Flujo Alternativo 2 — Error de servidor:**
- Guardado falla (500, timeout).
- Modal de error: "✕ Error al guardar horario. Intente nuevamente o contacte soporte."
- Botones: "Reintentar" (reintenta guardado) | "Cancelar" (cierra modal, preserva datos en formulario).

**Flujo de Cancelación:**
- Usuario hace clic en "Cancelar" (botón terciario).
- Si hay datos ingresados: Confirmación "¿Descartar cambios?" → Sí | No.
- Si No: Permanece en formulario.
- Si Sí: Cierra formulario, vuelve a pantalla anterior.

---

### F-002: Ver Dashboard de Alertas (Coordinador)

**Trigger:** Coordinador accede al sistema (login).

**Precondiciones:** Usuario autenticado como coordinador.

**Happy Path:**

1. **Login exitoso** → Redirect a Dashboard.
2. **Dashboard:**
   - **Sección Alertas de Conflictos:**
     - Si hay conflictos detectados (horarios solapados):
       - Card por cada conflicto:
         - Título: "Conflicto de ambiente" o "Conflicto de instructor"
         - Detalle: "LAB-101 | 2026-06-15 | 08:00-10:00 | Ficha 2491234 vs Ficha 2491235"
         - Acción: Botón "Resolver" → abre modal con ambos horarios para editar.
     - Si NO hay conflictos:
       - Empty state: "✓ No hay conflictos detectados. Todo en orden."
   - **Sección Próximos Horarios:**
     - Lista de próximos 5 horarios (fecha más cercana a hoy).
     - Cada item: Ficha | Ambiente | Instructor | Fecha | Hora.
   - **Acciones rápidas:**
     - Botón primario: "Crear Horario"
     - Botón secundario: "Ver Calendario"

**Flujo Alternativo — Sin horarios creados:**
- Sección Próximos Horarios vacía:
  - Empty state: "No hay horarios programados. Crea el primero."
  - Botón: "Crear Horario" (CTA).

---

### F-003: Consultar Calendario Semanal (Coordinador / Aprendiz)

**Trigger:** Usuario quiere ver horarios de una ficha en semana específica.

**Precondiciones:** Al menos 1 ficha con horarios creados.

**Happy Path:**

1. **Dashboard o Menú** → Usuario hace clic en "Calendario".
2. **Pantalla Calendario:**
   - **Filtros:**
     - Dropdown "Ficha" (obligatorio si múltiples fichas).
     - Date picker "Semana" (muestra lunes-domingo).
   - Usuario selecciona ficha y semana.
3. **Vista Calendario Semanal:**
   - **Grid 7 columnas (Lu-Do) × N filas (franjas horarias 06:00-22:00).**
   - Cada horario renderizado como **card en celda correspondiente:**
     - Fondo con color según estado:
       - Verde claro: PROGRAMADO
       - Azul claro: REALIZADO
       - Gris: CANCELADO
     - Contenido del card:
       - Hora inicio - fin (08:00 - 10:00)
       - Ambiente (LAB-101)
       - Instructor (J. Pérez)
       - Tema (si existe, truncado)
   - **Interacción:**
     - Hover: Card se eleva (shadow).
     - Click: Abre modal con detalle completo + observaciones.
4. **Navegación temporal:**
   - Botones "< Semana anterior" | "Semana siguiente >".
   - Botón "Hoy" (vuelve a semana actual).

**Flujo Alternativo — Semana sin horarios:**
- Grid vacío.
- Empty state centrado: "No hay horarios programados para esta semana."
- Botón (solo coordinador): "Crear Horario".

---

### F-004: Consultar Agenda Instructor (Instructor)

**Trigger:** Instructor quiere ver sus horarios asignados.

**Preconditions:** Usuario autenticado como instructor.

**Happy Path:**

1. **Login** → Redirect a Agenda Personal.
2. **Pantalla Agenda:**
   - **Filtro:** Date picker "Rango de fechas" (default: semana actual).
   - Usuario puede extender rango (mes, trimestre).
3. **Lista de Horarios:**
   - Ordenado cronológicamente (fecha+hora).
   - Cada item (card):
     - Fecha | Hora inicio - fin
     - Ficha (código + nombre)
     - Ambiente (código + nombre)
     - Tema
     - Estado (badge: programado/realizado/cancelado)
   - **Interacción:**
     - Click: Abre modal detalle con observaciones.
4. **Filtro adicional (opcional v1.1):**
   - Checkbox "Mostrar cancelados" (default: ocultos).

**Flujo Alternativo — Sin horarios asignados:**
- Empty state: "No tienes horarios asignados en este rango."
- Ilustración + mensaje amigable.

---

### F-005: Modificar Horario (Coordinador)

**Trigger:** Coordinador necesita cambiar datos de horario existente.

**Preconditions:**
- Horario existe con estado PROGRAMADO.
- Usuario autenticado como coordinador.

**Happy Path:**

1. **Calendario o Lista Horarios** → Usuario hace clic en horario a modificar.
2. **Modal Detalle Horario:**
   - Botón "Editar" (si usuario es coordinador).
3. **Usuario hace clic "Editar":**
   - Modal se transforma en formulario editable (mismos campos que Crear).
   - Valores actuales pre-poblados.
4. **Usuario modifica campos:**
   - Cambia ambiente, instructor, fecha, hora, tema.
   - Hace clic en "Validar".
5. **Validación de conflictos** (igual que F-001 paso 3).
6. **Usuario guarda:**
   - Clic en "Guardar cambios".
   - Modal confirmación: "¿Guardar cambios en este horario?" → Confirmar | Cancelar.
   - Si Confirmar:
     - Loading spinner.
     - Toast verde: "✓ Horario actualizado exitosamente."
     - Modal se cierra, calendario/lista se actualiza.

**Flujo Alternativo — Horario con estado REALIZADO o CANCELADO:**
- Botón "Editar" deshabilitado.
- Tooltip: "No se puede editar un horario realizado/cancelado."

---

### F-006: Cancelar Horario (Coordinador)

**Trigger:** Clase no se dictará (instructor enfermo, falta de aprendices, etc.).

**Preconditions:**
- Horario existe con estado PROGRAMADO.
- Usuario autenticado como coordinador.

**Happy Path:**

1. **Modal Detalle Horario** → Usuario hace clic en "Cancelar" (botón destructivo rojo).
2. **Modal Confirmación:**
   - Título: "¿Cancelar horario?"
   - Detalle: Muestra resumen del horario.
   - Textarea obligatorio: "Motivo de cancelación" (min 10 caracteres).
   - Botones: "Confirmar Cancelación" (rojo) | "Volver" (gris).
3. **Usuario ingresa motivo y confirma:**
   - Sistema actualiza estado a CANCELADO.
   - Registra observación con tipo INCIDENCIA + motivo.
   - Toast amarillo: "⚠️ Horario cancelado."
   - Modal se cierra, vista se actualiza (horario con fondo gris, badge CANCELADO).

**Regla de negocio:** Horarios cancelados NO participan en validación de conflictos (no "ocupan" ambiente/instructor).

---

### F-007: Agregar Observación (Coordinador / Instructor)

**Trigger:** Usuario quiere agregar nota sobre un horario.

**Preconditions:**
- Horario existe.
- Usuario autenticado.

**Happy Path:**

1. **Modal Detalle Horario** → Sección "Observaciones" al final.
2. **Usuario hace clic "Agregar Observación":**
   - Textarea aparece inline.
   - Dropdown "Tipo" (CAMBIO | INCIDENCIA | NOTA) — default NOTA.
3. **Usuario escribe observación y selecciona tipo:**
   - Clic en "Guardar".
   - Loading inline.
   - Observación aparece en lista (más reciente arriba).
   - Metadata: Autor | Fecha/hora | Tipo (badge).

**Regla de negocio:** Observaciones son inmutables — NO hay botón "Editar" o "Eliminar".

---

### F-008: CRUD Ambiente (Coordinador)

**Trigger:** Gestionar ambientes disponibles.

**Happy Path:**

1. **Menú** → "Recursos" → "Ambientes".
2. **Lista Ambientes:**
   - Tabla con columnas: Código | Nombre | Tipo | Capacidad | Estado | Acciones.
   - Filtros: Estado (ACTIVO/INACTIVO) | Tipo (dropdown).
   - Búsqueda: Por código o nombre (autocomplete).
   - Botón primario: "Crear Ambiente".
3. **Crear Ambiente:**
   - Modal con formulario:
     - Código (text, obligatorio, único)
     - Nombre (text, obligatorio, único)
     - Tipo (dropdown: AULA/LABORATORIO/TALLER/AUDITORIO)
     - Capacidad (number, >0)
     - Descripción (textarea, opcional)
   - Validación inline.
   - Botón "Guardar" → Toast verde → Lista se actualiza.
4. **Editar Ambiente:**
   - Clic en icono "Editar" en fila.
   - Modal con formulario pre-poblado.
   - Cambios guardan con confirmación.
5. **Desactivar Ambiente:**
   - Clic en icono "Desactivar".
   - Confirmación: "Ambientes con horarios activos no pueden desactivarse."
   - Si sin horarios: Estado cambia a INACTIVO.

---

### F-009: CRUD Instructor (Coordinador) — Simétrico a F-008

Flujo idéntico a F-008 con campos específicos de instructor.

---

### F-010: CRUD Ficha (Coordinador) — Simétrico a F-008

Flujo idéntico a F-008 con campos específicos de ficha.

---

## 3. Pantallas Identificadas

| ID | Nombre Pantalla | Perfiles | Flujos |
|----|----------------|----------|--------|
| P-001 | Dashboard | Coordinador | F-002 |
| P-002 | Formulario Crear/Editar Horario | Coordinador | F-001, F-005 |
| P-003 | Calendario Semanal | Coordinador, Aprendiz | F-003 |
| P-004 | Agenda Instructor | Instructor | F-004 |
| P-005 | Detalle Horario (Modal) | Todos | F-001, F-005, F-006, F-007 |
| P-006 | Lista Ambientes | Coordinador | F-008 |
| P-007 | Formulario Crear/Editar Ambiente | Coordinador | F-008 |
| P-008 | Lista Instructores | Coordinador | F-009 |
| P-009 | Formulario Crear/Editar Instructor | Coordinador | F-009 |
| P-010 | Lista Fichas | Coordinador | F-010 |
| P-011 | Formulario Crear/Editar Ficha | Coordinador | F-010 |
| P-012 | Login | Todos | — |

**Total:** 12 pantallas principales para MVP.

---

## 4. Navegación y Arquitectura de Información

### Menú Principal (Coordinador)

```
┌─────────────────────────────────────┐
│ 🏠 Dashboard                        │
│ 📅 Calendario                       │
│ 🗂️  Recursos ▾                      │
│    ├─ Ambientes                     │
│    ├─ Instructores                  │
│    └─ Fichas                        │
│ 👤 Perfil                           │
│ 🚪 Cerrar Sesión                    │
└─────────────────────────────────────┘
```

### Menú Principal (Instructor)

```
┌─────────────────────────────────────┐
│ 📅 Mi Agenda                        │
│ 👤 Perfil                           │
│ 🚪 Cerrar Sesión                    │
└─────────────────────────────────────┘
```

### Menú Principal (Aprendiz)

```
┌─────────────────────────────────────┐
│ 📅 Calendario de mi Ficha           │
│ 👤 Perfil                           │
│ 🚪 Cerrar Sesión                    │
└─────────────────────────────────────┘
```

---

## 5. Estados Críticos por Pantalla

| Pantalla | Loading | Empty | Error | Success |
|----------|---------|-------|-------|---------|
| Dashboard | Skeleton de cards | "No hay alertas" + empty illustration | Error toast + retry | — |
| Calendario | Skeleton de grid | "No hay horarios" + CTA crear | Error modal + contactar soporte | — |
| Formulario Horario | Spinner en botón Guardar | — | Mensajes inline + toast | Toast verde + redirect |
| Lista Ambientes | Skeleton de tabla | "No hay ambientes" + CTA crear | Error banner + retry | — |
| Agenda Instructor | Skeleton de lista | "Sin horarios asignados" + ilustración | Error modal + retry | — |

---

## 6. Responsive Behavior (Web)

### Breakpoints

- **sm:** 640px (móvil)
- **md:** 768px (tablet)
- **lg:** 1024px (desktop)
- **xl:** 1280px (desktop grande)

### Comportamiento por Pantalla

**Dashboard:**
- sm/md: Cards en 1 columna
- lg+: Cards en 2 columnas
- xl: Cards en 3 columnas

**Calendario Semanal:**
- sm: Vista de lista (no grid) — día por día, expandible.
- md: Grid 3 días visibles, scroll horizontal.
- lg+: Grid completo 7 días.

**Formulario Horario:**
- sm/md: Modal full-screen.
- lg+: Modal centrado (max-width 600px).

**Tablas (Ambientes/Instructores/Fichas):**
- sm: Cards en lugar de tabla.
- md+: Tabla completa con scroll horizontal si necesario.

---

## 7. Accesibilidad (WCAG 2.1 AA)

### Contraste

- Texto normal (16px): Mínimo 4.5:1
- Texto grande (≥18px bold o ≥24px): Mínimo 3:1
- Elementos UI (botones, bordes): Mínimo 3:1

### Navegación por Teclado

- Tab order lógico (top-to-bottom, left-to-right).
- Focus visible: Outline azul 2px (nunca `outline: none` sin alternativa).
- Modales: Focus trap (Tab/Shift+Tab cicla dentro del modal).
- Escape: Cierra modales.

### ARIA Labels

- Botones con iconos: `aria-label` obligatorio.
- Formularios: `<label>` asociado a cada `<input>`.
- Alertas: `role="alert"` con `aria-live="polite"`.

### Targets Táctiles (Móvil)

- Mínimo 44x44px para botones/enlaces.
- Espaciado entre targets: ≥8px.

---

## 8. Feedback y Micro-interacciones

| Acción | Feedback Inmediato (<100ms) |
|--------|----------------------------|
| Click en botón | Ripple effect o color más oscuro |
| Hover en card | Elevación (shadow) |
| Envío de formulario | Botón muestra spinner, texto cambia a "Guardando..." |
| Validación de conflicto | Mensaje aparece con animación fade-in |
| Agregar observación | Observación aparece con slide-down |
| Cancelar horario | Card cambia color a gris instantáneamente, luego toast |

---

## 9. Validaciones y Mensajes

### Mensajes de Validación de Formulario

| Campo Vacío | Mensaje |
|-------------|---------|
| Ficha | "Seleccione una ficha" |
| Ambiente | "Seleccione un ambiente" |
| Instructor | "Seleccione un instructor" |
| Fecha | "Ingrese una fecha válida" |
| Hora inicio | "Ingrese hora de inicio" |
| Hora fin | "Ingrese hora de fin" |
| Hora fin ≤ inicio | "Hora de fin debe ser posterior a hora de inicio" |

### Mensajes de Conflicto

- **Ambiente ocupado:** "✕ Conflicto: El ambiente {codigo} ya está asignado el {fecha} de {hora_inicio} a {hora_fin}."
- **Instructor ocupado:** "✕ Conflicto: El instructor {nombre} ya está asignado el {fecha} de {hora_inicio} a {hora_fin}."
- **Warning capacidad:** "⚠️ Advertencia: Capacidad del ambiente ({capacidad}) es menor que aprendices de la ficha ({num_aprendices})."

### Mensajes de Confirmación

- **Eliminar horario:** "¿Está seguro de cancelar este horario? Esta acción no se puede deshacer."
- **Descartar cambios:** "¿Descartar cambios sin guardar?"
- **Desactivar recurso:** "¿Desactivar este {recurso}? No podrá asignarse a horarios nuevos."

---

## Aprobaciones

| Rol | Nombre | Fecha | Firma |
|-----|--------|-------|-------|
| UX Designer | A03 | 2026-05-20 | ✅ |
| Product Owner | A05 | Revisión | ⏸️ Debe validar que flujos cubren HUs del PRD |
| Frontend Dev | A08 | Input | ⏸️ Implementará interfaces siguiendo esta spec |

**Estado:** ✅ **APROBADO — INPUT PARA ui-spec.md Y A08**

---

**Producido:** 2026-05-20  
**Última revisión:** 2026-05-20  
**Próxima revisión:** Post-Sprint 3 (después de UAT con usuarios reales)
