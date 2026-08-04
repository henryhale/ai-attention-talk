<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref, watch } from 'vue'

type AttentionMode = 'full' | 'sliding' | 'flash' | 'sparse'

const props = defineProps<{
  tokens: string[]
  scores: number[][]
  weights: number[][]
  masks: boolean[][]
  queryIndex: number
  mode: AttentionMode
}>()

const emit = defineEmits<{
  'select-query': [index: number]
}>()

const TILE_SIZE = 3
const flashTile = ref(0)
const matrixRoot = ref<HTMLElement | null>(null)
const tooltip = ref({ visible: false, left: 0, top: 0, text: '' })
let flashTimer: ReturnType<typeof setInterval> | undefined

const matrixStyle = computed(() => ({
  gridTemplateColumns: `50px repeat(${Math.max(1, props.tokens.length)}, minmax(15px, 1fr))`,
  gridTemplateRows: `38px repeat(${Math.max(1, props.tokens.length)}, minmax(14px, 1fr))`,
}))

function compactToken(token: string, max = 7) {
  return token.length > max ? `${token.slice(0, max - 1)}…` : token
}

function isFlashActive(queryIndex: number, keyIndex: number) {
  if (props.mode !== 'flash' || !props.tokens.length) return false
  const tilesPerAxis = Math.ceil(props.tokens.length / TILE_SIZE)
  const tileRow = Math.floor(flashTile.value / tilesPerAxis) % tilesPerAxis
  const tileColumn = flashTile.value % tilesPerAxis
  return Math.floor(queryIndex / TILE_SIZE) === tileRow
    && Math.floor(keyIndex / TILE_SIZE) === tileColumn
}

function restartFlashTimer() {
  if (flashTimer) clearInterval(flashTimer)
  flashTimer = undefined
  flashTile.value = 0

  const reducedMotion = typeof window !== 'undefined'
    && window.matchMedia('(prefers-reduced-motion: reduce)').matches
  if (props.mode !== 'flash' || reducedMotion || !props.tokens.length) return

  flashTimer = setInterval(() => {
    const tilesPerAxis = Math.max(1, Math.ceil(props.tokens.length / TILE_SIZE))
    flashTile.value = (flashTile.value + 1) % (tilesPerAxis * tilesPerAxis)
  }, 620)
}

function positionTooltip(event: PointerEvent) {
  if (!matrixRoot.value) return
  const bounds = matrixRoot.value.getBoundingClientRect()
  tooltip.value.left = Math.min(bounds.width - 138, Math.max(4, event.clientX - bounds.left + 8))
  tooltip.value.top = Math.min(bounds.height - 48, Math.max(4, event.clientY - bounds.top + 8))
}

function showTooltip(event: PointerEvent, queryIndex: number, keyIndex: number) {
  const visible = props.masks[queryIndex]?.[keyIndex] ?? false
  const score = props.scores[queryIndex]?.[keyIndex] ?? 0
  const weight = props.weights[queryIndex]?.[keyIndex] ?? 0
  tooltip.value.text = visible
    ? `${props.tokens[queryIndex]} → ${props.tokens[keyIndex]} · score ${score.toFixed(2)} · weight ${weight.toFixed(3)}`
    : `${props.tokens[queryIndex]} → ${props.tokens[keyIndex]} · masked`
  tooltip.value.visible = true
  positionTooltip(event)
}

function hideTooltip() {
  tooltip.value.visible = false
}

watch(() => [props.mode, props.tokens.length], restartFlashTimer)
onMounted(restartFlashTimer)
onUnmounted(() => {
  if (flashTimer) clearInterval(flashTimer)
})
</script>

<template>
  <section ref="matrixRoot" class="matrix-card">
    <header class="matrix-header">
      <div>
        <p>CONNECTIVITY + INTENSITY</p>
        <h2>See every connection.</h2>
      </div>
      <span>{{ tokens.length }} × {{ tokens.length }}</span>
    </header>

    <div v-if="tokens.length" class="matrix-grid" :style="matrixStyle">
      <span class="matrix-corner">Q \ K</span>
      <span
        v-for="(token, index) in tokens"
        :key="`column-${index}`"
        class="matrix-label column"
        :class="{ active: index === queryIndex }"
        :title="token"
      >
        {{ compactToken(token) }}
      </span>

      <template v-for="(query, rowIndex) in tokens" :key="`row-${rowIndex}`">
        <button
          type="button"
          class="matrix-label row"
          :class="{ active: rowIndex === queryIndex }"
          :title="`Select query ${query}`"
          @click="emit('select-query', rowIndex)"
        >
          {{ compactToken(query) }}
        </button>
        <button
          v-for="(_, keyIndex) in tokens"
          :key="`cell-${rowIndex}-${keyIndex}`"
          type="button"
          class="attention-cell"
          :class="{
            masked: !masks[rowIndex]?.[keyIndex],
            'query-row': rowIndex === queryIndex,
            'flash-active': isFlashActive(rowIndex, keyIndex),
          }"
          :style="{ '--weight': weights[rowIndex]?.[keyIndex] ?? 0 }"
          :aria-label="`${query} attends to ${tokens[keyIndex]} with weight ${(weights[rowIndex]?.[keyIndex] ?? 0).toFixed(3)}`"
          @click="emit('select-query', rowIndex)"
          @pointerenter="showTooltip($event, rowIndex, keyIndex)"
          @pointermove="positionTooltip"
          @pointerleave="hideTooltip"
          @blur="hideTooltip"
        ></button>
      </template>
    </div>

    <div v-else class="matrix-empty">Type a sentence to build the matrix.</div>

    <footer class="matrix-legend">
      <span>masked / low</span>
      <i></i>
      <span>high weight</span>
      <b v-if="mode === 'flash'">tiled · exact</b>
    </footer>

    <div
      class="matrix-tooltip"
      :class="{ visible: tooltip.visible }"
      :style="{ left: `${tooltip.left}px`, top: `${tooltip.top}px` }"
      role="status"
      aria-live="polite"
    >
      {{ tooltip.text }}
    </div>
  </section>
