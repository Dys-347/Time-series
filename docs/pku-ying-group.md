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
| 2026 | HLinear | **投稿中**（代码已公开） | Hilbert 引导预测：从"数值"到"振荡态"；岳文振为合作者（三作） |

> 注：该组还有一条**多模态支线**（郭若浩主导）：OV-AVSS（ACM MM 2024 Oral）、AVIS（CVPR 2025），做音频-视觉分割，与时序线无关，不在本文分析范围内。

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

### 线三：变换域——从固定基到数据自适应基，再到振荡态

- **批评对象**：Fourier/小波变换用**固定基**（正弦波），不能利用数据集特有的时间相关结构。
- **OrthoTrans（OLinear）**：对**时间维 Pearson 相关矩阵**做特征分解，得到数据自适应的正交基；把序列投影上去，得到解耦的特征域，再用线性层编码。
- **FreEformer**：另一条变换域路线——频域的全局视角 + 变量级 token（类似 iTransformer，但建模的是频谱间的变量依赖）。
- **HLinear（2026，投稿中）**：变换域线的最新延续——**Hilbert 变换**提取振荡态先验（包络+相位），从"数值"到"振荡态"；Hilbert 引导的通道编码（HGCE）做振荡感知的变量间交互。注意同期还有独立工作 AOSNET（arXiv:2606.06010）也在做 Hilbert 引导预测，说明"振荡态"正在成为一个小热点。

一句话：**先换个"看数据的角度"（变换域），再建模**——而这个"角度"本身越来越数据自适应：固定傅里叶基 → 数据自适应正交基 → Hilbert 振荡态。

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

## 4. 评价：可借鉴与可质疑

- **可借鉴**：
  - 即插即用的叙事：每个模块都能单独插进现有模型，降低了别人试用的门槛，是放大影响力的有效写法。
  - OrthoTrans 的思想：数据自适应的解耦变换，对输入处理有普遍启发。
- **可质疑**：
  - 定理假设偏强（Laplace、一阶 Markov），定理更多是解释性的，不宜当作方法有效的主要证据。
  - OrthoTrans 要对相关矩阵做特征分解，大规模数据和在线场景下的代价，论文着墨不多。
  - 可学习混合矩阵的形状与变量数绑定（见 5.5 第 2 条），"即插即用"在变量数变化时需要重新初始化。
- **走向判断**：该组的重心正从架构转向目标函数，下一步可能进入概率预测。这是推测，需要后续论文验证。

## 5. 代码实证：HLinear 的实现细节（2026-10-13 读码）

> 代码：github.com/1004qiong/hlinear，`models/HLinear.py`（242 行）+ `exp/exp_main.py`。以下均为读码确认的事实。

### 5.1 NormLin 复用实锤

`LinearEncoder_Multihead` 就是 OLinear 的 NormLin，一字不差：
```python
A = F.softplus(self.weight_mat)   # 正数化
A = F.normalize(A, p=1, dim=-1)   # 行归一
```
该组**模块复用是工程习惯**：NormLin（OLinear）→ HLinear 的通道混合器 → vecTrans（vLinear）是它的 rank-1 激进版。

### 5.2 训练损失：vLinear Theorem 2 的工程实现版

实际使用的不是 MSE，而是 `HeteroLaplace_loss`（`WeightedL1Loss` 被注释掉了）：
```python
b = sigma_c * l^alpha + eps   # scale 随 horizon 幂律增长，alpha 可学习
loss = |e| / b + log(b)       # Laplace 负对数似然
```
- 这正是 vLinear Theorem 2（Laplace 假设下 horizon-weighted MAE 是 MLE 最优）的**可学习异方差实现**：vLinear 给理论 + 固定权重，HLinear 把它做成 scale 随 horizon 幂律增长、per-channel 可学习的 NLL。
- `sigma_c` 用 GT 的 MAD 初始化（`b_init = MAD/ln2`，Laplace 分布下 MAD = b·ln2，统计上是 principled 的）。
- **连接点**：岳文振是 vLinear 一作 + HLinear 三作——Laplace 思想是同一个人带过来的，不是巧合。

### 5.3 "Hilbert 引导"的真实形态：加法注入 + 全局门

- 引导方式是**加法**：`routed_embed = x_embed + sigmoid(query_alpha) * osc_prior`，不是注意力调制。"Guided"略有包装成分，实质是"Hilbert-augmented"。
- 门控存在但粗糙：`query_alpha` 初始化 -4.0（sigmoid≈0.018），先验初始几乎关闭、靠学习打开——设计是聪明的；但它是**全局标量**，不是 per-variable/per-sample 的。
- 另有一个 per-channel 的残差门 `rel_gate`，调制范围仅 [0.9, 1.1]，作用温和。

### 5.4 工程亮点（扎实的部分）

- 相位处理小心：cos/sin 双通道编码（避免相位 wrap 不连续）、瞬时频率用 `atan2(sin, cos)` 正确处理环绕、用 log amplitude 而非原始振幅。
- `HilbertSummary` 用 mean/std/last/slope 四个统计量 → MLP → 64 维 guide，简单有效。

### 5.5 诚实的局限（读码确认）

1. **Hilbert 直接作用于原始序列**，没有先做 EMD/VMD 分解。对多分量信号，瞬时频率的物理意义存疑（Bedrosian 定理条件不满足）。
2. **`feature_num=21` 硬编码**（Weather 数据集的变量数）：可学习混合矩阵的形状天然 dataset-specific，换数据集要改。所谓"即插即用"在变量数变化时需要重新初始化——这是 NormLin/vecTrans 类方法的固有代价。
3. **仍是点预测**：模型只输出均值，Laplace 的 scale 是 loss 内部参数，没有输出给用户。
4. `HilbertSummary` 里 `nn.Dropout(0.5)` 写死，没用传入的 dropout 参数——小瑕疵。

## 6. 跟踪清单

- [ ] vLinear 录用状态（arXiv:2601.13768，目前仅 v1、代码匿名）
- [ ] HLinear 录用状态（2026 投稿中，代码已公开：github.com/1004qiong/hlinear）
- [ ] "振荡态/Hilbert"是否会成为该组下一条主线（AOSNET 是独立验证信号）
- [ ] 该组下一步是否进入概率预测
- [ ] FreEformer 增强 attention 的后续引用情况
