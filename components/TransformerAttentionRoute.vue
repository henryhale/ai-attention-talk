<script setup lang="ts">
import { computed, ref } from 'vue'

const tokens = ['The', 'cat', 'sat', 'on', 'the']
const positions = [36, 108, 180, 252, 324]
const attentionWeights = [
  [0.52, 0.14, 0.10, 0.06, 0.18],
  [0.13, 0.35, 0.30, 0.08, 0.14],
  [0.10, 0.31, 0.34, 0.15, 0.10],
  [0.09, 0.15, 0.26, 0.35, 0.15],
  [0.12, 0.31, 0.28, 0.08, 0.21],
]

const queryIndex = ref(4)
const weights = computed(() => attentionWeights[queryIndex.value])
const strongestKeys = computed(() => new Set(weights.value
  .map((weight, index) => ({ weight, index }))
  .sort((left, right) => right.weight - left.weight)
  .slice(0, 3)
  .map(item => item.index)))

function selectQuery(index: number) {
  queryIndex.value = index
}

function routePath(keyIndex: number) {
  const queryX = positions[queryIndex.value]
  const keyX = positions[keyIndex]
  const bend = Math.max(32, Math.abs(queryX - keyX) * 0.3)
  const direction = keyX < queryX ? -1 : 1
  return `M ${queryX} 121 C ${queryX + direction * bend} 101, ${keyX - direction * bend} 79, ${keyX} 52`
}

function labelX(keyIndex: number) {
  return (positions[queryIndex.value] + positions[keyIndex]) / 2
}

function labelY(keyIndex: number) {
  return 82 + Math.abs(positions[queryIndex.value] - positions[keyIndex]) * 0.025
}
</script>

<template>
  <section class="example-panel attention-route-example">
    <div class="example-label">SELF-ATTENTION · SELECT A QUERY TOKEN</div>

    <svg
      class="route-stage"
      viewBox="0 0 360 150"
      role="img"
      :aria-label="`Attention routes from query ${tokens[queryIndex]}`"
    >
      <text x="5" y="9" class="stage-caption">KEYS / VALUES</text>

      <path
        v-for="(_, keyIndex) in tokens"
        :key="`route-${keyIndex}`"
        :d="routePath(keyIndex)"
        class="attention-link"
        :style="{
          opacity: Math.min(0.96, 0.2 + weights[keyIndex] * 2.2),
          strokeWidth: 1 + weights[keyIndex] * 9,
        }"
      />

      <text
        v-for="(_, keyIndex) in tokens"
        v-show="strongestKeys.has(keyIndex)"
        :key="`weight-${keyIndex}`"
        :x="labelX(keyIndex)"
        :y="labelY(keyIndex)"
        text-anchor="middle"
        class="weight-label"
      >
        {{ Math.round(weights[keyIndex] * 100) }}%
      </text>

      <g
        v-for="(token, index) in tokens"
        :key="`token-${index}`"
        class="stage-token"
        :class="{ selected: index === queryIndex }"
        role="button"
        tabindex="0"
        :aria-label="`Use ${token} as the query token`"
        @click.stop="selectQuery(index)"
        @keydown.enter="selectQuery(index)"
        @keydown.space.prevent="selectQuery(index)"
      >
        <rect :x="positions[index] - 25" y="25" width="50" height="27" rx="7" />
        <text :x="positions[index]" y="42" text-anchor="middle">{{ token }}</text>
      </g>

      <text x="5" y="142" class="stage-caption">CURRENT QUERY</text>
      <circle :cx="positions[queryIndex]" cy="121" r="25" class="query-ring" />
      <rect :x="positions[queryIndex] - 31" y="108" width="62" height="27" rx="8" class="query-core" />
      <text :x="positions[queryIndex]" y="125" text-anchor="middle" class="query-label">
        {{ tokens[queryIndex] }}
      </text>
    </svg>

    <p class="example-note">
      Click a token. Wider means <b>stronger</b>.
    </p>

    <p v-click class="route-principle">
      <span><b>Queries</b> find useful keys.</span>
      <span><b>Values</b> carry the context.</span>
    </p>
  </section>
</template>

<style scoped>
.attention-route-example {
  display: flex;
  flex-direction: column;
}

.route-stage {
  display: block;
  width: 100%;
  height: 168px;
  margin-top: -4px;
  overflow: visible;
}

.attention-link {
  fill: none;
  stroke: var(--accent);
  stroke-linecap: round;
  stroke-dasharray: 7 9;
  vector-effect: non-scaling-stroke;
  animation: route-flow 1.3s linear infinite;
  transition: opacity 0.2s ease, stroke-width 0.2s ease;
}

@keyframes route-flow {
  to { stroke-dashoffset: -32; }
}

.stage-caption {
  fill: #6f7870;
  font: 700 6.5px var(--deck-mono);
  letter-spacing: 0.09em;
}

.stage-token {
  cursor: pointer;
  outline: none;
}

.stage-token rect {
  fill: #111512;
  stroke: #394039;
  transition: fill 0.16s ease, stroke 0.16s ease, transform 0.16s ease;
}

.stage-token text {
  fill: #aeb5af;
  font: 700 7.5px var(--deck-mono);
  pointer-events: none;
}

.stage-token:hover rect,
.stage-token:focus-visible rect {
  stroke: var(--accent);
  transform: translateY(-2px);
}

.stage-token.selected rect {
  fill: var(--accent);
  stroke: var(--accent);
}

.stage-token.selected text {
  fill: #0b0e0c;
}

.query-ring {
  fill: rgba(119, 217, 255, 0.08);
  stroke: rgba(119, 217, 255, 0.32);
}

.query-core {
  fill: #eef1eb;
  stroke: rgba(119, 217, 255, 0.42);
}

.query-label {
  fill: #0b0e0c;
  font: 800 8px var(--deck-mono);
}

.weight-label {
  fill: #cfd5cf;
  font: 750 6.5px var(--deck-mono);
  paint-order: stroke;
  stroke: #121613;
  stroke-linejoin: round;
  stroke-width: 3px;
}

.route-principle {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 7px;
  margin: 4px 0 0;
}

.route-principle span {
  padding: 8px 9px;
  color: #8f999e;
  background: rgba(255, 255, 255, 0.022);
  border: 1px solid #293137;
  border-radius: 6px;
  font-size: 7px;
  line-height: 1.35;
}

.route-principle b {
  color: var(--accent);
}

@media (prefers-reduced-motion: reduce) {
  .attention-link {
    animation: none;
  }

  .stage-token rect,
  .attention-link {
    transition: none;
  }
}
</style>