</template>

<style scoped>
.matrix-card {
  position: relative;
  display: flex;
  flex-direction: column;
  min-width: 0;
  min-height: 0;
  padding: 14px 15px 12px;
  overflow: hidden;
  background:
    radial-gradient(circle at 90% 3%, rgba(119, 217, 255, 0.08), transparent 34%),
    rgba(255, 255, 255, 0.026);
  border: 1px solid #292f2a;
  border-radius: 11px;
}

.matrix-header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 12px;
  margin-bottom: 7px;
}

.matrix-header p {
  margin: 0 0 4px;
  color: #7e877e;
  font-size: 7px;
  font-weight: 750;
  letter-spacing: 0.16em;
}

.matrix-header h2 {
  margin: 0;
  color: var(--ink);
  font-size: 15px;
  font-weight: 620;
  letter-spacing: -0.025em;
}

.matrix-header > span {
  padding: 5px 7px;
  color: var(--accent);
  background: rgba(119, 217, 255, 0.055);
  border: 1px solid rgba(119, 217, 255, 0.22);
  border-radius: 999px;
  font: 700 7px/1 var(--deck-mono);
}

.matrix-grid {
  flex: 1 1 auto;
  display: grid;
  gap: 3px;
  min-height: 0;
  align-items: stretch;
}

.matrix-corner,
.matrix-label {
  display: grid;
  place-items: center;
  min-width: 0;
  overflow: hidden;
  color: #737c74;
  background: transparent;
  border: 0;
  font: 700 6.5px/1 var(--deck-mono);
  text-overflow: ellipsis;
  white-space: nowrap;
}

.matrix-corner {
  letter-spacing: 0.05em;
}

.matrix-label.column {
  align-items: end;
  padding-bottom: 5px;
  writing-mode: vertical-rl;
  transform: rotate(180deg);
}

.matrix-label.row {
  justify-content: end;
  padding: 0 6px 0 2px;
  cursor: pointer;
}

.matrix-label.active {
  color: var(--accent);
  font-weight: 850;
}

.attention-cell {
  position: relative;
  min-width: 0;
  min-height: 0;
  padding: 0;
  background: rgba(119, 217, 255, calc(0.035 + var(--weight) * 0.86));
  border: 1px solid rgba(119, 217, 255, 0.08);
  border-radius: 3px;
  cursor: pointer;
  transition: transform 0.12s ease, opacity 0.16s ease, background 0.18s ease;
}

.attention-cell:hover,
.attention-cell:focus-visible {
  z-index: 3;
  outline: 1px solid var(--accent);
  outline-offset: 1px;
  transform: scale(1.1);
}

.attention-cell.masked {
  opacity: 0.42;
  background:
    linear-gradient(135deg, transparent 45%, rgba(225, 116, 116, 0.12) 46%, rgba(225, 116, 116, 0.12) 54%, transparent 55%),
    rgba(255, 255, 255, 0.016);
  border-color: #282d29;
}

.attention-cell.query-row {
  box-shadow: 0 0 0 1px rgba(119, 217, 255, 0.42);
}

.attention-cell.flash-active {
  z-index: 2;
  outline: 1px solid #f0a65b;
  outline-offset: 1px;
}

.matrix-empty {
  display: grid;
  flex: 1;
  place-items: center;
  color: #737c74;
  border: 1px dashed #303631;
  border-radius: 8px;
  font-size: 8px;
}

.matrix-legend {
  display: flex;
  align-items: center;
  gap: 7px;
  margin-top: 8px;
  color: #687068;
  font-size: 6.5px;
}

.matrix-legend i {
  width: 65px;
  height: 5px;
  background: linear-gradient(90deg, rgba(119, 217, 255, 0.04), rgba(119, 217, 255, 0.88));
  border-radius: 999px;
}

.matrix-legend b {
  margin-left: auto;
  color: #f0a65b;
  font: 700 6.5px/1 var(--deck-mono);
}

.matrix-tooltip {
  position: absolute;
  z-index: 20;
  max-width: 138px;
  padding: 5px 7px;
  color: #e8ece8;
  background: rgba(20, 24, 21, 0.96);
  border: 1px solid #394039;
  border-radius: 5px;
  font: 600 6.5px/1.35 var(--deck-mono);
  opacity: 0;
  pointer-events: none;
  transform: translateY(2px);
  transition: opacity 0.1s ease, transform 0.1s ease;
}

.matrix-tooltip.visible {
  opacity: 1;
  transform: translateY(0);
}

@media (prefers-reduced-motion: reduce) {
  .attention-cell,
  .matrix-tooltip {
    transition: none;
  }
}
</style>
