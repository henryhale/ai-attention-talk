<script setup lang="ts">
import { computed, ref } from 'vue'

const tokens = ['The', 'cat', 'sat', 'on', 'the', 'mat']
const nextPredictions = [
  { token: 'cat', confidence: 0.42 },
  { token: 'sat', confidence: 0.55 },
  { token: 'on', confidence: 0.63 },
  { token: 'the', confidence: 0.69 },
  { token: 'mat', confidence: 0.76 },
  { token: '.', confidence: 0.81 },
]
const step = ref(0)

function tokenSeed(token: string) {
  return [...token.toLocaleLowerCase()].reduce((sum, character, index) => sum + (character.codePointAt(0) || 0) * (index + 3), 17)
}

function inputVector(token: string) {
  const seed = tokenSeed(token)
  return Array.from({ length: 6 }, (_, index) => Math.sin(seed * (index + 1) * 0.071))
}

const hiddenStates = computed(() => {
  const states: number[][] = []
  let previous = Array(6).fill(0) as number[]
  for (const token of tokens) {
    const input = inputVector(token)
    const current = input.map((value, index) => {
      const recurrent = previous[index] * 0.66 + previous[(index + 5) % previous.length] * 0.18
      return Math.tanh(value * 0.58 + recurrent)
    })
    states.push(current)
    previous = current
  }
  return states
})

const activeState = computed(() => hiddenStates.value[step.value])
const previousState = computed(() => step.value ? hiddenStates.value[step.value - 1] : Array(6).fill(0))
const prediction = computed(() => nextPredictions[step.value])

function move(delta: number) {
  step.value = Math.max(0, Math.min(tokens.length - 1, step.value + delta))
}
</script>

<template>
  <div class="slide-shell history-slide">
    <header class="history-header">
      <div class="history-title">
        <span class="step-index">03</span>
        <h1>RNNs carry one summary forward.</h1>
      </div>
      <span class="era-chip">1990</span>
    </header>

    <main class="history-grid interactive-grid">
      <section class="concept-panel rnn-concept">
        <p class="panel-label">STEP THROUGH IT</p>
        <h2>Each token rewrites the memory.</h2>
        <p v-click>One state carries everything<br>the model remembers.</p>

        <div class="step-readout">
          <span>STEP</span>
          <strong>{{ step + 1 }} / {{ tokens.length }}</strong>
          <b>{{ tokens[step] }}</b>
        </div>

        <div class="step-controls" aria-label="RNN step controls">
          <button type="button" @click.stop="step = 0" :disabled="step === 0">
            <carbon-reset aria-hidden="true" /> Reset
          </button>
          <button type="button" @click.stop="move(-1)" :disabled="step === 0">
            <carbon-chevron-left aria-hidden="true" /> Previous
          </button>
          <button type="button" class="primary" @click.stop="move(1)" :disabled="step === tokens.length - 1">
            Next <carbon-chevron-right aria-hidden="true" />
          </button>
        </div>

        <p v-click class="tradeoff"><span>Limit</span> Old clues weaken every step.</p>
      </section>

      <section class="example-panel rnn-example">
        <div class="example-label">NEXT TOKEN · STEP THROUGH THE HIDDEN STATE</div>

        <div class="interactive-rnn-chain">
          <template v-for="(token, index) in tokens" :key="token">
            <button
              type="button"
              :class="{ processed: index < step, active: index === step, future: index > step }"
              :aria-pressed="index === step"
              :title="`Jump to step ${index + 1}`"
              @click.stop="step = index"
            >
              <span>{{ token }}</span>
              <small>h{{ index + 1 }}</small>
            </button>
            <i v-if="index < tokens.length - 1" :class="{ lit: index < step }">→</i>
          </template>
        </div>

        <div class="state-flow">
          <div class="state-node muted">
            <span>PREVIOUS</span>
            <strong>h{{ step }}</strong>
            <small>{{ previousState.map(value => value.toFixed(1)).slice(0, 3).join(' · ') }}</small>
          </div>
          <i>+</i>
          <div class="state-node input-node">
            <span>INPUT</span>
            <strong>{{ tokens[step] }}</strong>
            <small>x{{ step + 1 }}</small>
          </div>
          <i>→</i>
          <div class="state-node active-node">
            <span>NEW STATE</span>
            <strong>h{{ step + 1 }}</strong>
            <div class="state-vector" aria-label="Toy hidden state vector">
              <b
                v-for="(value, index) in activeState"
                :key="index"
                :class="{ negative: value < 0 }"
                :style="{ height: `${5 + Math.abs(value) * 19}px` }"
                :title="value.toFixed(3)"
              ></b>
            </div>
          </div>
        </div>

        <div class="rnn-prediction">
          <small>h{{ step + 1 }} → SOFTMAX → NEXT TOKEN</small>
          <span>{{ prediction.token }}</span>
          <strong>{{ Math.round(prediction.confidence * 100) }}%</strong>
          <i><b :style="{ width: `${prediction.confidence * 100}%` }"></b></i>
        </div>

        <p v-click class="example-note">The state is a summary—not a <b>copy</b>.</p>
      </section>
    </main>

    <footer class="history-footer">
      <span>HISTORY OF SEQUENCE MODELING</span>
      <span class="slide-progress">08 / 20</span>
      <ResourceLink href="https://doi.org/10.1207/s15516709cog1402_1" label="Elman · Finding Structure in Time (1990)" />
    </footer>
  </div>
</template>

<style scoped>
.rnn-concept {
  padding: 16px 17px;
}

