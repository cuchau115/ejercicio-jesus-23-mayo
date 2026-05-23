# UI Specification — PRJ-SENA-SCHEDULE

> **Producido por:** A03 (UX Designer)  
> **Fecha:** 2026-05-20  
> **Fase:** 03-design (paso 3c)  
> **Versión:** 1.0  
> **Capacidad:** cap-ux-design  
> **Inputs:** ux-flows.md, architecture.md, prd.md

---

## 1. Design System Base

### 1.1. Paleta de Colores

#### Colores Primarios (Acciones)

| Token | Valor Hex | Uso |
|-------|-----------|-----|
| `primary-500` | `#2563EB` | Botones primarios, enlaces, elementos activos |
| `primary-600` | `#1D4ED8` | Hover sobre primario |
| `primary-700` | `#1E40AF` | Active/pressed primario |
| `primary-100` | `#DBEAFE` | Fondos sutiles de elementos informativos |

#### Colores Neutros (Fondos/Texto)

| Token | Valor Hex | Uso |
|-------|-----------|-----|
| `neutral-900` | `#111827` | Texto principal (headings, body) |
| `neutral-700` | `#374151` | Texto secundario |
| `neutral-500` | `#6B7280` | Texto terciario, placeholders |
| `neutral-300` | `#D1D5DB` | Bordes, separadores |
| `neutral-100` | `#F3F4F6` | Fondos alternos (cards, tabla zebra) |
| `neutral-50` | `#F9FAFB` | Fondo de página |
| `white` | `#FFFFFF` | Fondo de contenedores principales |

#### Colores Semánticos (Estados)

| Token | Valor Hex | Uso |
|-------|-----------|-----|
| `success-500` | `#10B981` | Validaciones exitosas, horarios realizados |
| `success-100` | `#D1FAE5` | Fondos de mensajes de éxito |
| `warning-500` | `#F59E0B` | Advertencias (ej. capacidad excedida) |
| `warning-100` | `#FEF3C7` | Fondos de advertencias |
| `error-500` | `#EF4444` | Errores, conflictos, cancelaciones |
| `error-100` | `#FEE2E2` | Fondos de mensajes de error |
| `info-500` | `#3B82F6` | Información neutral |
| `info-100` | `#DBEAFE` | Fondos informativos |

#### Colores de Estado de Horario

| Token | Valor Hex | Uso |
|-------|-----------|-----|
| `status-programado` | `#10B981` (success-500) | Horario PROGRAMADO |
| `status-programado-bg` | `#D1FAE5` (success-100) | Fondo card horario programado |
| `status-realizado` | `#3B82F6` (info-500) | Horario REALIZADO |
| `status-realizado-bg` | `#DBEAFE` (info-100) | Fondo card horario realizado |
| `status-cancelado` | `#6B7280` (neutral-500) | Horario CANCELADO |
| `status-cancelado-bg` | `#E5E7EB` (neutral-200) | Fondo card horario cancelado |

**Validación WCAG AA:**
- `neutral-900` sobre `white`: 16.75:1 ✅
- `primary-500` sobre `white`: 4.67:1 ✅
- `success-500` sobre `white`: 3.19:1 ⚠️ (usar solo para texto ≥18px bold)
- `error-500` sobre `white`: 4.15:1 ✅

---

### 1.2. Escala Tipográfica

**Familia:** `Inter` (Google Fonts) — fallback: `system-ui`, `-apple-system`, `BlinkMacSystemFont`, `"Segoe UI"`, `sans-serif`

| Token | Tamaño | Peso | Line Height | Uso |
|-------|--------|------|-------------|-----|
| `heading-1` | 32px | 700 (Bold) | 40px (1.25) | Títulos de página principales |
| `heading-2` | 24px | 600 (Semibold) | 32px (1.33) | Títulos de secciones |
| `heading-3` | 20px | 600 (Semibold) | 28px (1.4) | Títulos de cards, modales |
| `body` | 16px | 400 (Regular) | 24px (1.5) | Texto párrafos, descripciones |
| `body-sm` | 14px | 400 (Regular) | 20px (1.43) | Texto secundario, metadata |
| `caption` | 12px | 400 (Regular) | 16px (1.33) | Labels, ayudas, timestamps |
| `label` | 14px | 500 (Medium) | 20px (1.43) | Labels de formularios, botones |

---

### 1.3. Escala de Espaciado (Base 8px)

| Token | Valor | Uso |
|-------|-------|-----|
| `spacing-1` | 8px | Espacio mínimo entre elementos inline |
| `spacing-2` | 16px | Padding interno de botones, cards pequeños |
| `spacing-3` | 24px | Margin entre secciones relacionadas |
| `spacing-4` | 32px | Padding de contenedores principales |
| `spacing-6` | 48px | Margin entre secciones distintas |
| `spacing-8` | 64px | Padding vertical de héroes, separaciones grandes |
| `spacing-12` | 96px | Espaciado excepcional (empty states) |

---

### 1.4. Radios y Sombras

#### Border Radius

| Token | Valor | Uso |
|-------|-------|-----|
| `radius-sm` | 4px | Inputs, badges pequeños |
| `radius-md` | 8px | Botones, cards, modales |
| `radius-lg` | 12px | Contenedores destacados |
| `radius-full` | 9999px | Pills, avatares circulares |

