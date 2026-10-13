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
  "era": "era3",
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
| `era` | ✓ | 所属主线，4 选 1：`foundation`（奠基与生成式源头）`era1`（主线一·输入处理）`era2`（主线二·架构）`era3`（主线三·目标与推理）。主线是并行主题，年份可重叠；定义在 `papers.json` 的 `meta.eras` 里 |
| `new_cognition` | ✓ | **核心**：这篇论文让领域相信了什么新观点 |
| `corrects` | ✓ | **核心**：它修正或推进了之前哪篇的什么观点。被修正的观点必须能在原论文中找到出处；找不到就写成“此前主流做法是……”，不要构造无人主张过的稻草人 |
| `corrected_by` | | 后续哪篇修正了它（形成思想链） |
| `problem` / `method` / `insight` | | P/M/I 三元组 |
| `links` | | `arxiv` / `doi` / `github` 链接 |
| `tags` | | 标签，如 `under-review` 表示投稿中 |

## 维护习惯

读完一篇论文，顺手花 10 分钟加一条。重点写好 `new_cognition` 和 `corrects`——
这是这个仓库区别于普通论文列表的全部价值。`venue` 如实标注预印本 / 投稿中；单篇结论不要写进阶段的"共识"描述。

## 思想链

`index.html` 里的 `CHAINS` 数组定义了几条主线的修正链（后一篇回答前一篇的问题），
有新主线时直接加。主线色带由 `papers.json` 的 `meta.eras` 驱动，加新主线时同步加。

## 课题组深挖

`docs/` 目录下是课题组思想线分析（如 `docs/pku-ying-group.md`），
记录一个组连续几篇论文背后的统一思路、方法论签名和跟踪清单。

## 论文写法拆解

`docs/paper-writing-analysis.md` 拆解论文"怎么讲"：问题怎么命名、借来的知识是否点名来源、对手怎么设定、证据怎么组织、局限怎么写。每加一篇按其中的 7 项填写。
