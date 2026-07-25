---
theme: seriph
layout: center
class: text-center
highlighter: shiki
lineNumbers: false
info: |
  Hungarian Algorithm Presentation
drawings:
  persist: false
transition: slide-left
mdc: true
# 全局浅色背景 + 主题色
background: '#F7F2E7'
colorSchema: light
fonts:
  sans: 'Noto Sans SC'
  serif: 'Noto Serif SC'
  mono: 'Fira Code'
---

# Hungarian Algorithm

Rock & Wilson & Michael & Neel

<div class="absolute bottom-10 left-0 right-0 text-center">
  <a
    href="https://aaaa0ggmc.github.io/Presentation-HungarianAlgorithm"
    target="_blank"
    class="site-link"
  >
    aaaa0ggmc.github.io/Presentation-HungarianAlgorithm
  </a>
</div>


---
layout: default
---

<script setup>
import { ref } from 'vue'
const open02 = ref(false)
</script>

# The Problem

<div class="mt-8 space-y-6 text-lg leading-relaxed">

<div class="para"><span class="para-num">01</span>

In previous courses, we learned how to match partners via the <span class="term">Greedy</span> algorithm, and how to make a stable marriage with the aid of the <span class="term">Gale–Shapley</span> algorithm.

</div>

<div class="para para-collapsible" :class="{ 'para-open': open02 }">
  <div class="para-header" @click="open02 = !open02">
    <span class="para-num">02</span>
    <div class="para-intro">

But in *our* problem — given $n$ guys and $n$ girls — we want to find a <span class="term">maximum-score perfect matching</span>. For this, the previous methods fall short.

  </div>
    <span class="para-toggle">{{ open02 ? '−' : '+' }}</span>
  </div>

  <div v-if="open02" class="para-body" @wheel.stop>
    <div class="bipartite-wrap">
      <BipartiteGraph
        :left-count="4"
        :right-count="4"
        left-title="Boys"
        right-title="Girls"
        :width="380"
        :height="240"
        :node-radius="14"
        :edges="[
          { from: 0, to: 3, weight: 3 },
          { from: 1, to: 2, weight: 5 },
          { from: 2, to: 0, weight: 6 },
          { from: 2, to: 2, weight: 4 },
          { from: 3, to: 0, weight: 7 },
          { from: 3, to: 3, weight: 6 },
          { from: 0, to: 1, weight: 9, highlighted: true },
          { from: 1, to: 0, weight: 8, highlighted: true },
          { from: 2, to: 3, weight: 7, highlighted: true },
          { from: 3, to: 2, weight: 9, highlighted: true },
        ]"
      />
      <div class="bipartite-question">
        How to get the <em>maximum-score</em> matching?
      </div>
    </div>
  </div>
</div>

<div class="para"><span class="para-num">03</span>

We need a new algorithm, designed by two Hungarian mathematicians — the <span class="term-star">Hungarian Algorithm</span>.

</div>

</div>

---
layout: default
---

# Love Scores Matching Problem

<div class="mt-8 space-y-6 text-lg leading-relaxed">

<div class="para"><span class="para-num">01</span>

**Goal.** Given $n$ boys and $n$ girls, every potential couple $(b_i, g_j)$ has a <span class="term">love score</span> $w(b_i,g_j) \in \mathbb{R}$.

</div>

<div class="para"><span class="para-num">02</span>

A <span class="term">perfect matching</span> $M$ is a set of $n$ disjoint boy–girl pairs — everyone is matched to exactly one partner.

</div>

<div class="para"><span class="para-num">03</span>

Find a perfect matching $M^*$ whose **total score** is the greatest among all perfect matchings.

</div>

</div>

---
layout: default
---

# The Bipartite Model

<div class="graph-center-wrap">
  <BipartiteGraph
    :left-count="3"
    :right-count="3"
    :left-labels="['B_1','B_2','B_3']"
    :left-labels-latex="true"
    :right-labels="['G_1','G_2','G_3']"
    :right-labels-latex="true"
    :width="340"
    :height="240"
    :node-radius="16"
    :edges="[
      { from: 0, to: 0, weight: 3 },
      { from: 0, to: 1, weight: 5 },
      { from: 0, to: 2, weight: 1 },
      { from: 1, to: 0, weight: 2 },
      { from: 1, to: 1, weight: 7 },
      { from: 1, to: 2, weight: 4 },
      { from: 2, to: 0, weight: 6 },
      { from: 2, to: 1, weight: 2 },
      { from: 2, to: 2, weight: 5 },
    ]"
  />
  <div class="graph-caption">
    <MathText text="$G = (V,E)$, $w(M)$: total score of matching $M$" />
  </div>
</div>

---

# Problem v0.0 & Solution

> What if each boy's best-lover is unique (disjoint)?

If all boys' highest-score edges go to different girls, we can just pick them all — greedy works!

<GreedyDemo />

So $w(M^*) = \sum_{i=1}^{n} \max_{j} w(b_i,g_j)$ when the maxima are disjoint.

---

# Shortcomings of Solution v0.0

In general, best choices **overlap** — two boys may share the same top girl.

<div class="flex gap-8 items-start mt-6">
<div class="flex-1">

**Starting from the top**
The cherry-pick strategy gives us an **upper bound** on the total score. It's a natural starting point.

**Why not low-pick?**
Starting from low scores and climbing up is inefficient — we'd be far from the optimum.

**Why not medium-pick?**
A middle-ground start leaves two directions to explore, doubling the search space.

</div>
<div class="flex-1 text-lg leading-relaxed p-4" style="border-left:3px solid var(--c-accent,#b45309);background:var(--c-bg-soft,#f1ead9);border-radius:0 8px 8px 0">

**Keep the cherry-pick, add labels.**

We still aim high, but use **vertex labels** to systematically resolve conflicts — reducing some boys' expectations while raising some girls' to make room for better matches.

</div>
</div>

---

# Labels

Each vertex gets a **default** label (call it $h$ or $\ell$):

$$
h(b_i) = \max_{g_j \in G}\, w(b_i,g_j) \qquad\qquad h(g_j) = 0
$$

**Key insight** — from Solution v0.0, when every edge in $M$ satisfies $h(b_i)+h(g_j)=w(b_i,g_j)$, we have:

$$
w(M) = \sum_{(b_i,g_j)\in M} w(b_i,g_j) = \sum_{i} h(b_i) + \sum_{j} h(g_j)
$$

