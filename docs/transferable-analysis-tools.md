# 大模型分析工具向时序预测的迁移

> 2026-10-13 记录。核心判断：**分析工具可以搬，分析结论要重验。**
> 大模型领域（LLM/扩散模型）沉淀了大量 attention 分析与扩散理论，其中一部分已在时序预测中被使用，一部分仍是空白，一部分搬不动。

## 1. 已搬的（成功先例）

### 1.1 Attention 低秩性 → vecTrans / NormLin
- LLM 领域对 attention 矩阵低秩结构的分析已成体系（最新如 Lorsa，arXiv:2504.20938，把 attention 头稀疏分解为可解释的原子操作）。
- 北大应翔华组直接拿"低秩"结论：在时序里把 attention 换成 rank-1 可学习矩阵（vLinear 的 vecTrans）/ 正数行归一线性层（OLinear 的 NormLin）。
- **迁移模式**：结论级迁移——不需要重证低秩，只需要验证"时序的 attention 也低秩"（该组已做）。

### 1.2 扩散/FM 理论 → TSFlow 等时序生成模型
- Rectified flow、OT 路径、probability flow ODE 全套从图像/大模型领域搬来。
- rectified flow 的收敛与"直度"理论（arXiv:2410.14949，*On the Convergence and Straightness of Rectified Flow*）：给出 Wasserstein 收敛分析，并讨论 rectification 次数与路径直度的关系，可作为少步/单步推理方法的理论参考。时序论文中是否已有工作系统引用，尚未系统检索。

## 2. 可搬的（空白机会，按含金量排序）

### 2.1 时序 Transformer 的 attention 可解释性分析
- LLM 侧弹药：Attention Sink survey（arXiv:2604.10098）——attention 会塌缩到无意义 token（BOS、背景 patch），已有"利用/解释/缓解"全套路数；Transformer Circuits——把 attention 拆成 QK（决定看哪）/ OV（搬什么）两个低秩电路。
- 时序侧：**尚未检索到对 PatchTST / iTransformer attention 形态的系统分析**（未做系统检索，需先查重）。
  - 有没有 sink？（attention 塌缩到某些 patch/变量？）
  - 有没有"趋势头""周期头""突变头"？（QK/OV 分解视角）
- 产出形态：分析论文；若发现可缓解的病理 → 新方法。

### 2.2 Manifold hypothesis → 能否解释 DLinear 现象（待验证）
- **先纠正一个误读**：arXiv:2410.09046（*Linear Convergence of Diffusion Models Under the Manifold Hypothesis*）证明的是：扩散模型在 KL 意义下收敛所需的**采样步数**与内在维度 d 成线性关系。它讲的是生成要走多少步，不是模型要多大，也不是预测能多准，因此**不能直接解释** DLinear 现象。
- 更贴切的理论工具：
  - 流形上的非参数回归：误差收敛率依赖内在维度 d 而非环境维度，约为 n^(-2s/(2s+d))；
  - 流形上的神经网络逼近：所需参数量随 d 增长（约束的是参数总量，不是某个宽度）；
  - Takens 延迟嵌入：确定性动力系统的滑动窗口落在低维流形上，是把流形假设用到时序窗口的最直接依据。
- "时序数据集中在低维结构附近"是需要测量的假设，不是前提：真实 benchmark 有噪声且不平稳，内在维度估计依赖尺度。应先做多尺度 ID 估计、训练/测试段对比和噪声下限估计，再讨论它和模型容量的关系。
- 与 OLinear 的 OrthoTrans（数据自适应低维基）直觉相近，但后者是工程方法，不构成对流形假设的验证。

### 2.3 扩散收敛理论 → 时序 FM 的误差界
- DDPM 收敛理论（arXiv:2406.01320）、score-SDE 框架下的各类保证——时序 flow matching 论文目前只管"能用"，没人写"为什么收敛、误差界是多少"。
- 适合作为单步/概率预测方法的理论章节。

## 3. 搬不动的（诚实边界）

1. **尺度不对**：LLM 分析针对 4k–128k 超长序列、百亿参数；时序通常 96–720 长度、小模型。Attention sink 在小模型短序列上**可能根本不存在**——先验证再搬，不硬套。
2. **数据性质不对**：LLM token 离散有语义，时序 patch/变量连续。很多可解释性结论依赖离散语义，迁移会水土不服。
3. **理论假设不对**：扩散收敛理论一般假设数据分布满足正则条件；时序的非平稳、重尾可能直接违反假设，搬之前先检查假设。

## 4. 行动清单

- [ ] 跑一个 PatchTST/iTransformer 的 attention 可视化，看有没有 sink / 结构化模式（2.1 的第一步，成本低）
- [ ] 精读 arXiv:2410.14949（rectified flow straightness），评估其假设是否适用于时序（1.2/2.3）
- [ ] 在 ETTh1/electricity 上做多尺度内在维度估计与噪声下限估计，检验 2.2 的前提是否成立
