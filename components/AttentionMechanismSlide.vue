<script setup lang="ts">
import { computed, ref } from 'vue'

type Mechanism = 'full' | 'flash' | 'sliding' | 'dsa' | 'kda' | 'qwen'

const props = defineProps<{ mechanism: Mechanism }>()

const queryIndex = ref(7)
const windowSize = ref(3)
const flashTile = ref(0)
const kdaStep = ref(0)
const causal = ref(true)

const tokens = ['The', 'small', 'robot', 'crossed', 'the', 'bridge', 'because', 'it']
const kdaTokens = ['Maya', 'learned', 'Swahili', 'at', 'home']

const configs = {
  full: {
    slide: 11,
    eyebrow: 'DENSE CONNECTIVITY',
    era: 'TRANSFORMER',
    title: 'Full attention keeps every past token reachable.',
    lead: 'Each query scores every visible key. Nothing is removed before relevance is learned.',
    metric: 'O(n²) connections',
    takeaway: 'Maximum interaction; cost grows with every pair.',
    source: 'https://arxiv.org/abs/1706.03762',
    sourceLabel: 'Vaswani et al. · Attention Is All You Need',
  },
  flash: {
    slide: 12,
    eyebrow: 'EXACT, IO-AWARE',
    era: 'FLASHATTENTION',
    title: 'FlashAttention changes the route through memory.',
    lead: 'The answer and the connections stay dense. Small tiles avoid repeatedly moving the full matrix to slow memory.',
    metric: 'Same exact attention',
    takeaway: 'Faster execution—not a sparser attention pattern.',
    source: 'https://arxiv.org/abs/2205.14135',
    sourceLabel: 'Dao et al. · FlashAttention',
  },
  sliding: {
    slide: 13,
    eyebrow: 'LOCAL CONNECTIVITY',
    era: 'SLIDING WINDOW',
    title: 'Sliding-window attention assumes nearby tokens matter most.',
    lead: 'Each query can see only a fixed band of recent history. The band moves as the sequence grows.',
    metric: 'O(n × window)',
    takeaway: 'Predictable cost; distant clues need another route.',
    source: 'https://arxiv.org/abs/2004.05150',
    sourceLabel: 'Beltagy et al. · Longformer',
  },
  dsa: {
    slide: 14,
    eyebrow: 'LEARNED SPARSITY',
    era: 'DEEPSEEK DSA',
    title: 'DSA learns which few past tokens deserve full attention.',
    lead: 'A lightweight indexer ranks the history. The main attention operation runs only on the selected keys and values.',
    metric: 'Index → select → attend',
    takeaway: 'Content chooses the sparse routes; a window does not.',
    source: 'https://arxiv.org/abs/2512.02556',
    sourceLabel: 'DeepSeek-AI · DeepSeek-V3.2',
  },
  kda: {
    slide: 15,
    eyebrow: 'RECURRENT LINEAR MEMORY',
    era: 'KIMI K3 · KDA',
    title: 'KDA turns the past into an editable memory.',
    lead: 'Most K3 layers update a fixed recurrent state instead of building a token-to-token attention matrix.',
    metric: '69 KDA + 24 Gated MLA',
    takeaway: 'Fast long-context memory, punctuated by direct retrieval.',
    source: 'https://arxiv.org/abs/2607.24653',
    sourceLabel: 'Moonshot AI · Kimi K3',
  },
  qwen: {
    slide: 16,
    eyebrow: 'SHARED KEY / VALUE HEADS',
    era: 'QWEN3-8B · GQA',
    title: 'Qwen3-8B shares memory across query heads.',
    lead: 'Grouped-query attention keeps many ways to ask, while groups of query heads reuse the same keys and values.',
    metric: '32 Q heads → 8 KV heads',
    takeaway: 'Full-token reach with four times fewer KV heads to cache.',
    source: 'https://huggingface.co/Qwen/Qwen3-8B',
    sourceLabel: 'Qwen · Qwen3-8B model card',
  },
} as const

const config = computed(() => configs[props.mechanism])
const isMatrix = computed(() => ['full', 'flash', 'sliding', 'dsa'].includes(props.mechanism))

