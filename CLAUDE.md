# CLAUDE.md — Guía del proyecto Mini-AoE

Este archivo da contexto a Claude (y a cualquier desarrollador) sobre el
proyecto. **Léelo al empezar cualquier sesión** y respeta las normas de abajo.

---

## 1. Qué es el proyecto

**Mini-AoE**: una versión *ultra básica* de Age of Empires para jugarse con
**pantalla táctil desde el navegador del iPad** (Safari). Todo el juego vive en
un único archivo **`index.html`** (Canvas 2D + JavaScript puro, sin
dependencias, sin servidor, sin proceso de compilación).

- Diseño completo: ver **`DESIGN.md`**.
- Mapa de archivos y estructura del código: ver **`filemap.md`**.
- Bitácora de avance: ver **`progress.md`**.

## 2. Stack y restricciones

- **Un solo archivo jugable**: `index.html` (HTML + CSS + JS embebido). No
  añadir build tools, frameworks ni dependencias externas salvo que se acuerde.
- **Táctil primero**: objetivos de toque ≥ 44 px; 1 dedo = órdenes/caja de
  selección, 2 dedos = paneo + zoom. Debe funcionar también con ratón/rueda en
  escritorio (para pruebas).
- **Render**: vista cenital, sprites a base de emoji y figuras (cero assets).
  Escalado por `devicePixelRatio` para nitidez Retina.
- **Idioma**: la UI y los comentarios del código están en **español**.

## 3. Flujo de trabajo (git / despliegue)

- Rama de desarrollo: **`claude/mini-aoe-browser-game-k5vf3r`**. No empujar a
  otra rama sin permiso explícito.
- Cada tanda de cambios: commit claro → push → **PR en borrador** hacia `main`.
- `main` es la rama que sirve **GitHub Pages**:
  `https://juandiegorodri.github.io/Ageofempires/`. Al fusionar a `main`, la
  web se actualiza sola.
- Tras fusionar una PR con *squash*, **sincroniza la rama con `main`**
  (`git fetch origin main` + `git reset --hard origin/main`) antes de la
  siguiente tanda, para que el próximo PR tenga un diff limpio.

## 4. Verificación

Antes de dar por terminada una funcionalidad, **probar en Chromium headless**
(Playwright ya está disponible en el entorno):
- Cargar `index.html`, iniciar partida y comprobar que **no hay errores de
  consola** (`pageerror` / `console.error`).
- Ejercitar la nueva lógica vía `page.evaluate(...)` accediendo a las funciones
  y al estado global (`entities`, `player`, etc.).
- Cuando aplique, capturar una **captura de pantalla** para revisar el aspecto.

Ejecutar con: `node test.cjs` usando
`require('/opt/node22/lib/node_modules/playwright')` y
`executablePath: '/opt/pw-browsers/chromium'`.

---

## 5. NORMAS DE DOCUMENTACIÓN (obligatorias)

> **Cada vez que se cree o modifique una funcionalidad, hay que actualizar la
> documentación en el mismo cambio (mismo commit/PR).**

### 5.1 `CLAUDE.md` (este archivo) — listado de funcionalidades
Mantener actualizada la sección **«6. Listado de funcionalidades»** con TODAS
las funcionalidades del juego. Al añadir una nueva, agrégala a la lista con una
línea descriptiva. Al cambiar uno comportamiento existente, edítalo.

### 5.2 `filemap.md` — mapa de archivos
Describe cada archivo del repositorio y, para `index.html`, las secciones
principales del código (definiciones de datos, estado, lógica, render, entrada,
UI). Al añadir una sección/sistema nuevo de código, **actualiza `filemap.md`**.

### 5.3 `progress.md` — bitácora de avance
Registro cronológico. Por cada tanda de trabajo añade una entrada **al final**
con: fecha, número de PR (si aplica), y lista de cambios. No borres el
historial; solo se agrega.

> Regla rápida: **funcionalidad nueva ⇒ actualizar `CLAUDE.md` (lista) +
> `filemap.md` (si hubo cambios de estructura) + `progress.md` (siempre).**

---

## 6. Listado de funcionalidades

### Núcleo del juego
- **Motor**: Canvas 2D + JS puro en un solo `index.html`, optimizado para táctil
  de iPad; escalado Retina; bloqueo de gestos del navegador.
- **Cámara**: paneo con dos dedos, zoom por pinch; rueda y flechas en
  escritorio; botón «Centrar» (⌂) sobre la base propia.
- **Selección**: toque simple (unidad/edificio), arrastre de un dedo = caja de
  selección (prioriza militares), doble toque = todas las unidades del mismo
  tipo visibles, toque en vacío = deseleccionar.
- **Órdenes contextuales**: con unidades seleccionadas, tocar terreno = mover,
  recurso = recolectar (aldeanos), enemigo = atacar (militares), cimientos
  propios = construir.
- **Recursos (4)**: comida 🍖, madera 🪵, oro 💰, piedra 🪨. Nodos finitos; la
  recolección se suma directa al marcador (sin viaje de retorno). Al agotarse un
  nodo, el aldeano busca el siguiente del mismo tipo.
