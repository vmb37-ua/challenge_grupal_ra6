;;==============================================================
;; Challenge 6 · Ejercicio 11 · Implementa sys_animation_one
;;==============================================================
;;
;; CÓMO TRABAJAR
;;  1. Copia tu código del papel TAL CUAL en cada `◉◉YOUR CODE◉◉`. El resto ya está escrito.
;;  2. Prueba: en main.asm, DEF TEST EQUS "test_ej11" y make. BGB para en cada `ld b, b` (P1, P2).
;;     Compara CPU y MEM con el RESULTADO ESPERADO del final de este fichero.
;;  3. Rellena el resumen del final.
;;
;; DÓNDE ESTÁ
;;   `sys_animation_one` está en src/sys/sys_animation.asm. Allí solo hay un salto: tu código va AQUÍ, en `ej11::`.
;;
;; QUÉ HAY QUE HACER
;;   Distinto al entrenamiento: `sys_animation_one` ya no calcula la imagen: solo decide el `TICK` y salta a `sys_animation_image` (ejercicio 12). Y una entidad con `PHYSICS` que se para no cambia su `TICK`: se queda en el paso en el que se paró.
;;
;;   `DE` = entidad, que tiene `ANIMATION`. `TICK` cuenta los frames que lleva caminando.
;;   1. Una entidad sin `PHYSICS` (el portal) se anima siempre: suma 1 a su `TICK`.
;;   2. Una entidad con `PHYSICS` se anima solo mientras se mueve (`VY` o `VX` distintos de 0): suma 1 a su `TICK`. Si está quieta, no lo cambia.
;;   3. En todos los casos termina saltando a `sys_animation_image`, con `DE` = la entidad.
;;
;; PISTAS
;;   - `ld a, [hl+]` y `or [hl]` dejan en `A` el resultado de `VY | VX`: `Z` si la entidad está quieta.
;;==============================================================
INCLUDE "engine/hardware.inc"
INCLUDE "assets/gfx/gfx.inc"
INCLUDE "man/man_entity.inc"
INCLUDE "man/man_map.inc"
INCLUDE "sys/sys_render.inc"
INCLUDE "engine/ui/hud.inc"
INCLUDE "scn/scn_game.inc"
INCLUDE "assets/levels/levels.inc"

SECTION "Ejercicio 11", ROM0

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;; Animation system, for ONE entity: the TICK. TICK counts the frames it has walked.
;; An entity with PHYSICS only animates while it moves (VY or VX not 0): when it stands
;; still, its TICK does not change and it keeps the image of the step where it stopped.
;; The others (a portal) always animate. Then it picks the image (sys_animation_image).
;; ⚠️ Precondition: the entity has CMP_ANIMATION.
;; 📥 DE = entity
;; ❌ Modifies A, BC, HL
ej11::   ;; = `sys_animation_one`, que está en src/sys/sys_animation.asm
   ld h, d
   ld l, e
   ld a, [hl]                                ;; Fix N1: Sin completar la instrucción
   and CMP_ANIMATION
   jr z, .salir

   ld a, [hl]                                ;; Fix E1: Instrucción errónea
   and CMP_PHYSICS
   jr nz, .fisica

   ld a, l
   add a, 10                                 ;; Fix E2
   ld l, a
   inc [hl]
   jr .salir

   .fisica:
   ld a, e
   add a, 5
   ld l, a
   ld a, 0                                  ;; Fix E4
   cp [hl]
   jr nz, .mov
   inc hl
   cp [hl]
   jr nz, .mov
   jr .salir

   .mov:
   ld l, e
   ld a, l
   add a, 10                                 ;; Fix E2
   ld l, a
   inc [hl]
   .salir:
   call sys_animation_image
   ret ;; ◀━REMOVE RET IF NEEDED

;; ==========================================================
;;  ENSAMBLA: SI (SÍ / NO)
;; === CASOS DE PRUEBA   (test_ej11 en main.asm)
;;  * Comprobar que CPU y MEM coinciden con el valor esperado
;; ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
;; * CASO 1: OK  (OK / Falla)
;;
;;        >>>>> RESULTADO ESPERADO (Comprobar) <<<<<
;; [PREVIO]> Entidad 0 con PHYSICS: VX = −1 (FF), DIR_LEFT, TICK = 7, SHEET = 80, ATTR = 0. Entidad 1, el portal, SIN PHYSICS: DIR_DOWN, TICK = 7, SHEET = 136. Se llama con cada una (DE = su entidad). La imagen la elige la rutina del proyecto.
;; [CPU]
;;    de = C010
;; [MEM] entity_array (32 bytes)
;;         00 01 02 03 04 05 06 07|08 09 0A 0B 0C 0D 0E 0F|
;;   C000  A3 00 00 60 20 00 FF 00|02 50 08 00 00 00 00 00|
;;   C010  A1 00 00 88 00 00 00 00|00 88 08 00 00 00 00 00|
;;  Entidad 0, camina: TICK pasa de 7 a 8. Después, `sys_animation_image` elige la imagen: el bit 2 de 8 vale 0, así que el paso es 0. TILE = 80 + 16 = 96 (60 en hexadecimal) y ATTR = 20 (DIR_LEFT). Entidad 1, el portal: no tiene PHYSICS, así que se anima siempre: TICK = 8 y TILE = 136 + 0 = 136 (88).
;; ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
;; * CASO 2: OK  (OK / Falla)
;;
;;        >>>>> RESULTADO ESPERADO (Comprobar) <<<<<
;; [PREVIO]> Entidad 0 con PHYSICS: VY = VX = 0, DIR_UP, TICK = 13, SHEET = 80, ATTR = $30. DE = entidad 0.
;; [CPU]
;;    de = C000
;; [MEM] entity_array (16 bytes)
;;         00 01 02 03 04 05 06 07|08 09 0A 0B 0C 0D 0E 0F|
;;   C000  A3 00 00 5C 10 00 00 00|01 50 0D 00 00 00 00 00|
;;  Está quieta: TICK sigue en 13 (0D). `sys_animation_image` elige la imagen: el bit 2 de 13 vale 1, así que el paso es 4. TILE = 80 + 8 + 4 = 92 (5C). ATTR pasa de 30 a 10 (mira hacia arriba).
;; ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
;; ==========================================================
;; ==========================================================
;; === FIXES
;;  Tipo: Cant.
;;  *  N:    1     (no existe)
;;  *  E:    3     (errónea)
;;  *  S:    __     (sintaxis)
;;  *  F:    __     (falta algo)
;;
;; EXPLICACIONES ADICIONALES DE FIXES (si es necesario)
;;  Varios fixes no están contabilizados porque vienen de un mismo error propagado o son esencialmente iguales.
;;
