# AGENTS.md — Guía técnica de "Mis Cartas"

Documento para cualquier persona o IA que vaya a mantener, mejorar o escalar esta app.
**Léelo completo antes de cambiar código.** Las secciones marcadas con 🚨 son reglas que, si se rompen, **hacen que mamá pierda sus datos**.

---

## 1. Contexto del proyecto

- **Usuaria única:** la mamá del dueño del repo. Vive en Estados Unidos, usa **iPhone** (teléfono, no computadora), no es técnica.
- **Negocio:** le pagan por escribir y enviar cartas físicas a 3 casinos: **Chumba Casino**, **LuckyLand** y **Global Poker**.
- **Objetivo:** llevar el control de:
  1. Material e inventario (hojas, sobres, estampillas, lapiceros, etc.) y gastos.
  2. Por cada casino: cartas enviadas, pago por carta, total esperado, pagos recibidos, lo que le deben.
  3. Resumen general: invertido, cobrado, pendiente, ganancia neta, costo y ganancia por carta.
- **Idioma de la interfaz:** español (neutro/latino, trato de "tú"). **Moneda:** solo USD.
- **Alcance:** pequeño, funcional, cómodo e intuitivo. **No** es un producto público ni multiusuario.

## 2. Decisiones de arquitectura (y por qué)

| Decisión | Motivo | Alternativas descartadas |
|---|---|---|
| **Un solo archivo `index.html`** con HTML + CSS + JS inline, sin build, sin dependencias | Mínimo mantenimiento, se puede abrir/editar con cualquier editor, imposible que "se rompa el build" | React/Vite (exceso para el tamaño), app nativa (requiere tiendas/firmas) |
| **Datos en `localStorage`** del navegador | Sin servidor, sin cuentas, sin costo, funciona offline, privacidad total | Firebase/Supabase (requiere login y cuentas para mamá), Google Sheets (incómodo en celular) |
| **Copia de seguridad manual a archivo JSON** (Web Share API en móvil, descarga en escritorio) + recordatorio cada 7 días | `localStorage` puede perderse; esta es la red de seguridad | Sincronización en la nube (complejidad y login) |
| **Hosting en GitHub Pages** (`https://bers3rks.github.io/mis-cartas/`) | Gratis, HTTPS, URL estable, deploy con `git push` | Netlify Drop (requiere cuenta para no expirar), artefactos de Claude (privados, mamá no puede entrar sin cuenta) |
| **"Agregar a pantalla de inicio"** en iOS | Las web apps instaladas en inicio quedan exentas del borrado de almacenamiento de Safari (ITP, 7 días sin uso) | — |
| **"Kit por carta"**: al registrar un envío se descuenta automáticamente el material | Pedido explícito; evita que mamá anote cada hoja | Descuento manual total |
| **Uso manual de material** además del kit | Pedido explícito: para hojas dañadas, lapiceros acabados, etc. | — |
| **Pago por carta manual por envío** (puede quedar vacío) | No se conoce la tarifa de cada casino y podría variar | Tarifa fija por casino |
| **Envíos y pagos registrados por separado** | Los casinos no pagan al momento; así se calcula "te debe" | Un solo registro "venta" |
| **Tipografía Nunito** (Google Fonts) con fallback a fuentes del sistema | Redondeada, amigable y muy legible para una persona mayor | — |
| **Modo claro/oscuro**: sigue al sistema por defecto + botón manual 🌙/☀️ | Pedido explícito | — |

## 3. Estructura de archivos

```
index.html          ← TODA la app (estilos, marcado y lógica)
README.md           ← Guía para humanos (instalación en iPhone, copias, uso)
AGENTS.md           ← Este documento (guía técnica y reglas)
CLAUDE.md           ← Apunta a AGENTS.md (para Claude Code)
.claude/launch.json ← Servidor local de vista previa (py -m http.server 8765)
```

Dentro de `index.html`, el `<script>` está ordenado en bloques comentados:
`Helpers` → `Data` → `Navigation` → `Views` → `Sheet (forms)` → `Backup` → `Light / dark mode` → `Actions`.

## 4. 🚨 Reglas que NO se pueden romper

