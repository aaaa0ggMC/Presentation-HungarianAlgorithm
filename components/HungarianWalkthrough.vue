<script setup lang="ts">
import { ref, nextTick } from 'vue'
import katex from 'katex'

const step = ref(0)

interface StepInfo {
  title: string
  desc: string
  leftLabels: string[]
  rightLabels: string[]
  eqIds: number[]
  matchIds: number[]
  caption: string
}

const steps: StepInfo[] = [
  {
    title: 'Initial Labels',
    desc: 'Each boy gets his highest score; each girl gets 0.',
    leftLabels: ['5','7','6'],
    rightLabels: ['0','0','0'],
    eqIds: [],
    matchIds: [],
    caption: '$\Sigma h = 18$ (upper bound)',
  },
  {
    title: 'Equality Graph',
    desc: 'Edges where $h(b_i)+h(g_j)=w(b_i,g_j)$ are highlighted.',
    leftLabels: ['5','7','6'],
    rightLabels: ['0','0','0'],
    eqIds: [1, 4, 6],
    matchIds: [],
    caption: '3 equality edges: B₁G₂, B₂G₂, B₃G₁',
  },
  {
    title: 'Maximum Matching',
    desc: 'Size 2 — not perfect. Unmatched: B₂.',
    leftLabels: ['5','7','6'],
    rightLabels: ['0','0','0'],
    eqIds: [1, 4, 6],
    matchIds: [1, 6],
    caption: '$M = \\{(B_1,G_2), (B_3,G_1)\\}$',
  },
  {
    title: 'Update Labels (Δ=2)',
    desc: '$S=\\{B_1,B_2\\},T=\\{G_2\\}$. Lower boys in $S$, raise G₂.',
    leftLabels: ['3','5','6'],
    rightLabels: ['0','2','0'],
    eqIds: [0, 1, 4, 6],
    matchIds: [1, 6],
    caption: 'New: B₁G₁ also equality! $\Sigma h = 16$',
  },
  {
    title: 'Update Again (Δ=1)',
    desc: '$S=\\{B_1,B_2,B_3\\},T=\\{G_1,G_2\\}$. One more adjustment.',
    leftLabels: ['2','4','5'],
    rightLabels: ['1','3','0'],
    eqIds: [0, 1, 4, 5, 6, 8],
    matchIds: [1, 6],
    caption: 'Equality graph expands: B₂G₃, B₃G₃ appear',
  },
  {
    title: 'Perfect Matching!',
    desc: 'B₃→G₃ is equality & G₃ unmatched → augment. Done!',
    leftLabels: ['2','4','5'],
    rightLabels: ['1','3','0'],
    eqIds: [0, 1, 4, 5, 6, 8],
    matchIds: [0, 4, 8],
    caption: '$w(M)=3+7+5=15 = \\Sigma h$ ✅',
  },
]

const allEdges = [
  { from: 0, to: 0, weight: 3 },
  { from: 0, to: 1, weight: 5 },
  { from: 0, to: 2, weight: 1 },
  { from: 1, to: 0, weight: 2 },
  { from: 1, to: 1, weight: 7 },
  { from: 1, to: 2, weight: 4 },
  { from: 2, to: 0, weight: 6 },
  { from: 2, to: 1, weight: 2 },
  { from: 2, to: 2, weight: 5 },
]

const graphRef = ref<any>(null)

function applyStep(s: StepInfo) {
  if (!graphRef.value) return
  const g = graphRef.value
  g.clearHighlights()
  g.setOuterLabel('left', 0, s.leftLabels[0], true)
  g.setOuterLabel('left', 1, s.leftLabels[1], true)
  g.setOuterLabel('left', 2, s.leftLabels[2], true)
  g.setOuterLabel('right', 0, s.rightLabels[0], true)
  g.setOuterLabel('right', 1, s.rightLabels[1], true)
  g.setOuterLabel('right', 2, s.rightLabels[2], true)
  s.eqIds.forEach(id => g.highlightEdge(id))
  if (s.matchIds.length > 0) {
    g.setMatching(s.matchIds, 120)
  }
}

function next() {
  if (step.value >= steps.length - 1) return
  step.value++
  nextTick(() => applyStep(steps[step.value]))
}

