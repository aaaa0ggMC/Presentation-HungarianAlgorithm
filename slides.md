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

# Solution v0.0

> What if each boy's best-lover is unique (disjoint)?

If all boys' highest-score edges go to different girls, we can just pick them all — greedy works!

<GreedyDemo />

So $w(M^*) = \sum_{i=1}^{n} \max_{j} w(b_i,g_j)$ when the maxima are disjoint.

---

# Problem v0.0

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

Each vertex gets a label (call it $h$ or $\ell$):

$$
h(b_i) = \max_{g_j \in G}\, w(b_i,g_j) \qquad\qquad h(g_j) = 0
$$

**Key insight** — from Solution v0.0, if every edge in $M$ satisfies $h(b_i)+h(g_j)=w(b_i,g_j)$, then:

$$
w(M) = \sum_{(b_i,g_j)\in M} w(b_i,g_j) = \sum_{i} h(b_i) + \sum_{j} h(g_j)
$$

The **total score** of the matching equals the **sum of all labels**. If we can *keep this true* while improving the matching, we win.

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
    <MathText text="$\Sigma h = 5+7+6 = 18$ &ensp;→&ensp; upper bound of $w(M^*)$" />
  </div>
</div>

---