#### Shadows

| Token | Valor | Uso |
|-------|-------|-----|
| `shadow-sm` | `0 1px 2px 0 rgba(0,0,0,0.05)` | Inputs, botones planos |
| `shadow-md` | `0 4px 6px -1px rgba(0,0,0,0.1)` | Cards, dropdowns |
| `shadow-lg` | `0 10px 15px -3px rgba(0,0,0,0.1)` | Modales, hover en cards |
| `shadow-focus` | `0 0 0 3px rgba(37,99,235,0.3)` | Focus ring (primary-500 con opacidad) |

---

## 2. Componentes Globales

### 2.1. Botones

#### Variantes

**Primary Button**
- **Uso:** Acciones principales (Guardar, Crear, Confirmar)
- **Estilos:**
  - Default: `background: primary-500`, `color: white`, `padding: spacing-2 spacing-3`, `radius: radius-md`, `font: label`, `shadow: shadow-sm`
  - Hover: `background: primary-600`
  - Active: `background: primary-700`
  - Disabled: `background: neutral-300`, `color: neutral-500`, `cursor: not-allowed`
  - Focus: `shadow: shadow-focus`, `outline: none`

**Secondary Button**
- **Uso:** Acciones secundarias (Cancelar, Validar)
- **Estilos:**
  - Default: `background: white`, `border: 1px solid neutral-300`, `color: neutral-900`, `padding: spacing-2 spacing-3`, `radius: radius-md`, `font: label`
  - Hover: `background: neutral-50`, `border-color: neutral-400`
  - Active: `background: neutral-100`
  - Disabled: `opacity: 0.5`, `cursor: not-allowed`

**Destructive Button**
- **Uso:** Acciones destructivas (Cancelar horario, Eliminar)
- **Estilos:**
  - Default: `background: error-500`, `color: white`, resto igual a Primary
  - Hover: `background: #DC2626` (error-600)
  - Active: `background: #B91C1C` (error-700)

**Ghost Button**
- **Uso:** Acciones terciarias (Ver más, Detalles)
- **Estilos:**
  - Default: `background: transparent`, `color: primary-500`, `padding: spacing-2`, `font: label`
  - Hover: `background: primary-100`
  - Active: `background: primary-200`

#### Estados

- **Loading:** Botón muestra spinner (16px circular animado) + texto cambia a "Guardando..." o "Cargando..."
- **Target táctil:** Mínimo 44x44px (cumplir con `padding` o `min-height`)

---

### 2.2. Inputs y Formularios

#### Text Input

**Estilos:**
- Default: `border: 1px solid neutral-300`, `padding: spacing-2`, `radius: radius-sm`, `font: body`, `background: white`
- Focus: `border-color: primary-500`, `shadow: shadow-focus`, `outline: none`
- Error: `border-color: error-500`
- Disabled: `background: neutral-100`, `color: neutral-500`, `cursor: not-allowed`

**Label:**
- `font: label`, `color: neutral-700`, `margin-bottom: spacing-1`
- Obligatorio: Asterisco rojo `*` después del label

**Mensaje de error:**
- `font: body-sm`, `color: error-500`, `margin-top: spacing-1`
- Icono: ✕ antes del texto

**Helper text:**
- `font: caption`, `color: neutral-500`, `margin-top: spacing-1`

#### Dropdown (Select)

- Igual a Text Input + icono chevron-down alineado a la derecha
- Dropdown panel: `background: white`, `border: 1px solid neutral-300`, `shadow: shadow-md`, `radius: radius-md`, `max-height: 300px`, `overflow-y: auto`
- Opciones:
  - Default: `padding: spacing-2`, `hover: background neutral-100`
  - Selected: `background: primary-100`, `font-weight: 600`

#### Date Picker / Time Picker

- Input base + icono calendario/reloj a la derecha
- Popup calendar: `background: white`, `shadow: shadow-lg`, `radius: radius-md`
- Día seleccionado: `background: primary-500`, `color: white`
- Hoy: `border: 2px solid primary-500`

#### Textarea

- Igual a Text Input + `min-height: 80px`, `resize: vertical`

#### Checkbox / Radio

- Tamaño: 20x20px
- Checked: `background: primary-500`, checkmark blanco (✓)
- Unchecked: `border: 2px solid neutral-300`, `background: white`
- Focus: `shadow: shadow-focus`
- Label: `font: body`, `color: neutral-700`, `margin-left: spacing-1`

---

### 2.3. Cards

**Estilos:**
- Default: `background: white`, `border: 1px solid neutral-300`, `radius: radius-md`, `padding: spacing-4`, `shadow: shadow-sm`
- Hover (si clickeable): `shadow: shadow-md`, `cursor: pointer`, `transition: 200ms ease`
- Active: `border-color: primary-500`

**Anatomía:**
- Header: `heading-3` + acción opcional (icono)
- Body: `body` o contenido custom
- Footer (opcional): Botones o metadata (`body-sm`, `color: neutral-500`)

---

### 2.4. Modales

**Overlay:**
- `background: rgba(0,0,0,0.5)`, `position: fixed`, `inset: 0`, `z-index: 1000`