const dsaScores = [
  { index: 5, score: 0.94, reason: 'bridge' },
  { index: 2, score: 0.87, reason: 'robot' },
  { index: 6, score: 0.78, reason: 'because' },
  { index: 7, score: 0.72, reason: 'it' },
  { index: 3, score: 0.39, reason: 'crossed' },
  { index: 4, score: 0.26, reason: 'the' },
  { index: 1, score: 0.18, reason: 'small' },
  { index: 0, score: 0.11, reason: 'The' },
]

const dsaSelected = computed(() => new Set(dsaScores.slice(0, 3).map(item => item.index)))

function isVisible(row: number, column: number) {
  const supportsLookAhead = props.mechanism === 'full' || props.mechanism === 'flash'
  if (column > row && (causal.value || !supportsLookAhead)) return false
  if (props.mechanism === 'sliding') return row - column < windowSize.value
  if (props.mechanism === 'dsa') {
    if (row === queryIndex.value) return dsaSelected.value.has(column)
    return column === row || column === Math.max(0, row - 1) || column === Math.max(0, row - 3)
  }
  return true
}

function cellOpacity(row: number, column: number) {
  if (!isVisible(row, column)) return 0
  const distance = Math.abs(row - column)
  const semanticBoost = (row === 7 && [2, 5, 6].includes(column)) ? 0.38 : 0
  return Math.min(0.92, 0.18 + (1 / (distance + 1)) * 0.38 + semanticBoost)
}

function isFlashTile(row: number, column: number) {
  if (props.mechanism !== 'flash') return false
  const tileRow = Math.floor(flashTile.value / 3)
  const tileColumn = flashTile.value % 3
  return Math.floor(row / 3) === tileRow && Math.floor(column / 3) === tileColumn
}

function advanceFlash() {
  flashTile.value = (flashTile.value + 1) % 9
}

function advanceKda() {
  kdaStep.value = kdaStep.value === kdaTokens.length ? 0 : kdaStep.value + 1
}

const kdaMemory = computed(() => {
  const slots = [
    { label: 'PERSON', value: '—' },
    { label: 'ACTION', value: '—' },
    { label: 'LANGUAGE', value: '—' },
    { label: 'PLACE', value: '—' },
  ]
  if (kdaStep.value >= 1) slots[0].value = 'Maya'
  if (kdaStep.value >= 2) slots[1].value = 'learned'
  if (kdaStep.value >= 3) slots[2].value = 'Swahili'
  if (kdaStep.value >= 5) slots[3].value = 'home'
  return slots
})
</script>

