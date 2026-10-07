# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Proyecto

Tetris en JavaScript vanilla + HTML5 Canvas. Sin dependencias, sin `package.json`, sin build, sin linter ni tests. El idioma del proyecto (UI, README, comentarios) es español.

## Ejecutar

Abrir `index.html` directamente en el navegador, o servirlo estáticamente desde la raíz del repo:

```bash
python -m http.server 8000   # luego http://localhost:8000
```

No hay suite de pruebas: los cambios se verifican jugando en el navegador.

## Arquitectura

Toda la lógica vive en `game.js` (script clásico con `'use strict'`, no módulo ES), cargado al final de `index.html`. Depende de IDs del DOM definidos en `index.html` (`board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`); renombrar uno exige cambiar ambos archivos.

- **Estado global mutable**: `board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc. son `let` a nivel de módulo. `init()` los reinicia todos y es también el handler del botón de reinicio; cualquier variable de estado nueva debe resetearse ahí.
- **Representación**: `board` es una matriz `ROWS × COLS` de enteros; `0` = vacío, `1–7` = tipo de pieza. Ese mismo entero es el índice en `COLORS` y el valor que guardan las celdas de las matrices de `PIECES`, así que tipo, forma y color están acoplados por índice (el índice 0 de ambos arrays es `null`).
- **Pieza**: `{ type, shape, x, y }`, donde `shape` es una copia de la matriz de `PIECES` que se reemplaza al rotar (`rotateCW` + kicks horizontales en `tryRotate`). `collide(shape, ox, oy)` es la única comprobación de límites/solapamiento y la usan movimiento, rotación, caída, ghost y spawn.
- **Ciclo de vida**: `lockPiece()` → `merge()` → `clearLines()` (actualiza líneas, puntuación, nivel y `dropInterval`) → `spawn()` (que dispara `endGame()` si la nueva pieza colisiona).
- **Bucle**: `loop()` con `requestAnimationFrame` acumula `dt` en `dropAccum` y aplica gravedad cuando supera `dropInterval`; redibuja todo el tablero cada frame. Pausa y game over se implementan con `cancelAnimationFrame(animId)`, y un mismo `#overlay` sirve para ambos estados.
- **Render**: `drawBlock()` es compartido entre el tablero principal y el canvas de "siguiente pieza" (recibe el contexto y el tamaño de celda).

Si se cambian `COLS`, `ROWS` o `BLOCK`, hay que ajustar `width`/`height` de `<canvas id="board">` en `index.html` (`COLS×BLOCK` × `ROWS×BLOCK`). El README documenta controles, puntuación y constantes ajustables; mantenerlo sincronizado al cambiar mecánicas.
