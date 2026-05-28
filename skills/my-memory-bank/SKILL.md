---
name: my-memory-bank
description: 类 Cline 风格的零依赖、纯文件跨会话记忆系统。支持全局和项目级记忆的自动与手动存取，内置 index 导航、log 时间线、lint 健康检查与答案回写。
---

# Memory Bank 行为指南 (SKILL)

本 Skill 为 AI 代理提供一套零依赖、纯 Markdown 文件的跨会话记忆存取规范。记忆是**持续累积的复利资产**——每次保存都让下一次检索更精准，而不是每次重新推导。

---

## 一、存储结构与目录划分

记忆分为 **全局记忆** 和 **项目记忆** 两层：

### 1. 全局记忆 (跨项目通用)
- **存储路径**: `~/.claude/cc-memory-bank/`
- **内容文件**:
  - `preferences.md` — 用户个人偏好、编码风格、开发习惯、常用命令。
  - `conventions.md` — 通用规范、命名约定、最佳实践、通用安全规则。
  - `decisions.md` — 跨项目的重要技术决策与思考沉淀。
- **导航文件**:
  - `index.md` — 所有内容文件的目录，每个文件一行摘要，供检索时优先读取。
  - `log.md` — 操作时间线，追加式记录每次保存、回忆、导入、lint 操作。

### 2. 项目记忆 (当前工作区独立)
- **存储路径**: `<当前工作区根目录>/.cc-memory-bank/`
- **内容文件**:
  - `brief.md` — 项目概览、核心需求、一句话愿景。
  - `product.md` — 产品目标、用户体验、"为什么"要做这个产品。
  - `architecture.md` — 架构决策、系统设计模式、模块化划分。
  - `tech.md` — 技术栈、核心依赖、环境配置、编译部署命令。
  - `progress.md` — 整体进度：已完成（Completed）与待办（Todo）。
  - `active.md` — 当前工作焦点、近期变更、会话交接上下文。
  - `insights/` — （可选）存放值得复用的分析结论、对比报告、重要答案。
- **导航文件**:
  - `index.md` — 所有内容文件的目录摘要。
  - `log.md` — 操作时间线。

---

## 二、检索优先原则 (CRITICAL)

**每次 recall 必须先读 index.md，再按需精准读取，禁止盲目读取全部文件。**

```
recall 流程:
1. Read index.md (全局 + 项目)          ← 了解有什么、在哪里
2. 根据当前任务判断哪些文件相关         ← 按需定向
3. Read 相关文件 (通常 2-4 个)           ← 精准加载
4. 默默吸收，在后续行动中体现           ← 不复述，直接用
```

index.md 是检索的入口，它的质量直接决定记忆能否被找到。每次写入内容文件后，**必须同步更新 index.md**。

---

## 三、核心行为与指令触发

### 1. 快捷保存 (`/cc` 或 `cc`)
- **触发**: 发送单独的快捷输入 `/cc`、`cc`，或触发 `/cc-memory-bank` 命令行/Skill 时（**重要：若用户输入 `/cc` 导致系统自动匹配并触发了 `cc-memory-bank` 这个 Skill，AI 应当作快捷保存触发，立即开始执行保存流程**）。
- **流程**:
  1. 审视会话历史，提炼值得记录的内容：
     - 架构/技术决策 → `architecture.md`
     - 技术栈/依赖变更 → `tech.md`
     - 已完成工作/新待办 → `progress.md` / `active.md`
     - 用户偏好 → 全局 `preferences.md`
  2. `Read` 目标文件 → `Edit` 增量写入（禁止整体覆盖）。
  3. 更新 `index.md`（若该文件摘要有变化）。
  4. 向 `log.md` 追加一条记录：`## [YYYY-MM-DD] save | <本次保存内容的一句话摘要>`。
  5. 输出简洁确认：
     > ✅ **记忆已保存**
     > - `architecture.md` — 记录了 xxx
     > - `log.md` — 已追加操作记录

