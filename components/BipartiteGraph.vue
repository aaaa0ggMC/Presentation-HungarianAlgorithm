<script setup lang="ts">
import { computed, reactive, ref, watch, nextTick, onMounted } from 'vue'
import katex from 'katex'
import 'katex/dist/katex.min.css'

// ============== Types ==============
export interface Edge {
  from: number
  to: number
  weight?: number | string
  weightPos?: number             // 0~1,沿边位置
  weightLatex?: boolean          // 权重是否用 KaTeX 渲染
  color?: string
  width?: number
  dashed?: boolean
  highlighted?: boolean
  labelOffset?: { dx?: number; dy?: number }
  id?: number                    // 内部 id,addEdge 返回
}

interface Node {
  index: number
  label: string                  // 内部文字
  labelLatex?: boolean
  outerLabel?: string            // 外侧 label(顶标)
  outerLabelLatex?: boolean
  highlighted?: boolean
  highlightColor?: string
  x: number
  y: number
}

interface Props {
  leftCount?: number
  rightCount?: number
  leftLabels?: string[]
  rightLabels?: string[]
  leftLabelsLatex?: boolean
  rightLabelsLatex?: boolean
  leftOuterLabels?: string[]
  rightOuterLabels?: string[]
  outerLabelsLatex?: boolean
  leftColor?: string
  rightColor?: string
  leftBorderColor?: string
  rightBorderColor?: string
  title?: string
  leftTitle?: string
  rightTitle?: string
  width?: number
  height?: number
  nodeRadius?: number
  outerLabelOffset?: number
  edges?: Edge[]
  autoAvoidOverlap?: boolean
  animationDuration?: number     // 动画基础时长 ms
}

const props = withDefaults(defineProps<Props>(), {
  leftCount: 4,
  rightCount: 4,
  leftColor: '#dde5d4',
  rightColor: '#f4dcd4',
  leftBorderColor: '#6b7c5e',
  rightBorderColor: '#a0614f',
  width: 480,
  height: 300,
  nodeRadius: 18,
  outerLabelOffset: 22,
  autoAvoidOverlap: true,
  animationDuration: 400,
})

// ============== State ==============
const leftNodes = reactive<Node[]>([])
const rightNodes = reactive<Node[]>([])
const edgeList = reactive<Edge[]>([])
let nextEdgeId = 0

// ============== Layout ==============
const padding = 50          // 左右 padding
const topPadding = 50       // 顶部空间(列标题 + 间距)
const bottomPadding = 30
const titleHeight = props.title ? 36 : 0

function computeY(count: number, index: number): number {
  const innerHeight = props.height - topPadding - bottomPadding - titleHeight
  if (count === 1) return titleHeight + topPadding + innerHeight / 2
  const gap = innerHeight / (count - 1)
  return titleHeight + topPadding + index * gap
}

const leftX = computed(() => padding + props.nodeRadius)
const rightX = computed(() => props.width - padding - props.nodeRadius)

function initNodes() {
  leftNodes.length = 0
  rightNodes.length = 0
  for (let i = 0; i < props.leftCount; i++) {
    leftNodes.push({
      index: i,
      label: props.leftLabels?.[i] ?? `b${i + 1}`,
      labelLatex: props.leftLabelsLatex,
      outerLabel: props.leftOuterLabels?.[i],
      outerLabelLatex: props.outerLabelsLatex,
      x: leftX.value,
      y: computeY(props.leftCount, i),
    })
  }
  for (let i = 0; i < props.rightCount; i++) {
    rightNodes.push({
      index: i,
      label: props.rightLabels?.[i] ?? `g${i + 1}`,
      labelLatex: props.rightLabelsLatex,
      outerLabel: props.rightOuterLabels?.[i],
      outerLabelLatex: props.outerLabelsLatex,
      x: rightX.value,
      y: computeY(props.rightCount, i),
    })
  }
}

