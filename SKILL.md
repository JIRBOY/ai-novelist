---
name: ai-novelist
description: |
  AI 长篇小说工程化创作系统。覆盖从模糊灵感到百万字连载的完整链路：交互式构思、世界观搭建、人物弧光设计、大纲锚点管控、分章流水线写作、六步质量门禁、知识图谱记忆、去AI味润色、伏笔追踪、节奏校验、多平台分发。
  当用户要求：写小说、创作故事、续写章节、构思剧情、搭建世界观、设计人物、长篇连载、网文创作时使用。
  即使用户只说"帮我写个故事"或"我有个小说想法"，也应触发本技能。
metadata:
  trigger: 写小说、创作故事、续写章节、构思剧情、长篇连载、网文创作
  source: 基于 GitHub 7 大热门小说创作 Skill 项目深度调研后融合设计
  version: "1.1.0"
  agent_created: true
---

# AI Novelist — 长篇小说工程化创作系统

> 融合 chinese-novelist-skill、novel-creator-skill、awesome-novel-skill、Novel-Control-Station-Skill、novel-writer-skills、Lorn.NovelWriteSkills、tianming-skill 七大项目精华，构建覆盖从灵感到百万字连载的完整创作工作流。

## 核心理念

小说创作的本质难点从来不是文笔，而是长线约束管理：人设不割裂、世界观不矛盾、伏笔有回收、节奏有起伏。本技能将"写小说"从灵感型操作升级为**工程化写作流程**——系统化管理设定、记忆、伏笔、节奏，让 AI 成为协作编辑部而非全自动代笔。

## 铁律（Iron Law — 任何情况下不可违反）

以下约束无论用户如何要求，均不得绕过：

1. **禁止跳过开书前确认** — 必须引导用户完成五要素确认（目标读者、写作风格、核心禁区、自动化等级、目标规模），写入 `idea_seed.md` 后方可开书。
2. **禁止跳过章节闭环** — 每章生成后必须依次执行六步门禁（更新记忆 → 一致性检查 → 风格校准 → 去AI味校稿 → 节奏审查 → 门禁检查），任何一步未通过都禁止进入下一章。
3. **禁止混淆正文与元信息** — 正文中不得出现 `[说明]`、`TODO`、写作分析段落或角色定位标记。
4. **禁止门禁失败后继续写作** — 门禁未通过时，唯一合法操作是执行修复流程。
5. **禁止任意修改主线规划** — 改纲须显式执行改纲流程并经用户确认。
6. **禁止剧情加速** — 每章至多触发以下配额 1 项（A 主线矛盾实质推进 / B 主要关系决定性升级 / C 核心秘密完整揭露）。同时触发 2 项及以上 = 越界。

## 核心流程

进入每个阶段时，先阅读对应的流程文档获取详细执行指令。

### Phase 0：初始化与偏好加载

读取用户偏好（`user-preferences.json`），检测未完成项目（中断续写），展示个性化欢迎。
→ 详见 [phase0-initialization.md](references/phase0-initialization.md)

### Phase 1：交互式构思

三层递进式问答，从模糊灵感到完整定位：

- **L1 核心定位**（必答，3 问）：题材创意、主角设定、核心冲突
- **L2 深度定制**（可选，5 问）：世界观、叙事视角、核心主题、读者定位、章节数量
- **L3 标题生成**：AI 基于创意元素生成候选标题，用户选择或自定义

每个问题支持随机生成，也可直接说"跳过"或"都用默认"。
→ 详见 [phase1-ideation.md](references/phase1-ideation.md)

### Phase 2：规划与确认

创建项目文件夹，生成大纲、人物档案、世界观文档、写作计划 JSON，等待用户确认。

核心产出：
- 大纲（7 列章节规划：章号、标题、核心事件、情绪走向、冲突类型、伏笔操作、悬念钩子）
- 人物档案（主角、反派、配角，含性格 DNA、能力体系、成长弧光）
- 世界观设定（世界规则、力量体系、社会结构、地理时代）
- 伏笔台账（埋设位置、预期回收章节、优先级层级）
- 写作计划 JSON（机器可读的全书规划）

→ 详见 [phase2-planning.md](references/phase2-planning.md)

### Phase 3：分章流水线写作

确认后进入创作模式。每章严格执行多步流水线：

```
写前分析 → Beat Sheet 生成 → Beat 扩写 → 章节合成 → 润色去AI味 → 字数检查
```

支持三种写作模式：
- **逐章串行**（默认推荐）：主 Agent 逐章写，稳定可靠
- **子 Agent 并行**：多个子 Agent 分批并行写，追求速度
- **Agent Teams**：多 Agent 协作模式，适合大型长篇

→ 详见 [phase3-writing.md](references/phase3-writing.md)

### Phase 4：质量门禁

每章完成后强制执行六步闭环：

