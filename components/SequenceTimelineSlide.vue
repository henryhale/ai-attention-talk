<script setup lang="ts">
const milestones = [
  { year: '1948', title: 'N-grams', verb: 'COUNT', detail: 'Short fixed windows', tone: 'past' },
  { year: '1956', title: 'Rules', verb: 'MATCH', detail: 'Hand-built patterns', tone: 'past' },
  { year: '1990', title: 'RNNs', verb: 'COMPRESS', detail: 'Running hidden state', tone: 'past' },
  { year: '1997', title: 'LSTMs', verb: 'GATE', detail: 'Selective memory', tone: 'past' },
  { year: '2017', title: 'Transformers', verb: 'ATTEND', detail: 'Direct token access', tone: 'turn' },
  { year: 'NOW', title: 'Frontier', verb: 'COMBINE', detail: 'Dense · sparse · state · retrieval', tone: 'now' },
]
</script>

<template>
  <div class="slide-shell sequence-timeline-slide">
    <header class="timeline-header">
      <p class="eyebrow"><span class="pulse-dot"></span> THE MAP</p>
      <span class="timeline-progress">05 / 20</span>
    </header>

    <section class="timeline-title">
      <div>
        <span class="timeline-kicker">ONE TASK · SEVEN DECADES</span>
        <h1>Same prediction.<br><span>Better access to context.</span></h1>
      </div>
      <p v-click>Every era changes what “history” means—and how much of it the model can use.</p>
    </section>

    <main class="timeline-track" aria-label="Timeline of sequence-modeling approaches">
      <div class="timeline-line" aria-hidden="true"></div>
      <article
        v-for="milestone in milestones"
        v-click
        :key="milestone.year"
        class="milestone"
        :class="milestone.tone"
      >
        <span class="milestone-year">{{ milestone.year }}</span>
        <i class="milestone-dot" aria-hidden="true"></i>
        <div class="milestone-card">
          <small>{{ milestone.verb }}</small>
          <strong>{{ milestone.title }}</strong>
          <p>{{ milestone.detail }}</p>
        </div>
      </article>
    </main>

    <section v-click class="timeline-frontier">
      <span>FRONTIER QUESTION</span>
      <strong>What context is useful—and what is the cheapest way to reach it?</strong>
      <i>→</i>
    </section>

    <footer class="timeline-footer">
      <span>NEXT: BEGIN WITH THE SMALLEST MEMORY</span>
      <strong>N-GRAMS →</strong>
    </footer>
  </div>
</template>

<style scoped>
.sequence-timeline-slide {
  padding: 34px 46px 24px;
}

.timeline-header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  padding-bottom: 14px;
  border-bottom: 1px solid #292e29;
}

.timeline-header .eyebrow {
  font-size: 9px;
}

.timeline-progress {
  color: var(--accent);
  font: 750 9px/1 var(--deck-mono);
  letter-spacing: 0.12em;
}

.timeline-title {
  display: flex;
  align-items: end;
  justify-content: space-between;
  padding: 19px 0 12px;
}

.timeline-kicker {
  color: var(--accent);
  font-size: 7px;
  font-weight: 750;
  letter-spacing: 0.15em;
}

.timeline-title h1 {
  margin-top: 7px;
  font-size: 36px;
  line-height: 0.98;
}

.timeline-title h1 span {
  color: var(--accent);
}

.timeline-title > p {
  width: 250px;
  margin: 0 0 2px;
  color: var(--muted);
  font-size: 9.5px;
  line-height: 1.45;
  text-align: right;
}

.timeline-track {
  position: relative;
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  gap: 8px;
  min-height: 139px;
  padding-top: 3px;
}

.timeline-line {
  position: absolute;
  top: 35px;
  right: 7.5%;
  left: 7.5%;
  height: 1px;
  background: linear-gradient(90deg, #353c36 0 68%, rgba(119, 217, 255, 0.65) 86%, var(--accent));
}

.milestone {
  position: relative;
  display: grid;
  grid-template-rows: 18px 28px 1fr;
  justify-items: center;
  min-width: 0;
}

.milestone-year {
  color: #717a72;
  font: 700 7px/1 var(--deck-mono);
}

.milestone-dot {
  position: relative;
  z-index: 1;
  display: block;
  width: 7px;
  height: 7px;
  margin-top: 11px;
  background: #111512;
  border: 2px solid #697269;
  border-radius: 999px;
}

.milestone.turn .milestone-dot,
.milestone.now .milestone-dot {
  border-color: var(--accent);
}

.milestone.now .milestone-dot {
  background: var(--accent);
  box-shadow: 0 0 17px rgba(119, 217, 255, 0.48);
}

.milestone-card {
  width: 100%;
  box-sizing: border-box;
  padding: 9px 9px 8px;
  background: rgba(255, 255, 255, 0.018);
  border: 1px solid #292f2a;
  border-radius: 8px;
}

.milestone.turn .milestone-card {
  border-color: rgba(119, 217, 255, 0.25);
}

.milestone.now .milestone-card {
  background:
    radial-gradient(circle at 100% 0, rgba(119, 217, 255, 0.13), transparent 48%),
    rgba(119, 217, 255, 0.035);
  border-color: rgba(119, 217, 255, 0.48);
}

.milestone-card small {
  color: #737c74;
  font-size: 5.5px;
  font-weight: 750;
  letter-spacing: 0.12em;
}

.milestone.turn .milestone-card small,
.milestone.now .milestone-card small {
  color: var(--accent);
}

.milestone-card strong {
  display: block;
  margin-top: 6px;
  color: var(--ink);
  font-size: 11px;
  font-weight: 650;
}

.milestone-card p {
  margin: 4px 0 0;
  color: #747d75;
  font-size: 6.5px;
  line-height: 1.3;
}

.timeline-frontier {
  display: grid;
  grid-template-columns: 100px 1fr 18px;
  align-items: center;
  gap: 14px;
  margin-top: 11px;
  padding: 11px 14px;
  background: rgba(119, 217, 255, 0.035);
  border: 1px solid rgba(119, 217, 255, 0.25);
  border-radius: 8px;
}

.timeline-frontier span {
  color: var(--accent);
  font-size: 6px;
  font-weight: 750;
  letter-spacing: 0.12em;
}

.timeline-frontier strong {
  color: #cbd1cc;
  font-size: 10px;
  font-weight: 550;
}

.timeline-frontier i {
  color: var(--accent);
  font-style: normal;
}

.timeline-footer {
  display: flex;
  align-items: end;
  justify-content: space-between;
  margin-top: auto;
  padding-top: 10px;
  color: #646d65;
  border-top: 1px solid #292e29;
  font-size: 6.5px;
  font-weight: 700;
  letter-spacing: 0.1em;
}

.timeline-footer strong {
  color: var(--accent);
  font-family: var(--deck-mono);
}
</style>
