<script setup lang="ts">
const tasks = [
  {
    label: 'SELECTED TASK',
    title: 'Next-token prediction',
    detail: 'Choose one token from the vocabulary.',
    active: true,
  },
  {
    label: 'OTHER TASK',
    title: 'Sequence classification',
    detail: 'Choose one label for the whole sequence.',
    active: false,
  },
  {
    label: 'OTHER TASK',
    title: 'Sequence-to-sequence',
    detail: 'Transform one sequence into another.',
    active: false,
  },
]
</script>

<template>
  <div class="slide-shell sequence-intro-slide">
    <header class="sequence-intro-header">
      <p class="eyebrow"><span class="pulse-dot"></span> THE PROBLEM</p>
      <span class="sequence-intro-progress">03 / 20</span>
    </header>

    <main class="sequence-intro-main">
      <section class="sequence-intro-copy">
        <span class="sequence-kicker">SEQUENCE MODELING</span>
        <h1>Use what came before<br>to decide <span>what comes next.</span></h1>
        <p v-click>Language arrives in order. Each new token changes the context available for the next decision.</p>

        <div v-click class="sequence-question" aria-label="Next-token prediction example">
          <small>THE QUESTION WE WILL FOLLOW</small>
          <div>
            <span>The model reads the</span>
            <strong>context</strong>
            <i>→</i>
            <b>?</b>
          </div>
        </div>
      </section>

      <section class="task-stack" aria-label="Sequence modeling tasks">
        <article
          v-for="task in tasks"
          v-click
          :key="task.title"
          class="task-card"
          :class="{ active: task.active }"
        >
          <span class="task-radio" aria-hidden="true"><i></i></span>
          <div>
            <small>{{ task.label }}</small>
            <h2>{{ task.title }}</h2>
            <p>{{ task.detail }}</p>
          </div>
          <carbon-arrow-right v-if="task.active" class="task-arrow" aria-hidden="true" />
        </article>
      </section>
    </main>

    <footer class="sequence-intro-footer">
      <span>ONE OBJECTIVE · MANY WAYS TO REPRESENT HISTORY</span>
      <strong>CONTEXT → DISTRIBUTION → TOKEN</strong>
    </footer>
  </div>
</template>

<style scoped>
.sequence-intro-slide {
  padding: 38px 52px 28px;
}

.sequence-intro-header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  padding-bottom: 15px;
  border-bottom: 1px solid #292e29;
}

.sequence-intro-header .eyebrow {
  font-size: 9px;
}

.sequence-intro-progress {
  color: var(--accent);
  font: 750 9px/1 var(--deck-mono);
  letter-spacing: 0.12em;
}

.sequence-intro-main {
  flex: 1;
  display: grid;
  grid-template-columns: minmax(0, 1.04fr) minmax(0, 0.96fr);
  align-items: center;
  gap: 48px;
  min-height: 0;
  padding: 23px 0 18px;
}

.sequence-kicker {
  color: var(--accent);
  font-size: 8px;
  font-weight: 750;
  letter-spacing: 0.16em;
}

.sequence-intro-copy h1 {
  margin-top: 12px;
  font-size: 45px;
  line-height: 0.98;
}

.sequence-intro-copy h1 span {
  color: var(--accent);
}

.sequence-intro-copy > p {
  max-width: 390px;
  margin: 17px 0 0;
  color: var(--muted);
  font-size: 12px;
  line-height: 1.5;
}

.sequence-question {
  margin-top: 23px;
  padding: 12px 14px;
  background: rgba(255, 255, 255, 0.022);
  border: 1px solid #2b312c;
  border-radius: 9px;
}

.sequence-question small {
  color: #737c74;
  font-size: 6px;
  font-weight: 750;
  letter-spacing: 0.13em;
}

.sequence-question div {
  display: flex;
  align-items: center;
  gap: 7px;
  margin-top: 8px;
  font-family: var(--deck-mono);
  font-size: 9px;
}

.sequence-question div > span {
  color: #8b948c;
}

.sequence-question strong {
  padding: 5px 7px;
  color: var(--ink);
  background: #151a16;
  border: 1px solid #3a423b;
  border-radius: 5px;
  font-weight: 600;
}

.sequence-question i {
  color: #535c54;
  font-style: normal;
}

.sequence-question b {
  display: grid;
  width: 28px;
  height: 28px;
  place-items: center;
  color: #0b0e0c;
  background: var(--accent);
  border-radius: 6px;
  font-size: 13px;
}

.task-stack {
  display: grid;
  gap: 9px;
}

.task-card {
  display: grid;
  grid-template-columns: 23px 1fr 16px;
  align-items: center;
  gap: 12px;
  min-height: 74px;
  padding: 12px 14px;
  color: #737c74;
  background: rgba(255, 255, 255, 0.014);
  border: 1px solid #252a26;
  border-radius: 10px;
}

.task-card.active {
  color: var(--ink);
  background:
    radial-gradient(circle at 100% 0, rgba(119, 217, 255, 0.1), transparent 40%),
    rgba(119, 217, 255, 0.035);
  border-color: rgba(119, 217, 255, 0.38);
}

.task-radio {
  display: grid;
  width: 17px;
  height: 17px;
  place-items: center;
  border: 1px solid #485049;
  border-radius: 999px;
}

.task-card.active .task-radio {
  border-color: var(--accent);
}

.task-card.active .task-radio i {
  width: 7px;
  height: 7px;
  background: var(--accent);
  border-radius: inherit;
}

.task-card small {
  color: #656e66;
  font-size: 5.5px;
  font-weight: 750;
  letter-spacing: 0.12em;
}

.task-card.active small {
  color: var(--accent);
}

.task-card h2 {
  margin: 4px 0 0;
  color: currentColor;
  font-size: 14px;
  font-weight: 650;
}

.task-card p {
  margin: 3px 0 0;
  color: #707971;
  font-size: 7.5px;
}

.task-arrow {
  width: 13px;
  color: var(--accent);
}

.sequence-intro-footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-top: 12px;
  color: #646c65;
  border-top: 1px solid #292e29;
  font-size: 7px;
  font-weight: 700;
  letter-spacing: 0.1em;
}

.sequence-intro-footer strong {
  color: #8d968e;
  font-family: var(--deck-mono);
  font-weight: 650;
}
</style>
