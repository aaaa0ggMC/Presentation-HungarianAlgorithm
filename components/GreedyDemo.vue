<script setup lang="ts">
import { ref, computed, nextTick } from 'vue'

const step = ref(0)
const totalSteps = 4

const allEdges = [
  { from: 0, to: 0, weight: 3 },
  { from: 0, to: 1, weight: 5 },
  { from: 0, to: 2, weight: 8 },
  { from: 0, to: 3, weight: 2 },
  { from: 1, to: 0, weight: 7 },
  { from: 1, to: 1, weight: 4 },
  { from: 1, to: 2, weight: 1 },
  { from: 1, to: 3, weight: 6 },
  { from: 2, to: 0, weight: 2 },
  { from: 2, to: 1, weight: 9 },
  { from: 2, to: 2, weight: 5 },
  { from: 2, to: 3, weight: 4 },
  { from: 3, to: 0, weight: 2 },
  { from: 3, to: 1, weight: 3 },
  { from: 3, to: 2, weight: 4 },
  { from: 3, to: 3, weight: 7 },
]

const bestEdgeIds = [2, 4, 9, 15]
const bestWeights = [8, 7, 9, 7]

const graphRef = ref<any>(null)

const statusText = computed(() => {
  if (step.value === 0) return 'Click to start'
  const b = step.value - 1
  return `B${b+1} → G${bestEdgeIds[b]}  (score ${bestWeights[b]})`
})

const totalScore = computed(() => {
  if (step.value === 0) return ''
  let sum = 0
  for (let i = 0; i < step.value; i++) sum += bestWeights[i]
  return `Total: ${sum}`
})

function next() {
  if (step.value >= totalSteps) return
  step.value++
  nextTick(() => {
    if (!graphRef.value) return
    graphRef.value.clearHighlights()
    for (let i = 0; i < step.value; i++) {
      graphRef.value.highlightEdge(bestEdgeIds[i])
    }
  })
}

function reset() {
  step.value = 0
  if (graphRef.value) graphRef.value.clearHighlights()
}
</script>

<template>
  <div class="greedy-demo">
    <BipartiteGraph
      ref="graphRef"
      :left-count="4"
      :right-count="4"
      :left-labels="['B_1','B_2','B_3','B_4']"
      :left-labels-latex="true"
      :right-labels="['G_1','G_2','G_3','G_4']"
      :right-labels-latex="true"
      :width="320"
      :height="240"
      :node-radius="14"
      :edges="allEdges"
    />
    <div class="greedy-controls">
      <div class="greedy-status">{{ statusText }}</div>
      <div v-if="totalScore" class="greedy-score">{{ totalScore }}</div>
      <div class="greedy-buttons">
        <button v-if="step < totalSteps" class="greedy-btn" @click="next">
          {{ step === 0 ? 'Select Best Edges →' : 'Next Boy →' }}
        </button>
        <button v-if="step > 0" class="greedy-btn greedy-btn-reset" @click="reset">
          Reset
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.greedy-demo {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  gap: 1.5rem;
  margin: 1rem auto;
  max-width: 100%;
}

.greedy-controls {
  text-align: left;
  display: flex;
  flex-direction: column;
  gap: 0.6rem;
  min-width: 160px;
}

.greedy-status {
  font-size: 1rem;
  font-weight: 600;
  color: var(--c-text, #2b241a);
}

.greedy-score {
  font-size: 0.9rem;
  color: var(--c-accent, #b45309);
  font-weight: 600;
  margin: 0.2rem 0;
}

.greedy-buttons {
  display: flex;
  gap: 0.5rem;
  justify-content: center;
  flex-wrap: wrap;
}

.greedy-btn {
  padding: 0.35rem 1rem;
  border: 1.5px solid var(--c-primary, #7c5a2e);
  border-radius: 8px;
  background-color: var(--c-primary-soft, #f0e6d2);
  color: var(--c-primary, #7c5a2e);
  font-weight: 600;
  font-size: 0.9rem;
  cursor: pointer;
  transition: all 0.15s ease;
  white-space: nowrap;
}

.greedy-btn:hover {
  background-color: var(--c-primary, #7c5a2e);
  color: #fff;
}

.greedy-btn-reset {
  border-color: var(--c-text-dim, #7a6f5d);
  color: var(--c-text-dim, #7a6f5d);
  background-color: transparent;
}

.greedy-btn-reset:hover {
  background-color: var(--c-text-dim, #7a6f5d);
  color: #fff;
}
</style>
