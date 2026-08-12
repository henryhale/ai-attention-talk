<script setup lang="ts">
const groups = [
  {
    label: 'FOUNDATIONS',
    papers: [
      {
        year: '1948',
        title: 'A Mathematical Theory of Communication',
        authors: 'Claude Shannon',
        href: 'https://people.math.harvard.edu/~ctm/home/text/others/shannon/entropy/entropy.pdf',
      },
      {
        year: '1956',
        title: 'Representation of Events in Nerve Nets',
        authors: 'Stephen Kleene',
        href: 'https://hjemmesider.diku.dk/~henglein/bib/publications/kleene56.html',
      },
      {
        year: '1990',
        title: 'Finding Structure in Time',
        authors: 'Jeffrey Elman',
        href: 'https://doi.org/10.1207/s15516709cog1402_1',
      },
      {
        year: '1997',
        title: 'Long Short-Term Memory',
        authors: 'Hochreiter & Schmidhuber',
        href: 'https://direct.mit.edu/neco/article/9/8/1735/6109/Long-Short-Term-Memory',
      },
    ],
  },
  {
    label: 'ATTENTION AT SCALE',
    papers: [
      {
        year: '2017',
        title: 'Attention Is All You Need',
        authors: 'Vaswani et al.',
        href: 'https://arxiv.org/abs/1706.03762',
      },
      {
        year: '2019',
        title: 'Generating Long Sequences',
        authors: 'Child et al. · Sparse Transformer',
        href: 'https://arxiv.org/abs/1904.10509',
      },
      {
        year: '2020',
        title: 'Longformer',
        authors: 'Beltagy et al. · Local + global',
        href: 'https://arxiv.org/abs/2004.05150',
      },
      {
        year: '2022',
        title: 'FlashAttention',
        authors: 'Dao et al. · IO-aware exact attention',
        href: 'https://arxiv.org/abs/2205.14135',
      },
    ],
  },
  {
    label: 'INTERACTIVE EXPLAINERS',
    papers: [
      {
        year: 'LIVE',
        title: 'LLM Architecture Gallery',
        authors: 'Sebastian Raschka · Compare modern models',
        href: 'https://sebastianraschka.com/llm-architecture-gallery/',
      },
      {
        year: 'LIVE',
        title: 'How LLMs Work: TokenTown',
        authors: 'Laurentiu Gabriel · An LLM as a city',
        href: 'https://laurentiugabriel.github.io/token-town/',
      },
    ],
  },
]
</script>

<template>
  <div class="slide-shell resources-slide">
    <header class="resources-header">
      <p class="eyebrow"><span class="pulse-dot"></span> KEEP READING</p>
      <span class="resources-progress">19 / 20</span>
    </header>

    <div class="resources-title">
      <h1>The reading path follows<br><span>the model path.</span></h1>
      <p v-click>From counting sequences<br>to scaling direct retrieval.</p>
    </div>

    <main class="resource-groups">
      <section v-for="group in groups" :key="group.label" class="resource-group">
        <h2>{{ group.label }}</h2>
        <div class="paper-list">
          <a
            v-for="paper in group.papers"
            v-click
            :key="paper.title"
            class="paper-card"
            :href="paper.href"
            target="_blank"
            rel="noopener noreferrer"
          >
            <span class="paper-year">{{ paper.year }}</span>
            <span class="paper-icon"><carbon-document-pdf aria-hidden="true" /></span>
            <span class="paper-copy">
              <strong>{{ paper.title }}</strong>
              <small>{{ paper.authors }}</small>
            </span>
            <carbon-arrow-up-right class="paper-arrow" aria-hidden="true" />
          </a>
        </div>
      </section>
    </main>

    <footer class="resources-footer">
      <a href="https://github.com/henryhale" target="_blank" rel="noopener noreferrer">
        <carbon-logo-github aria-hidden="true" /> @henryhale
      </a>
      <span>19 / 20</span>
    </footer>
  </div>
</template>

<style scoped>
.resources-slide {
  padding: 28px 42px 20px;
}

.resources-header {
  flex: 0 0 39px;
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  border-bottom: 1px solid #292e29;
}

.resources-header .eyebrow {
  font-size: 9px;
}

.resources-progress {
  color: var(--accent);
  font: 750 9px/1 var(--deck-mono);
  letter-spacing: 0.12em;
}

.resources-title {
  display: flex;
  align-items: end;
  justify-content: space-between;
  padding: 14px 0 13px;
}

.resources-title h1 {
  font-size: 34px;
  line-height: 0.98;
}

.resources-title h1 span {
  color: var(--accent);
}

.resources-title p {
  margin: 0;
  color: var(--muted);
  font-size: 10px;
  line-height: 1.4;
  text-align: right;
}

.resource-groups {
  flex: 1;
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 10px;
  min-height: 0;
}

.resource-group {
  min-width: 0;
}

.resource-group h2 {
  margin: 0 0 7px;
  color: #737c74;
  font-size: 7px;
  font-weight: 750;
  letter-spacing: 0.14em;
}

.paper-list {
  display: grid;
  gap: 6px;
}

.paper-card {
  display: grid;
  grid-template-columns: 29px 24px 1fr 9px;
  align-items: center;
  gap: 7px;
  min-height: 54px;
  padding: 7px 8px;
  color: inherit;
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid #292f2a;
  border-radius: 8px;
  text-decoration: none;
  transition: border-color 0.16s ease, transform 0.16s ease;
}

.paper-card:hover,
.paper-card:focus-visible {
  border-color: rgba(119, 217, 255, 0.36);
  outline: none;
  transform: translateX(2px);
}

.paper-year {
  color: var(--accent);
  font: 700 7px/1 var(--deck-mono);
}

.paper-icon {
  display: grid;
  width: 26px;
  height: 26px;
  place-items: center;
  color: #8e978f;
  background: #121613;
  border-radius: 6px;
}

.paper-icon svg {
  width: 12px;
}

.paper-copy {
  display: grid;
  min-width: 0;
  gap: 3px;
}

.paper-copy strong {
  overflow: hidden;
  color: var(--ink);
  font-size: 10px;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.paper-copy small {
  overflow: hidden;
  color: #788079;
  font-size: 7px;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.paper-arrow {
  width: 10px;
  color: #555d56;
}

.paper-card:hover .paper-arrow,
.paper-card:focus-visible .paper-arrow {
  color: var(--accent);
}

.resources-footer {
  flex: 0 0 22px;
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  padding-top: 8px;
  color: #666e67;
  border-top: 1px solid #292e29;
  font: 650 6.5px/1 var(--deck-mono);
  letter-spacing: 0.08em;
}

.resources-footer a {
  display: flex;
  align-items: center;
  gap: 5px;
  color: #747d75;
  text-decoration: none;
}

.resources-footer a:hover,
.resources-footer a:focus-visible {
  color: var(--accent);
  outline: none;
}

.resources-footer svg {
  width: 10px;
}

.resources-footer > span {
  color: var(--accent);
}
</style>
