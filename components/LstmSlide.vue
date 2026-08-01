<script setup lang="ts">
import { computed, ref } from 'vue'

const sequence = [
  { token: 'The', forget: 0.86, input: 0.18, output: 0.28, candidate: 0.04, next: 'cat' },
  { token: 'cat', forget: 0.94, input: 0.92, output: 0.55, candidate: 0.96, next: 'wandered' },
  { token: 'wandered', forget: 0.91, input: 0.16, output: 0.34, candidate: 0.03, next: 'quietly' },
  { token: 'quietly', forget: 0.93, input: 0.10, output: 0.26, candidate: 0.02, next: 'sat' },
  { token: 'sat', forget: 0.96, input: 0.32, output: 0.78, candidate: 0.08, next: 'on' },
  { token: 'mat', forget: 0.98, input: 0.14, output: 0.91, candidate: 0.05, next: '.' },
]

const step = ref(0)

const memoryStates = computed(() => {
  const states: number[] = []
  let memory = 0
  for (const item of sequence) {
    memory = Math.min(1, memory * item.forget + item.input * item.candidate)
    states.push(memory)
  }
  return states
})

const active = computed(() => sequence[step.value])
const previousMemory = computed(() => step.value ? memoryStates.value[step.value - 1] : 0)
const currentMemory = computed(() => memoryStates.value[step.value])
const revealedMemory = computed(() => currentMemory.value * active.value.output)

function move(delta: number) {
  step.value = Math.max(0, Math.min(sequence.length - 1, step.value + delta))
}
</script>

<template>
  <div class="slide-shell history-slide">
    <header class="history-header">
      <div class="history-title">
        <span class="step-index">04</span>
        <h1>Gates protect the useful signal.</h1>
      </div>
      <span class="era-chip">1997</span>
    </header>

    <main class="history-grid interactive-grid">
      <section class="concept-panel lstm-concept">
        <p class="panel-label">STEP THROUGH IT</p>
        <h2>Keep. Write. Reveal.</h2>
        <p v-click>Three gates control<br>one protected memory.</p>

        <div class="gate-guide">
          <div><span>F</span><strong>Forget</strong><small>keep old memory</small></div>
          <div><span>I</span><strong>Input</strong><small>write a new clue</small></div>
          <div><span>O</span><strong>Output</strong><small>reveal the state</small></div>
        </div>

        <div class="step-readout">
          <span>STEP {{ step + 1 }} / {{ sequence.length }}</span>
          <strong>{{ active.token }}</strong>
        </div>

        <div class="step-controls" aria-label="LSTM step controls">
          <button type="button" :disabled="step === 0" @click.stop="step = 0">
            <carbon-reset aria-hidden="true" /> Reset
          </button>
          <button type="button" :disabled="step === 0" @click.stop="move(-1)">
            <carbon-chevron-left aria-hidden="true" /> Previous
          </button>
          <button type="button" class="primary" :disabled="step === sequence.length - 1" @click.stop="move(1)">
            Next <carbon-chevron-right aria-hidden="true" />
          </button>
        </div>

        <p v-click class="tradeoff"><span>Gain</span> Useful clues survive distractions.</p>
      </section>

      <section class="example-panel lstm-example">
        <div class="example-label">GATED MEMORY · STEP THROUGH THE SEQUENCE</div>

        <div class="lstm-token-chain">
          <button
            v-for="(item, index) in sequence"
            :key="`${item.token}-${index}`"
            type="button"
            :class="{ processed: index < step, active: index === step, future: index > step }"
            :aria-pressed="index === step"
            :title="`Jump to ${item.token}`"
            @click.stop="step = index"
          >
            <span>{{ item.token }}</span><small>t{{ index + 1 }}</small>
          </button>
        </div>

        <div class="gate-dashboard">
          <div class="gate-meter forget">
            <header><span>FORGET</span><strong>{{ Math.round(active.forget * 100) }}%</strong></header>
            <i><b :style="{ width: `${active.forget * 100}%` }"></b></i>
            <small>retain prior state</small>
          </div>
          <div class="gate-meter input">
            <header><span>INPUT</span><strong>{{ Math.round(active.input * 100) }}%</strong></header>
            <i><b :style="{ width: `${active.input * 100}%` }"></b></i>
            <small>write candidate clue</small>
          </div>
          <div class="gate-meter output">
            <header><span>OUTPUT</span><strong>{{ Math.round(active.output * 100) }}%</strong></header>
            <i><b :style="{ width: `${active.output * 100}%` }"></b></i>
            <small>expose hidden state</small>
          </div>
        </div>

        <div class="cell-state-flow">
          <div class="memory-node previous">
            <span>PREVIOUS CELL</span>
            <strong>{{ Math.round(previousMemory * 100) }}%</strong>
            <i><b :style="{ width: `${previousMemory * 100}%` }"></b></i>
          </div>
          <span class="flow-arrow">× F + I × clue →</span>
          <div class="memory-node current">
            <span>CURRENT CELL</span>
            <strong>{{ Math.round(currentMemory * 100) }}%</strong>
            <i><b :style="{ width: `${currentMemory * 100}%` }"></b></i>
            <small>subject memory: “cat”</small>
          </div>
        </div>

        <div class="lstm-output">
          <div>
            <small>REVEALED STATE · C × O</small>
            <strong>{{ Math.round(revealedMemory * 100) }}%</strong>
          </div>
          <span>predict</span>
          <b>{{ active.next }}</b>
        </div>

        <p v-click class="example-note">Write <b>cat</b>. Keep it. Reveal it.</p>
      </section>
    </main>

    <footer class="history-footer">
      <span>HISTORY OF SEQUENCE MODELING</span>
      <span class="slide-progress">09 / 20</span>
      <ResourceLink href="https://direct.mit.edu/neco/article/9/8/1735/6109/Long-Short-Term-Memory" label="Hochreiter & Schmidhuber · Long Short-Term Memory (1997)" />
    </footer>
  </div>