function initEdges() {
  edgeList.length = 0
  nextEdgeId = 0
  if (props.edges) {
    props.edges.forEach(e => {
      edgeList.push({ ...e, id: nextEdgeId++ })
    })
  }
}

initNodes()
initEdges()

// ============== Geometry ==============
function getNode(side: 'left' | 'right', index: number): Node | undefined {
  return side === 'left' ? leftNodes[index] : rightNodes[index]
}

function edgeEndpoints(e: Edge) {
  const from = getNode('left', e.from)
  const to = getNode('right', e.to)
  if (!from || !to) return null
  return { x1: from.x, y1: from.y, x2: to.x, y2: to.y }
}

function edgeWeightXY(e: Edge): { x: number; y: number } {
  const pts = edgeEndpoints(e)
  if (!pts) return { x: 0, y: 0 }
  const t = e.weightPos ?? autoWeightPos(e)
  let x = pts.x1 + (pts.x2 - pts.x1) * t
  let y = pts.y1 + (pts.y2 - pts.y1) * t
  if (e.labelOffset) {
    x += e.labelOffset.dx ?? 0
    y += e.labelOffset.dy ?? 0
  }
  return { x, y }
}

// 按 from 分组,组内按 to 排序,分配 0.25/0.5/0.75 等槽位
function autoWeightPos(e: Edge): number {
  const siblings = edgeList.filter(x => x.from === e.from && x.weight !== undefined)
  if (siblings.length <= 1) return 0.5
  const idx = siblings.findIndex(x => x.id === e.id)
  const slots = [0.3, 0.55, 0.78, 0.42, 0.66, 0.85]
  return slots[idx % slots.length]
}

function edgeAngle(e: Edge): number {
  const pts = edgeEndpoints(e)
  if (!pts) return 0
  return Math.atan2(pts.y2 - pts.y1, pts.x2 - pts.x1) * 180 / Math.PI
}

function edgeLength(e: Edge): number {
  const pts = edgeEndpoints(e)
  if (!pts) return 0
  return Math.hypot(pts.x2 - pts.x1, pts.y2 - pts.y1)
}

// ============== Anti-overlap for weights ==============
// 纯函数:给定已放置的 chips,为当前边沿边找一个不撞的位置
function placeWeightAlongEdge(
  e: Edge,
  placed: { x: number; y: number; r: number }[],
): { x: number; y: number } {
  const pts = edgeEndpoints(e)
  if (!pts) return { x: 0, y: 0 }

  const r = 7  // chip 半径(视觉 approx),跟 weightChipSize/2 接近
  const margin = 12

  const userFixed = e.weightPos !== undefined
  const baseT = e.weightPos ?? autoWeightPos(e)

  // 严格沿边滑动的候选 t,从「离 baseT 最近」到「离 baseT 最远」排
  const tCandidates = userFixed
    ? [baseT]
    : [
        baseT,
        0.5, 0.35, 0.65, 0.25, 0.75, 0.2, 0.8, 0.15, 0.85,
        0.42, 0.58, 0.3, 0.7, 0.12, 0.88,
      ].filter((v, i, a) => a.indexOf(v) === i)

  for (const t of tCandidates) {
    let x = pts.x1 + (pts.x2 - pts.x1) * t
    let y = pts.y1 + (pts.y2 - pts.y1) * t
    if (e.labelOffset) {
      x += e.labelOffset.dx ?? 0
      y += e.labelOffset.dy ?? 0
    }
    if (x < margin || x > props.width - margin) continue
    if (y < margin || y > props.height - margin) continue
    const collides = placed.some(p => Math.hypot(p.x - x, p.y - y) < p.r + r + 2)
    if (!collides) return { x, y }
  }

  // 兜底:硬放 baseT
  const x = Math.min(Math.max(pts.x1 + (pts.x2 - pts.x1) * baseT, margin), props.width - margin)
  const y = Math.min(Math.max(pts.y1 + (pts.y2 - pts.y1) * baseT, margin), props.height - margin)
  return { x, y }
}

