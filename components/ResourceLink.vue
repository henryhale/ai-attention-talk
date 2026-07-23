<script setup lang="ts">
defineProps<{
  href: string
  label: string
}>()
</script>

<template>
  <a
    class="resource-link"
    :href="href"
    target="_blank"
    rel="noopener noreferrer"
    :aria-label="label"
    :data-tooltip="label"
  >
    <carbon-document-pdf aria-hidden="true" />
  </a>
</template>

<style scoped>
.resource-link {
  position: relative;
  display: inline-grid;
  place-items: center;
  width: 22px;
  height: 22px;
  color: #858e86;
  border: 1px solid #303630;
  border-radius: 6px;
  text-decoration: none;
  transition: color 0.16s ease, border-color 0.16s ease, background 0.16s ease;
}

.resource-link:hover,
.resource-link:focus-visible {
  color: var(--accent);
  background: rgba(119, 217, 255, 0.06);
  border-color: rgba(119, 217, 255, 0.36);
  outline: none;
}

.resource-link svg {
  width: 12px;
  height: 12px;
}

.resource-link::after {
  content: attr(data-tooltip);
  position: absolute;
  right: 0;
  bottom: calc(100% + 7px);
  z-index: 20;
  width: max-content;
  max-width: 230px;
  padding: 6px 8px;
  color: #d9ded9;
  background: #171b18;
  border: 1px solid #343a35;
  border-radius: 6px;
  font: 600 8px/1.35 var(--deck-sans);
  letter-spacing: 0;
  text-align: left;
  text-transform: none;
  white-space: normal;
  opacity: 0;
  pointer-events: none;
  transform: translateY(3px);
  transition: opacity 0.14s ease, transform 0.14s ease;
}

.resource-link:hover::after,
.resource-link:focus-visible::after {
  opacity: 1;
  transform: translateY(0);
}
</style>
