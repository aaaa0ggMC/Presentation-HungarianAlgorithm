<script setup lang="ts">
import MathText from './MathText.vue'
</script>

<template>
  <svg viewBox="0 0 240 100" class="analysis-diagram">
    <rect x="5" y="2" width="100" height="96" rx="6" fill="#dbeafe" fill-opacity="0.35" stroke="#60a5fa" stroke-width="1.5" stroke-dasharray="4,3" />
    <text x="12" y="16" font-size="9" fill="#3b82f6" font-weight="600" font-family="'Noto Sans SC',sans-serif">F</text>

    <line x1="68" y1="38" x2="172" y2="38" stroke="#b45309" stroke-width="2.5" />
    <line x1="68" y1="76" x2="172" y2="76" stroke="#d1d5db" stroke-width="1.2" />
    <circle cx="68" cy="38" r="14" fill="#dde5d4" stroke="#6b7c5e" stroke-width="2" />
    <circle cx="68" cy="76" r="14" fill="#dde5d4" stroke="#d1d5db" stroke-width="1.2" />
    <circle cx="172" cy="38" r="14" fill="#f4dcd4" stroke="#d1d5db" stroke-width="1.2" />
    <circle cx="172" cy="76" r="14" fill="#f4dcd4" stroke="#d1d5db" stroke-width="1.2" />
    <text x="68" y="42" font-size="12" text-anchor="middle" fill="#333" font-weight="700" font-family="'Noto Sans SC',sans-serif">b</text>
    <text x="172" y="42" font-size="12" text-anchor="middle" fill="#333" font-weight="700" font-family="'Noto Sans SC',sans-serif">g</text>

    <rect x="102" y="31" width="18" height="14" rx="3" fill="#fef3c7" stroke="#b45309" stroke-width="1" />
    <text x="111" y="42" font-size="8" text-anchor="middle" fill="#b45309" font-weight="700" font-family="'Noto Sans SC',sans-serif">−δ</text>

    <text x="120" y="90" font-size="9" fill="#b45309" text-anchor="middle" font-weight="600" font-family="'Noto Sans SC',sans-serif">sum drops by δ</text>
  </svg>
</template>