// 纯 computed:无副作用,每次重算结果一致
const weightPositions = computed(() => {
  const placed: { x: number; y: number; r: number }[] = []
  const map = new Map<number, { x: number; y: number }>()
  // 先放高亮的(优先占好位置),再放普通的;同优先级按 id 稳定排序,避免抖动
  const sorted = [...edgeList].sort((a, b) => {
    const hd = (b.highlighted ? 1 : 0) - (a.highlighted ? 1 : 0)
    if (hd !== 0) return hd
    return (a.id ?? 0) - (b.id ?? 0)
  })
  sorted.forEach(e => {
    if (e.weight === undefined || e.weight === null) return
    const pos = placeWeightAlongEdge(e, placed)
    placed.push({ x: pos.x, y: pos.y, r: 7 })
    map.set(e.id!, pos)
  })
  return map
})

// ============== KaTeX rendering ==============
const renderedLatex = reactive(new Map<string, string>())

function renderLatex(key: string, tex: string) {
  if (renderedLatex.has(key)) return
  try {
    renderedLatex.set(key, katex.renderToString(tex, {
      throwOnError: false,
      output: 'html',
    }))
  } catch {
    renderedLatex.set(key, tex)
  }
}

function latexKey(prefix: string, ...args: (string | number | undefined)[]) {
  return `${prefix}:${args.join(':')}`
}

// 预渲染所有 latex — 同步执行,不依赖 watch
function prerenderAllLatex() {
  leftNodes.forEach((n, i) => {
    if (n.labelLatex && n.label) renderLatex(latexKey('nl', 'l', i, n.label), n.label)
    if (n.outerLabelLatex && n.outerLabel) renderLatex(latexKey('ol', 'l', i, n.outerLabel), n.outerLabel)
  })
  rightNodes.forEach((n, i) => {
    if (n.labelLatex && n.label) renderLatex(latexKey('nl', 'r', i, n.label), n.label)
    if (n.outerLabelLatex && n.outerLabel) renderLatex(latexKey('ol', 'r', i, n.outerLabel), n.outerLabel)
  })
  edgeList.forEach(e => {
    if (e.weightLatex && e.weight !== undefined) {
      renderLatex(latexKey('w', e.id, String(e.weight)), String(e.weight))
    }
  })
}
prerenderAllLatex()

// 后续动态 addNode/addEdge/setOuterLabel 时补渲染
watch([() => leftNodes.length, () => rightNodes.length, () => edgeList.length], () => {
  prerenderAllLatex()
}, { flush: 'post' })

// ============== Public API ==============
function addNode(side: 'left' | 'right', label?: string): number {
  const arr = side === 'left' ? leftNodes : rightNodes
  const idx = arr.length
  const newNode: Node = {
    index: idx,
    label: label ?? (side === 'left' ? `B${idx + 1}` : `G${idx + 1}`),
    x: side === 'left' ? leftX.value : rightX.value,
    y: 0,
  }
  arr.push(newNode)
  // 重新布局
  nextTick(() => {
    arr.forEach((n, i) => {
      n.index = i
      n.y = computeY(arr.length, i)
    })
  })
  return idx
}

function removeNode(side: 'left' | 'right', index: number) {
  const arr = side === 'left' ? leftNodes : rightNodes
  arr.splice(index, 1)
  arr.forEach((n, i) => {
    n.index = i
    n.y = computeY(arr.length, i)
  })
  // 清理相关边
  for (let i = edgeList.length - 1; i >= 0; i--) {
    const e = edgeList[i]
    if (side === 'left' && e.from === index) edgeList.splice(i, 1)
    else if (side === 'right' && e.to === index) edgeList.splice(i, 1)
    else {
      if (side === 'left' && e.from > index) e.from--
      if (side === 'right' && e.to > index) e.to--
    }
  }
}

