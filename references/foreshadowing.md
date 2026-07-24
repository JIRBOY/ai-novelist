# 伏笔追踪系统

## 三级伏笔管理

### 战略级（Tier-1）

- **定义**：影响主角命运/世界结局的伏笔
- **数量**：全书 3-5 个
- **生命周期**：跨越全书，在关键卷回收
- **示例**：主角的神秘身世、世界的终极秘密、命运的预言

### 战役级（Tier-2）

- **定义**：影响当前卷/派系的伏笔
- **数量**：每卷 5-8 个
- **生命周期**：跨卷或在当前卷回收
- **示例**：某势力的暗中计划、角色的隐藏动机、重要物品的来历

### 战术级（Tier-3）

- **定义**：影响局部冲突的伏笔
- **数量**：按需埋设
- **生命周期**：3-10 章内回收
- **示例**：角色的某项技能、场地的某个特征、对话中的暗示

## 伏笔台账格式

```json
{
  "foreshadowings": [
    {
      "id": "FS-001",
      "tier": "strategic",
      "description": "主角手上的神秘疤痕",
      "planted_chapter": 1,
      "planted_context": "主角在镜中看到疤痕，感到莫名不安",
      "expected_recall_chapter": 45,
      "actual_recall_chapter": null,
      "status": "planted",
      "priority": "high",
      "related_characters": ["char_001"],
      "related_events": [],
      "recall_plan": "在第45章揭示疤痕与主角身世的关系",
      "notes": "前20章每5章暗示一次，后逐渐加重"
    },
    {
      "id": "FS-002",
      "tier": "campaign",
      "description": "反派对主角的暗中观察",
      "planted_chapter": 5,
      "planted_context": "主角感觉有人在暗中注视",
      "expected_recall_chapter": 18,
      "actual_recall_chapter": 18,
      "status": "recalled",
      "priority": "medium",
      "related_characters": ["char_001", "char_002"],
      "related_events": ["evt_007"],
      "recall_plan": "第18章反派正式登场，揭示一直在观察主角",
      "notes": "回收成功"
    }
  ]
}
```

## 伏笔状态

| 状态 | 说明 |
|------|------|
| `planted` | 已埋设，待回收 |
| `hinted` | 已暗示（中期提醒读者伏笔存在） |
| `recalled` | 已回收 |
| `overdue` | 超期未回收（current_chapter > expected_recall_chapter + 5） |
| `abandoned` | 已放弃（改纲后不再需要） |

## 伏笔操作流程

### 埋设伏笔

1. 写作时发现需要埋设伏笔
2. 在 `foreshadowing_ledger.json` 中添加记录
3. 在章节控制卡中标记"新埋伏笔"
4. 设定预期回收章节和回收计划

### 回收伏笔

1. 写前检查台账中本章预期回收的伏笔
2. 在章节中自然回收（不生硬）
3. 更新台账：`actual_recall_chapter`、`status: "recalled"`

### 伏笔提醒

- 超期预警：`status: "overdue"` 的伏笔在写前高亮提醒
- 即将到期：`expected_recall_chapter - current_chapter <= 3` 的伏笔在写前提醒
- 战略级检查：每 10 章检查所有战略级伏笔状态

### 伏笔暗示

对于长周期伏笔（战略级），在埋设与回收之间需要定期暗示：
- 每 5-10 章以不同方式提醒读者伏笔存在
- 暗示方式：角色偶尔想起、场景中的线索、其他角色的提及
- 暗示记录在台账的 `notes` 字段

## 伏笔回收质量检查

门禁检查时验证伏笔回收质量：

| 检查项 | 标准 |
|--------|------|
| 自然度 | 回收不生硬，与前文逻辑一致 |
| 完整度 | 回收充分，无遗漏关键信息 |
| 时机 | 回收时机合理，不过早不过晚 |
| 影响 | 回收对剧情有实质影响 |

## 集中收束风险检测

当多根伏笔集中在同一章节回收时，检测收束风险：
- 单章回收 ≥3 根伏笔 → 警告
- 单章回收 ≥5 根伏笔 → 门禁失败
- 建议分散回收，避免信息过载