</template>

<style scoped>
.lstm-concept {
  padding: 15px 17px;
}

.lstm-concept h2 {
  margin-bottom: 8px;
  font-size: 20px;
}

.lstm-concept > p:not(.panel-label):not(.tradeoff) {
  font-size: 12.5px !important;
  line-height: 1.4 !important;
}

.gate-guide {
  display: grid;
  gap: 5px;
  margin-top: 12px;
}

.gate-guide > div {
  display: grid;
  grid-template-columns: 25px 52px 1fr;
  align-items: center;
  min-height: 27px;
  padding: 4px 7px;
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid #2c322d;
  border-radius: 5px;
}

.gate-guide span {
  display: grid;
  width: 19px;
  height: 19px;
  place-items: center;
  color: #0b0e0c;
  background: var(--accent);
  border-radius: 4px;
  font: 800 7px/1 var(--deck-mono);
}

.gate-guide strong {
  color: #c9ceca;
  font-size: 9.5px;
}

.gate-guide small {
  color: #788079;
  font-size: 7.5px;
}

.step-readout {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-top: 10px;
  padding: 7px 9px;
  background: #0d100e;
  border: 1px solid #303631;
  border-radius: 5px;
}

.step-readout span {
  color: #737c74;
  font-size: 7px;
  font-weight: 750;
  letter-spacing: 0.09em;
}

.step-readout strong {
  color: var(--accent);
  font: 750 10px/1 var(--deck-mono);
}

.step-controls {
  display: grid;
  grid-template-columns: 0.8fr 1fr 1fr;
  gap: 5px;
  margin-top: 7px;
}