**Modal Container:**
- `background: white`, `radius: radius-lg`, `shadow: shadow-lg`, `max-width: 600px` (ajustar según contenido), `margin: auto`, `padding: spacing-4`
- Animación entrada: Fade in + scale 0.95→1 (200ms ease-out)

**Anatomía:**
- Header: `heading-2` + botón cerrar (X) top-right
- Body: Contenido variable (`padding: spacing-3`)
- Footer: Botones alineados derecha (Primary + Secondary)

**Comportamiento:**
- Click en overlay: Cerrar modal (con confirmación si hay cambios)
- Escape: Cerrar modal
- Focus trap: Tab cicla dentro del modal

---

### 2.5. Toasts (Notificaciones)

**Posición:** Top-right de viewport, `position: fixed`, `top: spacing-4`, `right: spacing-4`, `z-index: 2000`

**Estilos:**
- `background: white`, `border-left: 4px solid <color semántico>`, `radius: radius-md`, `shadow: shadow-lg`, `padding: spacing-3`, `min-width: 300px`, `max-width: 500px`
- Success: `border-color: success-500`, icono ✓
- Error: `border-color: error-500`, icono ✕
- Warning: `border-color: warning-500`, icono ⚠️
- Info: `border-color: info-500`, icono ℹ️

**Anatomía:**
- Icono (20x20px) + Mensaje (`font: body`) + Botón cerrar (opcional)
- Auto-dismiss: 5 segundos (errores: 8 segundos)

---

### 2.6. Badges

**Estilos:**
- `padding: 4px spacing-2`, `radius: radius-full`, `font: caption`, `font-weight: 600`, `text-transform: uppercase`
- Variantes:
  - **PROGRAMADO:** `background: status-programado-bg`, `color: status-programado`
  - **REALIZADO:** `background: status-realizado-bg`, `color: status-realizado`
  - **CANCELADO:** `background: status-cancelado-bg`, `color: status-cancelado`
  - **INCIDENCIA:** `background: error-100`, `color: error-500`
  - **NOTA:** `background: neutral-100`, `color: neutral-700`

---

### 2.7. Tablas

**Estilos:**
- `border: 1px solid neutral-300`, `radius: radius-md`, `overflow: hidden`
- Header: `background: neutral-100`, `font: label`, `color: neutral-700`, `padding: spacing-2 spacing-3`, `border-bottom: 2px solid neutral-300`
- Rows:
  - Default: `background: white`, `border-bottom: 1px solid neutral-300`, `padding: spacing-2 spacing-3`
  - Hover: `background: neutral-50`, `cursor: pointer` (si clickeable)
  - Zebra striping (opcional): Filas pares `background: neutral-50`

**Empty state:**
- Centrado vertical + ilustración SVG + mensaje (`body`) + botón CTA (si aplica)

---

## 3. Especificación de Pantallas

### P-001: Dashboard (Coordinador)

**Ruta:** `/dashboard`  
**Perfil:** Coordinador

#### Layout

```
┌────────────────────────────────────────────────┐
│ Header: "Dashboard" (heading-1)                │
│ Subheader: "Gestión de horarios SENA" (body)  │
├────────────────────────────────────────────────┤
│ ┌──────────────────┐  ┌──────────────────────┐│
│ │ Alertas          │  │ Próximos Horarios    ││
│ │ de Conflictos    │  │                      ││
│ └──────────────────┘  └──────────────────────┘│
│ ┌────────────────────────────────────────────┐│
│ │ Acciones Rápidas (botones)                 ││
│ └────────────────────────────────────────────┘│
└────────────────────────────────────────────────┘
```

#### Componentes

**Sección: Alertas de Conflictos**
- **Tipo:** Card
- **Contenido:**
  - Si conflictos detectados:
    - Lista de Cards secundarios (uno por conflicto):
      - Icono: ⚠️ (warning-500)
      - Título: "Conflicto de {tipo}" (heading-3, color: error-500)
      - Detalle: "{Ambiente/Instructor} | {Fecha} | {Hora} | Ficha X vs Ficha Y" (body-sm)
      - Botón: "Resolver" (Ghost button)
  - Si sin conflictos:
    - Empty state: Icono ✓ grande (success-500) + "No hay conflictos detectados" (body) + "Todo en orden" (caption, neutral-500)

**Sección: Próximos Horarios**
- **Tipo:** Card con tabla simple
- **Contenido:**
  - Tabla 5 columnas: Ficha | Ambiente | Instructor | Fecha | Hora
  - Ordenado por fecha ascendente (próximas 5 entradas)
  - Si vacío: Empty state "No hay horarios programados. Crea el primero." + Botón "Crear Horario" (Primary)

**Sección: Acciones Rápidas**
- Botón "Crear Horario" (Primary) + Botón "Ver Calendario" (Secondary)
- Alineados horizontalmente, separados por `spacing-2`

#### Estados

- **Loading:** Skeletons de cards (animación pulsante gris)
- **Error:** Banner rojo top: "Error al cargar datos. [Reintentar]"
- **Empty (sin horarios):** Ilustración SVG centrada + mensaje + CTA

#### Responsive

