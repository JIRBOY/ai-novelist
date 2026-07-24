# Phase 2：规划与确认

## 规划产出

确认创作种子后，AI 自动生成以下规划文档，展示给用户确认。

### 1. 总大纲

Phase 2 同时生成两个大纲文件，分工不同：

- **`02_outline/master-outline.md`**：完整大纲文档（人类可读），包含故事概述、卷划分、7 列章节规划表
- **`00_memory/novel_plan.md`**：Agent 工作大纲索引（精简格式），包含章节列表、锚点进度，写前必读

采用 7 列章节规划模板：

```markdown
# 《{title}》总大纲

## 故事概述
{一段话概括全书核心故事}

## 章节规划

| 章号 | 标题 | 核心事件 | 情绪走向 | 冲突类型 | 伏笔操作 | 悬念钩子 |
|------|------|----------|----------|----------|----------|----------|
| 1 | {标题} | {本章发生什么} | {读者情绪} | {冲突类型} | {埋设/回收} | {章末悬念} |
| 2 | ... | ... | ... | ... | ... | ... |
| ... | ... | ... | ... | ... | ... | ... |

## 卷划分（长篇）
- 第一卷：{卷名}（第1-20章）— {本卷核心弧光}
- 第二卷：{卷名}（第21-40章）— {本卷核心弧光}
```

**情绪走向**值域：压抑 → 提升 → 打脸 → 装逼 / 紧张 → 逼近 → 震惊 → 舒缓

**冲突类型**值域：生死存亡 / 查明真相 / 复仇雪恨 / 成长突破 / 爱情阻碍 / 权力争夺 / 守护保护

### 2. 人物档案（characters/）

每个角色一个文件，包含完整档案：

```markdown
# 角色档案：{角色名}

## 基本信息
- 角色：主角 / 反派 / 配角 / 导师
- 年龄：{age}
- 外貌：{简要外貌描述}
- 身份：{社会身份}

## 性格 DNA
- 核心特质：{2-3个核心性格词}
- 说话方式：{语言风格特征}
- 行为模式：{典型行为倾向}
- 弱点/缺陷：{性格弱点}

## 能力体系
- 核心能力：{主要能力}
- 成长潜力：{未来发展方向}
- 限制条件：{能力使用代价或限制}

## 成长弧光
- 起点：{故事开始时的状态}
- 转折：{关键转变节点}
- 终点：{故事结束时的状态}

## 关系网络
- 与{角色B}：{关系类型} — {关系描述}
- 与{角色C}：{关系类型} — {关系描述}

## 情绪弧线
- 第1-5章：{情绪状态}
- 第6-15章：{情绪状态}
- ...
```

### 3. 世界观设定（world-setting.md）

```markdown
# 世界观设定

## 世界基础
- 时代背景：{时代}
- 地理环境：{主要地点}
- 社会结构：{社会形态}

## 特殊规则
- 力量体系：{如有，详细描述}
- 核心设定：{区别于现实世界的关键设定}
- 规则限制：{设定的边界与代价}

## 世界观法则（不可违反）
1. {法则1}
2. {法则2}
3. ...
```

### 4. 伏笔台账（foreshadowing_ledger.json）

```json
{
  "foreshadowings": [
    {
      "id": "FS-001",
      "description": "主角手上的神秘疤痕",
      "planted_chapter": 1,
      "expected_recall_chapter": 15,
      "actual_recall_chapter": null,
      "tier": "strategic",
      "status": "planted",
      "related_characters": ["主角"],
      "notes": "与主角身世之谜相关"
    }
  ]
}
```

**伏笔层级**：
- `strategic`（战略级）：影响主角命运/世界结局，全书 3-5 个
- `campaign`（战役级）：影响当前卷/派系，每卷 5-8 个
- `tactical`（战术级）：影响局部冲突，按需埋设

### 5. 写作计划 JSON

存放在 `00_memory/writing-plan.json`，Agent 与子 Agent 共享读取：

```json
{
  "title": "{title}",
  "genre": "{genre}",
  "total_chapters": 20,
  "words_per_chapter": { "min": 3000, "max": 5000 },
  "target_total_words": 80000,
  "volumes": [
    {
      "name": "第一卷",
      "chapters": [1, 20],
      "core_arc": "{本卷核心弧光}"
    }
  ],
  "pov": "{pov}",
  "tone": "{tone}",
  "automation_level": "manual",
  "created_at": "{timestamp}",
  "status": "planning"
}
```

### 6. 大纲锚点（outline_anchors.json）

```json
{
  "total_chapters": 20,
  "current_chapter": 0,
  "current_arc": "第一卷",
  "chapters_in_arc": 20,
  "quota": {
    "min_progress": 0.00,
    "max_progress": 0.05
  },
  "arc_anchors": [
    { "arc": "第一卷", "start": 1, "end": 20, "core_conflict_resolution": "第20章" }
  ]
}
```

## 确认流程

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
规划完成！请确认：

基本信息
  题材：{genre}  |  主角：{protagonist}  |  冲突：{conflict}
  章节数：{n}章  |  视角：{pov}  |  基调：{tone}

章节规划（前5章）
  第1章：{title} - {summary}
  第2章：{title} - {summary}
  ...

主要角色
  主角：{name} - {brief}
  反派：{name} - {brief}
  配角：{name} - {brief}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
回复"确认" → 选择写作模式 → 立即开始
回复"修改" → 说明需要调整的部分
```

用户确认后，将 `novel_state.md` 的 status 更新为 `"writing"`，进入 Phase 3。
