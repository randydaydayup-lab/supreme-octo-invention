---
name: mind-analyst
description: >
  百度健康C端用户心智分析专项技能（文档编号：SOP-BDHC-MIND-001 衍生版）。
  当用户提及"用户心智"、"心智分析"、"心智画像"、"用户调研"、"用户心理"、
  "心智提取"、"用户洞察报告"、"VOC分析"，或提到处理 .ark/tmp/manual_input/
  目录下的评论数据时，必须立即调用本技能。即便用户只说"帮我分析这批用户评论"
  但上下文涉及百度健康产品或医疗健康平台，也应触发本技能。
  禁止调用任何外部搜索或抓取 API，所有分析基于本地文件。
compatibility:
  tools:
    - bash
    - view
    - create_file
    - str_replace
---

# 百度健康用户心智分析 Skill

> **角色**：具备医疗产品背景的**资深用户研究员**，擅长从碎片化用户评论中提炼心智规律。  
> **使命**：将用户的情感表达转化为可量化的心智画像，驱动产品决策。  
> **核心原则**：证据先行 · 9维度结构化 · 禁止外部 API · 全流程本地执行

---

## Phase 0 · 环境初始化

执行以下检查，将结果存入工作上下文后进入 Phase 1：

```bash
# 读取关键词配置
cat config/mind-monitor-config.yaml 2>/dev/null || echo "[WARNING] 关键词配置文件不存在"

# 读取历史心智看板（纵向对比用）
cat MIND_INSIGHT_BOARD.md 2>/dev/null || echo "[INFO] 首次运行，无历史记录"

# 读取状态记录（避免重复分析）
cat MIND_MEMORY.md 2>/dev/null || echo "[INFO] 无状态记录，将创建新记录"

# 扫描待分析数据
ls .ark/tmp/manual_input/*_cleaned.json 2>/dev/null || \
ls .ark/tmp/manual_input/*_raw_data.json 2>/dev/null || \
echo "[WARNING] 未找到待分析数据文件，请先完成数据采集（阶段3）"

# 确认输出目录
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
OUTPUT_DIR=".ark/output/mind_monitor_${TIMESTAMP}"
mkdir -p "${OUTPUT_DIR}/charts"
echo "[INFO] 输出目录：${OUTPUT_DIR}"
```

---

## Phase 1 · 数据质检（对应 SOP 阶段4）

**前提**：输入文件为 `MIND-[任务编号]_raw_data.json` 或 `*_cleaned.json`

### 1.1 质检清单

| 检查项 | 标准 | 不合格处理 |
|--------|------|-----------|
| 样本量 | ≥ 50 条（常规监测） | 提示补充采集 |
| 时效性 | 90% 评论在最近 30 天内 | 提示替换过期样本 |
| 平台分布 | ≥ 3 个平台，单平台占比 < 60% | 提示补充弱势平台 |
| 内容质量 | 无乱码、无截断、无营销内容 | 自动过滤 |
| 格式规范 | 符合 JSON schema | 报告格式错误 |

### 1.2 自动过滤规则（`filtered_out=true`）

以下条目在分析时自动跳过：
1. 内容长度 < 10 字
2. 包含营销关键词（"扫码领券"、"限时优惠"、"点击购买"）
3. 无明确情感表达（纯事实陈述）
4. 语义不清或乱码

### 1.3 人工复核触发条件

- 某批次样本过滤率 > 30% → 提示返回数据采集阶段
- 情感强度全部为极端值（0 或 10）→ 提示检查关键词偏向性
- 心智分布严重失衡（某状态 > 80%）→ 提示补充对立样本

---

## Phase 2 · 9维度心智提取（对应 SOP 阶段5）

对每条有效评论，提取以下 9 个维度，输出结构化 `user_mind_profile`：

```json
{
  "user_mind_profile": {
    "user_type": "",
    "core_mindset": "",
    "emotion_intensity": 0,
    "primary_motivation": "",
    "primary_barrier": "",
    "decision_stage": "",
    "trust_level": "",
    "key_tags": [],
    "filtered_out": false
  }
}
```

### 2.1 维度提取标准

详细判断规则参见 → [`references/mindset-dimensions.md`](references/mindset-dimensions.md)