Since each $h(b_i)$ is $b_i$'s highest possible score, $\sum h$ is an **upper bound** that no perfect matching can surpass. So a perfect matching that satisfies the equation automatically hits the bound — and must be $M^*$


<div class="graph-center-wrap" style="margin-top:0.4rem">
  <BipartiteGraph
    :left-count="3"
    :right-count="3"
    :left-labels="['B_1','B_2','B_3']"
    :left-labels-latex="true"
    :right-labels="['G_1','G_2','G_3']"
    :right-labels-latex="true"
    :left-outer-labels="['5','7','6']"
    :right-outer-labels="['0','0','0']"
    :outer-labels-latex="true"
    :width="280"
    :height="180"
    :node-radius="12"
    :outer-label-offset="10"
    :edges="[
      { from: 0, to: 0, weight: 3, dashed: true },
      { from: 0, to: 1, weight: 5, highlighted: true },
      { from: 0, to: 2, weight: 1, dashed: true },
      { from: 1, to: 0, weight: 2, dashed: true },
      { from: 1, to: 1, weight: 7, highlighted: true },
      { from: 1, to: 2, weight: 4, dashed: true },
      { from: 2, to: 0, weight: 6, highlighted: true },
      { from: 2, to: 1, weight: 2, dashed: true },
      { from: 2, to: 2, weight: 5, dashed: true },
    ]"
  />
  <div class="graph-caption">
    <MathText text="$\Sigma h = 5+7+6 = 18$ — upper bound" />
  </div>
</div>

---

# Equality Graph

The edges of a maximum-score perfect matching must always satisfy $h(b_i) + h(g_j) = w(b_i, g_j)$.

What about the other edges that don't satisfy this?

**Just forget them.** Transform $G$ into $G_h$ — the **equality graph** of $G$ — keeping only edges where the condition holds:

$$
G_h = (V, E_h), \qquad E_h = \{\, (b_i,g_j) \mid h(b_i) + h(g_j) = w(b_i,g_j) \,\}
$$

If a perfect matching $M^*$ exists in $G_h$, then $M^*$ is also the maximum-score perfect matching in the original $G$.

---

# So Why?

<div class="eq-side">
  <div class="eq-col">
    <div class="eq-title"><MathText text="$G$" /></div>
    <BipartiteGraph
      :left-count="4"
      :right-count="4"
      :left-labels="['B_1','B_2','B_3','B_4']"
      :left-labels-latex="true"
      :right-labels="['G_1','G_2','G_3','G_4']"
      :right-labels-latex="true"
      :width="240"
      :height="150"
      :node-radius="9"
      :edges="[
        { from: 0, to: 0, weight: 5 }, { from: 0, to: 1, weight: 3 },
        { from: 0, to: 2, weight: 2 }, { from: 0, to: 3, weight: 1 },
        { from: 1, to: 0, weight: 2 }, { from: 1, to: 1, weight: 4 },
        { from: 1, to: 2, weight: 7 }, { from: 1, to: 3, weight: 3 },
        { from: 2, to: 0, weight: 1 }, { from: 2, to: 1, weight: 6 },
        { from: 2, to: 2, weight: 3 }, { from: 2, to: 3, weight: 4 },
        { from: 3, to: 0, weight: 3 }, { from: 3, to: 1, weight: 2 },
        { from: 3, to: 2, weight: 1 }, { from: 3, to: 3, weight: 8 },
      ]"
    />
  </div>
  <div class="eq-col">
    <div class="eq-title"><MathText text="$G_h$" /></div>
    <BipartiteGraph
      :left-count="4"
      :right-count="4"
      :left-labels="['B_1','B_2','B_3','B_4']"
      :left-labels-latex="true"
      :right-labels="['G_1','G_2','G_3','G_4']"
      :right-labels-latex="true"
      :left-outer-labels="['5','7','6','8']"
      :outer-labels-latex="true"
      :width="240"
      :height="150"
      :node-radius="9"
      :outer-label-offset="6"
      :edges="[
        { from: 0, to: 0, weight: 5, highlighted: true },
        { from: 1, to: 2, weight: 7, highlighted: true },
        { from: 2, to: 1, weight: 6, highlighted: true },
        { from: 3, to: 3, weight: 8, highlighted: true },
      ]"
    />
  </div>
</div>

<div class="eq-derivation">

$$
\begin{aligned}
w(M) &= \sum_{(b_i,g_j)\in M} w(b_i,g_j) \\
     &\le \sum_{(b_i,g_j)\in M} \bigl(h(b_i) + h(g_j)\bigr) \\
     &= \sum_{i} h(b_i) + \sum_{j} h(g_j) \\
     &= w(M^*)
\end{aligned}
$$

$\sum h = 5+7+6+8 = 26$ is the upper bound. The matching in $G_h$ hits it — so it is optimal.

</div>

---

# Solution v0.1

With labels and equality graph, we can now tackle the overlap problem.

> A boy can have **multiple best-lovers** — all scoring the same max value $h(b_i)$. There always exists a solution where every boy finds one of his best-lovers.

<div class="eq-side">
  <div class="eq-col">
    <div class="eq-title"><MathText text="$G$" /></div>
    <BipartiteGraph
      :left-count="4"
      :right-count="4"
      :left-labels="['B_1','B_2','B_3','B_4']"
      :left-labels-latex="true"
      :right-labels="['G_1','G_2','G_3','G_4']"
      :right-labels-latex="true"
      :width="230"
      :height="170"
      :node-radius="9"
      :edges="[
        { from: 0, to: 0, weight: 5 }, { from: 0, to: 1, weight: 5 },
        { from: 0, to: 2, weight: 2 }, { from: 0, to: 3, weight: 1 },
        { from: 1, to: 0, weight: 3 }, { from: 1, to: 1, weight: 7 },
        { from: 1, to: 2, weight: 7 }, { from: 1, to: 3, weight: 4 },
        { from: 2, to: 0, weight: 6 }, { from: 2, to: 1, weight: 3 },
        { from: 2, to: 2, weight: 3 }, { from: 2, to: 3, weight: 2 },
        { from: 3, to: 0, weight: 1 }, { from: 3, to: 1, weight: 2 },
        { from: 3, to: 2, weight: 2 }, { from: 3, to: 3, weight: 8 },
      ]"
    />
  </div>
  <div class="eq-col">
    <div class="eq-title"><MathText text="$G_h$" /></div>
    <BipartiteGraph
      :left-count="4"
      :right-count="4"
      :left-labels="['B_1','B_2','B_3','B_4']"
      :left-labels-latex="true"
      :right-labels="['G_1','G_2','G_3','G_4']"
      :right-labels-latex="true"
      :left-outer-labels="['5','7','6','8']"
      :outer-labels-latex="true"
      :width="230"
      :height="170"
      :node-radius="9"
      :outer-label-offset="6"
      :edges="[
        { from: 0, to: 0, weight: 5, highlighted: true },
        { from: 0, to: 1, weight: 5, highlighted: true },
        { from: 1, to: 1, weight: 7, highlighted: true },
        { from: 1, to: 2, weight: 7, highlighted: true },
        { from: 2, to: 0, weight: 6, highlighted: true },
        { from: 3, to: 3, weight: 8, highlighted: true },
      ]"
    />
  </div>
