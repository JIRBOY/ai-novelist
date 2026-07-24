# Phase 5：状态回写与归档

## 归档流程

每章门禁通过后，执行以下归档操作：

### 1. 正文归档

将润色后的正文定稿写入 `03_manuscript/第{NN}章-{标题}.md`

### 2. 角色状态更新

更新 `00_memory/character_states.json`：

```json
{
  "characters": [
    {
      "id": "char_001",
      "name": "{name}",
      "current_location": "{location}",
      "current_mood": "{mood}",
      "known_facts": ["{已知信息1}", "{已知信息2}"],
      "ability_state": "{能力状态}",
      "status": "alive",
      "arc_position": "{弧光阶段}",
      "state_history": [
        {
          "chapter": 1,
          "location": "{location}",
          "mood": "{mood}",
          "key_event": "{事件}",
          "relationships_change": ["{关系变化}"]
        }
      ]
    }
  ]
}
```

### 3. 伏笔状态更新

更新 `00_memory/foreshadowing_ledger.json`：

- 扫描本章新埋设的伏笔 → 添加到台账
- 标记本章回收的伏笔 → 更新 `actual_recall_chapter` 和 `status`
- 检测超期伏笔（`current_chapter > expected_recall_chapter + 5`）→ 标记 `status: "overdue"`

### 4. 知识图谱回写

更新 `00_memory/story_graph.json`：

- 新增角色节点（如有新角色出场）
- 新增事件节点
- 新增/更新关系边
- 更新时间线

**节点类型**：角色、地点、势力、物品、事件、伏笔、世界观规则
**边类型**：同盟、敌对、师徒、从属、情感、归属、位于、引发、铺垫、持有

### 5. 大纲锚点更新

更新 `00_memory/outline_anchors.json`：

- `current_chapter` +1
- 重新计算 `quota`（min_progress / max_progress）
- 检查是否进入新卷

### 6. 情绪弧线追加

在角色档案中追加情绪弧线记录：
- 本章角色情绪状态
- 情绪转折点
- 人际关系变化

### 7. 写作日志

追加到 `writing-log.md`：

```markdown
## 第{N}章 — {时间戳}
- 字数：{count}
- 节奏档位：{tier}
- 配额触发：{A/B/C}
- 伏笔操作：埋设{x}个，回收{y}个
- 悬念钩子：{type} — {brief}
- 门禁状态：通过
- 备注：{如有特殊事项}
```

### 8. 偏好学习

更新 `user-preferences.json`：

- 记录本次创作中的用户偏好信号（修改倾向、风格偏好、节奏偏好）
- 跨会话累积，下次创作自动应用

### 9. RAG 索引更新

更新 `00_memory/retrieval/`：
- `story_index.json`：建立本章的检索索引
- `entity_chapter_map.json`：更新实体-章节映射
- 生成 `next_plot_context.md`：为下一章提供推荐上下文

## 卷边界处理

当当前章为某卷最后一章时：
- 生成本卷总结（核心弧光完成情况、伏笔回收情况、角色成长情况）
- 检查本卷所有伏笔回收状态
- 生成下一卷的详细大纲（如未预先规划）
- 提示用户确认是否继续

## 全书完结处理

当所有章节完成时：
1. **全书体检**：执行全面一致性检查
2. **伏笔终检**：确认所有战略级伏笔已回收
3. **角色弧光终检**：确认所有主要角色弧光完整
4. **风格一致性终检**：全书风格偏差分析
5. **生成完结报告**：
   ```markdown
   # 《{title}》完结报告
   
   ## 基本信息
   - 总章数：{n}
   - 总字数：{count}
   - 创作周期：{start} - {end}
   
   ## 质量统计
   - 门禁通过率：{percentage}%
   - 平均修复轮次：{avg}
   - 伏笔回收率：{percentage}%
   
   ## 角色弧光完成度
   - {角色A}：{percentage}% — {说明}
   - {角色B}：{percentage}% — {说明}
   
   ## 遗留问题（如有）
   - {问题描述}
   ```
6. 更新 `novel_state.md` 的 status 为 `"completed"`
