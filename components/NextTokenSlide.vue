<script setup lang="ts">
const steps = [
  { context: ['<START>'], next: 'The' },
  { context: ['The'], next: 'curious' },
  { context: ['The', 'curious'], next: 'robot' },
  { context: ['The', 'curious', 'robot'], next: 'opened' },
  { context: ['The', 'curious', 'robot', 'opened'], next: 'the' },
  { context: ['The', 'curious', 'robot', 'opened', 'the'], next: 'door' },
]
</script>

<template>
  <div class="slide-shell next-token-slide">
    <header class="next-token-header">
      <div>
        <span class="step-index">THE REPEATED DECISION</span>
        <h1>Predict one token. Add it. Repeat.</h1>
      </div>
      <span class="next-token-progress">04 / 20</span>
    </header>

    <main class="next-token-main">
      <section class="triangle-panel">
        <div class="triangle-heading">
          <span>AVAILABLE HISTORY</span>
          <span>SELECTED NEXT TOKEN</span>
        </div>

        <div class="token-triangle" aria-label="A sentence growing one predicted token at a time">
          <div v-for="(step, rowIndex) in steps" v-click :key="step.next" class="triangle-row">
            <span class="row-index">0{{ rowIndex + 1 }}</span>
            <div class="history-tokens">
              <span
                v-for="(token, tokenIndex) in step.context"
                :key="`${token}-${tokenIndex}`"
                :class="{ start: token === '<START>' }"
              >{{ token }}</span>
            </div>
            <i>→</i>
            <strong>{{ step.next }}</strong>
          </div>
        </div>
      </section>

      <aside class="decision-panel">
        <p class="panel-label">AT EVERY STEP</p>
        <h2>History becomes a probability distribution.</h2>
        <p>The model scores every possible token, selects one, then folds it into the next context.</p>

        <div v-click class="distribution-card">
          <small>P( NEXT TOKEN | “THE CURIOUS ROBOT” )</small>
          <div class="distribution-row winner">
            <span>opened</span><i><b style="width: 72%"></b></i><strong>0.72</strong>
          </div>
          <div class="distribution-row">
            <span>waited</span><i><b style="width: 17%"></b></i><strong>0.17</strong>
          </div>
          <div class="distribution-row">
            <span>was</span><i><b style="width: 8%"></b></i><strong>0.08</strong>
          </div>
        </div>

        <div v-click class="decision-loop">
          <span>CONTEXT</span><i>→</i><span>SCORES</span><i>→</i><strong>TOKEN</strong><i>↺</i>
        </div>

        <p v-click class="decision-takeaway"><span>The approaches differ in one place:</span><br>how they represent and retrieve the history.</p>
      </aside>
    </main>

    <footer class="next-token-footer">
      <span>NEXT-TOKEN PREDICTION · CAUSAL CONTEXT</span>
      <span>THE LOWER TRIANGLE GROWS ONE ROW AT A TIME</span>
    </footer>
  </div>
</template>

<style scoped>
.next-token-slide {
  padding: 30px 42px 21px;
}

.next-token-header {
  flex: 0 0 52px;
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  border-bottom: 1px solid #292e29;
}

.next-token-header > div {
  display: flex;
  align-items: baseline;
  gap: 14px;
}

.next-token-header h1 {
  font-size: 34px;
  line-height: 1;
}

.next-token-progress {
  color: var(--accent);
  font: 750 9px/1 var(--deck-mono);
  letter-spacing: 0.12em;
}

.next-token-main {
  flex: 1;
  display: grid;
  grid-template-columns: minmax(0, 1.28fr) minmax(0, 0.72fr);
  gap: 16px;
  min-height: 0;
  padding: 15px 0 12px;
}

.triangle-panel,
.decision-panel {
  min-width: 0;
  min-height: 0;
  padding: 15px 17px;
  overflow: hidden;
  border: 1px solid #292f2a;
  border-radius: 11px;
}

.triangle-panel {
  background:
    linear-gradient(135deg, rgba(119, 217, 255, 0.035), transparent 42%),
    rgba(255, 255, 255, 0.018);
}