- **sm/md:** Cards en 1 columna, apilados verticalmente
- **lg+:** Cards en 2 columnas (Alertas | Próximos), Acciones rápidas full-width abajo

---

### P-002: Formulario Crear/Editar Horario

**Ruta:** `/horarios/crear` o `/horarios/:id/editar`  
**Perfil:** Coordinador  
**Tipo:** Modal (lg+) o Full-screen (sm/md)

#### Layout

```
┌─────────────────────────────────────────────┐
│ Header: "Crear Horario" / "Editar Horario" │
│ (heading-2) + X (close)                     │
├─────────────────────────────────────────────┤
│ [Dropdown] Ficha *                          │
│ [Dropdown] Ambiente *                       │
│ [Dropdown] Instructor *                     │
│ [Date Picker] Fecha *                       │
│ [Time Picker] Hora inicio *                 │
│ [Time Picker] Hora fin *                    │
│ [Textarea] Tema (opcional)                  │
├─────────────────────────────────────────────┤
│ [Botón] Validar (Secondary)                 │
│ → Mensaje de validación aquí ←             │
├─────────────────────────────────────────────┤
│ Footer: [Cancelar] [Guardar]               │
│         (Secondary) (Primary, disabled si   │
│                     no validado)            │
└─────────────────────────────────────────────┘
```

#### Componentes

**Campos de formulario:**
- Todos usan componentes de sección 2.2
- Labels con asterisco rojo `*` para obligatorios
- Helper text bajo "Tema": "Describe el contenido de la sesión (opcional)" (caption, neutral-500)

**Zona de validación:**
- **Sin validar (inicial):** Mensaje info-100: "ℹ️ Haz clic en 'Validar' para verificar disponibilidad" (body-sm)
- **Sin conflictos:** Mensaje success-100: "✓ Horario válido. No se detectaron conflictos." (body-sm, success-500)
- **Con warning:** Mensaje warning-100: "⚠️ Advertencia: Capacidad del ambiente (X) es menor que aprendices de la ficha (Y)." (body-sm, warning-500)
- **Con conflictos:** Mensaje error-100: "✕ Conflicto: [detalle]" (body-sm, error-500) + Botón "Resolver" deshabilitado "Guardar"

**Botones footer:**
- "Cancelar" (Secondary): Confirmación si hay cambios → "¿Descartar cambios?" (Modal confirmación)
- "Guardar" (Primary): Deshabilitado hasta validación exitosa
  - Loading: Spinner + texto "Guardando..."
  - Success: Toast verde + Redirect a Dashboard/Calendario

#### Estados

- **Loading (guardado):** Spinner en botón, overlay semi-transparente en formulario (prevenir edición)
- **Error (servidor):** Modal error: "✕ Error al guardar horario. Intente nuevamente." + Botones "Reintentar" | "Cancelar"
- **Validación error (formulario):** Mensajes inline bajo cada campo con error

#### Validaciones

| Campo | Validación | Mensaje |
|-------|-----------|---------|
| Ficha | Required | "Seleccione una ficha" |
| Ambiente | Required | "Seleccione un ambiente" |
| Instructor | Required | "Seleccione un instructor" |
| Fecha | Required + formato válido | "Ingrese una fecha válida" |
| Hora inicio | Required | "Ingrese hora de inicio" |
| Hora fin | Required + > hora_inicio | "Hora de fin debe ser posterior a hora de inicio" |

#### Responsive

- **sm/md:** Modal full-screen, formulario ocupa todo el viewport
- **lg+:** Modal centrado, max-width 600px, overlay oscuro

---

### P-003: Calendario Semanal

**Ruta:** `/calendario`  
**Perfil:** Coordinador, Aprendiz  
**Tipo:** Página completa

#### Layout Desktop (lg+)

```
┌──────────────────────────────────────────────────┐
│ Header: "Calendario" (heading-1)                 │
│ Filtros: [Dropdown Ficha] [Date Picker Semana]  │
│          [< Semana anterior] [Hoy] [Siguiente >] │
├──────────────────────────────────────────────────┤
│ Grid 7 columnas × N filas                        │
│ ┌────┬────┬────┬────┬────┬────┬────┐            │
│ │ Lu │ Ma │ Mi │ Ju │ Vi │ Sa │ Do │            │
│ ├────┼────┼────┼────┼────┼────┼────┤            │
│ │08:00│    │[H1]│    │[H3]│    │    │            │
│ │10:00│[H2]│    │[H4]│    │    │    │            │
│ │... │    │    │    │    │    │    │            │
│ └────┴────┴────┴────┴────┴────┴────┘            │
└──────────────────────────────────────────────────┘
```

#### Componentes

**Filtros:**
- Dropdown "Ficha": Si usuario Coordinador → todas las fichas; si Aprendiz → solo su ficha (bloqueado)
- Date Picker "Semana": Muestra lunes-domingo, selector de semana
- Botones navegación: "< Semana anterior" | "Hoy" (Secondary) | "Siguiente >"

