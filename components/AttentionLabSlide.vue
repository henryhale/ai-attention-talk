<script setup lang="ts">
import { computed, ref, watch } from 'vue'

type AttentionMode = 'full' | 'sliding' | 'flash' | 'sparse'

const MAX_TOKENS = 10
const DIMS = 12
const EPS = 1e-9

const modes: Array<{ value: AttentionMode; label: string }> = [
  { value: 'full', label: 'Full attention' },
  { value: 'sliding', label: 'Sliding window' },
  { value: 'flash', label: 'FlashAttention' },
  { value: 'sparse', label: 'Sparse top-k' },
]

const modeInfo: Record<AttentionMode, {
  complexity: string
  description: string
  resourceHref: string
  resourceLabel: string
}> = {
  full: {
    complexity: 'O(n²d)',
    description: 'Every token can reach every visible token.',
    resourceHref: 'https://arxiv.org/abs/1706.03762',
    resourceLabel: 'Vaswani et al. · Attention Is All You Need (2017)',
  },
  sliding: {
    complexity: 'O(nw)',
    description: 'Each token sees only nearby neighbors.',
    resourceHref: 'https://arxiv.org/abs/2004.05150',
    resourceLabel: 'Beltagy et al. · Longformer (2020)',
  },
  flash: {
    complexity: 'exact O(n²)',
    description: 'Same answer. Less memory movement.',
    resourceHref: 'https://arxiv.org/abs/2205.14135',
    resourceLabel: 'Dao et al. · FlashAttention (2022)',
  },
  sparse: {
    complexity: 'O(nk)',
    description: 'Keep only the strongest three routes.',
    resourceHref: 'https://arxiv.org/abs/1904.10509',
    resourceLabel: 'Child et al. · Generating Long Sequences with Sparse Transformers (2019)',
  },
}

const text = ref('The small robot crossed the bridge because it was strong')
const mode = ref<AttentionMode>('full')
const causal = ref(true)

