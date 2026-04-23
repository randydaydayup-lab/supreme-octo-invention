# 记忆文件格式规范（Memory Schema）

---

## MIND_INSIGHT_BOARD.md 结构

```markdown
# 用户心智洞察看板（Mind Insight Board）

> 由 `mind-analyst` Skill 自动维护。请勿手动修改编号。  
> 对应文档：SOP-BDHC-MIND-001

## 活跃洞察（未解决）

### [MI-001] [新趋势/持续存在] 洞察标题
- **首次发现**：YYYY-MM-DD
- **最近确认**：YYYY-MM-DD
- **任务编号**：MIND-YYYYMMDD-XXX
- **心智维度**：core_mindset / primary_barrier / decision_stage 等
- **支撑样本量**：N 条
- **核心证据**：
  > "[用户原话节选（脱敏）]" —— 来源：平台名，发布时间
- **产品建议**：[可落地的改进方向]
- **状态**：🔴 未处理 / 🟡 处理中 / 🟢 已解决

---

## 历史归档（已解决）

[已解决的洞察移至此处，保留记录]
```

---

## MIND_MEMORY.md 结构

```markdown
# 心智分析状态记录（Mind Analysis Memory）

> 由 `mind-analyst` Skill 自动维护，防止重复分析。

## 已分析任务记录

| 分析日期 | 任务编号 | 数据文件 | 数据时间范围 | 有效样本数 | 过滤率 | 报告路径 |
|---------|---------|---------|------------|----------|-------|---------|
| YYYY-MM-DD | MIND-XXX | xxx_cleaned.json | 2026-01-01~2026-01-31 | 63 | 18% | .ark/output/... |

## 洞察编号计数器

当前最大编号：MI-XXX（下次从 MI-XXX+1 开始）

## 最后更新

YYYY-MM-DD HH:MM
```

---

## 操作规范

### 新增洞察
1. 从 `MIND_MEMORY.md` 获取当前最大编号，+1 得到新编号
2. 在 `MIND_INSIGHT_BOARD.md` 的"活跃洞察"区块顶部插入
3. 更新 `MIND_MEMORY.md` 计数器

### 标记持续存在
1. 找到匹配条目，将标签改为 `[持续存在]`
2. 更新"最近确认"日期
3. 更新支撑样本量（累加）

### 标记已解决
1. 移至"历史归档"，记录解决日期和解决方案
2. 状态改为 `🟢 已解决`

### 每次分析后
在 `MIND_MEMORY.md` 的任务记录表追加一行。
