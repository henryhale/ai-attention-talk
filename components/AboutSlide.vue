<script setup lang="ts">
const projects = [
  {
    name: 'Embedding Explorer',
    description: 'Train word embeddings from scratch and explore them in an interactive 3D space',
    href: 'https://github.com/henryhale/embeddings',
  },
  {
    name: 'WhatsApp Analyzer',
    description: 'Analyze, summarize, and chat with your exported WhatsApp chats',
    href: 'https://github.com/henryhale/whatsapp-analyzer',
  },
  {
    name: 'DepGraph',
    description: 'Make unfamiliar codebases legible',
    href: 'https://github.com/henryhale/depgraph',
  },
  {
    name: 'Worstcase',
    description: 'Explain JavaScript complexity automatically',
    href: 'https://github.com/henryhale/worstcase',
  },
]
</script>

<template>
  <div class="slide-shell about-slide">
    <header class="about-header">
      <p class="eyebrow"><span class="pulse-dot"></span> BEFORE WE BEGIN</p>
      <span class="about-progress">02 / 20</span>
    </header>

    <main class="about-grid">
      <section class="portrait-column">
        <div class="portrait-frame">
          <img src="/photo.png" alt="Illustrated portrait of Henry Ssenono">
        </div>
        <a class="profile-link" href="https://github.com/henryhale" target="_blank" rel="noopener noreferrer">
          <carbon-logo-github aria-hidden="true" />
          <span>@henryhale</span>
          <carbon-arrow-up-right aria-hidden="true" />
        </a>
      </section>

      <section class="about-copy">
        <span class="about-kicker">HENRY SSENONO · CONWAY LABS</span>
        <h1>I build tools that make<br><span>systems legible.</span></h1>
        <p v-click>Open source. Developer tools.<br>Experiments that explain themselves.</p>

        <a
          v-click
          class="conway-card"
          href="https://conwaylabs.org"
          target="_blank"
          rel="noopener noreferrer"
        >
          <img src="/conwaylabs.svg" alt="Conway Labs logo">
          <span class="conway-copy">
            <small>CONWAY LABS · CONWAYLABS.ORG</small>
            <strong>Conway Canvas</strong>
            <p>Design neural networks visually with Conway Canvas.</p>
          </span>
          <span class="featured-product">FEATURED PRODUCT</span>
          <carbon-arrow-up-right class="conway-arrow" aria-hidden="true" />
        </a>

        <div v-click class="project-list" aria-label="Personal projects">
          <a
            v-for="project in projects"
            :key="project.name"
            class="project-card"
            :href="project.href"
            target="_blank"
            rel="noopener noreferrer"
          >
            <span class="project-icon"><carbon-code aria-hidden="true" /></span>
            <span class="project-copy">
              <strong>{{ project.name }}</strong>
              <small>{{ project.description }}</small>
            </span>
            <carbon-arrow-up-right class="project-arrow" aria-hidden="true" />
          </a>
        </div>
      </section>
    </main>
  </div>
</template>

<style scoped>
.about-slide {
  padding: 38px 52px 32px;
}

.about-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-bottom: 15px;
  border-bottom: 1px solid #292e29;
}

.about-header .eyebrow {
  font-size: 9px;
}

.about-progress {
  color: var(--accent);
  font: 750 9px/1 var(--deck-mono);
  letter-spacing: 0.12em;
}

.about-grid {
  flex: 1;
  display: grid;
  grid-template-columns: 0.72fr 1.28fr;
  align-items: center;
  gap: 54px;
  min-height: 0;
  padding-top: 20px;
}

.portrait-column {
  display: grid;
  justify-items: center;
  gap: 13px;
}

.portrait-frame {
  position: relative;
  width: 218px;
  height: 218px;
  padding: 8px;
  background: rgba(119, 217, 255, 0.035);
  border: 1px solid rgba(119, 217, 255, 0.28);
  border-radius: 28px;
  box-shadow: 0 26px 70px rgba(0, 0, 0, 0.32);
}

.portrait-frame::after {
  content: '';
  position: absolute;
  right: -16px;
  bottom: -16px;
  width: 62px;
  height: 62px;
  border-right: 2px solid var(--accent);
  border-bottom: 2px solid var(--accent);
  border-radius: 0 0 18px;
}