<template>
  <div class="slide-shell mechanism-slide">
    <header class="mechanism-header">
      <p class="eyebrow"><span class="pulse-dot"></span> {{ config.eyebrow }}</p>
      <div class="header-meta">
        <span class="era-chip">{{ config.era }}</span>
        <span>{{ config.slide }} / 20</span>
      </div>
    </header>

    <section class="mechanism-title">
      <div>
        <h1>{{ config.title }}</h1>
        <p>{{ config.lead }}</p>
      </div>
      <strong>{{ config.metric }}</strong>
    </section>

    <main v-if="isMatrix" class="matrix-layout">
      <section class="mechanism-explainer">
        <template v-if="mechanism === 'full'">
          <span class="section-label">ONE QUERY, ALL EARLIER KEYS</span>
          <div class="query-sentence">
            <button
              v-for="(token, index) in tokens"
              :key="`${token}-${index}`"
              type="button"
              :class="{ active: queryIndex === index }"
              @click.stop="queryIndex = index"
            >{{ token }}</button>
          </div>
          <ol>
            <li><b>Query</b><span>What does this token need?</span></li>
            <li><b>Keys</b><span>Which earlier tokens match?</span></li>
            <li><b>Values</b><span>Mix the useful information.</span></li>
          </ol>
        </template>

        <template v-else-if="mechanism === 'flash'">
          <span class="section-label">SAME MATRIX, SMALLER TRIPS</span>
          <div class="memory-route">
            <div><small>SLOW MEMORY</small><strong>Q · K · V</strong></div>
            <i>→</i>
            <div class="active"><small>FAST ON-CHIP TILE</small><strong>score + normalize</strong></div>
            <i>→</i>
            <div><small>SLOW MEMORY</small><strong>output</strong></div>
          </div>
          <button class="demo-button" type="button" @click.stop="advanceFlash">PROCESS NEXT TILE</button>
          <p class="micro-copy">The blue square moves. The lower triangle does not change.</p>
        </template>

        <template v-else-if="mechanism === 'sliding'">
          <span class="section-label">CHOOSE THE LOCAL HORIZON</span>
          <div class="window-control">
            <strong>{{ windowSize }} tokens</strong>
            <input v-model.number="windowSize" type="range" min="1" max="8" @click.stop>
          </div>
          <div class="mini-example">
            <span>“it” can inspect</span>
            <strong>{{ tokens.slice(Math.max(0, tokens.length - windowSize)).join(' · ') }}</strong>
          </div>
          <p class="micro-copy">Widen the window and watch the diagonal band grow.</p>
        </template>

        <template v-else>
          <span class="section-label">TOY INDEXER SCORES FOR “IT”</span>
          <div class="score-list">
            <div v-for="item in dsaScores.slice(0, 5)" :key="item.index" :class="{ selected: dsaSelected.has(item.index) }">
              <span>{{ item.reason }}</span>
              <i><b :style="{ width: `${item.score * 100}%` }"></b></i>
              <strong>{{ item.score.toFixed(2) }}</strong>
            </div>
          </div>
          <p class="micro-copy">Top 3 continue to the expensive attention step. Scores are illustrative.</p>
        </template>

        <button
          v-if="mechanism === 'full' || mechanism === 'flash'"
          class="causal-toggle"
          type="button"
          :aria-pressed="causal"
          @click.stop="causal = !causal"
        >
          <span>
            <strong>{{ causal ? 'CAUSAL ON' : 'LOOK AHEAD' }}</strong>
            <small>{{ causal ? 'Future tokens are masked' : 'Future tokens are visible' }}</small>
          </span>
          <i :class="{ active: !causal }"><b></b></i>
        </button>
      </section>

      <section class="matrix-panel">
        <header>
          <div><small>KEY / HISTORY →</small><strong>{{ tokens[queryIndex] }} asks what matters</strong></div>
          <span>{{ !causal && (mechanism === 'full' || mechanism === 'flash') ? 'FUTURE VISIBLE' : mechanism === 'flash' ? 'TILED EXECUTION' : mechanism === 'dsa' ? 'SELECTED ROUTES' : 'ALLOWED ROUTES' }}</span>
        </header>
        <div class="attention-matrix" :style="{ '--matrix-size': tokens.length }">
          <span class="matrix-corner">Q</span>
          <span v-for="(token, index) in tokens" :key="`head-${index}`" class="column-label">{{ token }}</span>
          <template v-for="(rowToken, row) in tokens" :key="`row-${row}`">
            <button
              type="button"
              class="row-label"
              :class="{ active: queryIndex === row }"
              @click.stop="queryIndex = row"
            >{{ rowToken }}</button>
            <span
              v-for="(_, column) in tokens"
              :key="`cell-${row}-${column}`"
              class="matrix-cell"
              :class="{
                masked: !isVisible(row, column),
                query: queryIndex === row,
                'flash-tile': isFlashTile(row, column),
              }"
              :style="{ '--cell-opacity': cellOpacity(row, column) }"
            ></span>
          </template>
        </div>
        <div class="matrix-legend"><span><i></i> reachable</span><span><i class="off"></i> removed before scoring</span></div>
      </section>
    </main>

    <main v-else-if="mechanism === 'kda'" class="kda-layout">
      <section class="kda-stream">
        <span class="section-label">STEP THROUGH THE SEQUENCE</span>
        <div class="token-stream">
          <span
            v-for="(token, index) in kdaTokens"
            :key="token"
            :class="{ seen: index < kdaStep, current: index === kdaStep - 1 }"
          >{{ token }}</span>
        </div>
        <div class="kda-cycle">
          <article><small>1 · READ</small><strong>{{ kdaStep ? kdaTokens[kdaStep - 1] : 'next token' }}</strong></article>
          <i>→</i>
          <article><small>2 · COMPARE</small><strong>what changed?</strong></article>
          <i>→</i>
          <article class="active"><small>3 · UPDATE</small><strong>erase + write</strong></article>
        </div>
        <button class="demo-button" type="button" @click.stop="advanceKda">
          {{ kdaStep === kdaTokens.length ? 'RESET MEMORY' : 'READ NEXT TOKEN' }}
        </button>
      </section>

      <section class="memory-panel">
        <header><small>FIXED-SIZE RECURRENT STATE</small><strong>Memory is edited, not extended.</strong></header>
        <div class="memory-slots">
          <article v-for="slot in kdaMemory" :key="slot.label" :class="{ filled: slot.value !== '—' }">
            <span>{{ slot.label }}</span><strong>{{ slot.value }}</strong>
          </article>
        </div>
        <div class="layer-strip" aria-label="Kimi K3 hybrid layer mix">
          <span v-for="index in 12" :key="index" :class="index % 4 === 0 ? 'mla' : 'kda'">{{ index % 4 === 0 ? 'MLA' : 'KDA' }}</span>
        </div>
        <p>K3 periodically restores direct token retrieval with Gated MLA layers.</p>
      </section>
    </main>

    <main v-else class="qwen-layout">
      <section class="gqa-example">
        <span class="section-label">ONE NEXT-TOKEN DECISION</span>
        <div class="gqa-sentence">
          <small>VISIBLE HISTORY</small>
          <p>The robot crossed the bridge because it was</p>
          <strong>?</strong>
        </div>
        <div class="question-list">
          <span>ILLUSTRATIVE QUERY HEADS</span>
          <article><b>Q₁</b><p>What describes “it”?</p></article>
          <article><b>Q₂</b><p>Which noun does “it” refer to?</p></article>
          <article><b>Q₃</b><p>What adjective fits the syntax?</p></article>
          <article><b>Q₄</b><p>What is most recent?</p></article>
        </div>
      </section>

      <section class="gqa-panel">
        <header>
          <div><small>GROUPED-QUERY ATTENTION</small><strong>Four query heads share one KV head.</strong></div>
          <span>8 GROUPS TOTAL</span>
        </header>
        <div class="head-groups">
          <article v-for="group in 8" :key="group" :class="{ featured: group === 1 }">
            <div class="query-heads">
              <span v-for="head in 4" :key="head">Q</span>
            </div>
            <i>→</i>
            <div class="kv-head"><span>K</span><span>V</span></div>
            <small>GROUP {{ group }}</small>
          </article>
        </div>
        <div class="gqa-comparison">
          <div><small>STANDARD 32-HEAD MHA</small><strong>32 KV heads cached</strong></div>
          <i>→</i>
          <div class="active"><small>QWEN3-8B GQA</small><strong>8 KV heads cached</strong></div>
        </div>
        <p>Sharing happens across heads—not across token positions. Every head can still inspect the full causal history.</p>
      </section>
    </main>

    <footer class="mechanism-footer">
      <span>{{ config.takeaway }}</span>
      <ResourceLink :href="config.source" :label="config.sourceLabel" />
    </footer>
  </div>