- **Unidades (5)**: Aldeano, Milicia, Piquetero, Arquero, Caballo.
- **Edificios (8)**: Centro Urbano, Casa, Cuartel, Galería de Tiro, Establo,
  Herrería, Torre y Castillo. Prerrequisitos: Galería y Establo requieren
  Cuartel; el Castillo requiere Edad Feudal (Era III).
- **Cuadrilátero de combate (×2)**: Arquero → Milicia → Piquetero → Caballo →
  Arquero (cada uno fuerte contra el siguiente).
- **Entrenamiento por cola**: cada edificio productor tiene cola con coste y
  tiempo; punto de reunión (rally) configurable tocando el terreno.
- **Construcción**: el jugador coloca cimientos (silueta que sigue el dedo) y
  los aldeanos los construyen con barra de progreso.
- **Mejoras de Herrería (4)**: Flechas de Punta de Hierro, Forja de Espadas,
  Escudos de Madera, Hachas Afiladas. Requieren Edad de las Herramientas.
- **Avance de Era**: cuatro edades (ver «Funcionalidades añadidas»); cada avance
  cuesta recursos y desbloquea contenido.
- **IA enemiga**: 3 dificultades (Fácil/Normal/Difícil); recolecta, construye,
  avanza de era, entrena ejército variado y lanza oleadas de ataque.
- **Población**: límite dinámico según edificios (ver «Población dinámica»),
  mostrado en la barra superior.
- **Auto-defensa**: las unidades militares inactivas atacan enemigos cercanos.
- **Victoria/Derrota**: gana quien destruye el Centro Urbano rival.
- **UI/UX**: barra superior de recursos, panel inferior de acciones contextual,
  avisos (hints), pausa (⏸), pantallas de inicio y de fin.

### Funcionalidades añadidas
- **Localizador de aldeanos inactivos** (PR #2): botón 👷 arriba a la derecha con
  contador en vivo; al tocarlo selecciona y centra la cámara en el siguiente
  aldeano sin tarea (rota entre ellos). En el mapa los inactivos muestran 💤.
- **Símbolo de recurso al recolectar** (PR #2): cada aldeano que recolecta
  muestra una insignia con el icono del recurso (🍒/🌳/💰/🪨).
- **Botón Deseleccionar** (PR #2): en el panel de acciones, para unidades y
  edificios; limpia la selección.
- **Torres defensivas** (PR #2): edificio Torre 🗼 construible por aldeanos
  (madera + piedra); dispara automáticamente a enemigos en rango (muestra su
  radio al seleccionarla). La IA construye torres en Normal/Difícil.
- **Producción acelerada por cantidad de edificios** (PR #3): cuantos más
  edificios del mismo tipo productor tenga un bando, más rápido entrena ese tipo
  de unidades. Multiplicador = 1 + 0.5 × (nº de edificios − 1); 1 = ×1, 2 = ×1.5,
  3 = ×2, … Se muestra en el panel del edificio.
- **Nivel de producción por recurso** (PR #3): junto a cada recurso de la barra
  superior se muestra la tasa actual de recolección (p. ej. «+1.4/s»), calculada
  según los aldeanos que están recolectando ese recurso. Se resalta en verde
  cuando hay producción activa.
- **Menú principal con opciones** (PR #4): antes de jugar se configura la
  partida: mapa, recursos iniciales (Bajo/Estándar/Alto), velocidad
  (Lenta/Normal/Rápida), inteligencia de la IA (Fácil/Normal/Difícil) y posición
  del jugador (Izquierda/Derecha/Aleatoria).
- **Mapas con temática** (PR #4): Llanura, Río (río central que bloquea
  construcción), Selva Negra (madera abundante por todo el mapa) y Riscos
  (bloques rocosos que bloquean construcción; abunda piedra y oro).
- **Población dinámica** (PR #4): se empieza con 20 de población. El Centro
  Urbano aporta 20, cada Casa 🏠 +5 y el Castillo 🏰 +50 (tope absoluto 200).
- **Edificios nuevos** (PR #4): Casa 🏠 (sube población) y Castillo 🏰 (sube +50
  población, defensa potente con auto-disparo; requiere Edad Feudal).
- **Cuatro edades** (PR #4): Inicial → Edad de las Herramientas → Edad Feudal →
  Edad Imperial. Cada avance cuesta más recursos y desbloquea contenido
  (mejoras de Herrería en Era II, Castillo en Era III, tecnologías económicas de
  nivel 2 en Era IV).
- **Tecnologías económicas por recurso** (PR #4): en el Centro Urbano se
  investigan tecnologías que aumentan la recolección de cada recurso (Molino,
  Aserradero, Minería de Oro, Cantera y sus versiones avanzadas). Cada tier suma
  un % a la tasa de ese recurso.
- **Velocidad de partida** (PR #4): multiplicador global del ritmo del juego
  (Lenta ×0.7, Normal ×1, Rápida ×1.6).
- **Cola de edificio editable** (PR #4): al seleccionar un edificio se ve su cola
  de unidades como iconos; tocar uno lo cancela y reembolsa su coste.
- **IA mejorada** (PR #4): el enemigo avanza de era, construye casas al acercarse
  al tope de población, levanta castillos (en Difícil/Era III), reparte sus
  aldeanos de forma ponderada y entrena la unidad que puede pagar.