1. 🚨 **No cambiar la clave de `localStorage` `misCartas.v1`.** Si se cambia, la app arranca vacía y los datos "desaparecen". Claves auxiliares: `misCartas.v1.tab` (pestaña activa) y `misCartas.v1.theme` (`light` | `dark`).
2. 🚨 **No cambiar la URL / origen de la app** (`bers3rks.github.io` + `/mis-cartas/`). `localStorage` está ligado al origen; otro dominio = datos vacíos. No renombrar el repo ni el usuario de GitHub, ni mover a otro hosting, **sin antes pedirle a mamá que haga "Guardar copia"** y luego "Recuperar copia" en la nueva dirección (y volver a agregarla a la pantalla de inicio).
3. 🚨 **No cambiar los `id` de casinos (`chumba`, `luckyland`, `globalpoker`) ni de materiales existentes** (`hojas`, `sobres`, `estampillas`, `lapiceros`, o los generados con `uid()`). Los registros los referencian por `id`. Los **nombres visibles** sí se pueden cambiar.
4. 🚨 **Cambios de esquema = migración.** Si se agrega/renombra un campo, subir `version` y escribir una función de migración en `load()` que convierta datos viejos. Nunca asumir que los datos guardados tienen la forma nueva. Hoy `load()` hace `Object.assign(defaults(), guardado)`, lo que rellena **colecciones de primer nivel** faltantes, pero **no** campos dentro de cada registro.
5. 🚨 **La copia de seguridad debe poder restaurarse siempre.** Cualquier cambio de formato debe seguir aceptando copias viejas (`importFile` valida que existan `casinos`, `shipments` y `purchases` como arrays).
6. **El stock nunca se guarda: se calcula** (`stock()`), a partir de compras − envíos×kit − usos. No agregar un campo "stock" guardado (se desincroniza).
7. **El kit se guarda congelado en cada envío** (`shipment.kit`). Cambiar el kit en ajustes afecta solo a envíos **nuevos**; los pasados conservan lo que consumieron. No recalcular envíos viejos con el kit actual.
8. **Escapar con `esc()` todo dato guardado** antes de meterlo en `innerHTML`: textos, pero también `id`, íconos y colores. Los datos pueden venir de un archivo de copia importado, así que no se consideran confiables.
9. **Todo en español** y con palabras simples (nada de "registro", "entidad", "sincronizar"…; preferir "anotar", "guardar", "te debe").
10. **Mobile-first:** objetivos táctiles ≥ 44px (los botones usan 54–60px), inputs ≥ 16px de fuente (si no, iOS hace zoom al enfocar), respetar `env(safe-area-inset-*)`.
11. **Colores solo vía tokens CSS** (`--bg`, `--card`, `--ink`, …). Todo token nuevo debe definirse **tres veces**: en `:root` (claro), en `@media (prefers-color-scheme: dark) :root:not([data-theme="light"])` y en `:root[data-theme="dark"]`.
12. **Sin dependencias externas** salvo la fuente de Google Fonts. La app debe funcionar sin internet (si la fuente no carga, usa la del sistema).

## 5. Modelo de datos

Un único objeto JSON `S` guardado en `localStorage['misCartas.v1']`:

```jsonc
{
  "version": 1,
  "lastBackup": 1760000000000,          // ms epoch de la última "Guardar copia", o null
  "materials": [
    { "id": "hojas", "name": "Hojas", "kit": 1, "alert": 20 }
    // kit   = unidades que consume cada carta (0 = no se descuenta solo)
    // alert = avisar cuando el stock sea <= a este número
  ],
  "casinos": [
    { "id": "chumba", "name": "Chumba Casino", "short": "Chumba", "icon": "🍀",
      "color": "#2f7d4f", "rate": 4 }   // rate = último pago por carta usado (para prellenar)
  ],
  "purchases": [                         // compras de material Y "otros gastos"
    { "id": "…", "ts": 0, "date": "2026-10-11",
      "materialId": "sobres",            // null = "Otro gasto" (gasolina, copias…)
      "desc": "",                        // solo se usa si materialId es null
      "qty": 100, "cost": 8.5, "note": "" }
  ],
  "usages": [                            // uso manual de material (hoja dañada, etc.)
    { "id": "…", "ts": 0, "date": "…", "materialId": "hojas", "qty": 3, "note": "" }
  ],
  "shipments": [                         // envíos de cartas a un casino
    { "id": "…", "ts": 0, "casinoId": "chumba", "date": "…",
      "letters": 5, "rate": 4,           // rate null = aún no se sabe el pago
      "kit": { "hojas": 1, "sobres": 1, "estampillas": 1 },  // kit congelado al crear
      "note": "" }
  ],
  "payments": [                          // pagos recibidos de un casino
    { "id": "…", "ts": 0, "casinoId": "chumba", "date": "…", "amount": 12, "note": "" }
  ]
}
```

- Fechas: `YYYY-MM-DD` en hora local (`today()`). `ts` es solo para desempatar el orden.
- Montos en dólares como `number` (no centavos enteros). Se formatean con `Intl.NumberFormat('en-US', USD)`.
- Los colores de degradado de cada casino están en la constante `CASINO_COLORS` (por `id`); `casino.color` es el respaldo para casinos sin entrada ahí.

## 6. Fórmulas (fuente de verdad)

| Concepto | Fórmula | Función |
|---|---|---|
| Stock de un material | Σ compras.qty − Σ (envío.letters × envío.kit[id]) − Σ usos.qty | `stock(id)` |
| Cartas que alcanzan | min sobre materiales con kit>0 de ⌊stock / kit⌋ | `lettersLeft()` |
| Esperado de un casino | Σ envío.letters × envío.rate (rate null cuenta 0) | `casinoStats()` |
| Cobrado de un casino | Σ pagos.amount | `casinoStats()` |
| Te debe | max(0, esperado − cobrado) | `casinoStats()` |
| Invertido total | Σ compras.cost (material + otros gastos) | `totals()` |
| **Ganancia hasta hoy** | cobrado total − invertido | `totals().net` |
| Ganancia si pagan todo | cobrado + pendiente − invertido | `totals().netIfPaid` |
| Costo por carta | invertido / cartas enviadas | `viewSummary()` |
| Ganas por carta | (cobrado + pendiente − invertido) / cartas | `viewSummary()` |

