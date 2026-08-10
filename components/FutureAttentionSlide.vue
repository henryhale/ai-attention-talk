<template>
  <div class="slide-shell future-slide">
    <header class="future-header">
      <p class="eyebrow"><span class="pulse-dot"></span> WHERE I THINK THIS GOES</p>
      <span class="future-progress">18 / 20</span>
    </header>

    <main class="future-main">
      <section class="future-title">
        <span>MY PREDICTION</span>
        <h1>The next paradigm will learn online—<br><strong>and remember selectively.</strong></h1>
        <p>A recurrent learner with content-addressed memory and multi-step predictive objectives.</p>
      </section>

      <section v-click class="today-flow" aria-label="Today's dominant language model design">
        <small>TODAY</small>
        <div>
          <span>ALL TOKEN STATES</span><i>→</i><span>ATTENTION LOOKUP</span><i>→</i><strong>NEXT-TOKEN DISTRIBUTION</strong>
        </div>
      </section>

      <section v-click class="replacement-stack" aria-label="Predicted replacement architecture">
        <header>
          <small>LIKELY SUCCESSOR</small>
          <span>THE REPLACEMENT STACK</span>
        </header>

        <article>
          <b>01</b>
          <div><strong>Online recurrent learner</strong><p>Update a compact streaming state with every token.</p></div>
        </article>

        <article>
          <b>02</b>
          <div><strong>Selective episodic memory</strong><p>Store useful or surprising events—not every activation.</p></div>
        </article>

        <article>
          <b>03</b>
          <div><strong>Hierarchical retrieval</strong><p>Search summaries first, then retrieve the relevant detail.</p></div>
        </article>

        <article>
          <b>04</b>
          <div><strong>Latent + multi-step prediction</strong><p>Learn toward future states, not only the immediate token.</p></div>
        </article>

        <div class="stack-output"><i>↓</i><strong>NEXT-TOKEN DISTRIBUTION</strong></div>
      </section>
    </main>
  </div>
</template>

<style scoped>
.future-slide {
  padding: 25px 42px 24px;
}

.future-header {
  flex: 0 0 38px;
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  border-bottom: 1px solid #292e33;
}

.future-header .eyebrow {
  font-size: 8.5px;
}

.future-progress {
  color: var(--accent);
  font: 750 8.5px/1 var(--deck-mono);
  letter-spacing: 0.12em;
}

.future-main {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: 0;
  padding-top: 13px;
}

.future-title > span {
  color: var(--accent);
  font-size: 6.5px;
  font-weight: 750;
  letter-spacing: 0.15em;
}

.future-title h1 {
  margin-top: 6px;
  font-size: 32px;
  line-height: 0.99;
}

.future-title h1 strong {
  color: var(--accent);
  font-weight: inherit;
}

.future-title p {
  margin: 8px 0 0;
  color: var(--muted);
  font-size: 9.5px;
  line-height: 1.4;
}

.today-flow {
  display: grid;
  grid-template-columns: 56px 1fr;
  align-items: center;
  gap: 12px;
  margin-top: 12px;
  padding: 9px 12px;
  background: rgba(255, 255, 255, 0.018);
  border: 1px solid #2a3237;
  border-radius: 7px;
}

.today-flow > small {
  color: #738087;
  font: 750 6px/1 var(--deck-mono);
  letter-spacing: 0.12em;
}

.today-flow > div {
  display: grid;
  grid-template-columns: 1fr 18px 1fr 18px 1.25fr;
  align-items: center;
  gap: 6px;
}

.today-flow span,
.today-flow strong {
  padding: 6px 8px;
  color: #8d989d;
  background: #101519;
  border-radius: 5px;
  font: 700 6px/1 var(--deck-mono);
  letter-spacing: 0.06em;
  text-align: center;
}

.today-flow strong {
  color: #b9c3c7;
}

.today-flow i {
  color: #56636a;
  font-style: normal;
  text-align: center;
}

.replacement-stack {
  flex: 1;
  display: grid;
  grid-template-rows: 22px repeat(4, 1fr) 31px;
  gap: 5px;
  min-height: 0;
  margin-top: 10px;
  padding: 10px 12px 9px;
  background:
    radial-gradient(circle at 100% 0, rgba(119, 217, 255, 0.09), transparent 38%),
    rgba(119, 217, 255, 0.022);
  border: 1px solid rgba(119, 217, 255, 0.3);
  border-radius: 10px;
}

.replacement-stack > header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-bottom: 1px solid #293239;
}

.replacement-stack > header small,
.replacement-stack > header span {
  font: 750 5.5px/1 var(--deck-mono);
  letter-spacing: 0.12em;
}

.replacement-stack > header small {
  color: var(--accent);
}

.replacement-stack > header span {
  color: #6e7b82;
}

.replacement-stack article {
  display: grid;
  grid-template-columns: 36px 1fr;
  align-items: center;
  gap: 11px;
  min-height: 0;
  padding: 7px 11px;
  background: #0f1417;
  border: 1px solid #2b353a;
  border-radius: 7px;
}

.replacement-stack article > b {
  display: grid;
  width: 27px;
  height: 27px;
  place-items: center;
  color: #071117;
  background: var(--accent);
  border-radius: 7px;
  font: 750 6px/1 var(--deck-mono);
}

.replacement-stack article > div {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  gap: 18px;
}

.replacement-stack article strong {
  flex: 0 0 230px;
  color: var(--ink);
  font-size: 10.5px;
}

.replacement-stack article p {
  flex: 1;
  margin: 0;
  color: #849097;
  font-size: 7.5px;
  line-height: 1.35;
}

.stack-output {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
}

.stack-output i {
  color: #607079;
  font-style: normal;
}

.stack-output strong {
  padding: 7px 13px;
  color: #071117;
  background: var(--accent);
  border-radius: 999px;
  font: 750 6.5px/1 var(--deck-mono);
  letter-spacing: 0.1em;
}
</style>
