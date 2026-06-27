# filemap.md — Mapa de archivos

Estructura del repositorio. Actualizar cuando se añadan archivos o secciones de
código nuevas (ver normas en `CLAUDE.md`).

## Archivos del repositorio

| Archivo | Propósito |
|---|---|
| `index.html` | **El juego completo**: HTML + CSS + JavaScript embebido. Es el único archivo necesario para jugar. |
| `DESIGN.md` | Documento de diseño (GDD) detallado del juego. |
| `CLAUDE.md` | Guía del proyecto, flujo de trabajo, normas y listado de funcionalidades. |
| `filemap.md` | Este archivo: mapa de archivos y estructura del código. |
| `progress.md` | Bitácora cronológica de avance. |
| `README.md` | Cómo jugar, controles y cómo desplegar. |

## Estructura interna de `index.html`

El archivo se organiza en estas secciones (en orden de aparición):

### `<head>` / `<style>`
- Estilos de la UI: barra superior (`#topbar`), panel de acciones (`#actions`),
  botones flotantes (`#util`: inactivos / centrar / pausa), overlays de inicio y
  fin, avisos (`#hint`), botones (`.btn`).

### `<body>` (marcado)
- `#game` (canvas), `#topbar` (recursos), `#util` (botones inactivos/centrar/
  pausa), `#hint`, `#actions` (panel inferior), `#startScreen`, `#endScreen`.

### `<script>` — lógica del juego
1. **Configuración / definiciones de datos**: constantes del mundo (`WORLD`,
   `POP_MAX`, `BONUS`, `AGGRO`, `MAX_AGE`), `STRONG_VS` (cuadrilátero), `UNIT`
   (unidades), `BLD` (edificios: incluye Casa y Castillo, con `pop`/`reqAge` y la
   Torre/Castillo con `atk/range/cd`), `AGES` (cuatro edades y sus costes),
   `ECON` (tecnologías económicas por recurso), `UPG` (mejoras de Herrería),
   `RES` (recursos), presets de partida (`RES_PRESETS`, `SPEED_PRESETS`, `MAPS`)
   y `gameConfig`.
2. **Estado global**: `cam`, `entities`, `selection`, `player`, `enemy`
   (con `mods.resMult`), `gameSpeed`, `mapTheme`, `terrain`, flags
   (`running`, `paused`, `gameOver`, `difficulty`).
3. **Utilidades**: `dist`, `clamp`, `find`, `radiusOf`, recursos/coste
   (`canAfford`, `pay`, `costStr`, `popCount`, `popCap`), `hasBuilding`,
   `countBuildings`, `prodSpeed` (bono de producción por nº de edificios).
4. **Creación de entidades**: `makeUnit`, `makeBuilding`, `makeResource`.
5. **Inicialización y mapas**: `startGame` (usa `gameConfig`),
   `spawnResourceCluster`, `generateMap` (genera recursos/terreno por tema) y
   `onObstacle` (colisión con río/riscos).
6. **Cámara**: `viewW/viewH`, `centerOn`, `clampCam`, conversiones
   `worldToScreen` / `screenToWorld`, `resize`.
7. **Economía / entrenamiento**: `queueUnit`, `countQueued`, `tryAdvanceAge`
   (multi-era), `buyUpgrade`, `buyEcon`/`nextEcon` (tecnologías de recursos),
   `cancelQueued` (cancela y reembolsa unidades de la cola), stats efectivas
   (`unitAtk`, `unitRange`, `unitArmor`, `gatherRate` por recurso), combate
   (`damage`).
8. **Lógica de unidades / IA**: `nearestEnemy`, `nearestResourceOfType`,
   `separate`.
9. **Bucle principal**: `loop`, `update` (unidades, edificios + **torres**,
   muertes, fin de partida), `stepToward`, `spawnTrained`, `removeEntity`,
   `enemyAI`.
10. **Render**: `render`, `drawTerrain` (río/riscos), `drawGround`, `onScreen`,
    `drawResource`, `drawBuilding` (incluye disparo y radio de torre/castillo),
    `drawUnit` (incluye insignia de recurso y de inactivo), `drawHpBar`,
    `roundRect`.
11. **Entrada táctil**: objeto `input`, manejadores `pointerdown/move/up/cancel`,
    `wheel`, teclado; `pickAt`, `handleTap`, `handleDoubleTap`,
    `finishBoxSelect`, `selectedUnits`, `selectedBuilding`; colocación
    (`placementValid`, `tryPlaceBuilding`).
12. **UI: panel de acciones**: `btnEl`, `clearActions` (limpia botones y filas de
    cola), `updateActionPanel` (multiplicador de producción, fila de cola
    cancelable y botón Deseleccionar), `buildingButtons` (incluye avance de era,
    tecnologías económicas y construcción de Casa/Castillo), `deselectAll`.
13. **Barra superior y utilidades de UI**: `updateTopbar` (incluye contador de
    inactivos y **tasa de producción por recurso**), `idleVillagers`,
    `selectNextIdle`, `showHint`, `endGame`.
14. **Menú principal y arranque**: `MAP_DESC`, `refreshMenu` y listeners de los
    grupos de opciones del menú (mapa, recursos, velocidad, IA, posición) +
    botón Empezar; listeners de fin/centrar/pausa/inactivos; bloqueo de gestos
    del navegador; refresco periódico del panel; `resize()` +
    `requestAnimationFrame(loop)`.
