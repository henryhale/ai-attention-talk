<script setup lang="ts">
import { computed, ref } from 'vue'

type Preset = {
  label: string
  pattern: string
  text: string
}

const presets: Preset[] = [
  {
    label: 'ENDING',
    pattern: String.raw`^The cat sat on the (mat|rug)\.$`,
    text: 'The cat sat on the mat.\nThe cat sat on the rug.\nThe cat sat on the floor.',
  },
  {
    label: 'REPEATED WORD',
    pattern: String.raw`\b([A-Za-z]+)\s+\1\b`,
    text: 'The cat cat sat down.\nWe learn by doing.\nThis is is a duplicate.',
  },
  {
    label: 'EMAIL',
    pattern: String.raw`[\w.+-]+@[\w.-]+\.[A-Za-z]{2,}`,
    text: 'Write to hello@example.com\nThis address is incomplete@\nTry team+ai@conway.dev',
  },
]

const pattern = ref(presets[0].pattern)
const testText = ref(presets[0].text)
const activePreset = ref(0)

function applyPreset(preset: Preset, index: number) {
  pattern.value = preset.pattern
  testText.value = preset.text
  activePreset.value = index
}

const compiledPattern = computed(() => {
  if (!pattern.value) return { regex: null, error: 'Enter a pattern to begin.' }
  try {
    return { regex: new RegExp(pattern.value, 'i'), error: '' }
  }
  catch (error) {
    return { regex: null, error: error instanceof Error ? error.message : 'Invalid regular expression.' }
  }
})

const results = computed(() => testText.value
  .split(/\n/)
  .filter(line => line.length)
  .map(line => {
    const regex = compiledPattern.value.regex
    if (!regex) return { line, matched: false, before: line, match: '', after: '' }
    const match = regex.exec(line)
    if (!match || match.index === undefined) return { line, matched: false, before: line, match: '', after: '' }
    const matchedText = match[0]
    return {
      line,
      matched: true,
      before: line.slice(0, match.index),
      match: matchedText,
      after: line.slice(match.index + matchedText.length),
    }
  }))

const matchCount = computed(() => results.value.filter(result => result.matched).length)
</script>

<template>
  <div class="slide-shell history-slide">
    <header class="history-header">
      <div class="history-title">
        <span class="step-index">02</span>
        <h1>Rules recognize only what we specify.</h1>
      </div>
      <span class="era-chip">1951 / 56</span>
    </header>

    <main class="history-grid interactive-grid">
      <section class="concept-panel regex-concept">
        <p class="panel-label">TRY IT</p>
        <h2>Write the pattern first.</h2>
        <p v-click>The answer is binary:<br><em>match or reject</em>.</p>

        <div class="preset-row" aria-label="Regular expression presets">
          <button
            v-for="(preset, index) in presets"
            :key="preset.label"
            type="button"
            :class="{ active: activePreset === index }"
            @click.stop="applyPreset(preset, index)"
          >
            {{ preset.label }}
          </button>
        </div>

        <label class="regex-field">
          <span>PATTERN · CASE INSENSITIVE</span>
          <input v-model="pattern" spellcheck="false" aria-label="Regular expression pattern" @input="activePreset = -1">
        </label>

        <label class="regex-field">
          <span>TEST TEXT · ONE EXAMPLE PER LINE</span>
          <textarea v-model="testText" rows="5" spellcheck="false" aria-label="Text to test" @input="activePreset = -1"></textarea>
        </label>

        <p v-click class="tradeoff"><span>Limit</span> Valid does not mean likely.</p>
      </section>

      <section class="example-panel regex-example">
        <div class="example-label">LIVE MATCHER · ACCEPT OR REJECT</div>

        <div class="pattern-card">
          <span>/</span><code>{{ pattern || '…' }}</code><span>/i</span>
        </div>

        <div v-if="compiledPattern.error" class="regex-error">
          <carbon-warning-alt aria-hidden="true" />
          <span>{{ compiledPattern.error }}</span>
        </div>

        <div v-else class="result-summary">
          <strong>{{ matchCount }}</strong>
          <span>of {{ results.length }} examples match</span>
          <i :style="{ width: `${results.length ? (matchCount / results.length) * 100 : 0}%` }"></i>
        </div>

        <div class="live-match-list">
          <div v-for="(result, index) in results" :key="`${result.line}-${index}`" :class="{ matched: result.matched }">
            <span class="match-status">{{ result.matched ? '✓' : '×' }}</span>
            <code>
              <span>{{ result.before }}</span><mark v-if="result.match">{{ result.match }}</mark><span>{{ result.after }}</span>
            </code>
            <b>{{ result.matched ? 'MATCH' : 'NO MATCH' }}</b>
          </div>
        </div>

        <p v-click class="example-note">Edit anything. Results update <b>instantly</b>.</p>
      </section>
    </main>

    <footer class="history-footer">
      <span>HISTORY OF SEQUENCE MODELING</span>
      <span class="slide-progress">07 / 20</span>
      <ResourceLink href="https://hjemmesider.diku.dk/~henglein/bib/publications/kleene56.html" label="Kleene · Representation of Events in Nerve Nets and Finite Automata (1956)" />
    </footer>
  </div>