.step-controls button {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 3px;
  padding: 6px 3px;
  color: #a3aaa4;
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

.lstm-concept .tradeoff {
  right: 17px;
  bottom: 12px;
  left: 17px;
  font-size: 8px;
}

.lstm-example {
  display: flex;
  flex-direction: column;
}

.lstm-token-chain {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  gap: 4px;
}

.lstm-token-chain button {
  display: grid;
  min-width: 0;
  padding: 0;
  overflow: hidden;
  color: #919992;
  background: #111512;
  border: 1px solid #303631;
  border-radius: 5px;
  cursor: pointer;
}

.lstm-token-chain span,
.lstm-token-chain small {
  padding: 5px 3px;
  overflow: hidden;
  font: 650 7px/1 var(--deck-mono);
  text-overflow: ellipsis;
  white-space: nowrap;
}

.lstm-token-chain small {
  color: #667068;
  background: #0c0f0d;
  border-top: 1px solid #292f2a;
  font-size: 5.5px;
}

.lstm-token-chain button.processed {
  border-color: #485242;
}

.lstm-token-chain button.active {
  color: #0b0e0c;
  background: var(--accent);
  border-color: var(--accent);
}

.lstm-token-chain button.active small {
  color: var(--accent);
}

.lstm-token-chain button.future {
  opacity: 0.4;
}

.gate-dashboard {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 6px;
  margin-top: 10px;
}

.gate-meter {
  padding: 7px;
  background: rgba(255, 255, 255, 0.018);
  border: 1px solid #2c322d;
  border-radius: 6px;
}

.gate-meter header {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.gate-meter header span {
  color: #737c74;
  font-size: 6px;
  font-weight: 750;
  letter-spacing: 0.08em;
}

.gate-meter header strong {
  color: #d0d5d0;
  font: 700 7px/1 var(--deck-mono);
}

.gate-meter > i,
.memory-node > i {
  display: block;
  height: 5px;
  margin-top: 6px;
  overflow: hidden;
  background: #262c27;
  border-radius: 999px;
}

.gate-meter > i b,
.memory-node > i b {
  display: block;
  height: 100%;
  background: var(--accent);
  border-radius: inherit;
  transition: width 0.18s ease;
}

.gate-meter.input > i b {
  background: #75a6df;
}

.gate-meter.output > i b {
  background: #f0a65b;
}

.gate-meter > small {
  display: block;
  margin-top: 5px;
  color: #687068;
  font-size: 6px;
}

.cell-state-flow {
  display: grid;
  grid-template-columns: 1fr auto 1fr;
  align-items: center;
  gap: 8px;
  margin-top: 10px;
}

.memory-node {
  padding: 7px 8px;
  background: #0d100e;
  border: 1px solid #303631;
  border-radius: 6px;
}

.memory-node.current {
  border-color: rgba(119, 217, 255, 0.34);
}

.memory-node > span {
  color: #737c74;
  font-size: 6px;
  font-weight: 750;
  letter-spacing: 0.08em;
}

.memory-node > strong {
  float: right;
  color: var(--accent);
  font: 750 8px/1 var(--deck-mono);
}

.memory-node > small {
  display: block;
  margin-top: 4px;
  color: #7a837b;
  font-size: 6px;
}

.flow-arrow {
  color: #788079;
  font: 650 6px/1.2 var(--deck-mono);
  text-align: center;
}

.lstm-output {
  display: grid;
  grid-template-columns: 1fr auto auto;
  align-items: center;
  gap: 9px;
  margin-top: 9px;
  padding: 7px 9px;
  background: rgba(119, 217, 255, 0.045);
  border: 1px solid rgba(119, 217, 255, 0.2);
  border-radius: 6px;
}

.lstm-output > div {
  display: flex;
  align-items: baseline;
  gap: 7px;
}

.lstm-output small,
.lstm-output > span {
  color: #737c74;
  font-size: 6px;
  font-weight: 700;
  letter-spacing: 0.07em;
}

.lstm-output strong,
.lstm-output > b {
  color: var(--accent);
  font: 750 9px/1 var(--deck-mono);
}

@media (prefers-reduced-motion: reduce) {
  .gate-meter > i b,
  .memory-node > i b {
    transition: none;
  }
}
</style>
