# Phase 0：初始化与偏好加载

## 执行流程

### 1. 加载用户偏好

读取 `user-preferences.json`（位于项目根目录或 `~/.workbuddy/skills/ai-novelist/`）：

```json
{
  "preferred_genres": ["悬疑推理", "都市现实"],
  "preferred_protagonist": "男性主角",
  "preferred_chapter_count": 20,
  "preferred_word_density": "中等",
  "preferred_pov": "第三人称限知",
  "preferred_tone": "紧张冷峻",
  "writing_history": [
    {"title": "午夜列车", "genre": "悬疑", "chapters": 20, "completed": true}
  ]
}
```

若文件不存在，使用默认偏好，并在首次创作后自动生成。

### 2. 检测中断续写

扫描当前工作目录下是否存在未完成项目：

- 检查是否存在 `00_memory/novel_state.md`
- 若存在且 `status != "completed"`，提示用户：
  > "检测到未完成的项目《{title}》，当前进度：第{current_chapter}章 / 共{total_chapters}章。是否继续？"

### 3. 个性化欢迎

基于偏好和历史，展示欢迎信息：

```
====================================
  AI Novelist — 长篇小说创作系统
====================================
欢迎回来！根据你的偏好：
  偏好题材：悬疑推理 | 都市现实
  常用视角：第三人称限知
  偏好基调：紧张冷峻

你可以：
  1. 输入「开书」开始新项目
  2. 输入「继续写」恢复未完成项目
  3. 直接描述你的小说想法
====================================
```

### 4. 项目目录初始化

确定项目名称后，创建标准目录结构：

```
{timestamp}-{小说名称}/
├── 00_memory/
├── 01_settings/
│   └── characters/
├── 02_outline/
│   └── volume-outlines/
├── 03_manuscript/
├── 04_editing/
│   └── gate_artifacts/
├── 05_control/
└── writing-log.md
```

### 5. 初始化记忆文件

创建空的记忆文件模板：

- `00_memory/novel_state.md`：初始化为 `status: "planning"`, `current_chapter: 0`
- `00_memory/story_graph.json`：初始化为空图 `{"nodes": [], "edges": [], "timeline": []}`
- `00_memory/foreshadowing_ledger.json`：初始化为空台账 `{"foreshadowings": []}`
- `00_memory/character_states.json`：初始化为空 `{"characters": []}`
- `00_memory/writing-plan.json`：初始化为空 `{"title": "", "total_chapters": 0, "status": "planning"}`
- `00_memory/outline_anchors.json`：初始化为空 `{"total_chapters": 0, "current_chapter": 0, "arc_anchors": []}`
- `00_memory/timeline.json`：初始化为空 `{"events": []}`
