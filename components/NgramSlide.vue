<script setup lang="ts">
import { computed, ref } from 'vue'

type NgramSize = 1 | 2 | 3
const sizes: NgramSize[] = [1, 2, 3]

const corpus = ref(`the cat sat on the mat
the cat slept on the rug
the dog sat on the rug
the cat sat on the floor`)
const prompt = ref('the cat sat on the')
const n = ref<NgramSize>(1)

function tokenize(value: string) {
  return (value.toLocaleLowerCase().match(/[\p{L}\p{N}]+(?:['’\-][\p{L}\p{N}]+)*/gu) || [])
}

const sequences = computed(() => corpus.value
  .split(/\n+/)
  .map(line => tokenize(line))
  .filter(sequence => sequence.length))
const promptTokens = computed(() => tokenize(prompt.value))
const context = computed(() => n.value === 1 ? [] : promptTokens.value.slice(-(n.value - 1)))
const modeLabel = computed(() => n.value === 1 ? 'UNIGRAM' : n.value === 2 ? 'BIGRAM' : 'TRIGRAM')

const examples = computed(() => {
  const windows: string[][] = []
  for (const sequence of sequences.value) {
    if (n.value === 1) {
      sequence.forEach(token => windows.push([token]))
      continue
    }
    for (let index = n.value - 1; index < sequence.length; index++) {
      const candidate = sequence.slice(index - n.value + 1, index + 1)
      const candidateContext = candidate.slice(0, -1)
      if (candidateContext.every((token, tokenIndex) => token === context.value[tokenIndex])) {
        windows.push(candidate)
      }
    }
  }
  return windows
})

const predictions = computed(() => {
  const counts = new Map<string, number>()
  for (const window of examples.value) {
    const candidate = window.at(-1)
    if (candidate) counts.set(candidate, (counts.get(candidate) || 0) + 1)
  }
  const total = [...counts.values()].reduce((sum, count) => sum + count, 0)
  return [...counts.entries()]
    .map(([token, count]) => ({ token, count, probability: total ? count / total : 0 }))
    .sort((left, right) => right.count - left.count || left.token.localeCompare(right.token))
    .slice(0, 4)
})

</script>

<template>
  <div class="slide-shell history-slide">
    <header class="history-header">
      <div class="history-title">
        <span class="step-index">01</span>
        <h1>N-grams predict by counting context.</h1>
      </div>
      <span class="era-chip">1948 →</span>
    </header>

    <main class="history-grid interactive-grid">
      <section class="concept-panel ngram-concept">
        <p class="panel-label">TRY IT</p>
        <h2>Choose how much history matters.</h2>
        <p v-click>More words mean better clues—<br>until matches disappear.</p>

        <div class="n-selector" aria-label="N-gram size">
          <button
            v-for="size in sizes"
            :key="size"
            type="button"
            :class="{ active: n === size }"
            :aria-pressed="n === size"
            @click.stop="n = size"
          >
            {{ size === 1 ? 'UNIGRAM' : size === 2 ? 'BIGRAM' : 'TRIGRAM' }} <small>n={{ size }}</small>
          </button>
        </div>

        <label class="field-label">
          <span>TRAINING CORPUS · ONE EXAMPLE PER LINE</span>
          <textarea v-model="corpus" rows="4" spellcheck="false" aria-label="Training corpus"></textarea>
        </label>
        <label class="field-label prompt-field">
          <span>PROMPT</span>
          <input v-model="prompt" aria-label="Prediction prompt">
        </label>
      </section>

      <section class="example-panel ngram-example">
        <div class="example-label">NEXT TOKEN · {{ modeLabel }}</div>

        <div class="context-card">
          <span>ACTIVE CONTEXT</span>
          <div v-if="context.length" class="context-tokens">
            <b v-for="(token, index) in context" :key="`${token}-${index}`">{{ token }}</b>
          </div>
          <strong v-else>∅ <small>no context</small></strong>
        </div>

        <div class="window-section">
          <span>MATCHING WINDOWS · {{ examples.length }}</span>
          <div v-if="examples.length" class="windows">
            <div v-for="(window, index) in examples.slice(0, 5)" :key="index">
              <span v-for="(token, tokenIndex) in window" :key="tokenIndex" :class="{ next: tokenIndex === window.length - 1 }">
                {{ token }}
              </span>
            </div>
          </div>
          <p v-else>No matching context. Change the prompt or corpus.</p>
        </div>

        <div class="prediction-list">
          <div v-for="prediction in predictions" :key="prediction.token" class="prediction-row">
            <span>{{ prediction.token }}</span>
            <i><b :style="{ width: `${prediction.probability * 100}%` }"></b></i>
            <strong>{{ Math.round(prediction.probability * 100) }}%</strong>
            <small>{{ prediction.count }}×</small>
          </div>
        </div>

        <p v-click class="example-note">More context helps—until matches <b>vanish</b>.</p>
      </section>
    </main>

    <footer class="history-footer">
      <span>HISTORY OF SEQUENCE MODELING</span>
      <span class="slide-progress">06 / 20</span>
      <ResourceLink href="https://ocw.mit.edu/courses/6-863j-natural-language-and-the-computer-representation-of-knowledge-spring-2003/c0ef4dd1f715f5575064e24de3413909_lecture503.pdf" label="MIT OCW · N-gram language models" />
    </footer>
  </div>
</template>

<style scoped>
.ngram-concept {
  padding: 14px 16px;
}

.ngram-concept h2 {
  margin-bottom: 7px;
  font-size: 20px;
}

.ngram-concept > p:not(.panel-label) {
  font-size: 12.5px !important;
  line-height: 1.4 !important;
}

.n-selector {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 4px;
  margin-top: 12px;
}

.n-selector button {
  padding: 5px 3px;
  color: #7e877e;
  background: #101411;
  border: 1px solid #2e342f;
  border-radius: 4px;
  font-size: 6.5px;
  font-weight: 750;
  cursor: pointer;
}

.n-selector button.active {
  color: #0b0e0c;
  background: var(--accent);
  border-color: var(--accent);
}

.n-selector small {
  display: block;
  margin-top: 2px;
  font: 650 5px/1 var(--deck-mono);
}

.field-label {
  display: grid;
  gap: 3px;
  margin-top: 7px;
}

.field-label > span {
  color: #6f7870;
  font-size: 6.2px;
  font-weight: 750;
  letter-spacing: 0.09em;
}

.field-label textarea,
.field-label input {
  width: 100%;
  box-sizing: border-box;
  padding: 5px 6px;
  color: #c7ccc8;
  background: #0c0f0d;
  border: 1px solid #303631;
  border-radius: 5px;
  outline: none;
  font: 7.5px/1.3 var(--deck-mono);
}

.field-label textarea {
  height: 43px;
  resize: none;
}

.field-label input {
  height: 22px;
}

.field-label textarea:focus,
.field-label input:focus {
  border-color: rgba(119, 217, 255, 0.48);
}

.ngram-example {
  display: flex;
  flex-direction: column;
}

.context-card {
  display: flex;
  align-items: center;
  gap: 9px;
  padding: 7px 9px;
  background: #0d100e;
  border: 1px solid #2c322d;
  border-radius: 6px;
}

.context-card > span,
.window-section > span {
  color: #747d75;
  font-size: 5.5px;
  font-weight: 750;
  letter-spacing: 0.1em;
}

.context-card > strong {
  color: var(--accent);
  font-size: 10px;
}

.context-card > strong small {
  color: #737c74;
  font-size: 6px;
  font-weight: 600;
}

.context-tokens {
  display: flex;
  gap: 4px;
}

.context-tokens b {
  padding: 4px 6px;
  color: #0b0e0c;
  background: var(--accent);
  border-radius: 4px;
  font: 700 7px/1 var(--deck-mono);
}

.window-section {
  margin-top: 9px;
}

.windows {
  display: flex;
  flex-wrap: wrap;
  gap: 4px;
  margin-top: 5px;
}

.windows > div {
  display: flex;
  overflow: hidden;
  border: 1px solid #2e342f;
  border-radius: 4px;
}

.windows span {
  padding: 4px 5px;
  color: #8f9790;
  background: #111512;
  font: 650 6px/1 var(--deck-mono);
}

.windows span.next {
  color: var(--accent);
  background: rgba(119, 217, 255, 0.07);
  border-left: 1px solid #343b34;
}

.window-section p {
  margin: 8px 0 0;
  color: #8b6f6f;
  font-size: 7px;
}

.prediction-list {
  display: grid;
  gap: 5px;
  margin-top: 10px;
}

.prediction-row {
  display: grid;
  grid-template-columns: 43px 1fr 27px 18px;
  align-items: center;
  gap: 6px;
  font: 650 7px/1 var(--deck-mono);
}

.prediction-row > span {
  overflow: hidden;
  color: #aab1ab;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.prediction-row i {
  height: 5px;
  overflow: hidden;
  background: #272d28;
  border-radius: 999px;
}

.prediction-row i b {
  display: block;
  height: 100%;
  background: var(--accent);
  border-radius: inherit;
}

.prediction-row strong {
  color: #d0d5d0;
  text-align: right;
}

.prediction-row small {
  color: #697169;
  font-size: 5.5px;
  text-align: right;
}
</style>