</div>

Now the problem reduces to finding a **perfect matching** in an unweighted graph $G_h$ — scores no longer matter.

---

# Finding M* in $G_h$

Since every edge in $G_h$ already gives a boy his best score, the weights are irrelevant — we just need to match as many pairs as possible.

<div class="flex gap-6 items-start mt-4">
<div class="flex-1">

**M-Augmenting Path** — gradually enlarge the matching by finding alternating paths.

We start with a simple greedy initialization:

```text
M = ∅
for each boy b in L
  if b has an unmatched neighbor g in R
    M = M ∪ {(b, g)}
return M
```

</div>
<div class="flex-1 text-lg leading-relaxed p-4" style="border-left:3px solid var(--c-accent,#b45309);background:var(--c-bg-soft,#f1ead9);border-radius:0 8px 8px 0">

This may not give a perfect matching — but it's a solid foundation. We then use **augmenting paths** to flip edges and increase the matching size until it's perfect.

</div>
</div>

---

<script setup>
import { ref, watch } from 'vue'

const step = ref(0)
const graphRef = ref(null)

const baseEdges = [
  { from: 0, to: 0, weight: 5, highlighted: true },
  { from: 0, to: 1, weight: 5 },
  { from: 1, to: 1, weight: 7, highlighted: true },
  { from: 1, to: 2, weight: 7 },
  { from: 2, to: 0, weight: 6 },
  { from: 3, to: 3, weight: 8, highlighted: true },
]

function resetStyle() {
  const g = graphRef.value
  if (!g) return
  for (let i = 0; i < 6; i++) {
    g.updateEdge(i, { highlighted: false, color: undefined, width: undefined, dashed: false })
  }
}

function applyStep(s) {
  const g = graphRef.value
  if (!g) return
  resetStyle()
  switch (s) {
    case 0:
      g.updateEdge(0, { highlighted: true })
      g.updateEdge(2, { highlighted: true })
      g.updateEdge(5, { highlighted: true })
      break
    case 1:
      g.updateEdge(0, { highlighted: true })
      g.updateEdge(2, { highlighted: true })
      g.updateEdge(5, { highlighted: true })
      g.updateEdge(4, { color: '#2563eb', width: 3 })
      break
    case 2:
      g.updateEdge(0, { highlighted: true })
      g.updateEdge(2, { highlighted: true })
      g.updateEdge(5, { highlighted: true })
      g.updateEdge(4, { color: '#2563eb', width: 3 })
      g.updateEdge(1, { color: '#2563eb', width: 3 })
      break
    case 3:
      g.updateEdge(0, { highlighted: true })
      g.updateEdge(2, { highlighted: true })
      g.updateEdge(5, { highlighted: true })
      g.updateEdge(4, { color: '#2563eb', width: 3 })
      g.updateEdge(1, { color: '#2563eb', width: 3 })
      g.updateEdge(3, { color: '#2563eb', width: 3 })
      break
    case 4:
      g.updateEdge(0, { color: '#9ca3af', width: 1.5, dashed: true })
      g.updateEdge(1, { highlighted: true })
      g.updateEdge(2, { color: '#9ca3af', width: 1.5, dashed: true })
      g.updateEdge(3, { highlighted: true })
      g.updateEdge(4, { highlighted: true })
      g.updateEdge(5, { highlighted: true })
      break
  }
}

watch(step, s => applyStep(s))
</script>

# Demo of M-Augmenting

<div class="flex flex-col items-center">
  <BipartiteGraph
    ref="graphRef"
    :left-count="4" :right-count="4"
    :left-labels="['B_1','B_2','B_3','B_4']" :left-labels-latex="true"
    :right-labels="['G_1','G_2','G_3','G_4']" :right-labels-latex="true"
    :left-outer-labels="['5','7','6','8']" :outer-labels-latex="true"
    :width="300" :height="200" :node-radius="10" :outer-label-offset="6"
    :edges="baseEdges"
  />

  <div class="demo-caption">
    <span v-if="step === 0">
      <MathText text="Greedy: $\{B_1G_1, B_2G_2, B_4G_4\}$ — $B_3$ isolated" />
    </span>
    <span v-if="step === 1">
      <MathText text="$B_3$ reaches $G_1$ — but $G_1$ is taken by $B_1$" />
    </span>
    <span v-if="step === 2">
      <MathText text="$B_1$ looks for an alternative — $G_2$ is taken by $B_2$" />
    </span>
    <span v-if="step === 3">
      <MathText text="$B_2$ reaches $G_3$ — FREE! Augmenting path complete" />
    </span>
    <span v-if="step === 4">
      <MathText text="Flip! Perfect matching: $\{B_3G_1, B_1G_2, B_2G_3, B_4G_4\}$" />
    </span>
  </div>

  <div class="demo-nav" v-if="step < 4" @click="step++">
    Next →
  </div>
  <div class="demo-nav demo-nav-reset" v-if="step === 4" @click="step = 0">
    ↺ Reset
  </div>
</div>

---

# Summary for v0.1

Solution v0.1 handles cases where $G_h$ contains a perfect matching — we find it via augmenting paths.

But real problems are messier. Often $G_h$ **lacks** a perfect matching on the first try.

We know that keeping labels as high as possible lets $M^*$ emerge from $G_h$. So what do we do when $G_h$ has no perfect matching at all?

---

<script setup>
import { ref, watch, nextTick } from 'vue'
const pStep = ref(0)
const pGraph = ref(null)

const pEdges = [
  { from: 0, to: 0, weight: 5, highlighted: true },
  { from: 0, to: 1, weight: 5 },
  { from: 1, to: 1, weight: 7, highlighted: true },
  { from: 1, to: 3, weight: 7 },
  { from: 2, to: 0, weight: 6 },
  { from: 2, to: 1, weight: 6 },
  { from: 3, to: 3, weight: 8, highlighted: true },
]