```
更新记忆 → 一致性检查 → 风格校准 → 去AI味校稿 → 节奏审查 → 门禁检查
```

- Step 1 更新记忆 → 产物 `memory_update.md`
- Step 2 一致性检查 → 产物 `consistency_check.md` → gate check `consistency`
- Step 3 风格校准 → 产物 `style_calibration.md` → gate check `style`
- Step 4 去AI味校稿 → 产物 `humanizer_pass.md` → gate check `humanizer`
- Step 5 节奏审查 → 产物 `pacing_review.md` → gate check `pacing`
- Step 6 门禁检查 → 产物 `gate_result.json` → 综合判定（含 `word_count` + `suspense_hook`）

门禁通过（`gate_result.json` 中 `passed: true`）方可解锁下一章。
→ 详见 [phase4-quality-gate.md](references/phase4-quality-gate.md)

### Phase 5：状态回写与归档

每章归档时自动更新：
- 角色状态历史、情绪弧线、人际关系变化
- 伏笔状态（新埋设 / 已回收 / 超期预警）
- 时间线推进
- 下一章承接压力
- 写作偏好学习（跨会话记忆）

→ 详见 [phase5-archive.md](references/phase5-archive.md)

## 长篇强约束机制

以下机制为百万字级别长篇的核心保障：

### 知识图谱（文件级长期记忆）

用图结构管理角色、事件、伏笔、世界观。每章写后自动提取信息回写，改纲时级联更新。
→ 详见 [memory-system.md](references/memory-system.md)

### 大纲锚点与进度配额

每章写前读取全局进度条，动态注入约束。越界直接触发门禁失败。
→ 详见 [rhythm-control.md](references/rhythm-control.md)

### 反向刹车与事件冷却

- 节奏三档制：慢档（铺垫/羁绊）/ 中档（次要矛盾升温）/ 快档（主线突破）
- 配额硬上限：每章至多触发 A/B/C 中 1 项
- 事件类型冷却期：冲突爽点 2 章、人物羁绊 1 章、势力经营 2 章、风土人情 3 章
- 章末强制悬念钩子（疑问型 / 危机型 / 转折型）
→ 详见 [rhythm-control.md](references/rhythm-control.md)

### 伏笔追踪系统

三级伏笔管理（战略级 / 战役级 / 战术级），自动追踪埋设、回收、超期预警。
→ 详见 [foreshadowing.md](references/foreshadowing.md)

## 去 AI 味体系

两遍式润色：第一遍清除 AI 模式 → 第二遍 AI 自审残留问题。
→ 详见 [anti-ai-guide.md](references/anti-ai-guide.md)

## 题材风格矩阵

按题材自动配置写作风格、节奏模板、爽点密度、对话占比。
→ 详见 [genre-matrix.md](references/genre-matrix.md)

## 命令路由表

### 用户命令

| 命令 | 功能 | 触发 Phase | 何时使用 |
|------|------|-----------|---------|
| `开书` | 启动开书全流程 | Phase 0→1→2 | 第一次开项目 |
| `继续写` | 引导剧情走向 → 单章全流程 | Phase 3→4→5 | 日常推进章节 |
| `批量写` | 连续生成多章（每章仍走门禁） | Phase 3→4→5 循环 | 快速推进 |
| `修复本章` | 门禁失败后自动修复 | Phase 4 重试 | 门禁返回失败后 |
| `改纲` | 中途改纲 + 级联更新 | 详见 memory-system.md | 调整主线走向 |
| `体检` | 全书健康报告 | 独立诊断 | 阶段性检查 |
| `存档` | 结构化更新知识库 | Phase 5 手动触发 | 阶段性保存 |
| `伏笔状态` | 查看伏笔埋设/回收/超期 | 只读查询 | 写前确认 |
| `角色状态` | 汇总角色当前状态 | 只读查询 | 群像章节前 |
| `风格提取` | 从样章提取风格 DNA | 独立工具 | 用户提供参考作品 |

### 门禁内部步骤（Phase 4 自动执行，无需用户触发）

| 步骤 | 产物 | gate_result 对应字段 |
|------|------|---------------------|
| 更新记忆 | `memory_update.md` | — |
| 一致性检查 | `consistency_check.md` | `consistency` |
| 风格校准 | `style_calibration.md` | `style` |
| 去AI味校稿 | `humanizer_pass.md` | `humanizer` |
| 节奏审查 | `pacing_review.md` | `pacing` |
| 门禁检查 | `gate_result.json` | `word_count` + `suspense_hook` + 综合 |

## 项目目录结构