</template>

<style scoped>
.mechanism-slide {
  padding: 25px 38px 17px;
}

.mechanism-header {
  flex: 0 0 37px;
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  border-bottom: 1px solid #292e33;
}

.mechanism-header .eyebrow { font-size: 8.5px; }
.header-meta { display: flex; align-items: center; gap: 13px; color: var(--accent); font: 750 8px/1 var(--deck-mono); letter-spacing: .1em; }
.header-meta .era-chip { font-size: 7px; }

.mechanism-title {
  display: grid;
  grid-template-columns: minmax(0, 1fr) 150px;
  align-items: end;
  gap: 20px;
  padding: 12px 0 11px;
}

.mechanism-title h1 { font-size: 31px; line-height: 1; letter-spacing: -.04em; }
.mechanism-title p { max-width: 690px; margin: 7px 0 0; color: var(--muted); font-size: 9px; line-height: 1.45; }
.mechanism-title > strong { justify-self: end; padding: 8px 10px; color: var(--accent); background: rgba(119, 217, 255, .055); border: 1px solid rgba(119, 217, 255, .22); border-radius: 7px; font: 700 7px/1 var(--deck-mono); letter-spacing: .08em; text-align: center; }

.matrix-layout,
.kda-layout,
.qwen-layout {
  flex: 1;
  display: grid;
  grid-template-columns: minmax(0, .78fr) minmax(0, 1.22fr);
  gap: 14px;
  min-height: 0;
}

.mechanism-explainer,
.matrix-panel,
.kda-stream,
.memory-panel,
.gqa-example,
.gqa-panel {
  min-width: 0;
  padding: 13px;
  background: rgba(255, 255, 255, .018);
  border: 1px solid #292f34;
  border-radius: 9px;
}