function applyPStep(s) {
  const g = pGraph.value
  if (!g) return
  for (let i = 0; i < 7; i++)
    g.updateEdge(i, { highlighted: false, color: undefined, width: undefined, dashed: false })
  g.updateEdge(0, { highlighted: true })
  g.updateEdge(2, { highlighted: true })
  g.updateEdge(6, { highlighted: true })
  if (s === 1) {
    g.updateEdge(1, {})
    g.updateEdge(3, {})
    g.updateEdge(4, {})
    g.updateEdge(5, {})
  } else if (s === 2) {
    g.updateEdge(4, { color: '#ef4444', width: 2, dashed: true })
    g.updateEdge(1, { color: '#ef4444', width: 2, dashed: true })
    g.updateEdge(3, { color: '#ef4444', width: 2, dashed: true })
    g.updateEdge(5, {})
  } else if (s === 3) {
    g.updateEdge(1, {})
    g.updateEdge(4, {})
    g.updateEdge(3, { color: '#ef4444', width: 2, dashed: true })
    g.updateEdge(5, { color: '#ef4444', width: 2, dashed: true })
  }
}

watch(pStep, s => { if (s >= 1) nextTick(() => applyPStep(s)) })
</script>

# Problem v0.5

<div class="flex flex-col items-center">
  <BipartiteGraph
    v-if="pStep === 0"
    :left-count="4" :right-count="4"
    :left-labels="['B_1','B_2','B_3','B_4']" :left-labels-latex="true"
    :right-labels="['G_1','G_2','G_3','G_4']" :right-labels-latex="true"
    :width="340" :height="220" :node-radius="11"
    :edges="[
      { from: 0, to: 0, weight: 5 }, { from: 0, to: 1, weight: 5 },
      { from: 0, to: 2, weight: 2 }, { from: 0, to: 3, weight: 1 },
      { from: 1, to: 0, weight: 3 }, { from: 1, to: 1, weight: 7 },
      { from: 1, to: 2, weight: 4 }, { from: 1, to: 3, weight: 7 },
      { from: 2, to: 0, weight: 6 }, { from: 2, to: 1, weight: 6 },
      { from: 2, to: 2, weight: 3 }, { from: 2, to: 3, weight: 4 },
      { from: 3, to: 0, weight: 3 }, { from: 3, to: 1, weight: 2 },
      { from: 3, to: 2, weight: 2 }, { from: 3, to: 3, weight: 8 },
    ]"
  />
  <BipartiteGraph
    v-if="pStep >= 1"
    ref="pGraph"
    :left-count="4" :right-count="4"
    :left-labels="['B_1','B_2','B_3','B_4']" :left-labels-latex="true"
    :right-labels="['G_1','G_2','G_3','G_4']" :right-labels-latex="true"
    :left-outer-labels="['5','7','6','8']" :outer-labels-latex="true"
    :width="340" :height="220" :node-radius="11" :outer-label-offset="6"
    :edges="pEdges"
  />

  <div class="demo-caption">
    <span v-if="pStep === 0">
      <MathText text="$G$ — full bipartite graph with all scores" />
    </span>
    <span v-if="pStep === 1">
      <MathText text="$G_h$ — max matching size 3, $B_3$ isolated" />
    </span>
    <span v-if="pStep === 2">
      <MathText text="Try $G_1$ path: $B_3\!\rightarrow\!G_1\!\rightarrow\!B_1\!\rightarrow\!G_2\!\rightarrow\!B_2\!\rightarrow\!G_4\!\rightarrow\!B_4$ — $B_4$ has no other edge. Dead end!" />
    </span>
    <span v-if="pStep === 3">
      <MathText text="Try $G_2$ path: $B_3\!\rightarrow\!G_2\!\rightarrow\!B_2\!\rightarrow\!G_4\!\rightarrow\!B_4$ — same dead end. No augmenting path exists!" />
    </span>
  </div>

  <div class="demo-nav" v-if="pStep === 0" @click="pStep = 1">
    Build G<sub>h</sub> →
  </div>
  <div class="demo-nav" v-if="pStep === 1" @click="pStep = 2">
    Search →
  </div>
  <div class="demo-nav" v-if="pStep === 2" @click="pStep = 3">
    Try Other Path →
  </div>
  <div class="demo-nav demo-nav-reset" v-if="pStep === 3" @click="pStep = 0">
    ↺ Reset
  </div>
</div>

<div class="eq-derivation" style="margin-top:0.6rem">

<MathText text="It's impossible to cover all vertices with the current $G_h$. What if we could $\textbf{add\ more\ edges}$ into $G_h$? Too many edges are ignored — without enough information, even the best algorithm gets stuck." />

</div>

---

# Search Tree

During the search, we build an **alternating tree** — tracing all explored paths from the unmatched vertex.

<div class="st-wrap">
  <BipartiteGraph
    :left-count="4" :right-count="4"
    :left-labels="['B_1','B_2','B_3','B_4']" :left-labels-latex="true"
    :right-labels="['G_1','G_2','G_3','G_4']" :right-labels-latex="true"
    :left-outer-labels="['5','7','6','8']" :outer-labels-latex="true"
    :width="220" :height="180" :node-radius="9" :outer-label-offset="5"
    :edges="[
      { from: 0, to: 0, weight: 5, highlighted: true },
      { from: 0, to: 1, weight: 5 },
      { from: 1, to: 1, weight: 7, highlighted: true },
      { from: 1, to: 3, weight: 7 },
      { from: 2, to: 0, weight: 6 },
      { from: 2, to: 1, weight: 6 },
      { from: 3, to: 3, weight: 8, highlighted: true },
    ]"
  />

  <div class="st-tree">
    <div class="st-node st-root" style="margin-bottom:0.1rem">B₃</div>
    <div class="st-bridge"></div>
    <div class="st-branches">
      <div class="st-arm">
        <div class="st-edge">E\M</div>
        <div class="st-node st-r">G₁</div>
        <div class="st-edge">M</div>
        <div class="st-node st-l">B₁</div>
        <div class="st-edge">E\M</div>
        <div class="st-node st-r">G₂</div>
        <div class="st-edge">M</div>
        <div class="st-node st-l">B₂</div>
        <div class="st-edge">E\M</div>
        <div class="st-node st-r">G₄</div>
        <div class="st-edge">M</div>
        <div class="st-node st-l st-dead">B₄ ✗</div>
      </div>
      <div class="st-arm">
        <div class="st-edge">E\M</div>
        <div class="st-node st-r">G₂</div>
        <div class="st-edge">M</div>
        <div class="st-node st-l st-dim">B₂ (visited)</div>
      </div>
    </div>
  </div>

  <div class="st-def">
    <strong>Alternating Tree</strong>
    <br>
    <code>F = (V<sub>F</sub>, E<sub>F</sub>)</code> — rooted at the unmatched vertex, alternating unmatched/matched edges.
    <br><br>
    <code>F<sub>L</sub> = V<sub>F</sub> ∩ L</code> •
    <code>F<sub>R</sub> = V<sub>F</sub> ∩ R</code>
    <br><br>
    When no augmenting path exists, <code>F<sub>L</sub></code> and <code>F<sub>R</sub></code> tell us which labels to adjust.
    <br><br>
    <span style="font-size:0.8rem;color:var(--c-text-dim)">Multiple unmatched vertices → a <strong>forest</strong> (one tree per root).</span>
  </div>