function addEdge(edge: Edge): number {
  const id = nextEdgeId++
  edgeList.push({ ...edge, id })
  return id
}

function removeEdge(id: number) {
  const idx = edgeList.findIndex(e => e.id === id)
  if (idx >= 0) edgeList.splice(idx, 1)
}

function updateEdge(id: number, partial: Partial<Edge>) {
  const e = edgeList.find(e => e.id === id)
  if (e) Object.assign(e, partial)
}

function highlightEdge(id: number) {
  const e = edgeList.find(e => e.id === id)
  if (e) e.highlighted = true
}

function unhighlightEdge(id: number) {
  const e = edgeList.find(e => e.id === id)
  if (e) e.highlighted = false
}

function clearHighlights() {
  edgeList.forEach(e => { e.highlighted = false })
  leftNodes.forEach(n => { n.highlighted = false; n.highlightColor = undefined })
  rightNodes.forEach(n => { n.highlighted = false; n.highlightColor = undefined })
}

const pendingTimers = new Set<ReturnType<typeof setTimeout>>()

function setMatching(ids: number[], staggerMs = 100) {
  clearHighlights()
  ids.forEach((id, i) => {
    const t = setTimeout(() => {
      pendingTimers.delete(t)
      highlightEdge(id)
    }, i * staggerMs)
    pendingTimers.add(t)
  })
}

function highlightNode(side: 'left' | 'right', index: number, color?: string) {
  const n = getNode(side, index)
  if (n) {
    n.highlighted = true
    if (color) n.highlightColor = color
  }
}

function unhighlightNode(side: 'left' | 'right', index: number) {
  const n = getNode(side, index)
  if (n) {
    n.highlighted = false
    n.highlightColor = undefined
  }
}

function setOuterLabel(side: 'left' | 'right', index: number, text: string, latex?: boolean) {
  const n = getNode(side, index)
  if (!n) return
  n.outerLabel = text
  if (latex !== undefined) n.outerLabelLatex = latex
  if (n.outerLabelLatex) {
    renderLatex(latexKey('ol', side === 'left' ? 'l' : 'r', index, text), text)
  }
}

function clearOuterLabel(side: 'left' | 'right', index: number) {
  const n = getNode(side, index)
  if (n) n.outerLabel = undefined
}

function reset() {
  edgeList.length = 0
  nextEdgeId = 0
  clearHighlights()
  leftNodes.forEach(n => { n.outerLabel = undefined })
  rightNodes.forEach(n => { n.outerLabel = undefined })
  pendingTimers.forEach(t => clearTimeout(t))
  pendingTimers.clear()
}

defineExpose({
  addNode, removeNode,
  addEdge, removeEdge, updateEdge,
  highlightEdge, unhighlightEdge,
  highlightNode, unhighlightNode,
  clearHighlights, setMatching,
  setOuterLabel, clearOuterLabel,
  reset,
})

// ============== Animation helpers ==============
const animDuration = computed(() => `${props.animationDuration}ms`)

// 只在挂载后的第一帧播一次入场动画,之后数据更新不重播
const enableEnterAnim = ref(false)
// 权重 chip 等节点/边就位后再显示,避免动画期间位置算出错的扇形
const weightsReady = ref(false)
onMounted(() => {
  requestAnimationFrame(() => {
    enableEnterAnim.value = true
    // 播完就关,防止后续重渲再触发
    setTimeout(() => {
      enableEnterAnim.value = false
      weightsReady.value = true
    }, props.animationDuration + 50)
  })
})

function edgeStyle(e: Edge) {
  const len = edgeLength(e)
  return {
    '--edge-len': `${len}`,
  } as Record<string, string>
}

function edgeClass(e: Edge) {
  return {
    'bg-edge': true,
    'bg-edge-enter': enableEnterAnim.value,
    'bg-edge-highlighted': e.highlighted,
    'bg-edge-dashed': e.dashed,
  }
}

