<template>
  <div class="slide-shell position-slide">
    <header class="position-header">
      <p class="eyebrow"><span class="pulse-dot"></span> MY POSITION</p>
      <span class="position-progress">17 / 20</span>
    </header>

    <main class="position-main">
      <div class="position-title">
        <h1>Full attention wins small.<br><span>Hybrid scales farther.</span></h1>
      </div>

      <section class="context-ladder" aria-label="Attention strategy by context size">
        <article v-click class="context-card full">
          <span class="card-number">01</span>
          <small>SMALL CONTEXT</small>
          <strong>Full attention</strong>
          <p>Let every token reach every earlier token.</p>
        </article>

        <i>→</i>

        <article v-click class="context-card flash">
          <span class="card-number">02</span>
          <small>MEDIUM CONTEXT</small>
          <strong>Full + Flash</strong>
          <p>Keep the same connections with less memory traffic.</p>
        </article>

        <i>→</i>

        <article v-click class="context-card hybrid">
          <span class="card-number">03</span>
          <small>LARGE CONTEXT</small>
          <strong>Hybrid attention</strong>
          <p>Combine local routes with selected global context.</p>
        </article>
      </section>
    </main>
  </div>
</template>

<style scoped>
.position-slide {
  padding: 27px 42px 28px;
}

.position-header {
  flex: 0 0 39px;
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  border-bottom: 1px solid #292e33;
}

.position-header .eyebrow {
  font-size: 9px;
}

.position-progress {
  color: var(--accent);
  font: 750 9px/1 var(--deck-mono);
  letter-spacing: 0.12em;
}

.position-main {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  min-height: 0;
  padding-top: 9px;
}

.position-title {
  padding-left: 4px;
  text-align: left;
}

.position-title h1 {
  font-size: 44px;
  line-height: 0.98;
}

.position-title h1 span {
  color: var(--accent);
}

.context-ladder {
  display: grid;
  grid-template-columns: 1fr 28px 1fr 28px 1fr;
  align-items: stretch;
  gap: 8px;
  margin-top: 23px;
}

.context-ladder > i {
  align-self: center;
  display: grid;
  width: 26px;
  height: 26px;
  place-items: center;
  color: var(--accent);
  background: #11171b;
  border: 1px solid #303a40;
  border-radius: 50%;
  font-style: normal;
  font-size: 11px;
  text-align: center;
}

.context-card {
  position: relative;
  display: flex;
  min-height: 162px;
  flex-direction: column;
  justify-content: center;
  padding: 24px 21px 20px;
  overflow: hidden;
  background:
    linear-gradient(145deg, rgba(255, 255, 255, 0.035), transparent 58%),
    #0d1114;
  border: 1px solid #30383d;
  border-radius: 11px;
  box-shadow: 0 16px 32px rgba(0, 0, 0, 0.2);
}

.context-card::before {
  content: '';
  position: absolute;
  top: 0;
  right: 0;
  left: 0;
  height: 3px;
  background: #46545c;
}

.card-number {
  position: absolute;
  top: 12px;
  right: 15px;
  color: rgba(255, 255, 255, 0.06);
  font: 750 34px/1 var(--deck-mono);
}

.context-card small {
  color: var(--accent);
  font: 750 7.5px/1 var(--deck-mono);
  letter-spacing: 0.12em;
}

.context-card strong {
  margin-top: 16px;
  color: var(--ink);
  font-size: 21px;
  line-height: 1.05;
}

.context-card p {
  margin: 12px 0 0;
  color: #929da2;
  font-size: 10px;
  line-height: 1.45;
}

.context-card.full,
.context-card.hybrid {
  border-color: rgba(119, 217, 255, 0.34);
}

.context-card.full::before,
.context-card.hybrid::before {
  background: var(--accent);
}

.context-card.full strong,
.context-card.hybrid strong {
  color: var(--accent);
}

.context-card.hybrid {
  background:
    radial-gradient(circle at 100% 0, rgba(119, 217, 255, 0.13), transparent 46%),
    linear-gradient(145deg, rgba(119, 217, 255, 0.05), transparent 62%),
    #0d1114;
}
</style>