**快速参考：**

**user_type（用户类型）**

| 类型 | 判断依据 | 典型表达 |
|------|----------|----------|
| `first_time_user` | 明确提到"第一次"、"首次" | "第一次用线上问诊" |
| `regular_user` | 描述使用经历，未提及离开 | "用了三次了" |
| `potential_user` | 处于观望期，尚未使用 | "考虑要不要用" |
| `churned_user` | 明确表示不再使用或已转投竞品 | "现在改用 XX 了" |

**core_mindset（核心心智）**

| 心智状态 | 情感强度范围 | 典型表达 |
|----------|--------------|----------|
| `trusting` | 6-8 | "挺靠谱的"、"相信平台" |
| `hesitant` | 3-5 | "不知道靠不靠谱"、"有点担心" |
| `frustrated` | 7-10 | "气死了"、"太失望了" |
| `satisfied` | 6-9 | "很满意"、"超出预期" |
| `indifferent` | 2-4 | "无所谓"、"差不多" |
| `confused` | 3-6 | "不太懂"、"搞不清楚" |

**emotion_intensity（情感强度，0-10）**

| 分值 | 描述 | 典型表达 |
|------|------|----------|
| 0-2 | 几乎无情感 | 纯事实陈述 |
| 3-4 | 轻微情感 | "还行"、"一般般" |
| 5-6 | 中等情感 | "挺好的"、"有点失望" |
| 7-8 | 强烈情感 | "非常满意"、"很生气" |
| 9-10 | 极端情感 | "爱死了"、"被气死" |

**decision_stage（决策阶段）**

`awareness` → `consideration` → `usage` → `evaluation` → `churn`

### 2.2 质量控制

- 有效样本率目标：≥ 70%（`filtered_out=false` 的比例）
- 所有 9 个维度均需填充（不可为空，无法判断时填 `unknown`）
- 情感强度分布需合理，避免全为极端值

---

## Phase 3 · 心智洞察提炼（对应 SOP 阶段6）

### 3.1 核心分析维度

基于提取结果，执行以下分析：

**① 高风险用户识别**
- `core_mindset == 'frustrated'` 且 `emotion_intensity >= 7` 的用户群体
- `user_type == 'churned_user'` 的流失原因聚类

**② 高频障碍因素 Top 3**
- 统计 `primary_barrier` 字段出现频次
- 对出现频次 ≥ 5 的障碍因素重点分析

**③ 决策阶段流失分析**
- 统计每个 `decision_stage` 的分布比例
- 识别流失集中发生的阶段

**④ 信任度分析**
- 按 `trust_level` 分组，计算均值和分布
- 交叉分析：`platform` × `trust_level` 的关系

### 3.2 洞察质量标准（SADC 原则）

好的洞察必须满足：

| 原则 | 要求 | 反例 | 正例 |
|------|------|------|------|
| **S**urprising | 揭示非显而易见的发现 | "用户对响应速度不满" | "响应超 5 分钟的问诊，流失率提升 40%（n=23）" |
| **A**ctionable | 能转化为具体改进项 | "提升用户满意度" | "优先优化高峰期医生调度，预期减少等待投诉 35%" |
| **D**ata-Driven | 有明确数字支撑 | "很多用户担心隐私" | "25-30 岁女性中 68% 因隐私保护选择线上问诊" |
| **C**entered | 关注用户真实需求 | "平台需要更多功能" | "用户在报告解读环节产生最高焦虑（均分 7.2/10）" |

每个洞察的样本量要求：**≥ 10 条支撑评论**。

### 3.3 纵向对比

读取 `MIND_INSIGHT_BOARD.md`，对每个新发现：
- **若与历史匹配**：标注 `[持续存在]`，记录首次发现日期，紧迫度升级
- **若为新发现**：标注 `[新趋势]`，记录发现日期

---

## Phase 4 · 生成产出物（对应 SOP 阶段6 输出）

### 4.1 产出物 A：结构化分析结果

**路径**：`${OUTPUT_DIR}/detailed_results.json`

每条评论输出格式参见 → [`references/output-schema.md`](references/output-schema.md)

### 4.2 产出物 B：洞察报告

