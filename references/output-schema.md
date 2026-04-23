# 输出文件格式规范（Output Schema）

---

## detailed_results.json

每条评论处理后的完整输出结构：

```json
[
  {
    "id": "评论唯一标识符",
    "content": "用户原文",
    "platform": "小红书 / 微博 / 知乎 / 豆瓣",
    "publish_time": "YYYY-MM-DD HH:MM:SS",
    "user_mind_profile": {
      "user_type": "first_time_user | regular_user | potential_user | churned_user | unknown",
      "core_mindset": "trusting | hesitant | frustrated | satisfied | indifferent | confused | unknown",
      "emotion_intensity": 0,
      "primary_motivation": "convenience | cost_saving | privacy | speed | expertise | information | unknown",
      "primary_barrier": "trust_concern | price_concern | quality_concern | privacy_concern | ux_friction | response_time | competitor_pull | none | unknown",
      "decision_stage": "awareness | consideration | usage | evaluation | churn",
      "trust_level": 0,
      "key_tags": [],
      "filtered_out": false,
      "filter_reason": "若 filtered_out=true，说明过滤原因；否则为空字符串"
    },
    "analysis_confidence": "high | medium | low",
    "analyst_note": "分析时的备注（可选）"
  }
]
```

---

## insights_report.json

汇总统计结果：

```json
{
  "task_id": "MIND-YYYYMMDD-001",
  "analysis_date": "YYYY-MM-DD",
  "data_range": {
    "start": "YYYY-MM-DD",
    "end": "YYYY-MM-DD"
  },
  "sample_stats": {
    "total_raw": 0,
    "valid_count": 0,
    "filtered_count": 0,
    "filter_rate": 0.0
  },
  "platform_distribution": {
    "小红书": 0,
    "知乎": 0,
    "微博": 0,
    "豆瓣": 0
  },
  "mindset_distribution": {
    "trusting": 0,
    "hesitant": 0,
    "frustrated": 0,
    "satisfied": 0,
    "indifferent": 0,
    "confused": 0
  },
  "decision_stage_funnel": {
    "awareness": 0,
    "consideration": 0,
    "usage": 0,
    "evaluation": 0,
    "churn": 0
  },
  "emotion_intensity_stats": {
    "mean": 0.0,
    "median": 0.0,
    "high_risk_count": 0
  },
  "top_barriers": [],
  "top_motivations": [],
  "top_key_tags": [],
  "key_insights": [
    {
      "insight_id": "IN-001",
      "title": "",
      "supporting_samples": 0,
      "trend": "新趋势 | 持续存在",
      "recommendation": {
        "short_term": "",
        "mid_term": "",
        "long_term": ""
      }
    }
  ]
}
```

---

## 文件命名规范

| 文件 | 路径 | 说明 |
|------|------|------|
| 原始数据 | `.ark/tmp/manual_input/MIND-[任务编号]_raw_data.json` | 采集后未清洗 |
| 清洗数据 | `.ark/tmp/manual_input/MIND-[任务编号]_cleaned.json` | 质检通过后 |
| 详细结果 | `.ark/output/mind_monitor_[时间戳]/detailed_results.json` | 9维度提取结果 |
| 洞察报告JSON | `.ark/output/mind_monitor_[时间戳]/insights_report.json` | 汇总统计 |
| 最终报告 | `.ark/output/mind_monitor_[时间戳]/final_report.md` | 可交付文档 |
| 图表目录 | `.ark/output/mind_monitor_[时间戳]/charts/` | 可视化描述文件 |