</div>

---

# Thinking

To add more edges into $G_h$ while keeping the bound, we adjust the labels:

$$
v.h =
\begin{cases}
v.h - \delta & \text{if } v \in F_L \\[4pt]
v.h + \delta & \text{if } v \in F_R \\[4pt]
v.h & \text{otherwise}
\end{cases}
$$

- Boys in the tree ($F_L$) **lower** expectations — they've been aiming too high
- Girls in the tree ($F_R$) **rise** in value — they become more attractive
- Vertices outside the tree stay untouched

This keeps all matching edges valid (since $h(b)+h(g)$ is unchanged when both are in $F$), while new edges may enter $G_h$ from $F_L$ to girls outside $F_R$.

---

# Analysis

What happens to each type of edge when we adjust by $\delta$?

<div class="analysis-grid">
  <div class="analysis-card">
    <div class="analysis-case"><MathText text="$F\ \text{both}$" /></div>
    <div class="analysis-detail"><MathText text="$b \in F_L$" />, <MathText text="$g \in F_R$" /></div>
    <div class="analysis-change"><MathText text="$h(b) \downarrow \delta$" />, <MathText text="$h(g) \uparrow \delta$" /></div>
    <div class="analysis-sum" style="color:var(--c-primary)"><MathText text="$h(b)+h(g)$" /> unchanged ✓</div>
    <div class="analysis-note">Matching edges stay satisfied</div>
  </div>
  <div class="analysis-card">
    <div class="analysis-case"><MathText text="$F\ \text{one}$" /></div>
    <div class="analysis-detail"><MathText text="$b \in F_L$" />, <MathText text="$g \notin F_R$" /></div>
    <div class="analysis-change"><MathText text="$h(b)+h(g)$" /> drops by <MathText text="$\delta$" /></div>
    <div class="analysis-sum" style="color:#b45309">New edges may enter <MathText text="$G_h$" />!</div>
    <div class="analysis-note">This is exactly what we want</div>
  </div>
  <div class="analysis-card">
    <div class="analysis-case"><MathText text="$F\ \text{none}$" /></div>
    <div class="analysis-detail"><MathText text="$b,g \notin V_F$" /></div>
    <div class="analysis-change"><MathText text="$h(b)+h(g)$" /> unchanged</div>
    <div class="analysis-sum">Nothing changes</div>
    <div class="analysis-note">Irrelevant edges stay irrelevant</div>
  </div>
</div>

---

# Why

Why adjust **all** vertices in the forest — not just some?

Multiple edges could enter $G_h$ at once, and we don't know which one will lead to a perfect matching. Adjusting all of them keeps every option open.

Since $\delta$ doesn't affect the total score of already-established connections (both endpoints in $F$ → $h(b)+h(g)$ unchanged), there's **no downside** to adjusting the whole forest.

<div class="eq-derivation" style="margin-top:1rem;text-align:center">

<MathText text="Lower the $F_L$ boys' expectations, raise the $F_R$ girls' worth — and let new possibilities emerge." />

</div>

---

# Choose the Best $\delta$

Since $h(b_i)$ is the max weight from $b_i$, we have $h(b_i)+h(g_j)\ge w(b_i,g_j)$ for all edges — this is what makes $\sum h$ the upper bound. After adjusting labels, this **must still hold**.

The only case where the sum drops is $b\in F_L,\ g\notin F_R$:

$$
h_{\text{new}}(b)+h_{\text{new}}(g)=h(b)+h(g)-\delta
$$

So $\delta$ is squeezed between two constraints:

<div class="flex gap-4 justify-center items-stretch mt-4">
  <div class="flex-1 p-3" style="border:2px solid #dc2626;border-radius:8px;text-align:center">
    <div style="font-size:0.85rem;color:#dc2626;font-weight:600">If <MathText text="$\delta$" /> too large:</div>
    <div style="font-size:1.1rem;margin:0.3rem 0"><MathText text="$\delta > h(b)+h(g)-w(b,g)$" /></div>
    <div style="font-size:0.85rem;color:var(--c-text-dim)"><MathText text="$h(b)+h(g) < w(b,g)$" /> — invalid!</div>
  </div>
  <div class="flex-1 p-3" style="border:2px solid var(--c-primary);border-radius:8px;text-align:center">
    <div style="font-size:0.85rem;color:var(--c-primary);font-weight:600">Just right:</div>
    <div style="font-size:1.1rem;margin:0.3rem 0"><MathText text="$\delta = \min\bigl(h(b)+h(g)-w(b,g)\bigr)$" /></div>
    <div style="font-size:0.85rem;color:var(--c-text-dim)">Adds one edge ✓, stays valid ✓</div>
  </div>
  <div class="flex-1 p-3" style="border:2px solid #2563eb;border-radius:8px;text-align:center">
    <div style="font-size:0.85rem;color:#2563eb;font-weight:600">If <MathText text="$\delta$" /> too small:</div>
    <div style="font-size:1.1rem;margin:0.3rem 0"><MathText text="$\delta < h(b)+h(g)-w(b,g)$" /></div>
    <div style="font-size:0.85rem;color:var(--c-text-dim)">No new edge enters — useless</div>
  </div>
</div>

Only **one** value satisfies both:

$$
\boxed{\ \delta = \min_{b\in F_L,\ g\notin F_R} \bigl(h(b)+h(g)-w(b,g)\bigr)\ }
$$

---

<script setup>
import { ref, watch, nextTick } from 'vue'
const dStep = ref(0)
const dGraph = ref(null)

