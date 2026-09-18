# Boda Master Plan

Planeador de boda en una sola página. Sin dependencias, sin build, sin backend:
dos archivos HTML que funcionan abriéndolos con doble clic.

```
index.html   Planeador completo (privado)
guia.html    Guía del Día B — vista pública de solo lectura
```

---

## Qué incluye

**Panel** · cuenta regresiva, anillo de avance, presupuesto real contra plan, pagos
vencidos, confirmaciones, dona de gasto por categoría, próximas tareas y próximos pagos.

**Tareas** · 23 tareas precargadas en 6 etapas; las fechas límite se calculan solas a
partir del día de la boda.

**Dinero** · presupuesto (costo fijo o por invitado, estimado contra real), calendario de
pagos con vencimientos, directorio de proveedores con estado de contrato, y módulo de
ahorro con curva acumulada.

**Invitados** · RSVP por toque, acompañantes, restricciones alimentarias, importación
masiva desde Excel o CSV, **acomodo visual de mesas** (arrastrar en escritorio, tocar en
celular) y asignación de hospedaje.

**Día B** · cronograma minuto a minuto con bloque, responsable y proveedor a cargo —
con marcado telefónico directo — más la agenda de citas previas.

**Más** · música por momento con **lista negra** para el DJ, padrinos y cortejo, registro
de regalos y agradecimientos, tablero de inspiración, datos del lugar y cuaderno de notas.

**Reportes** · seis salidas imprimibles con diseño de papelería: cronograma, acomodo de
mesas, lista de invitados, música y lista negra, presupuesto y pagos, resumen general.

---

## Cómo se guardan los datos

El planeador escribe en dos lugares, en este orden:

1. **`localStorage` del navegador** — siempre, al instante.
2. **La nube** — solo cuando la página corre como Artifact en claude.ai, donde el
   entorno le inyecta `window.claude`. Ahí usa la capability `db`: seis documentos
   (`plan/settings`, `plan/tasks`, `plan/money`, `plan/people`, `plan/day`, `plan/extras`),
   con suscripción en vivo y cola de reintentos para los cambios hechos sin conexión.

**Fuera de claude.ai** (abriendo el archivo, o servido desde GitHub Pages) `window.claude`
no existe, `claude.use()` nunca resuelve y la app se queda en el paso 1: funciona
completa, pero **los datos viven solo en ese navegador** y no se sincronizan entre
dispositivos. El indicador de la barra superior lo muestra en ámbar, y Ajustes lo dice
explícitamente.

Para mover datos entre navegadores en ese modo: **Ajustes → Descargar respaldo**
(un `.json` con todo) y **Ajustes → Importar**. El importador también acepta el formato
del planeador anterior (`planeadorBodaMTY_v2`).

---

## La guía pública (`guia.html`)

Página de solo lectura para proveedores, coordinadora y familia. Su contenido vive en un
bloque JSON dentro del propio archivo:

```html
<script type="application/json" id="payload">{ … }</script>
```

Hay dos formas de actualizarla:

- **Como Artifact en claude.ai** — declara la capability `artifact`, así que quien tiene
  permiso de edición ve un botón *Actualizar*, pega el paquete de datos y la página se
  vuelve a publicar sola con esa información.
- **En un repo o en GitHub Pages** — edita el bloque `#payload` a mano y haz commit. El
  botón *Actualizar* no aparece (no hay `window.claude`), y así debe ser.

El paquete se genera desde el planeador en **🖨 → Vista pública con link**, con casillas
para elegir qué secciones se publican. Nunca incluye teléfonos ni correos de invitados,
presupuesto, pagos, ahorro, regalos, ni las aportaciones de los padrinos. Las
restricciones alimentarias vienen apagadas por defecto: son datos de salud de terceros.

Forma del paquete:

```json
{
  "v": 1, "couple": "", "date": "AAAA-MM-DD", "venue": "", "city": "", "updated": "AAAA-MM-DD",
  "timeline": [{ "t": "17:00", "e": "18:15", "title": "", "block": "", "owner": "", "vendor": "", "phone": "", "notes": "" }],
  "tables":   [{ "label": "", "capacity": 10, "guests": [{ "name": "", "extra": 0, "diet": "" }] }],
  "songs":    [{ "moment": "", "title": "", "artist": "", "status": "", "notes": "" }],
  "veto":     [{ "name": "", "tipo": "", "nivel": "", "reason": "" }],
  "sponsors": [{ "name": "", "role": "" }],
  "tasks":    [{ "title": "", "due": "AAAA-MM-DD", "phase": "", "done": false }],
  "contacts": [{ "name": "", "role": "", "phone": "" }]
}
```

Cualquier clave que falte simplemente no dibuja su sección.

---

## Publicar en GitHub Pages

```bash
git init
git add .
git commit -m "Planeador de boda"
git branch -M main
git remote add origin git@github.com:USUARIO/REPO.git
git push -u origin main
```

Luego **Settings → Pages → Source: `main` / raíz**. Queda en
`https://USUARIO.github.io/REPO/` (el planeador) y `…/guia.html` (la guía).

> **Piensa antes de hacer público el repo.** `index.html` no contiene datos —
> se leen de `localStorage` en el navegador de cada persona— pero `guia.html` **sí**
> lleva su JSON dentro del archivo. Si le pusiste el acomodo de mesas con nombres, esos
> nombres quedan en el historial de Git para siempre y son visibles para cualquiera. Para
> la guía con datos reales, repo privado o Pages solo para la guía.

---

## Detalles técnicos

- Vanilla JS, sin framework. Un solo `render()` que reescribe `#app`, preservando scroll,
  foco y posición del cursor.
- Tipografías: Bodoni Moda y Jost desde Google Fonts. Con la red caída, caen a Didot/Times
  y a la fuente del sistema sin romper la retícula.
- Tokens de color en `:root`, redefinidos para tema oscuro bajo
  `@media (prefers-color-scheme: dark)` y `[data-theme="dark"]`. El planeador tiene los dos
  temas; la guía está comprometida a claro a propósito, por ser papelería.
- Móvil primero: nav inferior con respeto a `safe-area-inset`, que en ≥900 px se convierte
  en riel lateral.
- Gráficas dibujadas a mano en SVG (anillo, dona, curva de ahorro). Cero librerías.
- Accesible en lo básico: `aria-label` en los botones de icono, foco visible,
  `prefers-reduced-motion` respetado, Escape cierra cualquier hoja modal.
- Los reportes se construyen en un `<div id="report">` fuera de `#app` y solo existen
  durante la impresión.

## Licencia

Uso personal. Hazle lo que quieras.