.portrait-frame img {
  display: block;
  width: 100%;
  height: 100%;
  object-fit: contain;
  border-radius: 20px;
  filter: saturate(0.86) contrast(1.05);
}

.profile-link {
  display: flex;
  align-items: center;
  gap: 7px;
  color: #9ba29b;
  font: 700 9px/1 var(--deck-mono);
  text-decoration: none;
}

.profile-link:hover,
.profile-link:focus-visible {
  color: var(--accent);
  outline: none;
}

.profile-link svg {
  width: 13px;
}

.profile-link svg:last-child {
  width: 9px;
}

.about-copy {
  min-width: 0;
}

.about-kicker {
  color: var(--accent);
  font-size: 8px;
  font-weight: 750;
  letter-spacing: 0.16em;
}

.about-copy h1 {
  margin-top: 10px;
  font-size: 42px;
  line-height: 0.98;
}

.about-copy h1 span {
  color: var(--accent);
}

.about-copy > p {
  margin: 13px 0 0;
  color: var(--muted);
  font-size: 12.5px;
  line-height: 1.45;
}

.conway-card {
  display: grid;
  grid-template-columns: 48px minmax(0, 1fr) auto 12px;
  align-items: center;
  gap: 11px;
  min-height: 72px;
  margin-top: 13px;
  padding: 9px 12px;
  color: inherit;
  background:
    radial-gradient(circle at 100% 0, rgba(119, 217, 255, 0.12), transparent 42%),
    rgba(119, 217, 255, 0.045);
  border: 1px solid rgba(119, 217, 255, 0.42);
  border-radius: 10px;
  box-shadow: 0 12px 30px rgba(0, 0, 0, 0.18);
  text-decoration: none;
  transition: border-color 0.16s ease, transform 0.16s ease, box-shadow 0.16s ease;
}

.conway-card:hover,
.conway-card:focus-visible {
  border-color: var(--accent);
  box-shadow: 0 14px 34px rgba(119, 217, 255, 0.08);
  outline: none;
  transform: translateX(3px);
}

.conway-card > img {
  display: block;
  width: 46px;
  height: 46px;
  border-radius: 10px;
}

.conway-copy {
  display: grid;
  gap: 3px;
  min-width: 0;
}

.conway-copy small {
  color: var(--accent);
  font: 700 5.5px/1 var(--deck-mono);
  letter-spacing: 0.1em;
}

.conway-copy strong {
  color: var(--ink);
  font-size: 13px;
}

.conway-copy p {
  margin: 0;
  color: #9aa5aa;
  font-size: 7.5px;
  line-height: 1.35;
}

.featured-product {
  padding: 5px 6px;
  color: #071117;
  background: var(--accent);
  border-radius: 999px;
  font: 750 4.5px/1 var(--deck-mono);
  letter-spacing: 0.08em;
  white-space: nowrap;
}

.conway-arrow {
  width: 10px;
  color: var(--accent);
}

.project-list {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 7px;
  margin-top: 10px;
}

.project-card {
  display: grid;
  grid-template-columns: 30px minmax(0, 1fr) 10px;
  align-items: center;
  gap: 9px;
  min-height: 64px;
  padding: 9px 10px;
  color: inherit;
  background: rgba(255, 255, 255, 0.022);
  border: 1px solid #2b312c;
  border-radius: 8px;
  text-decoration: none;
  transition: border-color 0.16s ease, transform 0.16s ease;
}

.project-card:hover,
.project-card:focus-visible {
  border-color: rgba(119, 217, 255, 0.38);
  outline: none;
  transform: translateX(3px);
}

.project-icon {
  display: grid;
  width: 30px;
  height: 30px;
  place-items: center;
  color: var(--accent);
  background: rgba(119, 217, 255, 0.07);
  border-radius: 7px;
}

.project-icon svg {
  width: 14px;
}

.project-copy {
  display: grid;
  gap: 3px;
}

.project-copy strong {
  color: var(--ink);
  font-size: 10.5px;
}

.project-copy small {
  color: #7f8880;
  font-size: 7px;
  line-height: 1.35;
}

.project-arrow {
  width: 10px;
  color: #59615a;
}

.project-card:hover .project-arrow,
.project-card:focus-visible .project-arrow {
  color: var(--accent);
}
</style>
