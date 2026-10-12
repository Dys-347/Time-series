# 北大应翔华组：从 OLinear 到 vLinear 的思想线分析

> 分析对象：北京大学智能科学技术学院、通用人工智能国家重点实验室，应翔华组。
> 核心成员：应翔华（导师）、岳文振（一作主力）、石吉、郭若浩、邢博伟；合作者刘勇（清华）。
> 分析日期：2026-10-13。vLinear 尚未正式发表，结论以 arXiv 预印本为准。

## 1. 论文年表

| 时间 | 论文 |  venue | 一句话 |
|---|---|---|---|
| 2024 | Sub-adjacent Transformer | arXiv:2404.18948 | 时序异常检测（支线） |
| 2025-01 | FreEformer | IJCAI 2025（arXiv:2501.13989） | 频域增强 Transformer；发现频域下 attention 低秩 |
| 2025-05 | OLinear | **NeurIPS 2025**（arXiv:2505.08550） | 数据自适应正交变换 OrthoTrans + 类 attention 线性层 NormLin |
| 2026-01 | vLinear | arXiv 预印本（2601.13768） | vecTrans（rank-1 替代 attention）+ WFMLoss（监督最终序列） |

## 2. 五条思想线

### 线一：Attention 低秩 → 修 Attention → 扔 Attention

这是该组最完整的一条进化链：

1. **FreEformer 的发现**：把 Transformer 搬到频域后，vanilla attention 矩阵呈现低秩特性——原因是频域本身稀疏，加上 softmax 的"强值聚焦"，导致表达多样性受限。他们的修法是给 attention 矩阵加一个可学习矩阵再做行 L1 归一化，并用理论证明这样能提升特征多样性和梯度流。
2. **OLinear 的回应**：OrthoTrans 的 ablation 显示，把数据变换到解耦的正交域后，Transformer 中 attention 矩阵的**秩提升了**——侧面印证"纠缠的输入导致低秩"是病根。
3. **vLinear 的终局**：vecTrans 直接用**可学习 rank-1 矩阵**替代 attention，复杂度 O(N²)→O(N)，效果反超。

一句话：**从"修 attention"（FreEformer）到"换 attention"（OLinear 的 NormLin）再到"扔 attention"（vLinear 的 vecTrans）**。vLinear 论文 Figure 1 亲手画了这条链：MHSA → NormLin → vecTrans。

### 线二：线性化一切

- **NormLin（OLinear）**：观察到 attention 权重的数学形式——全正数、每行 L1 范数为 1。于是做一个线性层：权重经 Softplus 变正数，再按行归一化。**模仿 attention 的形式，不要 attention 的动态计算**。结果：效果超过 self-attention，FLOPs 减半。
- **vecTrans（vLinear）**：连"模仿"都省了，直接学一个 rank-1 混合矩阵。

哲学含义：该组认为 attention 的成功不在于"输入依赖的动态权重"，而在于"归一化的混合形式"——形式留下，动态计算扔掉。这是 DLinear（2023）"复杂度≠效果"论断在该组的延续和激进化。

### 线三：变换域——从固定基到数据自适应基

- **批评对象**：Fourier/小波变换用**固定基**（正弦波），不能利用数据集特有的时间相关结构。
- **OrthoTrans（OLinear）**：对**时间维 Pearson 相关矩阵**做特征分解，得到数据自适应的正交基；把序列投影上去，得到解耦的特征域，再用线性层编码。
- **FreEformer**：另一条变换域路线——频域的全局视角 + 变量级 token（类似 iTransformer，但建模的是频谱间的变量依赖）。

一句话：**先换个"看数据的角度"（变换域），再建模**。这跟 Autoformer/FEDformer 的"分解进架构"是一脉，但该组走得更远：基本身也是从数据里学的。

### 线四：从架构转向损失（vLinear 的转身）

前三篇都在回答"模型怎么算"（attention、变换域），vLinear 第一次把主战场移到"**模型学什么**"：

- **WFMLoss**：final-series oriented——直接监督最终预测序列，不监督速度场（+8.6%）。
- Theorem 2 用 Laplace + 一阶 Markov 假设给 horizon 加权配了 MLE 最优性故事（假设偏强，详见主站 vLinear 条目）。

这是该组研究重心的转移信号：**架构红利吃完后，开始吃目标函数的红利**。

### 线五：方法论签名（他们怎么写论文）

1. **即插即用**：每篇必带 plug-and-play 宣称——OrthoTrans 可插进现有预测器、NormLin 可替换 attention、vecTrans 可插进 Transformer、FreEformer 的增强 attention 可插进 SOTA、WFMLoss 可作为目标函数插进现有模型。这是刻意设计的影响力放大器。
2. **理论点缀 + 强实证**：每篇配 1–2 个定理（多样性/梯度流、秩、MLE 最优、线性推理简化），但定理假设普遍偏强；真正的证据是 benchmark 数量逐篇递增（18 → 22 benchmarks，124 settings）。
3. **大白话包装**：标题都是"Simple yet effective"、"Powerful"这类直给型，不绕弯子。

## 3. 统一命题（一句话）

> Transformer 的 attention 又贵又低秩，不如换成便宜的线性部件；与其在时域硬刚纠缠的依赖，不如先换到解耦的变换域；与其监督中间量，不如直接监督最终结果。

## 4. 对我们的启示

- **站位**：vLinear 是"改考试"（换监督目标），FlowCast 是"省考场"（单步直达）——同一条"去间接化"思想线的上下游。写 related work 时这是天然的承接句。
- **可偷的**：
  - 即插即用包装：AWL 也可以包装成 plug-and-play loss（"drop-in replacement for MSE/MAE in FM training"），这是该组验证过的有效叙事。
  - OrthoTrans 思想：数据自适应的解耦变换，对时序输入处理有启发（方向 1 的跨实体检索也许用得上）。
- **可质疑的**：
  - 定理假设强（Laplace、一阶 Markov）——rebuttal 级别不要拿定理当挡箭牌。
  - OrthoTrans 对相关矩阵做特征分解，大规模数据/在线场景的代价论文着墨不多。
- **竞争关系**：该组也在往"目标函数"走，下一步很可能碰**概率预测**（单步概率是空白）。方向 3 要抢时间。

## 5. 跟踪清单

- [ ] vLinear 录用状态（arXiv:2601.13768，目前仅 v1、代码匿名）
- [ ] 该组下一步是否进入概率预测 / 单步方向
- [ ] FreEformer 增强 attention 的后续引用情况