const init10 = [
  { from: 0, to: 0, weight: 5, highlighted: true },
  { from: 0, to: 1, weight: 5 },
  { from: 1, to: 1, weight: 7, highlighted: true },
  { from: 1, to: 3, weight: 7 },
  { from: 2, to: 0, weight: 6 },
  { from: 2, to: 1, weight: 6 },
  { from: 3, to: 3, weight: 8, highlighted: true },
  { from: 0, to: 2, weight: 2, color: '#2563eb', width: 3 },
  { from: 1, to: 2, weight: 4, color: '#2563eb', width: 3 },
  { from: 2, to: 2, weight: 3, color: '#2563eb', width: 3 },
]

function applyDStep(s) {
  const g = dGraph.value
  if (!g) return
  for (let i = 0; i < 10; i++)
    g.updateEdge(i, { highlighted: false, color: undefined, width: undefined, dashed: false })
  g.updateEdge(0, { highlighted: true })
  g.updateEdge(2, { highlighted: true })
  g.updateEdge(6, { highlighted: true })
  if (s === 0 || s === 1) {
    for (let i = 7; i < 10; i++)
      g.updateEdge(i, { color: '#2563eb', width: 3 })
    g.updateEdge(1, {})
    g.updateEdge(3, {})
    g.updateEdge(4, {})
    g.updateEdge(5, {})
  } else if (s === 2) {
    for (let i = 7; i < 10; i++)
      g.updateEdge(i, { color: '#2563eb', width: 3 })
    g.updateEdge(1, {})
    g.updateEdge(3, {})
    g.updateEdge(4, {})
    g.updateEdge(5, {})
    g.updateEdge(7, { color: '#2563eb', width: 3 })
    g.updateEdge(8, { color: '#2563eb', width: 3 })
    g.updateEdge(9, { color: '#2563eb', width: 3 })
  } else if (s === 3) {
    g.updateEdge(9, { color: '#2563eb', width: 4 })
    g.updateEdge(7, {})
    g.updateEdge(8, {})
    g.updateEdge(1, {})
    g.updateEdge(3, {})
    g.updateEdge(4, {})
    g.updateEdge(5, {})
  } else if (s === 4) {
    g.updateEdge(9, { highlighted: true })
    g.updateEdge(7, {})
    g.updateEdge(8, {})
    g.updateEdge(1, {})
    g.updateEdge(3, {})
    g.updateEdge(4, {})
    g.updateEdge(5, {})
  }
}

watch(dStep, s => { if (s >= 2) nextTick(() => applyDStep(s)) })
</script>

# Applying $\delta$ — Step by Step

<div class="flex flex-col items-center">
  <BipartiteGraph
    v-if="dStep <= 1"
    :left-count="4" :right-count="4"
    :left-labels="['B_1','B_2','B_3','B_4']" :left-labels-latex="true"
    :right-labels="['G_1','G_2','G_3','G_4']" :right-labels-latex="true"
    :left-outer-labels="['5','7','6','8']" :outer-labels-latex="true"
    :width="340" :height="220" :node-radius="11" :outer-label-offset="6"
    :edges="[
      { from: 0, to: 0, weight: 5, highlighted: true },
      { from: 0, to: 1, weight: 5 },
      { from: 1, to: 1, weight: 7, highlighted: true },
      { from: 1, to: 3, weight: 7 },
      { from: 2, to: 0, weight: 6 },
      { from: 2, to: 1, weight: 6 },
      { from: 3, to: 3, weight: 8, highlighted: true },
    ]"
  />
  <BipartiteGraph
    v-if="dStep >= 2"
    ref="dGraph"
    :left-count="4" :right-count="4"
    :left-labels="['B_1','B_2','B_3','B_4']" :left-labels-latex="true"
    :right-labels="['G_1','G_2','G_3','G_4']" :right-labels-latex="true"
      :left-outer-labels="['2','4','3','5']" :outer-labels-latex="true"
      :right-outer-labels="['3','3','0','3']"
    :width="340" :height="220" :node-radius="11" :outer-label-offset="6"
    :edges="init10"
  />

  <div class="demo-caption">
    <span v-if="dStep === 0">
      <MathText text="$G_h$: max matching size 3, $B_3$ isolated" />
    </span>
    <span v-if="dStep === 1">
      <MathText text="$\delta = \min\{ h(b)+h(g)-w(b,g) \mid b\in F_L,\ g\notin F_R \} = \min(3,3,3,6) = 3$" />
    </span>
    <span v-if="dStep === 2">
      <MathText text="$\delta = 3$ applied! New labels: ${\bf [2,4,3,5]}$, ${\bf [3,3,0,3]}$ — 3 new edges enter $G_h$!" />
    </span>
    <span v-if="dStep === 3">
      <MathText text="$B_3$ connects to $G_3$ via a new edge — $G_3$ is FREE! Augmenting path found!" />
    </span>
    <span v-if="dStep === 4">
      <MathText text="Perfect matching $\{B_1G_1, B_2G_2, B_3G_3, B_4G_4\}$ — all boys find their best lovers!" />
    </span>
  </div>

  <div class="demo-nav" v-if="dStep === 0" @click="dStep = 1">
    Compute <MathText text="$\delta$" /> →
  </div>
  <div class="demo-nav" v-if="dStep === 1" @click="dStep = 2">
    Apply <MathText text="$\delta$" /> →
  </div>
  <div class="demo-nav" v-if="dStep === 2" @click="dStep = 3">
    Find Path →
  </div>
  <div class="demo-nav" v-if="dStep === 3" @click="dStep = 4">
    Flip →
  </div>
  <div class="demo-nav demo-nav-reset" v-if="dStep === 4" @click="dStep = 0">
    ↺ Reset
  </div>
</div>

---

# Repeat

If $G_h$ still has no perfect matching, just repeat the process — each iteration adds at least one new edge.