function tokenize(value: string) {
  return (value.match(/[\p{L}\p{N}]+(?:['’\-][\p{L}\p{N}]+)*|[^\s\p{L}\p{N}]/gu) || [])
    .slice(0, MAX_TOKENS)
}

function hashString(value: string, seed = 2166136261) {
  let hash = seed >>> 0
  for (const character of value) {
    hash ^= character.codePointAt(0) ?? 0
    hash = Math.imul(hash, 16777619)
  }
  return hash >>> 0
}

function seededValue(seed: number) {
  let value = seed >>> 0
  value ^= value << 13
  value ^= value >>> 17
  value ^= value << 5
  return ((value >>> 0) / 4294967295) * 2 - 1
}

function normalize(vector: number[]) {
  const norm = Math.sqrt(vector.reduce((sum, value) => sum + value * value, 0)) || 1
  return vector.map(value => value / norm)
}

function tokenFeatures(token: string, index: number, total: number, kind: 'q' | 'k') {
  const lower = token.toLocaleLowerCase()
  const typeSeed = kind === 'q' ? 911 : 353
  const base = hashString(lower, 2166136261 ^ typeSeed)
  const vector = Array.from({ length: DIMS }, (_, dimension) => {
    const unigram = seededValue(base + Math.imul(dimension + 1, 2654435761))
    const charSeed = hashString(`${lower}:${dimension % Math.max(1, lower.length)}`, base ^ 374761393)
    const character = seededValue(charSeed)
    const position = Math.sin((index + 1) / Math.pow(10000, (2 * Math.floor(dimension / 2)) / DIMS))
    const relative = total > 1 ? (index / (total - 1)) * 2 - 1 : 0
    const lexical = /\p{L}/u.test(lower) ? Math.min(lower.length / 10, 1) : -0.35
    return unigram * 0.58 + character * 0.26 + position * 0.12 + relative * 0.07 + lexical * 0.08
  })
  return normalize(vector)
}

function dot(left: number[], right: number[]) {
  return left.reduce((sum, value, index) => sum + value * (right[index] ?? 0), 0)
}

function softmax(scores: number[], mask: boolean[]) {
  const visible = scores.map((score, index) => mask[index] ? score : -Infinity)
  const finite = visible.filter(Number.isFinite)
  if (!finite.length) return scores.map(() => 0)
  const max = Math.max(...finite)
  const exponents = visible.map(score => Number.isFinite(score)
    ? Math.exp(Math.max(-30, Math.min(30, score - max)))
    : 0)
  const total = exponents.reduce((sum, value) => sum + value, 0) || 1
  return exponents.map(value => value / total)
}

const tokens = computed(() => tokenize(text.value))
const rawTokenCount = computed(() => tokenize(text.value).length)
const inputWasTruncated = computed(() => {
  const allTokens = text.value.match(/[\p{L}\p{N}]+(?:['’\-][\p{L}\p{N}]+)*|[^\s\p{L}\p{N}]/gu) || []
  return allTokens.length > MAX_TOKENS
})
const queryIndex = ref(Math.floor(tokens.value.length / 2))

watch(tokens, (next, previous) => {
  if (!next.length) {
    queryIndex.value = 0
    return
  }
  if (!previous.length) {
    queryIndex.value = Math.floor(next.length / 2)
    return
  }
  queryIndex.value = Math.min(queryIndex.value, next.length - 1)
})

const attention = computed(() => {
  const currentTokens = tokens.value
  const count = currentTokens.length
  const queries = currentTokens.map((token, index) => tokenFeatures(token, index, count, 'q'))
  const keys = currentTokens.map((token, index) => tokenFeatures(token, index, count, 'k'))
  const scores: number[][] = []
  const masks: boolean[][] = []
  const weights: number[][] = []

  for (let rowIndex = 0; rowIndex < count; rowIndex++) {
    const rowScores = keys.map((key, keyIndex) => {
      const queryToken = currentTokens[rowIndex].toLocaleLowerCase()
      const keyToken = currentTokens[keyIndex].toLocaleLowerCase()
      let score = dot(queries[rowIndex], key) * 2.4
      if (queryToken === keyToken) score += 1.05
      if (queryToken.length > 2 && keyToken.length > 2 && queryToken[0] === keyToken[0]) score += 0.14
      if (/^[.,!?;:]$/.test(keyToken)) score -= 0.2
      return score
    })

    let rowMask = rowScores.map((_, keyIndex) => !causal.value || keyIndex <= rowIndex)

    if (mode.value === 'sliding') {
      rowMask = rowMask.map((visible, keyIndex) => visible && Math.abs(keyIndex - rowIndex) <= 2)
    }

    if (mode.value === 'sparse') {
      const kept = new Set(rowScores
        .map((score, keyIndex) => ({ score, keyIndex }))
        .filter(item => rowMask[item.keyIndex])
        .sort((left, right) => right.score - left.score)
        .slice(0, 3)
        .map(item => item.keyIndex))
      rowMask = rowMask.map((visible, keyIndex) => visible && kept.has(keyIndex))
    }

    scores.push(rowScores)
    masks.push(rowMask)
    weights.push(softmax(rowScores, rowMask))
  }

  return { scores, masks, weights }
})

const routeItems = computed(() => {
  const row = attention.value.weights[queryIndex.value] || []
  return row
    .map((weight, index) => ({ token: tokens.value[index], weight, index }))
    .filter(item => item.weight > EPS)
    .sort((left, right) => right.weight - left.weight)
    .slice(0, 4)
})

const activeInfo = computed(() => modeInfo[mode.value])
</script>

<template>
  <div class="slide-shell lab-slide">
    <header class="lab-header">
      <div class="lab-title">
        <span class="step-index">06</span>
        <h1>Attention makes a <span>visible choice.</span></h1>
      </div>
      <span class="era-chip">INTERACTIVE</span>
    </header>

    <main class="lab-workspace">
      <div class="lab-left">
        <section class="lab-card input-card">
          <header class="card-heading compact">
            <div><p>LIVE INPUT</p><h2>Change the words</h2></div>
            <span>{{ rawTokenCount }}{{ inputWasTruncated ? '+' : '' }} / {{ MAX_TOKENS }}</span>
          </header>
          <textarea v-model="text" rows="2" aria-label="Text to visualize" spellcheck="true"></textarea>
          <div class="token-strip" aria-label="Select a query token">
            <button
              v-for="(token, index) in tokens"
              :key="`${token}-${index}`"
              type="button"
              :class="{ selected: index === queryIndex }"
              :aria-pressed="index === queryIndex"
              :title="`Use ${token} as the query`"
              @click.stop="queryIndex = index"
            >
              {{ token }}<small>{{ index + 1 }}</small>
            </button>
          </div>
        </section>

        <div class="control-row">
          <label class="lab-card select-card">
            <span>ATTENTION TYPE</span>
            <span class="select-wrap">
              <select v-model="mode" aria-label="Attention type">
                <option v-for="option in modes" :key="option.value" :value="option.value">
                  {{ option.label }}
                </option>
              </select>
              <carbon-chevron-down aria-hidden="true" />
            </span>
            <small>{{ activeInfo.complexity }}</small>
          </label>

          <div class="lab-card mask-card">
            <span><b>CAUSAL MASK</b><small>Hide future tokens</small></span>
            <label class="switch">
              <input v-model="causal" type="checkbox" aria-label="Toggle causal mask">
              <i aria-hidden="true"></i>
            </label>
          </div>
        </div>

        <section class="lab-card route-card">
          <header class="card-heading route-heading">
            <div><p>ONE TOKEN'S ROUTE</p><h2>Follow its strongest links</h2></div>
          </header>
          <p class="mode-description">{{ activeInfo.description }}</p>
          <div v-if="tokens.length" class="route-content">
            <div class="query-node"><small>QUERY</small><strong>{{ tokens[queryIndex] }}</strong></div>
            <div class="route-list">
              <div v-for="item in routeItems" :key="item.index" class="route-item">
                <span>{{ item.token }}</span>
                <i><b :style="{ width: `${item.weight * 100}%` }"></b></i>
                <strong>{{ item.weight.toFixed(2) }}</strong>
              </div>
            </div>
          </div>
          <div v-else class="route-empty">Type text to create attention routes.</div>
        </section>
      </div>

      <ConnectivityIntensity
        :tokens="tokens"
        :scores="attention.scores"
        :weights="attention.weights"
        :masks="attention.masks"
        :query-index="queryIndex"
        :mode="mode"
        @select-query="queryIndex = $event"
      />
    </main>

    <footer class="lab-footer">
      <span>ATTENTION LAB · DETERMINISTIC TOY EMBEDDINGS</span>
      <span class="slide-progress">11 / 20</span>
      <ResourceLink :href="activeInfo.resourceHref" :label="activeInfo.resourceLabel" />
    </footer>
  </div>
</template>

<style scoped>
.lab-slide {
  width: 100%;
  max-width: none;
  padding: 22px 34px 17px;
}

.lab-header {
  flex: 0 0 43px;
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  border-bottom: 1px solid #292e29;
}

.lab-title {
  display: flex;
  align-items: baseline;
  gap: 12px;
}

.lab-title h1 {
  font-size: 27px;
  line-height: 1;
  letter-spacing: -0.04em;
}

.lab-title h1 span {
  color: var(--accent);
}

.lab-workspace {
  flex: 1 1 auto;
  display: grid;
  grid-template-columns: minmax(0, 46fr) minmax(0, 54fr);
  gap: 11px;
  min-height: 0;
  padding: 10px 0 9px;
}

.lab-left {
  display: grid;
  grid-template-rows: auto auto minmax(0, 1fr);
  gap: 7px;
  min-width: 0;
  min-height: 0;
}

.lab-card {
  min-width: 0;
  padding: 10px 11px;
  background: rgba(255, 255, 255, 0.022);
  border: 1px solid #292f2a;
  border-radius: 9px;
}

.card-heading {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 8px;
}

.card-heading p {
  margin: 0 0 3px;
  color: #7e877e;
  font-size: 6.5px;
  font-weight: 750;
  letter-spacing: 0.16em;
}

.card-heading h2 {
  margin: 0;
  color: var(--ink);
  font-size: 11px;
  font-weight: 620;
  letter-spacing: -0.02em;
}

.card-heading > span {
  color: #737c74;
  font: 700 6.5px/1.2 var(--deck-mono);
}

.input-card {
  padding-bottom: 9px;
}

.input-card textarea {
  display: block;
  width: 100%;
  height: 39px;
  margin-top: 7px;
  padding: 6px 8px;
  resize: none;
  color: #cdd2cd;
  background: #0c0f0d;
  border: 1px solid #303631;
  border-radius: 6px;
  outline: none;
  font: 8px/1.4 var(--deck-mono);
}

.input-card textarea:focus {
  border-color: rgba(119, 217, 255, 0.48);
  box-shadow: 0 0 0 2px rgba(119, 217, 255, 0.07);
}

.token-strip {
  display: flex;
  flex-wrap: wrap;
  gap: 3px;
  max-height: 31px;
  margin-top: 6px;
  overflow: hidden;
}

.token-strip button {
  display: flex;
  align-items: center;
  gap: 3px;
  min-width: 0;
  padding: 4px 5px;
  overflow: hidden;
  color: #8f9790;
  background: #111512;
  border: 1px solid #2c322d;
  border-radius: 4px;
  font: 650 6.5px/1 var(--deck-mono);
  text-overflow: ellipsis;
  white-space: nowrap;
  cursor: pointer;
}

.token-strip button.selected {
  color: #0b0e0c;
  background: var(--accent);
  border-color: var(--accent);
}

.token-strip button:focus-visible {
  outline: 1px solid var(--accent);
  outline-offset: 1px;
}

.token-strip small {
  opacity: 0.5;
  font-size: 5px;
}

.control-row {
  display: grid;
  grid-template-columns: 1.08fr 0.92fr;
  gap: 7px;
}

.select-card {
  position: relative;
  display: grid;
  grid-template-columns: 1fr auto;
  align-items: center;
  gap: 3px 7px;
  padding: 7px 9px;
}

.select-card > span:first-child,
.mask-card b {
  color: #7e877e;
  font-size: 6px;
  font-weight: 750;
  letter-spacing: 0.12em;
}

.select-wrap {
  position: relative;
  grid-column: 1;
}

.select-wrap select {
  width: 100%;
  padding: 3px 17px 3px 0;
  color: var(--ink);
  background: transparent;
  border: 0;
  outline: none;
  appearance: none;
  font-size: 8px;
  font-weight: 620;
  cursor: pointer;
}

.select-wrap svg {
  position: absolute;
  top: 50%;
  right: 0;
  width: 9px;
  color: var(--accent);
  pointer-events: none;
  transform: translateY(-50%);
}

.select-card > small {
  grid-column: 2;
  grid-row: 1 / 3;
  color: var(--accent);
  font: 700 6px/1 var(--deck-mono);
}

.mask-card {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  padding: 7px 9px;
}

.mask-card > span {
  display: grid;
  gap: 4px;
}

.mask-card small {
  color: #808881;
  font-size: 6.5px;
}

.switch {
  position: relative;
  display: inline-flex;
  flex: 0 0 auto;
  width: 31px;
  height: 17px;
}

.switch input {
  position: absolute;
  opacity: 0;
}

.switch i {
  position: absolute;
  inset: 0;
  background: #343a35;
  border-radius: 999px;
  cursor: pointer;
  transition: background 0.15s ease;
}

.switch i::after {
  content: '';
  position: absolute;
  top: 3px;
  left: 3px;
  width: 11px;
  height: 11px;
  background: #f1f3ef;
  border-radius: 50%;
  transition: transform 0.15s ease;
}

.switch input:checked + i {
  background: var(--accent);
}

.switch input:checked + i::after {
  transform: translateX(14px);
}

.switch input:focus-visible + i {
  outline: 1px solid var(--accent);
  outline-offset: 2px;
}

.route-card {
  display: flex;
  flex-direction: column;
  min-height: 0;
  padding: 9px 11px;
}

.mode-description {
  margin: 5px 0 6px;
  color: #737c74;
  font-size: 6.5px;
  line-height: 1.3;
}

.route-content {
  flex: 1;
  display: grid;
  grid-template-columns: 58px 1fr;
  align-items: center;
  gap: 9px;
  min-height: 0;
}

.query-node {
  display: grid;
  place-items: center;
  align-self: stretch;
  min-height: 47px;
  padding: 5px;
  overflow: hidden;
  background: #101411;
  border: 1px solid rgba(119, 217, 255, 0.25);
  border-radius: 6px;
  text-align: center;
}

.query-node small {
  color: #6e776f;
  font-size: 5.5px;
  letter-spacing: 0.1em;
}

.query-node strong {
  max-width: 48px;
  overflow: hidden;
  color: var(--accent);
  font: 700 8px/1 var(--deck-mono);
  text-overflow: ellipsis;
  white-space: nowrap;
}

.route-list {
  display: grid;
  gap: 4px;
}

.route-item {
  display: grid;
  grid-template-columns: 44px 1fr 22px;
  align-items: center;
  gap: 5px;
  color: #949c95;
  font: 650 6px/1 var(--deck-mono);
}

.route-item > span {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.route-item > i {
  display: block;
  height: 4px;
  overflow: hidden;
  background: #262c27;
  border-radius: 999px;
}

.route-item > i b {
  display: block;
  height: 100%;
  background: var(--accent);
  border-radius: inherit;
  transition: width 0.2s ease;
}

.route-item > strong {
  color: #cbd0cc;
  font-weight: 650;
  text-align: right;
}

.route-empty {
  display: grid;
  flex: 1;
  place-items: center;
  color: #737c74;
  font-size: 7px;
}

.lab-footer {
  flex: 0 0 22px;
  display: grid;
  grid-template-columns: 1fr auto 1fr;
  align-items: end;
  gap: 12px;
  padding-top: 7px;
  color: #666e67;
  border-top: 1px solid #292e29;
  font-size: 6.5px;
  font-weight: 650;
  letter-spacing: 0.1em;
}

.lab-footer > :last-child {
  justify-self: end;
}

@media (prefers-reduced-motion: reduce) {
  .switch i,
  .switch i::after,
  .route-item > i b {
    transition: none;
  }
}
</style>
