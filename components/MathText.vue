<script setup lang="ts">
import katex from 'katex'
import { computed } from 'vue'

const props = defineProps<{ text: string }>()

const html = computed(() => {
  const parts = props.text.split(/(\$[^$]+\$)/g)
  return parts.map(p => {
    if (p.startsWith('$') && p.endsWith('$')) {
      return katex.renderToString(p.slice(1, -1), { throwOnError: false })
    }
    return p
  }).join('')
})
</script>

<template>
  <span v-html="html" />
</template>