### 2. 完整保存 (`update memory bank` / `保存记忆` / `save memory`)
- **触发**: 输入上述短语。
- **流程**: 与快捷输入 `/cc` 相同，但额外主动询问用户是否有补充，并对全套文件做一次完整同步与整理，重建 `index.md`。

### 3. 全局保存 (`save to global` / `保存全局记忆`)
- **触发**: 用户明确要求保存到全局时。
- **流程**: 将当前会话中的通用偏好、规范、决策写入全局 `~/.claude/cc-memory-bank/` 对应文件，并更新全局 `index.md` 和 `log.md`。

### 4. 回忆 (`recall` / `回忆` / `load memory`)
- **触发**: 手动输入，或新会话开始时。
- **流程** (严格按顺序):
  1. `Read` 全局 `index.md` → 了解全局记忆概况。
  2. `Read` 项目 `index.md` → 了解项目记忆概况。
  3. 根据当前任务，选择性 `Read` 相关的 2-4 个内容文件。
  4. 将内容默默吸收为上下文背景，在后续行动中自然体现，无需向用户复述。
  5. 向 `log.md` 追加：`## [YYYY-MM-DD] recall | 加载了 <文件列表>`。

### 5. 答案回写 (`/file-insight: <标题>`)
- **触发**: 用户输入 `/file-insight: <标题>`，或 AI 判断当前对话产生了值得沉淀的分析结论。
- **流程**:
  1. 将当前对话中的分析结论、对比报告、重要推断整理为独立 Markdown 文件。
  2. 保存至 `.cc-memory-bank/insights/<YYYY-MM-DD>-<标题>.md`。
  3. 在 `index.md` 中为该文件新增一行摘要。
  4. 追加 `log.md` 记录：`## [YYYY-MM-DD] insight | <标题>`。
  5. 确认反馈：
     > 💡 **分析已归档**
     > - `insights/2026-05-27-xxx.md` 已创建

### 6. 跨项目导入 (`/cc-memory-bank: <项目路径>`)
- **触发**: 输入 `/cc-memory-bank:` 后跟本地项目路径。
- **流程**:
  1. 解析路径，展开 `~` 为绝对路径。
  2. 依次 `Read` 该路径 `.cc-memory-bank/` 下存在的所有内容文件，跳过缺失文件。
  3. 按映射规则合并写入全局 `~/.claude/cc-memory-bank/`：
     - `brief.md` / `product.md` / `progress.md` / `active.md` → 全局 `decisions.md`（以项目名为标题追加一节）
     - `architecture.md` → 全局 `decisions.md`（追加架构决策小节）
     - `tech.md` → 全局 `conventions.md`（追加技术规范小节）
  4. 每条内容标注来源：`<!-- 导入自: <路径> | YYYY-MM-DD -->`
  5. 更新全局 `index.md`，追加全局 `log.md`：`## [YYYY-MM-DD] import | <项目路径>`。
  6. 输出合并汇总。

### 7. 健康检查 (`/lint-memory`)
- **触发**: 用户输入 `/lint-memory`。
- **流程**:
  1. 读取 `index.md`，遍历所有已登记的内容文件。
  2. 逐一检查并报告以下问题：
     - **矛盾**: 不同文件中存在相互冲突的描述或决策。
     - **过期**: 有明确时间戳的条目与近期记录产生了逻辑矛盾。
     - **孤岛**: `index.md` 中登记但实际文件不存在，或文件存在但未登记。
     - **空洞**: `index.md` 中某文件摘要为空或意义不明。
     - **待补充**: 提示哪些主题值得深入记录。
  3. 对可自动修复的问题（如 `index.md` 补漏）直接修复。
  4. 对需要用户判断的矛盾，以列表形式呈现，等待用户指示。
  5. 追加 `log.md`：`## [YYYY-MM-DD] lint | 发现 <N> 个问题，修复 <M> 个`。

### 8. 任务完成自动归档 (自动)
- **触发**: 当前会话完成了复杂开发任务，或 `task.md` 中任务全部标记 `[x]`。
- **流程**:
  1. `Read` 项目 `progress.md` 和 `active.md`。
  2. 将完成的任务从"待办"移至"已完成"，更新 `active.md` 为下一步建议。
  3. 更新 `index.md`，追加 `log.md`。
  4. 在最终总结中轻描一句："已自动将任务进度归档至 Memory Bank。"

