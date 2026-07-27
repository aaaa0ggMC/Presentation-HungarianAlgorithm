<script setup lang="ts">
import MathText from './MathText.vue'
</script>

<template>
  <svg viewBox="0 0 240 100" class="analysis-diagram">
    <line x1="68" y1="38" x2="172" y2="38" stroke="#d1d5db" stroke-width="1.2" />
    <line x1="68" y1="76" x2="172" y2="76" stroke="#d1d5db" stroke-width="1.2" />
    <circle cx="68" cy="38" r="14" fill="#dde5d4" stroke="#d1d5db" stroke-width="1.2" />
    <circle cx="68" cy="76" r="14" fill="#dde5d4" stroke="#d1d5db" stroke-width="1.2" />
    <circle cx="172" cy="38" r="14" fill="#f4dcd4" stroke="#d1d5db" stroke-width="1.2" />
    <circle cx="172" cy="76" r="14" fill="#f4dcd4" stroke="#d1d5db" stroke-width="1.2" />
    <text x="68" y="42" font-size="12" text-anchor="middle" fill="#9ca3af" font-weight="700" font-family="'Noto Sans SC',sans-serif">b</text>
    <text x="172" y="42" font-size="12" text-anchor="middle" fill="#9ca3af" font-weight="700" font-family="'Noto Sans SC',sans-serif">g</text>

    <text x="120" y="55" font-size="10" fill="#9ca3af" text-anchor="middle" font-family="'Noto Sans SC',sans-serif">h(b)+h(g) unchanged</text>
    <text x="120" y="90" font-size="9" fill="#9ca3af" text-anchor="middle" font-weight="600" font-family="'Noto Sans SC',sans-serif">nothing changes</text>
  </svg>
</template>