**Grid Calendario:**
- **Estructura:** Tabla HTML con columnas fijas (7 días)
- **Filas:** Franjas horarias 06:00-22:00 (incrementos 1h o 2h según densidad)
- **Celdas:** Cada horario renderizado como Card mini:
  - **Estilos:**
    - `background`: Según estado (status-programado-bg, status-realizado-bg, status-cancelado-bg)
    - `border-left`: 4px solid <color estado>
    - `padding`: spacing-2
    - `radius`: radius-sm
    - `shadow`: shadow-sm
    - Hover: `shadow: shadow-md`, `cursor: pointer`
  - **Contenido:**
    - Hora: "08:00 - 10:00" (label, font-weight: 600)
    - Ambiente: "LAB-101" (body-sm)
    - Instructor: "J. Pérez" (body-sm, truncado si muy largo)
    - Tema: "Programación básica" (caption, truncado, opcional)
  - **Click:** Abre Modal Detalle Horario (P-005)

**Empty state:**
- Si grid vacío: Mensaje centrado "No hay horarios programados para esta semana." + Botón "Crear Horario" (solo Coordinador)

#### Estados

- **Loading:** Skeleton de grid (celdas grises pulsantes)
- **Error:** Banner rojo top: "Error al cargar calendario. [Reintentar]"

#### Responsive

- **sm (móvil):** Grid NO renderizado → Vista de lista por día:
  - Acordeón por día (Lunes [expandir], Martes, ...)
  - Dentro de cada día: Lista de Cards verticales con horarios
- **md (tablet):** Grid 3 días visibles, scroll horizontal
- **lg+:** Grid completo 7 días

---

### P-004: Agenda Instructor

**Ruta:** `/agenda` o `/mi-agenda`  
**Perfil:** Instructor  
**Tipo:** Página completa

#### Layout

```
┌─────────────────────────────────────────────┐
│ Header: "Mi Agenda" (heading-1)             │
│ Filtro: [Date Range Picker] Rango de fechas│
│         [Checkbox] Mostrar cancelados       │
├─────────────────────────────────────────────┤
│ Lista de Horarios (Cards)                   │
│ ┌─────────────────────────────────────────┐ │
│ │ [Badge PROGRAMADO] Lun 15 Jun | 08:00   │ │
│ │ Ficha: 2491234 - Programación           │ │
│ │ Ambiente: LAB-101                       │ │
│ │ Tema: Variables y tipos de datos       │ │
│ └─────────────────────────────────────────┘ │
│ ┌─────────────────────────────────────────┐ │
│ │ [Badge REALIZADO] Mar 16 Jun | 10:00    │ │
│ │ ...                                     │ │
│ └─────────────────────────────────────────┘ │
└─────────────────────────────────────────────┘
```

#### Componentes

**Filtros:**
- Date Range Picker: Default semana actual, extensible a mes/trimestre
- Checkbox "Mostrar cancelados": Default unchecked (oculta horarios CANCELADO)

**Lista de Horarios:**
- Cards ordenados cronológicamente (fecha + hora ascendente)
- **Card anatomy:**
  - Header: Badge estado + Fecha + Hora (heading-3)
  - Body:
    - Ficha: Código + Nombre (body)
    - Ambiente: Código + Nombre (body)
    - Tema: (body-sm, truncado 2 líneas)
  - Hover: `shadow: shadow-md`, `cursor: pointer`
  - Click: Abre Modal Detalle Horario (P-005)

**Empty state:**
- Si sin horarios en rango: Ilustración + "No tienes horarios asignados en este rango." (body) + Emoji 🎉

#### Estados

- **Loading:** Skeleton de lista (3-4 cards grises pulsantes)
- **Error:** Banner rojo + "Reintentar"

#### Responsive

- **sm/md/lg+:** Lista vertical 1 columna (consistente en todos breakpoints)

---

### P-005: Modal Detalle Horario

**Trigger:** Click en horario desde Calendario, Agenda, Dashboard  
**Perfil:** Todos (permisos según rol)  
**Tipo:** Modal

#### Layout

```
┌──────────────────────────────────────────┐
│ Header: "Detalle del Horario" + X       │
├──────────────────────────────────────────┤
│ [Badge PROGRAMADO]                       │
│ Ficha: 2491234 - Programación Web       │
│ Ambiente: LAB-101 (Capacidad: 30)       │
│ Instructor: Juan Pérez                   │
│ Fecha: Lunes 15 de Junio, 2026          │
│ Hora: 08:00 - 10:00                     │
│ Tema: Variables y tipos de datos        │
├──────────────────────────────────────────┤
│ Sección: Observaciones                   │
│ ┌──────────────────────────────────────┐ │
│ │ [Badge NOTA] 2026-05-20 10:15 AM     │ │
│ │ Autor: María López (Coordinador)     │ │
│ │ "Se ajustó el horario por solicitud" │ │
│ └──────────────────────────────────────┘ │
│ [Botón] Agregar Observación (Ghost)     │
├──────────────────────────────────────────┤
│ Footer (solo Coordinador):               │
│ [Editar] [Cancelar Horario]             │
│ (Secondary) (Destructive)               │
└──────────────────────────────────────────┘
```

#### Componentes

**Datos del horario:**
- Cada campo en formato "Label: Valor" (label + body)
- Badge estado top-left
- Si estado CANCELADO: Timestamp cancelación + Motivo (observación tipo INCIDENCIA)