---

## 四、index.md 规范

`index.md` 是检索入口，格式必须严格统一：

```markdown
# Memory Bank 索引

> 最后更新: YYYY-MM-DD

## 内容文件

| 文件 | 摘要 | 最后更新 |
|------|------|----------|
| [preferences.md](preferences.md) | 用户偏好：简洁回答、Vanilla CSS、模块化 JS | 2026-05-27 |
| [conventions.md](conventions.md) | 通用规范：命名约定、Git 提交格式 | 2026-05-27 |
| [decisions.md](decisions.md) | 跨项目决策：选择零依赖记忆方案 | 2026-05-27 |
| [insights/2026-05-27-xxx.md](insights/2026-05-27-xxx.md) | xxx 对比分析结论 | 2026-05-27 |
```

**更新规则**: 每次写入任何内容文件后，必须同步更新 `index.md` 对应行的摘要和日期。摘要控制在 20 字以内，突出最核心的关键词。

---

## 五、log.md 规范

`log.md` 是追加式时间线，**只追加，不修改历史记录**：

```markdown
# 操作日志

## [2026-05-27] save | 记录了 React 状态管理决策，更新了 progress.md
## [2026-05-27] recall | 加载了 index.md、architecture.md、active.md
## [2026-05-27] insight | 归档「Redux vs Zustand 对比分析」
## [2026-05-27] import | ~/Desktop/old-project/.cc-memory-bank/
## [2026-05-27] lint | 发现 2 个问题，修复 1 个（index.md 补漏）
```

格式固定为 `## [YYYY-MM-DD] <操作类型> | <一句话描述>`，便于用 `grep "^\#\# \[" log.md` 快速过滤。

---

## 六、Markdown 文件模版

### 项目级 `brief.md`
```markdown
# 项目概览 (Project Brief)

## 核心需求
- 简述项目的核心功能和解决的问题。

## 目标与愿景
- 产品的终极目标是什么。

## 相关文件
- [[architecture.md]] — 技术选型决策
- [[product.md]] — 产品目标详述
```

### 项目级 `architecture.md`
```markdown
# 架构决策 (Architecture Decisions)

## 2026-05-27: 选择零依赖 Markdown 记忆方案
- **背景**: 考虑过集成 claude-mem MCP 数据库，但存在外部依赖。
- **决定**: 改用纯本地 Markdown 读写，实现无缝迁移与 Git 版本控制。
- **影响**: 状态变化需要 AI 主动拦截 `/cc` 快捷输入进行增量更新。

## 相关文件
- [[tech.md]] — 具体技术栈
- [[brief.md]] — 项目背景
```

### 项目级 `progress.md`
```markdown
# 进度追踪 (Progress)

## 已完成 (Completed)
- [x] 2026-05-27: 完成 cc-memory-bank 插件基础架构设计

## 待做 (Todo)
- [ ] 测试 /cc 快捷保存功能
- [ ] 测试 recall 自动恢复记忆功能

## 相关文件
- [[active.md]] — 当前工作焦点
```

### 全局 `preferences.md`
```markdown
# 个人偏好 (Global Preferences)

## 编码风格
- 首选 Vanilla CSS，除非明确要求不使用 TailwindCSS。
- JavaScript 保持模块化，避免过度封装。

## 交互习惯
- 保持回答简练，不要废话。
- 任务完成时自动归档进度。

## 相关文件
- [[conventions.md]] — 通用规范
```

---

## 七、增量更新策略 (CRITICAL)

**严禁整体覆盖文件。** 每次更新必须：
1. `Read` 目标文件，识别现有层级结构。
2. `Edit` 在精确位置插入新内容，保留所有历史记录。
3. 新建文件时用 `Write`，以对应模版初始化后再写入内容。
4. 每条新记录标注时间戳 `YYYY-MM-DD`。
5. 写完内容文件后，立即更新 `index.md` 对应行。