function edgeLineStyle(e: Edge) {
  const style: Record<string, string> = {}
  if (e.color) style.stroke = e.color
  if (e.width) style.strokeWidth = String(e.width)
  return style
}

// 权重 chip 尺寸估算(基于文字长度)
function weightChipSize(w: number | string): { w: number; h: number } {
  const s = String(w)
  return { w: Math.max(14, s.length * 7 + 6), h: 14 }
}
</script>

<template>
  <div class="bipartite-graph" :style="{ width: `${width}px` }">
    <div v-if="title" class="bg-title">{{ title }}</div>
    <svg
      :viewBox="`0 0 ${width} ${height}`"
      :width="width"
      :height="height"
      class="bg-svg"
    >
      <!-- 列标题 -->
      <text
        v-if="leftTitle"
        :x="leftX"
        :y="titleHeight + 18"
        class="bg-col-title"
        text-anchor="middle"
      >{{ leftTitle }}</text>
      <text
        v-if="rightTitle"
        :x="rightX"
        :y="titleHeight + 18"
        class="bg-col-title"
        text-anchor="middle"
      >{{ rightTitle }}</text>

      <!-- 边 -->
      <g class="bg-edges">
        <line
          v-for="e in edgeList.filter(x => edgeEndpoints(x) !== null)"
          :key="e.id"
          :x1="edgeEndpoints(e)!.x1"
          :y1="edgeEndpoints(e)!.y1"
          :x2="edgeEndpoints(e)!.x2"
          :y2="edgeEndpoints(e)!.y2"
          :class="edgeClass(e)"
          :style="[edgeStyle(e), edgeLineStyle(e)]"
        />
      </g>

      <!-- 权重 chip 背景:等节点/边动画播完后再显示,保证位置算准 -->
      <g class="bg-weights" v-if="weightsReady">
        <g
          v-for="e in edgeList.filter(x => x.weight !== undefined && x.weight !== null && weightPositions.has(x.id!))"
          :key="`w-${e.id}`"
          :transform="`translate(${weightPositions.get(e.id!)!.x}, ${weightPositions.get(e.id!)!.y})`"
          :class="{ 'bg-weight': true, 'bg-weight-highlighted': e.highlighted }"
        >
          <rect
            :x="-weightChipSize(e.weight!).w / 2"
            :y="-weightChipSize(e.weight!).h / 2"
            :width="weightChipSize(e.weight!).w"
            :height="weightChipSize(e.weight!).h"
            rx="8"
            class="bg-weight-chip"
            :class="{ 'bg-weight-chip-highlighted': e.highlighted }"
            :style="{
              stroke: e.highlighted
                ? 'var(--c-accent, #b45309)'
                : (e.color || '#9c8b70'),
              fill: e.highlighted
                ? 'var(--c-accent, #b45309)'
                : '#fffdf7',
            }"
          />
          <!-- 文本权重 -->
          <text
            v-if="!e.weightLatex"
            class="bg-weight-text"
            :class="{ 'bg-weight-text-highlighted': e.highlighted }"
            :style="{
              fill: e.highlighted ? '#ffffff' : (e.color || '#7a6f5d'),
            }"
            text-anchor="middle"
            dominant-baseline="central"
          >{{ e.weight }}</text>
          <!-- LaTeX 权重:用 foreignObject 内嵌 HTML -->
          <foreignObject
            v-else
            :x="-weightChipSize(e.weight!).w / 2 - 4"
            :y="-weightChipSize(e.weight!).h / 2 - 4"
            :width="weightChipSize(e.weight!).w + 8"
            :height="weightChipSize(e.weight!).h + 8"
          >
            <div
              class="bg-weight-latex"
              v-html="renderedLatex.get(latexKey('w', e.id, String(e.weight))) || e.weight"
            />
          </foreignObject>
        </g>
      </g>

      <!-- 左节点 -->
      <g class="bg-nodes bg-nodes-left">
        <g
          v-for="n in leftNodes"
          :key="`l-${n.index}`"
          :class="{ 'bg-node': true, 'bg-node-enter': enableEnterAnim, 'bg-node-highlighted': n.highlighted }"
        >
          <circle
            :cx="n.x"
            :cy="n.y"
            :r="nodeRadius"
            class="bg-node-circle"
            :style="{
              fill: n.highlightColor || leftColor,
              stroke: leftBorderColor,
            }"
          />
          <text
            v-if="!n.labelLatex"
            :x="n.x"
            :y="n.y"
            class="bg-node-label"
            text-anchor="middle"
            dominant-baseline="central"
          >{{ n.label }}</text>
          <foreignObject
            v-else
            :x="n.x - nodeRadius"
            :y="n.y - nodeRadius"
            :width="nodeRadius * 2"
            :height="nodeRadius * 2"
          >
            <div
              class="bg-node-latex"
              v-html="renderedLatex.get(latexKey('nl', 'l', n.index, n.label)) || n.label"
            />
          </foreignObject>
          <!-- outer label -->
          <text
            v-if="n.outerLabel && !n.outerLabelLatex"
            :x="n.x - nodeRadius - outerLabelOffset"
            :y="n.y"
            class="bg-outer-label bg-outer-label-left"
            text-anchor="end"
            dominant-baseline="central"
          >{{ n.outerLabel }}</text>
          <foreignObject
            v-else-if="n.outerLabel && n.outerLabelLatex"
            :x="n.x - nodeRadius - outerLabelOffset - 60"
            :y="n.y - 12"
            :width="60"
            :height="24"
          >
            <div
              class="bg-outer-latex bg-outer-latex-left"
              v-html="renderedLatex.get(latexKey('ol', 'l', n.index, n.outerLabel)) || n.outerLabel"
            />
          </foreignObject>
        </g>
      </g>

      <!-- 右节点 -->
      <g class="bg-nodes bg-nodes-right">
        <g
          v-for="n in rightNodes"
          :key="`r-${n.index}`"
          :class="{ 'bg-node': true, 'bg-node-enter': enableEnterAnim, 'bg-node-highlighted': n.highlighted }"
        >
          <circle
            :cx="n.x"
            :cy="n.y"
            :r="nodeRadius"
            class="bg-node-circle"
            :style="{
              fill: n.highlightColor || rightColor,
              stroke: rightBorderColor,
            }"
          />
          <text
            v-if="!n.labelLatex"
            :x="n.x"
            :y="n.y"
            class="bg-node-label"
            text-anchor="middle"
            dominant-baseline="central"
          >{{ n.label }}</text>
          <foreignObject
            v-else
            :x="n.x - nodeRadius"
            :y="n.y - nodeRadius"
            :width="nodeRadius * 2"
            :height="nodeRadius * 2"
          >
            <div
              class="bg-node-latex"
              v-html="renderedLatex.get(latexKey('nl', 'r', n.index, n.label)) || n.label"
            />
          </foreignObject>
          <text
            v-if="n.outerLabel && !n.outerLabelLatex"
            :x="n.x + nodeRadius + outerLabelOffset"
            :y="n.y"
            class="bg-outer-label bg-outer-label-right"
            text-anchor="start"
            dominant-baseline="central"
          >{{ n.outerLabel }}</text>
          <foreignObject
            v-else-if="n.outerLabel && n.outerLabelLatex"
            :x="n.x + nodeRadius + outerLabelOffset"
            :y="n.y - 12"
            :width="60"
            :height="24"
          >
            <div
              class="bg-outer-latex bg-outer-latex-right"
              v-html="renderedLatex.get(latexKey('ol', 'r', n.index, n.outerLabel)) || n.outerLabel"
            />
          </foreignObject>
        </g>
      </g>
    </svg>
  </div>