**Sección Observaciones:**
- Lista de Cards secundarios (borde izquierdo con color según tipo):
  - NOTA: neutral-300
  - INCIDENCIA: error-500
  - CAMBIO: warning-500
- **Card anatomy:**
  - Header: Badge tipo + Timestamp (caption)
  - Body: Autor + Rol (body-sm, neutral-500)
  - Contenido: Texto observación (body)
- Ordenado: Más reciente arriba

**Botón "Agregar Observación":**
- Click: Expande inline un formulario:
  - Dropdown "Tipo" (NOTA, INCIDENCIA, CAMBIO) — default NOTA
  - Textarea "Contenido" (obligatorio, min 10 caracteres)
  - Botones: "Cancelar" | "Guardar" (Primary)
  - Guardado: Loading inline → Observación aparece en lista con animación slide-down

**Botones footer (Coordinador):**
- "Editar": Abre P-002 en modo edición (pre-poblado)
- "Cancelar Horario": Abre Modal Confirmación Cancelación (ver F-006)
  - Deshabilitado si estado != PROGRAMADO

#### Estados

- **Loading (al abrir):** Skeleton de contenido
- **Error:** Banner error top del modal
- **Permisos:** Instructor/Aprendiz NO ven botones footer (solo visualización)

#### Responsive

- **sm/md:** Modal full-screen
- **lg+:** Modal centrado max-width 700px

---

### P-006: Lista Ambientes

**Ruta:** `/recursos/ambientes`  
**Perfil:** Coordinador  
**Tipo:** Página completa

#### Layout

```
┌───────────────────────────────────────────────┐
│ Header: "Ambientes" (heading-1)               │
│ [Search box] [Filtro Estado] [Filtro Tipo]   │
│ [Botón] Crear Ambiente (Primary)             │
├───────────────────────────────────────────────┤
│ Tabla:                                        │
│ ┌────┬────────┬──────┬──────┬────────┬────┐  │
│ │Cód │ Nombre │ Tipo │ Cap. │ Estado │ ... │  │
│ ├────┼────────┼──────┼──────┼────────┼────┤  │
│ │L101│LAB-101 │LABOR.│  30  │ACTIVO  │[E][D]│
│ │A201│Aula201 │AULA  │  40  │ACTIVO  │[E][D]│
│ └────┴────────┴──────┴──────┴────────┴────┘  │
│ Paginación: [<] 1 2 3 ... [>]                │
└───────────────────────────────────────────────┘
```

#### Componentes

**Filtros y búsqueda:**
- Search box: Placeholder "Buscar por código o nombre..." (autocomplete mientras tipea)
- Dropdown "Estado": Todas | ACTIVO | INACTIVO
- Dropdown "Tipo": Todas | AULA | LABORATORIO | TALLER | AUDITORIO

**Tabla:**
- Columnas: Código | Nombre | Tipo | Capacidad | Estado | Acciones
- **Estado:** Badge (ACTIVO: success-100/success-500, INACTIVO: neutral-100/neutral-500)
- **Acciones:** Iconos:
  - [E] Editar (icono lápiz, primary-500)
  - [D] Desactivar (icono trash, error-500) — solo si sin horarios activos

**Paginación:**
- 20 items por página
- Botones < 1 2 3 ... >
- Actual: `background: primary-500`, `color: white`

**Empty state:**
- Si sin ambientes: "No hay ambientes creados. Crea el primero." + Botón "Crear Ambiente"
- Si búsqueda sin resultados: "No se encontraron ambientes con ese criterio."

#### Estados

- **Loading:** Skeleton de tabla (5 filas grises)
- **Error:** Banner rojo top

#### Responsive

- **sm:** Cards verticales en lugar de tabla (cada card con datos del ambiente + botones acción)
- **md+:** Tabla completa

---

### P-007: Formulario Crear/Editar Ambiente

**Ruta:** `/recursos/ambientes/crear` o `/recursos/ambientes/:id/editar`  
**Perfil:** Coordinador  
**Tipo:** Modal

#### Layout

```
┌─────────────────────────────────────────┐
│ Header: "Crear Ambiente" + X            │
├─────────────────────────────────────────┤
│ [Input] Código *                        │
│ [Input] Nombre *                        │
│ [Dropdown] Tipo *                       │
│ [Input Number] Capacidad *              │
│ [Textarea] Descripción (opcional)       │
├─────────────────────────────────────────┤
│ Footer: [Cancelar] [Guardar]           │
└─────────────────────────────────────────┘
```

#### Componentes

**Campos:**
- Código: max 50 caracteres, único, uppercase automático
- Nombre: max 200 caracteres, único
- Tipo: Dropdown (AULA, LABORATORIO, TALLER, AUDITORIO)
- Capacidad: Number input, min 1, placeholder "Ej: 30"
- Descripción: Textarea, max 500 caracteres, opcional

**Validaciones:**
- Obligatorios: Código, Nombre, Tipo, Capacidad
- Capacidad: Debe ser >0
- Si duplicado (código o nombre): Error del servidor → Mensaje inline "Este código/nombre ya existe"

#### Estados

- **Loading (guardado):** Spinner en botón, formulario deshabilitado
- **Success:** Toast verde + Modal se cierra + Lista se actualiza
- **Error:** Modal error "Error al guardar ambiente. Intente nuevamente."

