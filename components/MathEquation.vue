<script setup lang="ts">
import katex from 'katex'
import { computed } from 'vue'

const props = withDefaults(defineProps<{
  expression: string
  display?: boolean
  label?: string
}>(), {
  display: false,
  label: 'Mathematical equation',
})

const renderedEquation = computed(() => katex.renderToString(props.expression, {
  displayMode: props.display,
  output: 'htmlAndMathml',
  throwOnError: false,
  strict: false,
}))
</script>

<template>
  <span
    class="math-equation"
    :class="{ display }"
    :aria-label="label"
    v-html="renderedEquation"
  ></span>
</template>

<style scoped>
.math-equation {
  display: inline-flex;
  align-items: center;
  color: inherit;
}

.math-equation.display {
  display: flex;
  justify-content: center;
  width: 100%;
}

.math-equation :deep(.katex) {
  color: inherit;
  font-size: 1em;
}

.math-equation :deep(.katex-display) {
  margin: 0;
}
</style>