<div class="flex justify-center items-center gap-0 mt-6">
  <div class="flex flex-col items-center">
    <div class="p-2 px-3" style="background:var(--c-bg-soft);border:2px solid var(--c-border);border-radius:8px;font-size:0.9rem;font-weight:600">Search Tree</div>
    <div class="h-6 w-0" style="border-left:2px dashed var(--c-border)"></div>
    <div class="p-2 px-3" style="background:#fef3c7;border:2px solid #d97706;border-radius:8px;font-size:0.9rem;font-weight:600;color:#92400e">No Augmenting Path?</div>
    <div class="h-6 w-0" style="border-left:2px dashed var(--c-border)"></div>
    <div class="p-2 px-3" style="background:var(--c-primary-soft);border:2px solid var(--c-primary);border-radius:8px;font-size:0.9rem;font-weight:600">Compute <MathText text="$\delta$" /></div>
    <div class="h-6 w-0" style="border-left:2px dashed var(--c-border)"></div>
    <div class="p-2 px-3" style="background:var(--c-bg-soft);border:2px solid var(--c-border);border-radius:8px;font-size:0.9rem;font-weight:600">Adjust Labels</div>
    <div class="h-6 w-0" style="border-left:2px dashed var(--c-border)"></div>
    <div class="p-2 px-3" style="background:var(--c-bg-soft);border:2px solid var(--c-border);border-radius:8px;font-size:0.9rem;font-weight:600">New Edges in <MathText text="$G_h$" /></div>
  </div>
  <div class="flex flex-col items-center mx-4">
    <div class="text-2xl" style="color:var(--c-accent)">↻</div>
    <div style="font-size:0.8rem;color:var(--c-text-dim)">repeat</div>
  </div>
  <div class="flex flex-col items-center">
    <div class="p-2 px-3" style="background:var(--c-bg-soft);border:2px solid var(--c-border);border-radius:8px;font-size:0.9rem;font-weight:600">Build Search Tree</div>
    <div class="h-6 w-0" style="border-left:2px solid var(--c-primary)"></div>
    <div class="p-2 px-3" style="background:#dde5d4;border:2px solid var(--c-primary);border-radius:8px;font-size:0.9rem;font-weight:600;color:var(--c-primary)">Augmenting Path Found?</div>
    <div class="flex gap-2 mt-3">
      <div class="p-2 px-3" style="background:#dde5d4;border:2px solid var(--c-primary);border-radius:8px;font-size:0.9rem;font-weight:600;color:var(--c-primary)">Yes → Flip!</div>
      <div class="p-2 px-3" style="background:#fef3c7;border:2px solid #d97706;border-radius:8px;font-size:0.9rem;font-weight:600;color:#92400e">No → Repeat</div>
    </div>
  </div>
</div>

<div style="text-align:center;margin-top:1.2rem;font-size:0.95rem;color:var(--c-text-dim)">
The process always terminates — each <MathText text="$\delta$" /> adds an edge, and there are only finitely many edges.
</div>

---

# Hungarian Algorithm

Putting it all together — the algorithm is surprisingly compact:

<div class="flex gap-6 items-start mt-4">
<div class="flex-1" style="background:rgba(255,252,245,0.7);border:1px solid var(--c-border);border-radius:10px;padding:0.8rem 1rem;font-size:0.95rem;line-height:1.8">

```text
for each boy b:
  h(b) = max weight from b
for each girl g:
  h(g) = 0

repeat
  G_h = { edges where h(b)+h(g)=w(b,g) }
  M = greedy matching in G_h
  F = alternating tree from unmatched vertices

  if F reaches a free girl:
    augment M along the path
  else if F reaches no free girl:
    δ = min{ h(b)+h(g)-w(b,g) | b∈F_L, g∉F_R }
    for b in F_L:  h(b) -= δ
    for g in F_R:  h(g) += δ
    // new edges enter G_h → try again

until M is a perfect matching
return M
```

</div>
<div class="flex-1 text-lg leading-relaxed p-4" style="border-left:3px solid var(--c-accent,#b45309);background:var(--c-bg-soft,#f1ead9);border-radius:0 8px 8px 0">

The algorithm is **guaranteed to find the optimal matching** in polynomial time — each iteration either grows the matching or adds a new edge, and it never backtracks.

</div>
</div>

---

<script setup>
import { ref, nextTick, onMounted } from 'vue'

const n = ref(4)
const curStep = ref(0)
const tryGraph = ref(null)
const steps = ref([])

function genWeights(size) {
  const w = []
  for (let i = 0; i < size; i++) {
    w[i] = []
    for (let j = 0; j < size; j++) w[i][j] = Math.floor(Math.random() * 9) + 1
  }
  return w
}

function greedyMatch(ghEdges, size) {
  const mL = Array(size).fill(-1), mR = Array(size).fill(-1)
  for (let i = 0; i < size; i++)
    for (const e of ghEdges)
      if (e.from === i && mR[e.to] === -1) { mL[i] = e.to; mR[e.to] = i; break }
  return { matchL: mL, matchR: mR }
}

function findAugPath(ghEdges, matchL, matchR, root, size) {
  const visL = new Set([root]), visR = new Set()
  const parL = {}, parR = {}
  const queue = [root]
  while (queue.length) {
    const cur = queue.shift()
    for (const e of ghEdges) {
      if (e.from !== cur || visR.has(e.to)) continue
      visR.add(e.to)
      parR[e.to] = cur
      if (matchR[e.to] === -1) {
        // free girl — trace back
        const path = []
        let g = e.to, b = cur
        while (b !== undefined) {
          path.push({ from: b, to: g })
          if (parL[b] !== undefined) {
            g = parL[b]
            b = parR[g]
          } else b = undefined
        }
        return path.reverse()
      }
      const nb = matchR[e.to]
      if (!visL.has(nb)) { visL.add(nb); parL[nb] = e.to; queue.push(nb) }
    }
  }
  return null
}

function augment(path, matchL, matchR) {
  for (const e of path) {
    if (matchL[e.from] === e.to) { matchL[e.from] = -1; matchR[e.to] = -1 }
    else { matchL[e.from] = e.to; matchR[e.to] = e.from }
  }
}