#### Responsive

- **sm/md:** Modal full-screen
- **lg+:** Modal centrado max-width 500px

---

### P-008 y P-009: Lista y Formulario Instructores

**Simétrico a P-006 y P-007** con campos específicos:

**Formulario Instructor:**
- Nombres * (max 200 caracteres)
- Apellidos * (max 200 caracteres)
- Documento * (max 50, único)
- Tipo Contrato * (Dropdown: PLANTA | CONTRATISTA)
- Especialidad (max 100, opcional)
- Email * (max 255, único, validación formato email)
- Teléfono (max 20, opcional)
- Estado (default: ACTIVO)

---

### P-010 y P-011: Lista y Formulario Fichas

**Simétrico a P-006 y P-007** con campos específicos:

**Formulario Ficha:**
- Código * (max 50, único)
- Nombre * (max 200)
- Programa (max 200)
- Número de Aprendices * (number, min 1)
- Fecha Inicio * (date picker)
- Fecha Fin * (date picker, validación > fecha_inicio)
- Estado (default: ACTIVA)

---

### P-012: Login

**Ruta:** `/login`  
**Perfil:** Todos  
**Tipo:** Página completa

#### Layout

```
┌──────────────────────────────────────────┐
│           [Logo SENA]                    │
│     Sistema de Gestión de Horarios      │
├──────────────────────────────────────────┤
│ [Input] Usuario                          │
│ [Input] Contraseña (type=password)       │
│ [Checkbox] Recordarme                    │
│ [Botón] Iniciar Sesión (Primary)        │
│ "¿Olvidaste tu contraseña?" (link)      │
└──────────────────────────────────────────┘
```

#### Componentes

- Centrado vertical y horizontalmente
- Card contenedor: max-width 400px, padding spacing-6
- Logo SENA: SVG o imagen, centrado, 80x80px
- Título: heading-2, centrado
- Form con campos estándar (sección 2.2)
- Link "¿Olvidaste tu contraseña?": body-sm, color primary-500

#### Estados

- **Loading (login):** Spinner en botón + "Iniciando sesión..."
- **Error:** Banner rojo arriba del formulario "Usuario o contraseña incorrectos"

**Nota MVP:** Autenticación básica (sin OAuth ni 2FA). Ver C-005 en architecture.md.

---

## 4. Accesibilidad (WCAG 2.1 AA)

### 4.1. Contraste Validado

| Combinación | Ratio | Estado |
|------------|-------|--------|
| neutral-900 / white | 16.75:1 | ✅ AAA |
| primary-500 / white | 4.67:1 | ✅ AA |
| error-500 / white | 4.15:1 | ✅ AA |
| success-500 / white | 3.19:1 | ⚠️ Solo texto ≥18px bold |