**路径**：`${OUTPUT_DIR}/final_report.md`

使用模板 → [`assets/templates/mind-analysis-report.md`](assets/templates/mind-analysis-report.md)

报告必需章节：

```
# 百度健康用户心智分析报告 · YYYY-MM-DD

## 执行摘要（Executive Summary）
- 分析周期、样本量、核心发现 Top3

## 用户心智全景（Mindset Landscape）
- 6种心智状态分布（含占比数字）
- 决策阶段漏斗（各阶段样本量）
- 情感强度分布（均值、中位数、极端案例数）

## 关键洞察（Key Insights）
- 每条洞察：现象描述 + 典型评论引用 + 深层原因 + 产品建议

## 用户心声（Voice of Customer）
- 高频心智标签 Top 10
- 最满意 / 最失望用户原声各 3 条（脱敏）

## 行动建议（Recommendations）
- 短期优化（2周内）
- 中期迭代（1个月内）
- 长期战略（季度级）
```

### 4.3 产出物 C：数据可视化描述

**路径**：`${OUTPUT_DIR}/charts/`

必需图表（文字描述版，供人工制图或工具生成）：
1. **心智分布饼图**：6种状态 + 各自占比
2. **决策阶段漏斗图**：5个阶段样本量递减展示
3. **情感强度箱线图**：按平台/用户类型分组

可选图表：词云图、时间序列趋势图、情感强度×障碍因素散点图

### 4.4 更新记忆文件

```bash
# 追加到 MIND_INSIGHT_BOARD.md（新洞察）
# 更新 MIND_MEMORY.md（记录本次分析状态）
```

格式规范参见 → [`references/memory-schema.md`](references/memory-schema.md)

---

## Phase 5 · 终端摘要输出

分析完成后输出：

```
╔══════════════════════════════════════════════════╗
║         用户心智分析完成 · 摘要                   ║
╠══════════════════════════════════════════════════╣
║  任务编号：MIND-YYYYMMDD-XXX                     ║
║  有效样本：N 条 / 原始 M 条（过滤率：X%）          ║
║  数据时间范围：YYYY-MM-DD ~ YYYY-MM-DD            ║
║  平台分布：小红书 X% / 知乎 X% / 微博 X% / 其他   ║
╠══════════════════════════════════════════════════╣
║  心智分布 TOP3：                                  ║
║    1. [心智状态] X%                              ║
║    2. [心智状态] X%                              ║
║    3. [心智状态] X%                              ║
╠══════════════════════════════════════════════════╣
║  ⚠️  高风险（frustrated, intensity≥7）：N 条      ║
║  🔄 流失用户（churned_user）：N 条                ║
║  📌 新趋势洞察：N 个  ♻️  持续存在：N 个           ║
╠══════════════════════════════════════════════════╣
║  报告路径：.ark/output/mind_monitor_[时间戳]/     ║
╚══════════════════════════════════════════════════╝
```

---

## 异常处理快速指南

| 异常现象 | 原因 | 处理方式 |
|---------|------|---------|
| 过滤率 > 50% | 数据质量差或关键词不相关 | 返回数据采集阶段，优化关键词 |
| 心智分布严重失衡（某状态 > 80%） | 样本来源单一或关键词有偏向 | 补充对立样本，重新分析 |
| 某维度大量 `unknown` | 评论信息量不足 | 降低该批次可信度，标注说明 |
| JSON 格式错误 | 采集或填写时格式不规范 | 运行格式检查并手动修正 |

> ⚠️ **全程禁止调用任何外部搜索或抓取 API。** 所有分析基于本地文件。

---

## 执行 SOP 总览

```
Phase 0  环境初始化（读取配置、历史记忆、扫描数据）
   ↓
Phase 1  数据质检（5项检查 + 自动过滤规则）
   ↓
Phase 2  9维度心智提取（user_type / core_mindset / emotion_intensity 等）
   ↓
Phase 3  洞察提炼（高风险识别 / 障碍聚类 / 纵向对比）
   ↓
Phase 4  生成产出物（detailed_results.json + final_report.md + charts/）
         更新记忆（MIND_INSIGHT_BOARD.md + MIND_MEMORY.md）
   ↓
Phase 5  终端摘要输出
```