function computeSteps(w, size) {
  const s = []
  const hL = w.map(row => Math.max(...row)), hR = Array(size).fill(0)

  s.push({ leftLabels: [...hL], rightLabels: [...hR],
    ghEdges: (() => { const all = []; for (let i = 0; i < size; i++) for (let j = 0; j < size; j++) all.push({ from: i, to: j, weight: w[i][j] }); return all })(),
    matchEdges: [], newEdges: [], treeEdges: [],
    caption: 'Initialize: boys = max weight, girls = 0' })

  let ghEdges = []
  for (let i = 0; i < size; i++)
    for (let j = 0; j < size; j++)
      if (hL[i] + hR[j] === w[i][j]) ghEdges.push({ from: i, to: j, weight: w[i][j] })
  s.push({ leftLabels: [...hL], rightLabels: [...hR], ghEdges: ghEdges.map(e => ({ ...e })), matchEdges: [], newEdges: [], treeEdges: [],
    caption: `Build $G_h$: ${ghEdges.length} equality edges` })

  let { matchL, matchR } = greedyMatch(ghEdges, size)
  let matchEdges = ghEdges.filter(e => matchL[e.from] === e.to)
  s.push({ leftLabels: [...hL], rightLabels: [...hR], ghEdges: ghEdges.map(e => ({ ...e })), matchEdges: matchEdges.map(e => ({ ...e })), newEdges: [], treeEdges: [],
    caption: `Greedy matching: size ${matchEdges.length}/${size}` })

  let iter = 0
  while (matchEdges.length < size && iter < 30) {
    iter++
    const root = matchL.indexOf(-1)
    // BFS for display tree
    const visL = new Set([root]), visR = new Set()
    const queue = [{ v: root, s: 'L' }]
    const treeE = []
    let found = false
    while (queue.length && !found) {
      const c = queue.shift()
      if (c.s === 'L') {
        for (const e of ghEdges) {
          if (e.from === c.v && !visR.has(e.to)) {
            visR.add(e.to)
            treeE.push({ from: e.from, to: e.to, weight: e.weight })
            if (matchR[e.to] === -1) { found = true; break }
            queue.push({ v: e.to, s: 'R' })
          }
        }
      } else {
        const m = matchR[c.v]
        if (m !== -1 && !visL.has(m)) { visL.add(m); queue.push({ v: m, s: 'L' }) }
      }
    }

    const path = findAugPath(ghEdges, matchL, matchR, root, size)
    if (path) {
      s.push({ leftLabels: [...hL], rightLabels: [...hR], ghEdges: ghEdges.map(e => ({ ...e })), matchEdges: matchEdges.map(e => ({ ...e })), newEdges: [], treeEdges: treeE,
        caption: `Search from B_${root + 1}: augmenting path found!` })
      augment(path, matchL, matchR)
      matchEdges = ghEdges.filter(e => matchL[e.from] === e.to)
      s.push({ leftLabels: [...hL], rightLabels: [...hR], ghEdges: ghEdges.map(e => ({ ...e })), matchEdges: matchEdges.map(e => ({ ...e })), newEdges: [], treeEdges: [],
        caption: `Augmented! Matching size ${matchEdges.length}/${size}` })
    } else {
      let delta = Infinity
      for (const b of [...visL])
        for (let g = 0; g < size; g++)
          if (!visR.has(g))
            delta = Math.min(delta, hL[b] + hR[g] - w[b][g])
      if (delta === Infinity) break
      for (const b of [...visL]) hL[b] -= delta
      for (const g of [...visR]) hR[g] += delta

      const newEdges = []
      for (let i = 0; i < size; i++)
        for (let j = 0; j < size; j++)
          if (hL[i] + hR[j] === w[i][j] && !ghEdges.some(e => e.from === i && e.to === j))
            newEdges.push({ from: i, to: j, weight: w[i][j] })
      ghEdges.push(...newEdges)

      s.push({ leftLabels: [...hL], rightLabels: [...hR], ghEdges: ghEdges.map(e => ({ ...e })), matchEdges: matchEdges.map(e => ({ ...e })), newEdges: newEdges.map(e => ({ ...e })), treeEdges: treeE,
        caption: `δ=${delta} → adjust labels → ${newEdges.length} new edge${newEdges.length > 1 ? 's' : ''}` })
    }
  }

  s.push({ leftLabels: [...hL], rightLabels: [...hR], ghEdges: ghEdges.map(e => ({ ...e })), matchEdges: matchEdges.map(e => ({ ...e })), newEdges: [], treeEdges: [],
    caption: matchEdges.length === size ? 'Perfect matching found!' : `Max matching: size ${matchEdges.length}/${size}` })
  return s
}

function applyTryStep(s) {
  const g = tryGraph.value
  if (!g || !steps.value[s]) return
  g.reset()
  const st = steps.value[s]
  const size = st.leftLabels.length
  for (let i = 0; i < size; i++) {
    g.setOuterLabel('left', i, String(st.leftLabels[i]), true)
    g.setOuterLabel('right', i, String(st.rightLabels[i]), true)
  }
  const ids = {}
  for (const e of st.ghEdges) ids[`${e.from}-${e.to}`] = g.addEdge({ from: e.from, to: e.to, weight: e.weight })
  for (const e of st.matchEdges) { const id = ids[`${e.from}-${e.to}`]; if (id !== undefined) g.updateEdge(id, { highlighted: true }) }
  for (const e of st.newEdges) { const id = ids[`${e.from}-${e.to}`]; if (id !== undefined) g.updateEdge(id, { color: '#2563eb', width: 3 }) }
  for (const e of st.treeEdges) { const id = ids[`${e.from}-${e.to}`]; if (id !== undefined) g.updateEdge(id, { dashed: true }) }
}

function regenerate() {
  steps.value = computeSteps(genWeights(n.value), n.value)
  curStep.value = 0
  nextTick(() => applyTryStep(0))
}

function prevStep() { if (curStep.value > 0) { curStep.value--; nextTick(() => applyTryStep(curStep.value)) } }
function nextStep() { if (curStep.value < steps.value.length - 1) { curStep.value++; nextTick(() => applyTryStep(curStep.value)) } }

onMounted(() => regenerate())
</script>

# Let's Try It Out!!!

<div class="flex flex-col items-center">
  <BipartiteGraph
    :key="n"
    ref="tryGraph"
    :left-count="n"
    :right-count="n"
    :left-labels="Array.from({length:n},(_,i)=>`B_${i+1}`)"
    :left-labels-latex="true"
    :right-labels="Array.from({length:n},(_,i)=>`G_${i+1}`)"
    :right-labels-latex="true"
    :width="380" :height="Math.min(300, 80 + n * 22)" :node-radius="Math.max(4, 14 - n)" :outer-label-offset="6"
  />

  <div class="demo-caption" style="font-size:1rem">
    <MathText :text="steps[curStep]?.caption || ''" />
  </div>

  <div class="flex gap-3 items-center mt-2">
    <div class="flex items-center gap-2">
      <span style="font-size:0.8rem;color:var(--c-text-dim)">n = {{ n }}</span>
      <input type="range" min="3" max="10" v-model.number="n" class="try-slider" @change="regenerate()" />
    </div>
    <button class="try-btn try-regen" @click="regenerate">Regenerate</button>
    <div class="flex gap-1">
      <button class="try-btn" @click="prevStep" :disabled="curStep <= 0">← Prev</button>
      <button class="try-btn" @click="nextStep" :disabled="curStep >= steps.length - 1">Next →</button>
    </div>
  </div>
</div>