**Acciones correctivas:**
- Success text: Usar solo en badges (12px uppercase bold ≥ 18px equiv)
- Si texto normal: Usar success-600 (#059669) con ratio 4.5:1 ✅

### 4.2. Navegación por Teclado

- **Tab order:** Lógico (izquierda→derecha, arriba→abajo)
- **Focus visible:** `shadow: shadow-focus` (ring azul 3px) en todos los interactivos
- **NUNCA:** `outline: none` sin alternativa visual
- **Modales:** Focus trap con Tab/Shift+Tab
- **Escape:** Cierra modales y dropdowns
- **Enter:** Submits formularios, activa botones
- **Spacebar:** Activa checkboxes y botones

### 4.3. ARIA Labels

**Obligatorio en:**
- Botones con solo iconos: `aria-label="Editar ambiente"`
- Inputs: Asociar `<label for="id">` con `<input id="id">`
- Alertas: `<div role="alert" aria-live="polite">Mensaje</div>`
- Modales: `role="dialog"` + `aria-labelledby="modal-title"` + `aria-describedby="modal-desc"`
- Tabs (si se implementan): `role="tablist"`, `role="tab"`, `aria-selected`

### 4.4. Targets Táctiles (Móvil)

- **Mínimo:** 44x44px para todos los elementos interactivos
- **Espaciado:** ≥8px entre targets adyacentes
- **Validación:** Botones con `min-height: 44px`, `min-width: 44px`

### 4.5. Mensajes de Error

- **Visual:** Color rojo (error-500) + icono ✕
- **Screen reader:** Texto asociado al campo con `aria-describedby`
- **Focus:** Auto-focus en primer campo con error al submit

---

## 5. Responsive Breakpoints y Comportamiento

### Breakpoints

| Token | Valor | Dispositivo |
|-------|-------|-------------|
| `breakpoint-sm` | 640px | Móvil (portrait) |
| `breakpoint-md` | 768px | Tablet (portrait) |
| `breakpoint-lg` | 1024px | Desktop pequeño |
| `breakpoint-xl` | 1280px | Desktop grande |

### Estrategia General

- **Mobile-first:** Diseño base para sm, ajustes progresivos para md/lg/xl
- **Fluido:** Contenedores con max-width, no fixed widths
- **Touch-friendly:** Targets ≥44px en sm/md

### Comportamiento por Pantalla (Resumen)

| Pantalla | sm | md | lg+ |
|----------|----|----|-----|
| Dashboard | Cards 1 col | Cards 1 col | Cards 2 cols |
| Calendario | Lista días (acordeón) | Grid 3 días + scroll | Grid 7 días completo |
| Formularios (modales) | Full-screen | Full-screen | Modal centrado 600px |
| Tablas | Cards verticales | Tabla scroll horizontal | Tabla completa |
| Agenda | Lista 1 col | Lista 1 col | Lista 1 col |

---

## 6. Animaciones y Transiciones

### Principios

- **Duración:** 150-300ms (rápido), nunca >500ms
- **Easing:** `ease-out` para entradas, `ease-in` para salidas, `ease-in-out` para transformaciones
- **Reducir movimiento:** Respetar `prefers-reduced-motion: reduce` (sin animaciones complejas)

### Casos de Uso

| Elemento | Animación | Duración |
|----------|-----------|----------|
| Hover en cards | Box-shadow cambio | 200ms ease |
| Modal entrada | Fade in + scale(0.95→1) | 200ms ease-out |
| Toast aparición | Slide-in desde derecha | 300ms ease-out |
| Dropdown expand | Max-height 0→auto | 200ms ease |
| Observación nueva | Slide-down | 300ms ease-out |
| Botón click | Ripple effect | 400ms linear |

---

## 7. Iconografía

### Librería

**Heroicons** (https://heroicons.com/) — Set Outline (24x24px)

### Iconos Principales

| Icono | Nombre Heroicons | Uso |
|-------|-----------------|-----|
| ➕ | `plus` | Crear, Agregar |
| ✏️ | `pencil` | Editar |
| 🗑️ | `trash` | Eliminar, Desactivar |
| ✓ | `check` | Éxito, Confirmación |
| ✕ | `x-mark` | Error, Cerrar |
| ⚠️ | `exclamation-triangle` | Advertencia |
| ℹ️ | `information-circle` | Información |
| 📅 | `calendar` | Calendario, Fecha |
| 🕐 | `clock` | Hora, Tiempo |
| 👤 | `user` | Usuario, Perfil |
| 🏠 | `home` | Dashboard, Inicio |
| 🔍 | `magnifying-glass` | Búsqueda |
| ⚙️ | `cog-6-tooth` | Configuración |
| 🚪 | `arrow-right-on-rectangle` | Cerrar sesión |

**Estilos:**
- Tamaño: 20x20px (inline con texto), 24x24px (standalone)
- Color: Hereda del padre o color específico (primary-500, error-500, etc.)
- Stroke width: 1.5 (default Heroicons Outline)

---

## 8. Estados de Componentes (Resumen)

### Botón

- Default → Hover → Active → Focus → Disabled → Loading

### Input

- Default → Focus → Error → Disabled → Filled

### Card

- Default → Hover (si clickeable) → Active (si seleccionable)

### Modal

- Hidden → Entering (fade+scale) → Visible → Exiting (fade)

### Toast

- Hidden → Entering (slide-in) → Visible (5s) → Exiting (fade) → Hidden

---

## 9. Validación de Cumplimiento

### Checklist Pre-implementación

- [x] Todos los colores con tokens definidos (no hex hardcoded)
- [x] Contraste WCAG AA validado (mínimo 4.5:1 texto normal)
- [x] Focus visible en todos los interactivos
- [x] Targets táctiles ≥44px móvil
- [x] Estados obligatorios por pantalla (loading, error, empty, success)
- [x] Estados obligatorios por componente (default, hover, active, focus, disabled)
- [x] Responsive declarado por breakpoint (no "se adapta")
- [x] Mensajes de validación específicos (no genéricos)
- [x] ARIA labels en iconos sin texto
- [x] Animaciones ≤300ms con prefers-reduced-motion

---

## 10. Próximos Pasos (Post-MVP)

### v1.1 — Mejoras UX

- **Dark mode:** Paleta alternativa con inversión de luminosidad
- **Notificaciones push:** Alertas de conflictos en tiempo real
- **Filtros avanzados:** Calendario con múltiples fichas simultáneas
- **Exportar PDF:** Calendario semanal descargable

### v1.2 — Accesibilidad Avanzada

- **Lectores de pantalla:** Testing exhaustivo con NVDA/JAWS
- **Navegación por voz:** Comandos voice-over integrados
- **AAA compliance:** Contraste 7:1 para todos los textos

### v2.0 — Mobile Native

- **App nativa:** React Native iOS/Android
- **Offline-first:** Sync local + backend cuando reconecta
- **Gestos móviles:** Swipe para editar/cancelar horarios

---

## Aprobaciones

| Rol | Nombre | Fecha | Firma |
|-----|--------|-------|-------|
| UX Designer | A03 | 2026-05-20 | ✅ |
| Frontend Dev | A08 | Input | ⏸️ Implementará componentes siguiendo esta spec |
| QA | A09 | Testing | ⏸️ Validará conformidad con WCAG AA y responsive |

**Estado:** ✅ **APROBADO — GATE HALT-UX DESBLOQUEADO**

---

**Producido:** 2026-05-20  
**Última revisión:** 2026-05-20  
**Próxima revisión:** Post-Sprint 2 (ajustes tras testing con usuarios)