function prev() {
  if (step.value <= 0) return
  step.value--
  nextTick(() => applyStep(steps[step.value]))
}

function renderMath(text: string): string {
  return text.replace(/\$(.+?)\$/g, (_, m) => {
    try { return katex.renderToString(m, { throwOnError: false }) } catch { return m }
  })
}
</script>

<template>
  <div class="ha-demo">
    <div class="ha-main">
      <BipartiteGraph
        ref="graphRef"
        :left-count="3"
        :right-count="3"
        :left-labels="['B_1','B_2','B_3']"
        :left-labels-latex="true"
        :right-labels="['G_1','G_2','G_3']"
        :right-labels-latex="true"
        :left-outer-labels="['5','7','6']"
        :right-outer-labels="['0','0','0']"
        :outer-labels-latex="true"
        :width="260"
        :height="170"
        :node-radius="10"
        :outer-label-offset="10"
        :edges="allEdges"
      />
      <div class="ha-sidebar">
        <div class="ha-step-num">{{ step + 1 }} / {{ steps.length }}</div>
        <div class="ha-step-title">{{ steps[step].title }}</div>
        <div class="ha-step-desc" v-html="renderMath(steps[step].desc)" />
        <div class="ha-step-caption" v-html="renderMath(steps[step].caption)" />
        <div class="ha-dots">
          <span v-for="(s, i) in steps" :key="i"
            class="ha-dot"
            :class="{ 'ha-dot-active': i === step, 'ha-dot-done': i < step }"
            @click="step = i; nextTick(() => applyStep(s))"
          />
        </div>
        <div class="ha-buttons">
          <button class="ha-btn" :disabled="step === 0" @click="prev">Prev</button>
          <button class="ha-btn ha-btn-primary" :disabled="step === steps.length - 1" @click="next">
            {{ step === 0 ? 'Start' : 'Next' }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.ha-demo {
  margin: 0.3rem auto 0;
  max-width: 100%;
}

.ha-main {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  gap: 1rem;
}

.ha-sidebar {
  min-width: 140px;
  max-width: 180px;
}

.ha-step-num {
  font-size: 0.7rem;
  color: var(--c-text-dim, #7a6f5d);
  margin-bottom: 0.1rem;
}

.ha-step-title {
  font-size: 0.9rem;
  font-weight: 700;
  color: var(--c-text, #2b241a);
  margin-bottom: 0.2rem;
}

.ha-step-desc {
  font-size: 0.75rem;
  line-height: 1.35;
  color: var(--c-text-dim, #7a6f5d);
  margin-bottom: 0.2rem;
}

.ha-step-caption {
  font-size: 0.8rem;
  font-weight: 600;
  color: var(--c-accent, #b45309);
  margin-bottom: 0.4rem;
}

.ha-dots {
  display: flex;
  gap: 5px;
  margin-bottom: 0.5rem;
}

.ha-dot {
  width: 9px;
  height: 9px;
  border-radius: 50%;
  background-color: var(--c-border, #e2d9c3);
  cursor: pointer;
  transition: all 0.2s ease;
}

.ha-dot-active {
  background-color: var(--c-accent, #b45309);
  transform: scale(1.3);
}

.ha-dot-done {
  background-color: var(--c-primary, #7c5a2e);
}

.ha-buttons {
  display: flex;
  gap: 0.4rem;
}

.ha-btn {
  padding: 0.3rem 0.7rem;
  border: 1.5px solid var(--c-border, #e2d9c3);
  border-radius: 6px;
  background: transparent;
  color: var(--c-text-dim, #7a6f5d);
  font-weight: 600;
  font-size: 0.8rem;
  cursor: pointer;
  transition: all 0.15s ease;
}

.ha-btn:disabled {
  opacity: 0.35;
  cursor: default;
}

.ha-btn:not(:disabled):hover {
  border-color: var(--c-primary, #7c5a2e);
  color: var(--c-primary, #7c5a2e);
}

.ha-btn-primary {
  border-color: var(--c-primary, #7c5a2e);
  background-color: var(--c-primary-soft, #f0e6d2);
  color: var(--c-primary, #7c5a2e);
}

.ha-btn-primary:not(:disabled):hover {
  background-color: var(--c-primary, #7c5a2e);
  color: #fff;
}
</style>
