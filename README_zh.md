# 🦠 Isopods Simulation Under Artificial Light at Night

这是一个基于纯前端（Vanilla JS & Canvas）开发的数字虚拟行为学实验室，用于模拟和观察鼠妇（*Isopods*，又称潮虫）群体的自组织行为形态（Self-Organizing Behaviors）。

通过数学建模与行为学规则设计，本模拟真实还原了单只鼠妇在面对光照、同伴和拥挤环境时的底层本能，展示了群体是如何在没有任何宏观指挥的情况下，自发构建出稳态拓扑管道的。


[https://rustyduoedge.github.io/isopods-simulation-under-ALAN/](https://rustyduoedge.github.io/isopods-simulation-under-ALAN/)

## 🔬 鼠妇的两大核心行为动力学 (Core Dynamics)

在虚拟世界中，每只鼠妇（智能体 `SmartIsopod`）的移动方向和速度完全由**环境光向性力矩**与**多体邻域社会力矩**的合力矩（`total_omega`）共同驱动：

### 一、 光强敏感度与三区划分 (3-Phase Taxis)
实验室中央配有一盏光强自中心向外呈**费米-狄拉克平滑衰减（Fermi-Dirac smooth step decay）**分布的路灯。单只鼠妇通过前端 30° 角的左右两根感知触角（`I_left` 与 `I_right`）检测局部光强梯度，并表现出三种截然不同的行为反馈：
* **负趋光核心排斥区 ($I > I_{max}$，红色边界内)**：光强超过耐受极限。鼠妇产生强烈的恐光本能，产生巨大的转向力矩（`omega_light = k_light_turn * (I_right - I_left)`），驱动自身迅速掉头逃离核心。
* **正趋光并轨舒适区 ($I_{min} \le I \le I_{max}$，橙红之间的管道)**：这里的亮度最符合它们的心理预期。在此区域内，光向性力矩关闭（`omega_light = 0.0`），它们会安顿下来，沿着亮度切线轨道平稳地并轨爬行。
* **边缘忽略回归区 ($I_{threshold} \le I < I_{min}$，灰色与橙色之间)**：环境过于昏暗，它们会失去安全感，表现出微弱的正趋光性（`omega_light = k_light_turn * (I_left - I_right)`），调头往稍微有光亮的地方聚拢。

---

### 二、 多体邻域社会关系 (Inter-Agent Social Dynamics)
除了与光源的互动，智能体在局部感知半径（`R_c`）内会实时评估与其他所有邻居的距离与角度矢量。代码通过遍历多体邻域矩阵，复现了复杂的生物社会学和微观交通流行为：

#### 1. 局部从众效应 (Vicsek Polarized Alignment)
* **代码实现**：`omega_align += Math.sin(other.theta - this.theta)`
* **行为模式**：经典的维塞克（Vicsek）多体对齐模型。在敏感半径 `R_c` 内，单只鼠妇会持续计算邻居与自己的角度差。通过正弦力矩响应，它会表现出强烈的“跟风”本能，不断修正自己的朝向。
* **演化结果**：当纯暗环境没有路灯干扰时，这种相互从众的力矩会主导全局，让全画布散乱的鼠妇自发凝聚成方向高度统一、像鱼群或鸟群一样在环形拓扑空间中集体游弋的极化流。

#### 2. 刚体边界避障 (2D Avoidance Vector Masking)
* **代码实现**：`omega_repel += (u_x * (-Math.sin(this.theta)) + u_y * Math.cos(this.theta))`
* **行为模式**：每只鼠妇都有一个物理上的“硬壳核心半径”（`R_r = 0.42`）。一旦两个同伴的二维空间几何距离跌破这个刚性硬壳边界，系统会瞬间产生一个与彼此相对位置矢量彻底相反的排斥力矩（`omega_repel`），强迫它们的头部滑离对方。
* **演化结果**：这提供了二维平面上的水平防撞盾，保证了群体在无序乱跑时相互之间是“弹开”的，呈现出清晰的固体颗粒感。

#### 3. 跟车刹车机制 (Car-Following Braking Evaluation)
* **代码实现**：`let factor = 1.0 - Math.exp(-(dist - this.R_r) / 0.5); factor = Math.max(0.10, factor);`
* **行为模式**：当单只鼠妇发现正前方（角度余弦 `> 0.2`）有其他同伴在减速或爬行时，它会立刻激活类似高级驾驶辅助（ADAS）的**“智能跟车减速”**逻辑。它与前车的距离越近，其速度降额因子（`speed_modifier`）就越呈指数级骤降，最严重时会执行 `0.10` 倍率的强行重度刹车。
* **演化结果**：这种机制极大地避免了由于前方拥堵导致的智能体穿模失真。当并轨舒适区出现交通瓶颈时，你会看到后方的鼠妇像真实交通堵塞一样，排成一动不动的紧密车队。

#### 4. 拟三维叠罗汉 (Quasi-3D Trampling Bypass)
* **代码实现**：`let isStacking = Math.random() < trampleProbability; factor = Math.max(0.65, factor);`
* **行为模式**：当“跟车减速”与“刚体避障”导致管道发生极限拥堵、寸步难行时，代码中引入了一个**纵向自由度突变破局规则**。当触发踩踏概率（`trampleProbability`）时，当前鼠妇会无视上述的 2D 避障排斥力和 0.10 的重刹车惩罚，而是获得一个至少 `0.65` 的速度保留金牌，直接从前车背上踩过去。
* **演化结果**：此时智能体状态突变为 `isTrampling = true`，视觉上化为**霓虹粉色（Neon Pink）**且线宽加粗。通过这一“升维操作”，群体成功在高度拥阻的区域内依靠三维堆叠完成了交通分流，完美模拟了现实中潮虫在角落里挤成一坨的“叠罗汉”生态。

---

## 🎛️ 实验控制面板指南 (Control Panel Guide)

你可以通过页面两侧的滑块（Range Sliders）实时调整环境的物理规则或生物的基因本能，观察稳态平衡是如何被打破或重构的：

### ⚙️ 物理环境控制 (Side Panel Left)
* **播放速度 (`speedSlider`)**：调节整个实验室的时间流速倍率（`1.0x - 4.0x`）。拉大它可以让模拟加速，方便观察群体最终的汇聚形态。
* **鼠妇总数 (`countSlider`)**：动态调整画布中的智能体上限（`10 - 300` 只）。系统会自动进行动态扩容或裁剪数组（`isopods.splice`）。
* **光源物理半径 (`radiusSlider`)**：调整路灯核心光源的物理几何半径 $R_{disk}$。半径越大，光强梯度的覆盖范围越广。

### 🧬 生物本能微调 (Side Panel Right)
* **叠罗汉踩踏概率 (`trampleSlider`)**：调整遭遇拥堵时鼠妇爬向同伴背部的概率（`0% - 100%`）。设置为 `0%` 时，系统在狭窄交界处会因为高刚性碰撞发生严重的“重度刹车减速 lockups”；设置得越高，霓虹粉色的堆叠层级就越丰富。
* **1. 边缘忽略阈值 (`thresholdSlider`, 灰色虚线)**：控制最外围昏暗环境的忽略边界 $I_{threshold}$。
* **2. 正趋光阈值 (`iminSlider`, 橙色虚线)**：控制并轨舒适区的外部界线 $I_{min}$。
* **3. 负趋光阈值 (`imaxSlider`, 红色虚线)**：控制核心排斥区的临界点 $I_{max}$。

> ⚠️ **临界阻挫测试（Geometrical Frustration）**
>
> 如果你通过滑块将正趋光阈值 $I_{min}$ 与负趋光阈值 $I_{max}$ 的间距压缩得极窄（$\le 2.0$），拓扑管道将会闭合。鼠妇会由于逃跑本能与并轨本能在极短的时间和空间内发生极限拉扯，在界面顶部会触发状态警告：
> `⚠️ 临界阻挫！两区闭合，交界线乱哆嗦中！(Sliding mode chattering on boundary!)`

---

## 💡 灵感与致谢 (Inspiration & Acknowledgments)

* **学术原理灵感**：本项目的多体仿真逻辑和三区 Taxis 行为学建模，受到多体生态学研究论文 [*How do isopods aggregate? Formulating and simulating a model based on individual behavior* (Wiley Online Library)](https://onlinelibrary.wiley.com/doi/10.1002/ece3.73487) 的启发。
* **协作开发支持**：本项目全量核心前端 Canvas 动力学管线的搭建、双语本地化引擎（`i18nPack`）的重构、以及本技术说明文档的梳理，均在 Google Gemini 大语言模型的全程辅助协作下共同迭代编码完成。

---

## 🚀 快速本地运行 (Quick Start)

本项目采用纯 Vanilla JS 开发，零第三方依赖（No Node.js, No npm, No Webpack），打开即用：

```bash
# 1. 克隆实验室仓库
git clone github.com/RustyDuoEdge/isopods-simulation-under-ALAN.git
cd isopods-simulation-under-ALAN

# 2. 用任何现代浏览器直接打开 index.html 即可开始实验
open index.html
