# 多变量时间序列预测 · 认知演进史

一个领域的认知是由一篇篇论文组成的：每篇提出新观点、修正旧观点、完善 pipeline。
这个仓库记录每一次"认知修正"——一部用论文写成的思想史。

🌐 网页：https://dys-347.github.io/Time-series/ （GitHub Pages 自动部署）

## 怎么加一篇论文

所有数据在 `papers.json` 里。加一篇 = 在 `papers` 数组里加一个对象：

```json
{
  "id": "paper-id-2026",
  "title": "PaperTitle",
  "year": 2026,
  "venue": "ICLR",
  "authors": "Author et al.",
  "stage": "loss",
  "tags": ["flow-matching", "one-step"],
  "new_cognition": "它让领域相信了什么（一句话）",
  "corrects": "它修正了谁的什么旧观点（一句话）",
  "corrected_by": ["later-paper-id"],
  "problem": "解决什么问题",
  "method": "核心方法",
  "insight": "关键洞见",
  "links": { "arxiv": "https://arxiv.org/abs/xxxx.xxxxx" }
}
```

字段说明：

| 字段 | 必填 | 说明 |
|---|---|---|
| `id` | ✓ | 唯一标识，格式 `短名-年份` |
| `title` / `year` / `venue` / `authors` | ✓ | 基本信息 |
| `stage` | ✓ | 所属 pipeline 阶段，8 选 1：`problem`（问题定义）`input`（输入处理）`decomposition`（分解）`backbone`（建模主干）`variables`（变量间关系）`loss`（损失函数）`inference`（推理采样）`evaluation`（评估） |
| `new_cognition` | ✓ | **核心**：这篇论文让领域相信了什么新观点 |
| `corrects` | ✓ | **核心**：它修正/推翻了之前哪篇的什么旧观点 |
| `corrected_by` | | 后续哪篇修正了它（形成思想链） |
| `problem` / `method` / `insight` | | P/M/I 三元组 |
| `links` | | `arxiv` / `doi` / `github` 链接 |
| `tags` | | 标签，如 `author-work` 会显示"作者工作"徽标 |

## 维护习惯

读完一篇论文，顺手花 10 分钟加一条。重点写好 `new_cognition` 和 `corrects`——
这是这个仓库区别于普通论文列表的全部价值。

## 思想链与范式转移

`index.html` 里的 `CHAINS` 数组定义了几条主线的修正链（后一篇回答前一篇的问题），
`ERAS` 数组定义了"三次范式转移"（输入处理时代 → 架构时代 → 目标与推理时代），
有新主线时直接加。