Si se cambia una fórmula, actualizar esta tabla.

## 7. Interfaz

- **Navegación inferior** con 5 pestañas: `resumen`, `material`, y una por casino (`TABS()` las genera desde `S.casinos`). La grilla CSS es `repeat(5, 1fr)`: **si se agrega un casino hay que ajustar la grilla** (o pasar a scroll horizontal / menú "Más").
- **Formularios** en una hoja inferior (bottom sheet) reutilizable: `openSheet({ title, body, onSave, onDelete?, onInput? })`.
  - `onSave(datos)` devuelve `false` para no cerrar (validación) o un texto para el aviso (toast).
  - `onDelete` presente ⇒ aparece el botón "Borrar" (con `confirm`).
  - `onInput(form)` se ejecuta en cada cambio (vista previa en vivo).
- **Eventos**: delegación global con `data-act="nombre"` + `data-*`, despachada en el objeto `ACTIONS`. Para un botón nuevo: agregar `data-act` y la entrada en `ACTIONS`.
- Toda la vista se re-renderiza con `render()` después de cada guardado (la app es pequeña; no hace falta un DOM virtual).

### Sistema visual
- Fuente: **Nunito** 600–900. Base 19px.
- Paleta clara: fondo crema `#fbf6ef`, tarjetas blancas, marca morada `#6d3fc0 → #9a5ce6`.
- Paleta oscura: fondo `#17131c`, tarjetas `#231d2b`, marca `#9b6cf0`.
- Tarjeta de ganancia: verde si > 0, roja/coral si < 0, morada si = 0.
- Casinos: Chumba verde, LuckyLand dorado/naranja, Global Poker azul (`CASINO_COLORS`).
- El tema se aplica con `data-theme` en `<html>`; sin atributo = sigue al sistema.

## 8. Cómo hacer cambios comunes

**Agregar un casino**
1. Añadirlo en `defaults().casinos` con un `id` nuevo y único.
2. Como los usuarios existentes ya tienen `casinos` guardado, `defaults()` **no** lo agregará: hace falta una migración en `load()` (si no existe un casino con ese `id`, hacer `push`).
3. Añadir sus colores en `CASINO_COLORS`.
4. Ajustar la grilla de `.tabs` (ver §7).

**Agregar un material por defecto** → igual que casino: `defaults()` + migración. (Mamá también puede agregarlo desde ⚙️ en la app.)

**Agregar un campo nuevo a un registro** → subir `version`, migrar en `load()`, actualizar formularios, vistas, §5 y §6.

**Exportar a Excel/CSV, gráficas, varios usuarios, sincronizar entre dispositivos** → hoy fuera de alcance a propósito. Si se necesita sincronización, la opción menos intrusiva sería mantener `localStorage` como caché y añadir un backend opcional, sin obligar a mamá a iniciar sesión.

## 9. Probar

No hay tests automáticos (decisión consciente por tamaño). Antes de publicar, probar a mano en ancho de celular (375×812):

1. `py -m http.server 8765` → http://localhost:8765
2. Comprar material, usar material, otro gasto.
3. Enviar cartas con y sin pago por carta → ver el aviso "sin precio", el descuento del kit y "te debe".
4. Registrar pago → verificar resumen (ganancia, te deben, costo por carta).
5. Editar y borrar registros.
6. Modo claro y oscuro (botón 🌙/☀️).
7. Guardar copia → borrar `localStorage` → Recuperar copia → todo vuelve.
8. **Probar con datos de una versión anterior** (pegar un JSON viejo en `localStorage`) si se cambió el esquema.

Al terminar, borrar los datos de prueba del navegador local.

## 10. Publicar

- Repo: `https://github.com/bers3rks/mis-cartas`, rama `main`, GitHub Pages sirve la raíz de `main`.
- Publicar = `git commit` + `git push`. Pages se actualiza en ~1–2 min.
- El repo es **público** (requisito de GitHub Pages gratis). No contiene datos de mamá: esos viven solo en su teléfono. **Nunca subir copias de seguridad (`mis-cartas-copia-*.json`)** al repo (están en `.gitignore`).
- iOS puede tener en caché la versión vieja: cerrar la app desde el selector de apps y volver a abrirla.

## 11. Historial de decisiones

- **2026-10-11** — Versión 1: app de un archivo, `localStorage`, 5 pestañas, kit por carta, uso manual de material, otros gastos, pago por carta manual, copias JSON.
- **2026-10-11** — Rediseño: tipografía Nunito, paleta cálida morada, colores por casino, tarjeta de ganancia verde/roja, modo claro/oscuro con botón, mayor tamaño de texto y botones.
- **2026-10-11** — Publicada en GitHub Pages.