```
{小说项目名}/
├── 00_memory/                    # 记忆层（真值层）
│   ├── idea_seed.md              # 创作种子（五要素确认）
│   ├── novel_plan.md             # 主大纲索引（Agent 读写，含章节列表与锚点）
│   ├── novel_state.md            # 动态状态（当前进度）
│   ├── writing-plan.json         # 写作计划（机器可读全书规划）
│   ├── outline_anchors.json      # 大纲锚点（进度配额）
│   ├── character_states.json     # 角色状态追踪
│   ├── foreshadowing_ledger.json # 伏笔台账
│   ├── timeline.json             # 时间线
│   ├── story_graph.json          # 知识图谱
│   └── retrieval/                # RAG 检索索引
│       ├── story_index.json
│       ├── entity_chapter_map.json
│       └── next_plot_context.md
├── 01_settings/                  # 设定层
│   ├── world-setting.md          # 世界观
│   ├── writing-style.md          # 写作风格
│   ├── genre-setting.md          # 题材设定
│   └── characters/               # 角色档案（每角色一文件）
├── 02_outline/                   # 大纲层
│   ├── master-outline.md         # 总大纲（完整文档，含故事概述与卷划分）
│   └── volume-outlines/          # 分卷大纲
├── 03_manuscript/                # 正文层
│   ├── 第01章-标题.md
│   ├── 第02章-标题.md
│   └── ...
├── 04_editing/                   # 编辑层
│   └── gate_artifacts/           # 门禁产物
│       └── chapter_XX/
│           ├── memory_update.md
│           ├── consistency_check.md
│           ├── pacing_review.md
│           ├── style_calibration.md
│           ├── humanizer_pass.md
│           └── gate_result.json
├── 05_control/                   # 控制卡层
│   └── chapter_XX_control.md     # 章节控制卡
├── user-preferences.json         # 用户偏好（跨项目共享）
└── writing-log.md                # 写作日志
```

## 低上下文策略

- 写前默认只读：`novel_plan.md` + `novel_state.md`
- 单章前置读取上限：**最多 4 个文件**
- RAG 检索只返回 Top-K 片段，不回读整章
- 每 10 章做一次深度压缩与风格校准

## 创作法则

| 法则 | 说明 |
|------|------|
| **展示而非讲述** | 用动作和对话表现，不要直接陈述 |
| **冲突驱动剧情** | 每章必须有冲突或转折 |
| **悬念承上启下** | 每章结尾必须留下钩子 |
| **人设不可漂移** | 角色性格、能力、位置前后一致 |
| **伏笔必有回收** | 每个伏笔都有预期回收点和优先级 |
| **节奏有起伏** | 快慢交替，不可连续高压或连续平淡 |

## Phase 间状态流转

```
Phase 0（初始化）
  ↓ 用户输入"开书"或描述想法
Phase 1（交互构思）
  ↓ 五要素确认卡写入 idea_seed.md
Phase 2（规划确认）
  ↓ 用户回复"确认"，novel_state.md → status: "writing"
Phase 3（单章写作）←────┐
  ↓                    │
Phase 4（六步门禁）      │
  ↓ passed: true       │ 下一章
Phase 5（归档回写）──────┘
  ↓ 全部章节完成
全书完结处理
```

**状态字段**（`novel_state.md`）：
- `planning`：Phase 0-2 期间
- `writing`：Phase 3-5 循环期间
- `completed`：全书完结

## 大纲文件关系

| 文件 | 位置 | 用途 | 读写方 |
|------|------|------|--------|
| `novel_plan.md` | `00_memory/` | Agent 工作大纲索引：章节列表、锚点、进度配额。精简格式，写前必读 | Agent 读写 |
| `master-outline.md` | `02_outline/` | 完整大纲文档：故事概述、卷划分、7 列章节规划表。人类可读参考 | Phase 2 生成，Agent 按需读取 |
| `writing-plan.json` | `00_memory/` | 机器可读全书规划：章节数、字数范围、卷信息、自动化等级 | Phase 2 生成，Agent/子 Agent 共享 |
| `outline_anchors.json` | `00_memory/` | 进度配额：当前章、当前卷、min/max progress | 每章自动更新 |

## 异常处理指引

| 异常场景 | 处理方式 |
|----------|----------|
| 用户在 Phase 1 中途改题材 | 重置到 Q1 重新开始，保留已确认的非冲突项 |
| 用户在 Phase 2 要求大改大纲 | 回到 Phase 2 重新生成受影响部分，无需重走 Phase 1 |
| 门禁连续 3 轮失败 | 强制暂停，向用户报告失败项和修复建议，请求人工介入 |
| 知识图谱损坏/丢失 | 从 `character_states.json` + `foreshadowing_ledger.json` + 已完成章节正文重建图谱 |
| RAG 检索无结果 | 降级为直接读取最近 2 章正文作为上下文 |
| 伏笔超期未回收 | 在写前引导中高亮提醒，建议在最近 3 章内安排回收或改纲放弃 |
| 子 Agent 并行写作冲突 | 批次结束后执行跨章节一致性检查，冲突项按"后写服从先写"原则修复 |