.section-label { color: var(--accent); font-size: 6px; font-weight: 750; letter-spacing: .14em; }
.query-sentence { display: flex; flex-wrap: wrap; gap: 5px; margin-top: 14px; }
.query-sentence button,
.row-label { color: #909a9f; background: #11161a; border: 1px solid #2b3439; border-radius: 5px; font: 650 7px/1 var(--deck-mono); cursor: pointer; }
.query-sentence button { padding: 8px 7px; }
.query-sentence button.active,
.row-label.active { color: #081116; background: var(--accent); border-color: var(--accent); }
.mechanism-explainer ol { display: grid; gap: 7px; margin: 17px 0 0; padding: 0; list-style: none; }
.mechanism-explainer li { display: grid; grid-template-columns: 48px 1fr; gap: 8px; padding: 8px; background: #101417; border-radius: 6px; }
.mechanism-explainer li b { color: var(--accent); font-size: 8px; }
.mechanism-explainer li span { color: #929ba0; font-size: 7px; }

.matrix-panel { display: flex; flex-direction: column; }
.matrix-panel > header { display: flex; align-items: end; justify-content: space-between; margin-bottom: 8px; }
.matrix-panel header div { display: grid; gap: 3px; }
.matrix-panel header small { color: #747f84; font-size: 5.5px; letter-spacing: .12em; }
.matrix-panel header strong { color: var(--ink); font-size: 9px; }
.matrix-panel header > span { color: var(--accent); font: 700 5.5px/1 var(--deck-mono); letter-spacing: .1em; }
.attention-matrix { flex: 1; display: grid; grid-template-columns: 42px repeat(var(--matrix-size), 1fr); grid-template-rows: 20px repeat(var(--matrix-size), 1fr); gap: 3px; min-height: 0; }
.matrix-corner,
.column-label { display: grid; place-items: center; color: #687278; font: 650 5.5px/1 var(--deck-mono); overflow: hidden; }
.column-label { transform: rotate(-20deg); }
.row-label { display: grid; width: 100%; place-items: center; padding: 0; }
.matrix-cell { position: relative; border-radius: 3px; background: rgba(119, 217, 255, var(--cell-opacity)); border: 1px solid rgba(119, 217, 255, .08); }
.matrix-cell.masked { background: rgba(255, 255, 255, .018); border-color: #20262a; }
.matrix-cell.query:not(.masked) { box-shadow: inset 0 0 0 1px rgba(185, 235, 255, .7); }
.matrix-cell.flash-tile::after { content: ''; position: absolute; inset: -2px; border: 1px solid #fff; border-radius: 4px; box-shadow: 0 0 10px rgba(119, 217, 255, .45); }
.matrix-legend { display: flex; gap: 14px; margin-top: 7px; color: #737d82; font-size: 5.5px; }
.matrix-legend span { display: flex; align-items: center; gap: 5px; }
.matrix-legend i { width: 9px; height: 9px; background: rgba(119, 217, 255, .7); border-radius: 2px; }
.matrix-legend i.off { background: rgba(255, 255, 255, .025); border: 1px solid #252c30; }

.memory-route { display: grid; grid-template-columns: 1fr 10px 1fr 10px 1fr; align-items: center; gap: 4px; margin-top: 16px; }
.memory-route div { display: grid; gap: 7px; min-height: 78px; place-content: center; padding: 8px; background: #101417; border: 1px solid #293137; border-radius: 7px; text-align: center; }
.memory-route div.active { background: rgba(119, 217, 255, .055); border-color: rgba(119, 217, 255, .35); }
.memory-route small { color: #717c81; font-size: 5px; letter-spacing: .08em; }
.memory-route strong { color: var(--ink); font-size: 7px; line-height: 1.35; }
.memory-route i { color: #566168; font-style: normal; font-size: 8px; }
.demo-button { margin-top: 14px; padding: 9px 11px; color: #071117; background: var(--accent); border: 0; border-radius: 6px; font: 750 6px/1 var(--deck-mono); letter-spacing: .08em; cursor: pointer; }
.micro-copy { margin: 10px 0 0; color: #818b90; font-size: 7px; line-height: 1.45; }
.causal-toggle { display: flex; align-items: center; justify-content: space-between; width: 100%; margin-top: 11px; padding: 8px 10px; color: inherit; background: #101417; border: 1px solid #2a3338; border-radius: 7px; cursor: pointer; }
.causal-toggle > span { display: grid; gap: 3px; text-align: left; }
.causal-toggle strong { color: var(--accent); font: 750 6px/1 var(--deck-mono); letter-spacing: .09em; }
.causal-toggle small { color: #778288; font-size: 5.5px; }
.causal-toggle > i { position: relative; width: 31px; height: 17px; background: #263036; border: 1px solid #39454c; border-radius: 999px; transition: background .18s ease, border-color .18s ease; }
.causal-toggle > i b { position: absolute; top: 2px; left: 2px; width: 11px; height: 11px; background: #89949a; border-radius: 50%; transition: transform .18s ease, background .18s ease; }
.causal-toggle > i.active { background: rgba(119, 217, 255, .14); border-color: rgba(119, 217, 255, .48); }
.causal-toggle > i.active b { background: var(--accent); transform: translateX(14px); }
.window-control { display: grid; gap: 12px; margin-top: 21px; padding: 14px; background: #101417; border-radius: 8px; }
.window-control strong { color: var(--accent); font-size: 18px; }
.window-control input { width: 100%; accent-color: var(--accent); }
.mini-example { display: grid; gap: 7px; margin-top: 11px; padding: 11px; border: 1px solid #293137; border-radius: 7px; }
.mini-example span { color: #727c82; font-size: 6px; }
.mini-example strong { color: var(--ink); font: 650 7px/1.5 var(--deck-mono); }
.score-list { display: grid; gap: 6px; margin-top: 13px; }
.score-list div { display: grid; grid-template-columns: 46px 1fr 27px; align-items: center; gap: 7px; color: #768187; font: 650 6px/1 var(--deck-mono); }
.score-list div > i { height: 5px; background: #171d21; border-radius: 999px; overflow: hidden; }
.score-list div > i b { display: block; height: 100%; background: #52616a; }
.score-list div.selected { color: var(--ink); }
.score-list div.selected > i b { background: var(--accent); }

.kda-layout { grid-template-columns: 1fr 1fr; }
.token-stream { display: flex; gap: 7px; margin: 22px 0; }
.token-stream span { padding: 10px 9px; color: #616b70; background: #11161a; border: 1px solid #283036; border-radius: 6px; font: 650 8px/1 var(--deck-mono); }
.token-stream span.seen { color: #9ca6aa; border-color: #3a454b; }
.token-stream span.current { color: #071117; background: var(--accent); border-color: var(--accent); }
.kda-cycle { display: grid; grid-template-columns: 1fr 11px 1fr 11px 1fr; align-items: center; gap: 5px; }
.kda-cycle article { display: grid; gap: 6px; min-height: 72px; place-content: center; padding: 8px; background: #101417; border: 1px solid #293137; border-radius: 7px; text-align: center; }
.kda-cycle article.active { background: rgba(119, 217, 255, .055); border-color: rgba(119, 217, 255, .3); }
.kda-cycle small { color: var(--accent); font-size: 5px; letter-spacing: .1em; }
.kda-cycle strong { color: var(--ink); font-size: 7px; }
.kda-cycle i { color: #566168; font-style: normal; text-align: center; }
.memory-panel header { display: grid; gap: 4px; }
.memory-panel header small { color: var(--accent); font-size: 6px; letter-spacing: .13em; }
.memory-panel header strong { color: var(--ink); font-size: 11px; }
.memory-slots { display: grid; grid-template-columns: 1fr 1fr; gap: 7px; margin-top: 13px; }
.memory-slots article { display: grid; gap: 7px; padding: 12px; background: #101417; border: 1px solid #283036; border-radius: 7px; }
.memory-slots article.filled { background: rgba(119, 217, 255, .04); border-color: rgba(119, 217, 255, .28); }
.memory-slots span { color: #6f7a80; font-size: 5px; letter-spacing: .1em; }
.memory-slots strong { min-height: 13px; color: var(--ink); font: 700 9px/1 var(--deck-mono); }
.layer-strip { display: grid; grid-template-columns: repeat(12, 1fr); gap: 3px; margin-top: 13px; }
.layer-strip span { display: grid; height: 25px; place-items: center; border-radius: 3px; font: 700 4.5px/1 var(--deck-mono); }
.layer-strip .kda { color: #071117; background: var(--accent); }
.layer-strip .mla { color: #b4bdc1; background: #293238; }
.memory-panel > p { margin: 9px 0 0; color: #7d878c; font-size: 6.5px; }

.qwen-layout { grid-template-columns: .76fr 1.24fr; }
.gqa-sentence { position: relative; display: grid; gap: 8px; margin-top: 15px; padding: 13px 38px 13px 12px; background: #101417; border: 1px solid #293137; border-radius: 7px; }
.gqa-sentence small { color: #707b80; font-size: 5px; letter-spacing: .1em; }
.gqa-sentence p { margin: 0; color: var(--ink); font: 650 8px/1.45 var(--deck-mono); }
.gqa-sentence > strong { position: absolute; right: 12px; bottom: 10px; color: var(--accent); font-size: 22px; }
.question-list { display: grid; gap: 5px; margin-top: 12px; }
.question-list > span { margin-bottom: 1px; color: #707b80; font-size: 5px; letter-spacing: .1em; }
.question-list article { display: grid; grid-template-columns: 26px 1fr; align-items: center; gap: 7px; padding: 6px 8px; background: rgba(119, 217, 255, .025); border: 1px solid #293137; border-radius: 6px; }
.question-list b { color: var(--accent); font: 700 7px/1 var(--deck-mono); }
.question-list p { margin: 0; color: #8b969b; font-size: 6.5px; }
.gqa-panel { display: flex; flex-direction: column; }
.gqa-panel > header { display: flex; align-items: end; justify-content: space-between; }
.gqa-panel > header div { display: grid; gap: 3px; }
.gqa-panel > header small { color: var(--accent); font-size: 5.5px; letter-spacing: .12em; }
.gqa-panel > header strong { color: var(--ink); font-size: 9px; }
.gqa-panel > header > span { color: #778288; font: 700 5.5px/1 var(--deck-mono); letter-spacing: .1em; }
.head-groups { flex: 1; display: grid; grid-template-columns: repeat(4, 1fr); grid-template-rows: repeat(2, 1fr); gap: 7px; min-height: 0; margin-top: 10px; }
.head-groups article { position: relative; display: grid; grid-template-columns: 1fr 9px 31px; align-items: center; gap: 4px; padding: 8px; background: #101417; border: 1px solid #293137; border-radius: 7px; }
.head-groups article.featured { background: rgba(119, 217, 255, .04); border-color: rgba(119, 217, 255, .36); }
.query-heads { display: grid; grid-template-columns: 1fr 1fr; gap: 3px; }
.query-heads span { display: grid; width: 17px; height: 17px; place-items: center; color: #081116; background: var(--accent); border-radius: 50%; font: 750 5px/1 var(--deck-mono); }
.head-groups article > i { color: #566168; font-style: normal; font-size: 7px; text-align: center; }
.kv-head { display: grid; grid-template-columns: 1fr 1fr; gap: 2px; padding: 4px; background: #253038; border-radius: 5px; }
.kv-head span { display: grid; height: 24px; place-items: center; color: #c6d0d4; font: 750 5px/1 var(--deck-mono); }
.head-groups article > small { position: absolute; right: 5px; bottom: 3px; color: #566168; font-size: 3.8px; letter-spacing: .07em; }
.gqa-comparison { display: grid; grid-template-columns: 1fr 12px 1fr; align-items: center; gap: 6px; margin-top: 9px; }
.gqa-comparison div { display: grid; gap: 4px; padding: 8px 10px; background: #101417; border: 1px solid #293137; border-radius: 6px; }
.gqa-comparison div.active { background: rgba(119, 217, 255, .04); border-color: rgba(119, 217, 255, .3); }
.gqa-comparison small { color: #737e83; font-size: 4.8px; letter-spacing: .09em; }
.gqa-comparison strong { color: var(--ink); font-size: 7px; }
.gqa-comparison i { color: #59646a; font-style: normal; text-align: center; }
.gqa-panel > p { margin: 8px 0 0; color: #7f898e; font-size: 6px; line-height: 1.4; }

.mechanism-footer { flex: 0 0 27px; display: flex; align-items: end; justify-content: space-between; gap: 18px; padding-top: 8px; color: #768087; font-size: 6px; font-weight: 700; letter-spacing: .07em; text-transform: uppercase; }
.mechanism-footer > span { white-space: nowrap; }
</style>