</template>

<style scoped>
.bipartite-graph {
  display: inline-block;
}

.bg-title {
  text-align: center;
  font-weight: 700;
  font-size: 1.05rem;
  color: var(--c-text, #2b241a);
  padding: 0.5rem 0 0.25rem;
  letter-spacing: 0.02em;
}

.bg-svg {
  display: block;
  background-color: rgba(255, 252, 245, 0.7);
  border: 1px solid var(--c-border, #e2d9c3);
  border-radius: 10px;
}

.bg-col-title {
  font-size: 12px;
  font-weight: 700;
  fill: var(--c-text-dim, #7a6f5d);
  letter-spacing: 0.1em;
  text-transform: uppercase;
  font-family: 'Fira Code', monospace;
}

/* ===== Edges ===== */
.bg-edge {
  stroke: var(--c-border, #d6cdb8);
  stroke-width: 1.4;
  stroke-linecap: round;
  transition: stroke v-bind(animDuration) ease, stroke-width v-bind(animDuration) ease;
}

.bg-edge-enter {
  animation: bg-edge-draw v-bind(animDuration) ease-out both;
}

.bg-edge-highlighted {
  stroke: var(--c-accent, #b45309);
  stroke-width: 3;
}

.bg-edge-dashed {
  stroke-dasharray: 5, 4 !important;
  animation: none;
}

@keyframes bg-edge-draw {
  from {
    stroke-dasharray: var(--edge-len);
    stroke-dashoffset: var(--edge-len);
  }
  to {
    stroke-dasharray: var(--edge-len);
    stroke-dashoffset: 0;
  }
}

/* ===== Weights ===== */
.bg-weight-enter {
  animation: bg-pop v-bind(animDuration) ease-out both;
}

.bg-weight-chip {
  fill: var(--c-bg, #f7f2e7);
  stroke: var(--c-border, #d6cdb8);
  stroke-width: 0.8;
  transition: all v-bind(animDuration) ease;
}

.bg-weight-highlighted .bg-weight-chip {
  fill: #fff7ea;
  stroke: var(--c-accent, #b45309);
  stroke-width: 1.2;
}

.bg-weight-text {
  font-size: 10px;
  font-family: 'Fira Code', monospace;
  font-weight: 600;
  transition: fill v-bind(animDuration) ease;
}

.bg-weight-text-highlighted {
  font-weight: 700;
  font-size: 11px;
}

.bg-weight-latex {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
  font-size: 10px;
  color: var(--c-text-dim, #7a6f5d);
}

.bg-weight-highlighted .bg-weight-latex {
  color: var(--c-accent, #b45309);
  font-weight: 700;
}

/* ===== Nodes ===== */
.bg-node-enter {
  animation: bg-pop v-bind(animDuration) cubic-bezier(0.34, 1.56, 0.64, 1) both;
}

.bg-node-circle {
  stroke-width: 1.6;
  transition: all v-bind(animDuration) ease;
}

.bg-node-highlighted .bg-node-circle {
  stroke-width: 2.5;
  filter: drop-shadow(0 0 6px rgba(180, 83, 9, 0.5));
}

.bg-node-label {
  font-size: 13px;
  font-weight: 700;
  fill: var(--c-text, #2b241a);
  pointer-events: none;
}

.bg-node-latex,
.bg-outer-latex {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
  font-size: 13px;
  color: var(--c-text, #2b241a);
}

.bg-outer-latex-left {
  justify-content: flex-end;
}

.bg-outer-latex-right {
  justify-content: flex-start;
}

.bg-outer-label {
  font-size: 12px;
  font-weight: 600;
  fill: var(--c-text-dim, #7a6f5d);
  font-family: 'Fira Code', monospace;
}

/* ===== Animations ===== */
@keyframes bg-pop {
  0% {
    opacity: 0;
    transform: scale(0.6);
  }
  60% {
    transform: scale(1.08);
  }
  100% {
    opacity: 1;
    transform: scale(1);
  }
}
</style>
