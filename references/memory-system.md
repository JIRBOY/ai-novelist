# 长期记忆系统

## 问题背景

长篇小说（10 万字以上）面临的核心挑战：
1. **设定遗忘**：写到后面忘记前面的设定
2. **角色漂移**：角色性格、位置、能力前后矛盾
3. **剧情断裂**：伏笔忘记回收，支线悬而未决
4. **时间线错乱**：故事时间推进不合理

## 五层协同记忆架构

### 第一层：六步质量门禁（每章强制）

每章生成后强制执行六步闭环，确保每章都同步更新记忆。

### 第二层：RAG 剧情检索

**两级检索算法**：

1. **粗筛（BM25 风格 TF-IDF）**：从所有章节中筛选候选池（默认 8 个）
2. **精排（语义重排）**：对候选池进行语义分析，返回 Top-K（默认 4 个）

**条件触发**：
- 轻场景（日常/过渡）自动跳过检索
- 复杂剧情才执行检索

**实现步骤**（Phase 3 Step 1 写前分析时执行）：

1. **构建查询**：从当前章节控制卡提取关键词（角色名、地点名、伏笔 ID、事件关键词）
2. **粗筛**：读取 `story_index.json`，对每章的 TF-IDF 向量计算与查询的 BM25 相似度，取 Top-8
3. **精排**：对 Top-8 候选章节，读取 `entity_chapter_map.json` 中与查询实体共现的章节，按共现强度 + 时间近度重排，取 Top-4
4. **提取片段**：从 Top-4 章节正文中提取包含查询关键词的段落（每章最多 2 段，每段 ≤200 字）
5. **写入上下文**：将提取的片段写入 `00_memory/retrieval/next_plot_context.md`，供本章写作参考
6. **降级策略**：若 story_index.json 为空（前 3 章），直接读取最近 2 章全文作为上下文

**文件位置**：
```
00_memory/retrieval/
├── story_index.json          # 章节检索索引
├── entity_chapter_map.json   # 实体-章节映射
├── next_plot_context.md      # 当前写前上下文建议
└── chapter_meta/*.meta.json  # 章节元数据侧车文件
```

### 第三层：知识图谱

用图结构管理角色、事件、伏笔、世界观：

```json
{
  "nodes": [
    {
      "id": "char_001",
      "type": "character",
      "name": "李承乾",
      "status": "alive",
      "properties": {
        "age": 25,
        "faction": "太子党",
        "ability": "武道七品",
        "location": "京城",
        "mood": "隐忍"
      },
      "last_updated": 15
    },
    {
      "id": "evt_003",
      "type": "event",
      "name": "宫宴刺杀",
      "chapter": 12,
      "properties": { "resolved": false, "impact": "high" }
    },
    {
      "id": "fs_002",
      "type": "foreshadowing",
      "name": "神秘玉佩",
      "status": "planted",
      "planted_chapter": 3,
      "expected_recall": 20
    }
  ],
  "edges": [
    { "type": "ally", "source": "char_001", "target": "char_003", "strength": 0.8, "since_chapter": 5 },
    { "type": "holds", "source": "char_001", "target": "fs_002", "since_chapter": 3 },
    { "type": "involved_in", "source": "char_001", "target": "evt_003", "role": "target" }
  ],
  "timeline": [
    { "chapter": 1, "time": "建元三年春", "event": "主角入京" },
    { "chapter": 12, "time": "建元三年秋", "event": "宫宴刺杀" }
  ]
}
```

**节点类型**：角色、地点、势力、物品、事件、伏笔、世界观规则、力量体系
**边类型**：同盟、敌对、师徒、从属、情感、归属、位于、引发、铺垫、持有

**自动回写操作步骤**（每章门禁 Step 1 执行）：

1. **扫描本章正文**，提取以下信息：
   - 新出场角色 → 创建 character 节点（含 name, status, properties）
   - 角色位置/情绪/能力变化 → 更新对应节点 properties + last_updated
   - 新事件 → 创建 event 节点（含 chapter, properties.resolved）
   - 角色间互动 → 创建/更新 edge（含 type, strength, since_chapter）
   - 新埋设/回收伏笔 → 创建/更新 foreshadowing 节点
   - 时间推进 → 追加 timeline 条目
2. **写入 `story_graph.json`**：append 节点和边，更新 timeline
3. **标记变更**：新增/修改的节点设置 `last_updated: {chapter_number}`

**改纲级联**：修改大纲时自动计算影响范围并更新关联节点

### 第四层：大纲锚点

**进度配额约束**：

```json
{
  "total_chapters": 300,
  "current_chapter": 42,
  "current_arc": "第二卷：朝堂博弈",
  "chapters_in_arc": 40,
  "quota": {
    "min_progress": 0.14,
    "max_progress": 0.15
  }
}
```

每章写前读取全局进度条，动态注入约束。越界直接触发门禁失败。

### 第五层：跨 Agent 审核

**双智能体交叉验证**：

| 写作 Agent | 审核 Agent |
|-----------|-----------|
| 主 Agent | 子 Agent（独立审稿官人设） |

**三维度审核**：
1. 逻辑与连续性硬伤
2. 阅读体验与节奏把控
3. 文笔去 AI 化

**防死循环机制**：
- 单章最多 3 轮审核
- 连续 3 章"有条件通过"强制暂停请求人工介入

**并行写作冲突处理**（子 Agent 模式）：

当多个子 Agent 并行写作时，批次结束后执行跨章节一致性检查：

1. **冲突检测**：比对各子 Agent 产出的角色状态、伏笔操作、时间线，标记矛盾项
2. **修复优先级**：角色生死冲突 > 伏笔状态冲突 > 时间线冲突 > 细节冲突
3. **修复原则**：后写章节服从先写章节（先完成的章节状态为准真值），后写章节修改冲突部分
4. **回写同步**：修复完成后统一回写 `character_states.json` 和 `foreshadowing_ledger.json`
5. **批次隔离**：前一批次全部完成并通过一致性检查后，后一批次才能启动

## 低上下文策略

长篇创作的上下文管理至关重要：

- 写前默认只读：`novel_plan.md` + `novel_state.md`
- 单章前置读取上限：**最多 4 个文件**
- RAG 检索只返回 Top-K 片段，不回读整章
- 每 10 章做一次深度压缩与风格校准
- 每卷结束时做一次全书体检

## 改纲级联更新

当需要修改主线规划时：

1. **锚点重算**：备份当前锚点 → 从修改后的大纲重新计算所有锚点
2. **图谱级联标记**：将受影响的知识图谱节点标记为 `cascade_pending`
3. **RAG 索引重建**：全量重建检索索引
4. **影响报告**：生成改纲影响范围报告

```bash
# 改纲流程
1. 手动编辑 novel_plan.md
2. 确认改纲影响的起始章节
3. 执行锚点重算 → 图谱级联 → RAG 重建
4. 检查影响报告
5. 对 cascade_pending 节点做人工审核
6. 恢复正常写作流程
```
