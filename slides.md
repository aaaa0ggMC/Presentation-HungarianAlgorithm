---
layout: center
class: bg-white text-black
---

# Hungarian Algorithm

匈牙利算法:指派问题的多项式时间解法

<div class="absolute bottom-10 left-0 right-0 text-center">
  <a
    href="https://aaaa0ggmc.github.io/Presentation-HungarianAlgorithm"
    target="_blank"
    style="text-decoration: none; color: inherit; opacity: 0.6; font-size: 14px;"
    onmouseover="this.style.opacity='1'"
    onmouseout="this.style.opacity='0.6'"
  >
    aaaa0ggmc.github.io/Presentation-HungarianAlgorithm
  </a>
</div>

---
layout: center
transition: fade
---

# 问题:指派问题 (Assignment Problem)

$n$ 个任务分给 $n$ 个人,每个人做且只做一个任务

成本矩阵 $C = (c_{ij})_{n\times n}$,$c_{ij}$ 表示第 $i$ 个人做第 $j$ 个任务的成本

**目标**:找到一个指派方案,使总成本最小

$$
\min \sum_{i=1}^{n}\sum_{j=1}^{n} c_{ij} x_{ij}
\quad
\text{s.t. }
\sum_{j} x_{ij} = 1,\ 
\sum_{i} x_{ij} = 1,\ 
x_{ij} \in \{0,1\}
$$

---

# 核心思想

<v-clicks>

- 在成本矩阵的同一行(列)上加减一个常数,**最优解不变**
- 通过行/列变换,让矩阵中出现足够多的 **0**
- 如果能选出 $n$ 个不同行不同列的 0,就找到了最优指派
- 否则:用最少的线覆盖所有 0,调整矩阵,产生更多 0

</v-clicks>

---
layout: center
---

# 算法步骤 (矩阵形式)

1. **行变换**:每行减去该行最小值
2. **列变换**:每列减去该列最小值
3. **试指派**:寻找 $n$ 个独立 0
4. **覆盖**:用最少直线覆盖所有 0( König 定理:最少线数 = 最大独立 0 数)
5. **调整**:未被覆盖元素减去最小未覆盖值,交叉处加上该值,回到 3

直到找到 $n$ 个独立 0 —— 时间复杂度 $O(n^3)$

---
layout: center
class: bg-black text-white
---

# Thanks!

Edit [slides.md](./slides.md) to get started