.rnn-concept h2 {
  font-size: 20px;
}

.rnn-concept > p:not(.panel-label):not(.tradeoff) {
  font-size: 12.5px !important;
  line-height: 1.4 !important;
}

.step-readout {
  display: grid;
  grid-template-columns: auto auto 1fr;
  align-items: center;
  gap: 7px;
  margin-top: 17px;
  padding: 8px 9px;
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid #2c322d;
  border-radius: 6px;
}

.step-readout span {
  color: #737c74;
  font-size: 7px;
  font-weight: 750;
  letter-spacing: 0.1em;
}

.step-readout strong {
  color: var(--accent);
  font: 700 7px/1 var(--deck-mono);
}

.step-readout b {
  overflow: hidden;
  color: #d0d5d0;
  font: 700 9px/1 var(--deck-mono);
  text-align: right;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.step-controls {
  display: grid;
  grid-template-columns: 0.8fr 1fr 1fr;
  gap: 5px;
  margin-top: 8px;
}

.step-controls button {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 3px;
  padding: 7px 3px;
  color: #9aa29b;
  background: #101411;
  border: 1px solid #303631;
  border-radius: 5px;
  font-size: 7px;
  font-weight: 700;
  cursor: pointer;
}

.step-controls button.primary {
  color: #0b0e0c;
  background: var(--accent);
  border-color: var(--accent);
}

.step-controls button:disabled {
  opacity: 0.35;
  cursor: default;
}

.step-controls svg {
  width: 8px;
}

.rnn-concept .tradeoff {
  right: 17px;
  bottom: 14px;
  left: 17px;
  font-size: 9.5px;
}

.rnn-example {
  display: flex;
  flex-direction: column;
}

.interactive-rnn-chain {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 3px;
  margin-top: 3px;
}

.interactive-rnn-chain button {
  display: grid;
  width: 43px;
  overflow: hidden;
  padding: 0;
  color: #8f9790;
  background: #111512;
  border: 1px solid #303631;
  border-radius: 5px;
  cursor: pointer;
}

.interactive-rnn-chain button span,
.interactive-rnn-chain button small {
  padding: 5px 3px;
  overflow: hidden;
  font: 650 6.5px/1 var(--deck-mono);
  text-overflow: ellipsis;
  white-space: nowrap;
}

.interactive-rnn-chain button small {
  color: #667068;
  background: #0c0f0d;
  border-top: 1px solid #292f2a;
  font-size: 5.5px;
}

.interactive-rnn-chain button.processed {
  border-color: #4a5543;
}

.interactive-rnn-chain button.active {
  color: #0b0e0c;
  background: var(--accent);
  border-color: var(--accent);
  box-shadow: 0 0 15px rgba(119, 217, 255, 0.08);
}

.interactive-rnn-chain button.active small {
  color: var(--accent);
}

.interactive-rnn-chain button.future {
  opacity: 0.38;
}

.interactive-rnn-chain > i {
  color: #3e4640;
  font-size: 7px;
  font-style: normal;
}

.interactive-rnn-chain > i.lit {
  color: var(--accent);
}

.state-flow {
  display: grid;
  grid-template-columns: 1fr auto 0.85fr auto 1.15fr;
  align-items: center;
  gap: 6px;
  margin-top: 12px;
}

.state-flow > i {
  color: #59615a;
  font-size: 9px;
  font-style: normal;
}

.state-node {
  display: grid;
  min-height: 62px;
  place-items: center;
  padding: 7px;
  background: #101411;
  border: 1px solid #303631;
  border-radius: 6px;
  text-align: center;
}

.state-node > span {
  color: #707971;
  font-size: 5.5px;
  font-weight: 750;
  letter-spacing: 0.09em;
}

.state-node > strong {
  color: #c8cdc9;
  font: 750 10px/1 var(--deck-mono);
}

.state-node > small {
  max-width: 80px;
  overflow: hidden;
  color: #646d65;
  font: 600 5px/1 var(--deck-mono);
  text-overflow: ellipsis;
  white-space: nowrap;
}

.state-node.active-node {
  border-color: rgba(119, 217, 255, 0.34);
}

.state-node.active-node > strong {
  color: var(--accent);
}

.state-vector {
  display: flex;
  align-items: end;
  gap: 2px;
  height: 25px;
}

.state-vector b {
  display: block;
  width: 4px;
  max-height: 24px;
  background: var(--accent);
  border-radius: 2px 2px 1px 1px;
}

.state-vector b.negative {
  background: #75a6df;
}

.rnn-prediction {
  position: relative;
  display: grid;
  grid-template-columns: 1fr auto auto;
  align-items: center;
  gap: 9px;
  margin-top: 10px;
  padding: 8px 9px 10px;
  overflow: hidden;
  background: rgba(119, 217, 255, 0.045);
  border: 1px solid rgba(119, 217, 255, 0.2);
  border-radius: 6px;
}

.rnn-prediction small {
  color: #737c74;
  font-size: 5.5px;
  letter-spacing: 0.09em;
}

.rnn-prediction span {
  color: var(--accent);
  font: 750 9px/1 var(--deck-mono);
}

.rnn-prediction strong {
  color: #d0d5d0;
  font: 700 7px/1 var(--deck-mono);
}

.rnn-prediction > i {
  position: absolute;
  right: 0;
  bottom: 0;
  left: 0;
  height: 2px;
  background: #282e29;
}

.rnn-prediction > i b {
  display: block;
  height: 100%;
  background: var(--accent);
}
</style>