.triangle-heading {
  display: grid;
  grid-template-columns: 1fr 128px;
  padding: 0 9px 8px 34px;
  color: #6d766e;
  border-bottom: 1px solid #282e29;
  font-size: 6px;
  font-weight: 750;
  letter-spacing: 0.12em;
}

.triangle-heading span:last-child {
  text-align: right;
}

.token-triangle {
  display: grid;
  gap: 5px;
  margin-top: 9px;
}

.triangle-row {
  display: grid;
  grid-template-columns: 24px 1fr 14px 59px;
  align-items: center;
  gap: 7px;
  min-height: 33px;
}

.row-index {
  color: #505951;
  font: 650 6px/1 var(--deck-mono);
}

.history-tokens {
  display: flex;
  align-items: center;
  gap: 4px;
  min-width: 0;
}

.history-tokens span {
  padding: 6px 7px;
  color: #a3aba4;
  background: #111512;
  border: 1px solid #2d342e;
  border-radius: 5px;
  font: 600 7.5px/1 var(--deck-mono);
  white-space: nowrap;
}

.history-tokens span.start {
  color: #626b63;
  border-style: dashed;
}

.triangle-row > i {
  color: #4d564e;
  font-style: normal;
  font-size: 9px;
}

.triangle-row > strong {
  padding: 7px 6px;
  color: #0b0e0c;
  background: var(--accent);
  border-radius: 5px;
  font: 700 7.5px/1 var(--deck-mono);
  text-align: center;
  box-shadow: 0 0 18px rgba(119, 217, 255, 0.07);
}

.decision-panel {
  position: relative;
  background:
    radial-gradient(circle at 100% 0, rgba(119, 217, 255, 0.07), transparent 35%),
    rgba(255, 255, 255, 0.024);
}

.decision-panel h2 {
  margin: 0;
  font-size: 20px;
  line-height: 1.08;
}

.decision-panel > p:not(.panel-label):not(.decision-takeaway) {
  margin: 9px 0 0;
  color: var(--muted);
  font-size: 9px;
  line-height: 1.45;
}

.distribution-card {
  margin-top: 15px;
  padding: 10px;
  background: #0c0f0d;
  border: 1px solid #2d332e;
  border-radius: 7px;
}

.distribution-card > small {
  color: #717a72;
  font-size: 5.2px;
  font-weight: 750;
  letter-spacing: 0.08em;
}

.distribution-row {
  display: grid;
  grid-template-columns: 45px 1fr 25px;
  align-items: center;
  gap: 7px;
  margin-top: 7px;
  color: #7c857d;
  font: 600 7px/1 var(--deck-mono);
}

.distribution-row i {
  height: 4px;
  overflow: hidden;
  background: #242a25;
  border-radius: 999px;
}

.distribution-row i b {
  display: block;
  height: 100%;
  background: #606961;
  border-radius: inherit;
}

.distribution-row strong {
  font-weight: 600;
  text-align: right;
}

.distribution-row.winner {
  color: var(--ink);
}

.distribution-row.winner i b {
  background: var(--accent);
}

.decision-loop {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-top: 12px;
  padding: 8px 9px;
  color: #747d75;
  background: rgba(255, 255, 255, 0.018);
  border-radius: 6px;
  font: 700 6px/1 var(--deck-mono);
  letter-spacing: 0.07em;
}

.decision-loop i {
  color: #505850;
  font-style: normal;
}

.decision-loop strong {
  color: var(--accent);
}

.decision-takeaway {
  position: absolute;
  right: 17px;
  bottom: 14px;
  left: 17px;
  margin: 0;
  padding-top: 10px;
  color: #79827a;
  border-top: 1px solid #292f2a;
  font-size: 8.5px;
  line-height: 1.4;
}

.decision-takeaway span {
  color: var(--ink);
  font-weight: 650;
}

.next-token-footer {
  flex: 0 0 23px;
  display: flex;
  align-items: end;
  justify-content: space-between;
  padding-top: 9px;
  color: #646d65;
  border-top: 1px solid #292e29;
  font-size: 6.5px;
  font-weight: 700;
  letter-spacing: 0.1em;
}
</style>
