# 大模型分析工具向时序预测的迁移

> 2026-10-13 记录。核心判断：**分析工具可以搬，分析结论要重验。**
> 大模型领域（LLM/扩散模型）沉淀了大量 attention 分析与扩散理论，其中一部分已在时序预测中被使用，一部分仍是空白，一部分搬不动。

## 1. 已搬的（成功先例）

### 1.1 Attention 低秩性 → vecTrans / NormLin
- LLM 领域对 attention 矩阵低秩结构的分析已成体系（最新如 Lorsa，arXiv:2504.20938，把 attention 头稀疏分解为可解释的原子操作）。
- 北大应翔华组直接拿"低秩"结论：在时序里把 attention 换成 rank-1 可学习矩阵（vLinear 的 vecTrans）/ 正数行归一线性层（OLinear 的 NormLin）。
- **迁移模式**：结论级迁移——不需要重证低秩，只需要验证"时序的 attention 也低秩"（该组已做）。

### 1.2 扩散/FM 理论 → TSFlow / FlowCast
- Rectified flow、OT 路径、probability flow ODE 全套从图像/大模型领域搬来。
- ICLR 2025 的 rectified flow straightness 理论（arXiv:2410.14949）：Wasserstein 收敛 + "两次 rectification 就够直"——**可直接拿来给单步/少步推理方法做理论背书**，目前时序论文里还没人认真写这一笔。

## 2. 可搬的（空白机会，按含金量排序）

### 2.1 时序 Transformer 的 attention 可解释性分析
- LLM 侧弹药：Attention Sink survey（arXiv:2604.10098）——attention 会塌缩到无意义 token（BOS、背景 patch），已有"利用/解释/缓解"全套路数；Transformer Circuits——把 attention 拆成 QK（决定看哪）/ OV（搬什么）两个低秩电路。
- 时序侧空白：**没人系统看过 PatchTST / iTransformer 的 attention 长什么样**。
  - 有没有 sink？（attention 塌缩到某些 patch/变量？）
  - 有没有"趋势头""周期头""突变头"？（QK/OV 分解视角）
- 产出形态：分析论文；若发现可缓解的病理 → 新方法。

### 2.2 Manifold hypothesis → 解释 DLinear 现象
- 扩散理论（arXiv:2410.09046）：收敛速度只跟数据的**内在维度**线性相关，而非环境维度。
- 时序数据天然集中在低维流形上 → 这给"DLinear 现象"（小模型/线性模型够用）一个理论注脚：**复杂度≠效果，因为内在维度本来就小**。
- 与 OLinear 的 OrthoTrans（数据自适应低维基）是同一个直觉的不同表述。

### 2.3 扩散收敛理论 → 时序 FM 的误差界
- DDPM 收敛理论（arXiv:2406.01320）、score-SDE 框架下的各类保证——时序 flow matching 论文目前只管"能用"，没人写"为什么收敛、误差界是多少"。
- 适合作为单步/概率预测方法的理论章节。

## 3. 搬不动的（诚实边界）

1. **尺度不对**：LLM 分析针对 4k–128k 超长序列、百亿参数；时序通常 96–720 长度、小模型。Attention sink 在小模型短序列上**可能根本不存在**——先验证再搬，不硬套。
2. **数据性质不对**：LLM token 离散有语义，时序 patch/变量连续。很多可解释性结论依赖离散语义，迁移会水土不服。
3. **理论假设不对**：扩散收敛理论一般假设数据分布满足正则条件；时序的非平稳、重尾可能直接违反假设，搬之前先检查假设。

## 4. 行动清单

- [ ] 跑一个 PatchTST/iTransformer 的 attention 可视化，看有没有 sink / 结构化模式（2.1 的第一步，成本低）
- [ ] 精读 arXiv:2410.14949（rectified flow straightness），评估能否直接写进单步方法的理论部分（1.2/2.3）
- [ ] 精读 arXiv:2410.09046（manifold 收敛），看能否形式化 DLinear 现象的解释（2.2）