</template>

<style scoped>
.regex-concept {
  padding: 16px 17px;
}

.regex-concept h2 {
  margin-bottom: 8px;
  font-size: 20px;
}

.regex-concept > p:not(.panel-label):not(.tradeoff) {
  font-size: 12.5px !important;
  line-height: 1.4 !important;
}

.preset-row {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 4px;
  margin-top: 10px;
}

.preset-row button {
  padding: 6px 3px;
  color: #7e877e;
  background: #101411;
  border: 1px solid #2e342f;
  border-radius: 4px;
  font-size: 6.5px;
  font-weight: 750;
  cursor: pointer;
}

.preset-row button.active {
  color: #0b0e0c;
  background: var(--accent);
  border-color: var(--accent);
}

.regex-field {
  display: grid;
  gap: 4px;
  margin-top: 9px;
}

.regex-field > span {
  color: #6f7870;
  font-size: 6.2px;
  font-weight: 750;
  letter-spacing: 0.09em;
}

.regex-field input,
.regex-field textarea {
  width: 100%;
  box-sizing: border-box;
  padding: 6px 7px;
  color: #c7ccc8;
  background: #0c0f0d;
  border: 1px solid #303631;
  border-radius: 5px;
  outline: none;
  font: 8px/1.35 var(--deck-mono);
}

.regex-field input {
  height: 26px;
}

.regex-field textarea {
  height: 55px;
  resize: none;
}

.regex-field input:focus,
.regex-field textarea:focus {
  border-color: rgba(119, 217, 255, 0.48);
}

.regex-concept .tradeoff {
  right: 17px;
  bottom: 13px;
  left: 17px;
  font-size: 9.5px;
}

.regex-example {
  display: flex;
  flex-direction: column;
}

.pattern-card {
  display: grid;
  grid-template-columns: auto minmax(0, 1fr) auto;
  align-items: center;
  gap: 3px;
  padding: 8px 9px;
  color: var(--accent);
  background: #0b0e0c;
  border: 1px solid #2d332e;
  border-radius: 6px;
  font: 700 8px/1 var(--deck-mono);
}

.pattern-card code {
  overflow: hidden;
  color: #cbd1cc;
  font-size: 8px;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.regex-error {
  display: flex;
  align-items: center;
  gap: 7px;
  margin-top: 8px;
  padding: 8px 9px;
  color: #e38a8a;
  background: rgba(225, 116, 116, 0.06);
  border: 1px solid rgba(225, 116, 116, 0.22);
  border-radius: 6px;
  font-size: 7px;
}

.regex-error svg {
  flex: none;
  width: 11px;
}

.result-summary {
  position: relative;
  display: flex;
  align-items: baseline;
  gap: 6px;
  margin-top: 8px;
  padding: 6px 8px 8px;
  overflow: hidden;
  background: rgba(255, 255, 255, 0.018);
  border-radius: 5px;
}

.result-summary strong {
  color: var(--accent);
  font: 750 12px/1 var(--deck-mono);
}

.result-summary span {
  color: #747d75;
  font-size: 7px;
}

.result-summary i {
  position: absolute;
  bottom: 0;
  left: 0;
  height: 2px;
  background: var(--accent);
}

.live-match-list {
  display: grid;
  gap: 5px;
  margin-top: 9px;
}

.live-match-list > div {
  display: grid;
  grid-template-columns: 18px minmax(0, 1fr) auto;
  align-items: center;
  min-height: 29px;
  padding: 4px 7px;
  background: rgba(255, 255, 255, 0.018);
  border: 1px solid #292e2a;
  border-radius: 5px;
}

.live-match-list > div.matched {
  border-color: rgba(119, 217, 255, 0.22);
}

.match-status {
  color: #e17474;
  font-size: 11px;
  font-weight: 750;
}

.matched .match-status {
  color: var(--accent);
}

.live-match-list code {
  min-width: 0;
  overflow: hidden;
  color: #8f9790;
  font-size: 7.5px;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.live-match-list mark {
  padding: 2px 1px;
  color: #0b0e0c;
  background: var(--accent);
  border-radius: 2px;
}

.live-match-list b {
  color: #737c74;
  font-size: 5.5px;
  letter-spacing: 0.08em;
}

.matched b {
  color: #a8b58f;
}
</style>
