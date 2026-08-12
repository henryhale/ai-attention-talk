<script setup lang="ts">
const sections = [
  {
    index: '01',
    title: 'Patterns become memory',
    detail: 'Counts → rules → state → gates',
    slide: 6,
  },
  {
    index: '02',
    title: 'Attention retrieves context',
    detail: 'Any token can reach another',
    slide: 11,
  },
  {
    index: '03',
    title: 'You can watch it choose',
    detail: 'Change words. Watch routes move.',
    slide: 12,
  },
]
</script>

<template>
  <div class="slide-shell agenda-slide">
    <header class="agenda-header">
      <p class="eyebrow"><span class="pulse-dot"></span> THE ROUTE</p>
      <span class="agenda-progress">03 / 20</span>
    </header>

    <div class="agenda-copy">
      <h1>Memory compresses.<br><span class="accent">Attention selects.</span></h1>
      <p v-click>Three moves. One change in focus.</p>
    </div>

    <nav class="agenda-list" aria-label="Presentation agenda">
      <Link
        v-for="section in sections"
        v-click
        :key="section.index"
        class="agenda-item"
        :to="section.slide"
      >
        <span class="agenda-index">{{ section.index }}</span>
        <span class="agenda-item-copy">
          <strong>{{ section.title }}</strong>
          <small>{{ section.detail }}</small>
        </span>
        <carbon-arrow-right class="agenda-arrow" aria-hidden="true" />
      </Link>
    </nav>
  </div>
</template>

<style scoped>
.agenda-slide {
  display: grid;
  grid-template-columns: 0.83fr 1.17fr;
  grid-template-rows: auto 1fr;
  gap: 26px 54px;
  padding: 42px 58px;
}

.agenda-header {
  grid-column: 1 / -1;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-bottom: 15px;
  border-bottom: 1px solid #292e29;
}

.agenda-header .eyebrow {
  font-size: 9px;
}

.agenda-progress {
  color: var(--accent);
  font: 750 9px/1 var(--deck-mono);
  letter-spacing: 0.12em;
}

.agenda-copy {
  align-self: center;
}

.agenda-copy h1 {
  font-size: 46px;
  line-height: 0.96;
}

.agenda-copy p {
  max-width: 275px;
  margin: 20px 0 0;
  color: var(--muted);
  font-size: 12px;
  line-height: 1.5;
}

.agenda-list {
  align-self: center;
  display: grid;
  gap: 9px;
}

.agenda-item {
  display: grid;
  grid-template-columns: 30px 1fr 16px;
  align-items: center;
  gap: 13px;
  min-height: 67px;
  padding: 12px 15px;
  color: inherit;
  background: rgba(255, 255, 255, 0.022);
  border: 1px solid #292f2a;
  border-radius: 10px;
  text-decoration: none;
  transition: transform 0.16s ease, border-color 0.16s ease, background 0.16s ease;
}

.agenda-item:hover,
.agenda-item:focus-visible {
  background: rgba(119, 217, 255, 0.035);
  border-color: rgba(119, 217, 255, 0.35);
  outline: none;
  transform: translateX(3px);
}

.agenda-index {
  color: var(--accent);
  font: 750 9px/1 var(--deck-mono);
  letter-spacing: 0.08em;
}

.agenda-item-copy {
  display: grid;
  gap: 5px;
}

.agenda-item-copy strong {
  color: var(--ink);
  font-size: 14px;
  font-weight: 620;
}

.agenda-item-copy small {
  color: #7f8880;
  font-size: 8.5px;
}

.agenda-arrow {
  width: 13px;
  color: #5e675f;
}

.agenda-item:hover .agenda-arrow,
.agenda-item:focus-visible .agenda-arrow {
  color: var(--accent);
}
</style>
