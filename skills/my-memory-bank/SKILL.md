---
name: my-memory-bank
description: 类 Cline 风格的零依赖、纯文件跨会话记忆系统。支持全局和项目级记忆的自动与手动存取，内置目录分类、index 导航、log 时间线、lint 健康检查、答案回写与原始对话语料存档。
---

# Memory Bank 行为指南 (SKILL)

本 Skill 为 AI 代理提供一套零依赖、纯 Markdown 文件的跨会话记忆存取规范。记忆是**持续累积的复利资产**——每次保存都让下一次检索更精准，而不是每次重新推导。

---

## 一、存储结构与目录划分

记忆分为 **全局记忆** 和 **项目记忆** 两层，每层均使用**目录分类**组织文件：

### 1. 全局记忆 (跨项目通用)
- **存储路径**: `~/.claude/cc-memory-bank/`
- **预置目录**（首次使用时自动创建，见第零节）:

```
~/.claude/cc-memory-bank/
├── index.md                  # 检索入口
├── log.md                    # 操作时间线
├── preferences/              # 用户偏好：编码风格、开发习惯、审美倾向
├── conventions/              # 通用规范：命名约定、提交格式、安全规则
└── decisions/                # 跨项目决策：技术选型、方向沉淀
```

### 2. 项目记忆 (当前工作区独立)
- **存储路径**: `<当前工作区根目录>/.cc-memory-bank/`
- **预置目录**（首次使用时自动创建）:

```
.cc-memory-bank/
├── index.md                  # 检索入口
├── log.md                    # 操作时间线
├── overview/                 # 项目概览：brief、product 目标
├── engineering/              # 技术工程：architecture、tech 栈
├── progress/                 # 进度追踪：progress、active 焦点
├── insights/                 # 分析洞察：YYYY-MM-DD-<标题>.md
└── raw/                      # 原始对话语料：YYYY-MM-DD-HH-MM-<标题>.md（完整未提炼）
```

### 3. 动态目录扩展

当待保存内容**不属于任何现有目录**时，按以下规则动态创建新目录：

```
判断流程:
1. 识别内容类型关键词（如 "API 文档"、"安全策略"、"团队规范"、"运营数据"…）
2. 检查现有目录列表，判断是否有合适的归属
3. 若无匹配 → 推断语义最贴近的目录名（英文小写、连字符分隔）
4. mkdir 新目录 → 写入文件 → 更新 index.md 新增该目录条目
5. 向用户说明：「内容类型「xxx」在现有目录中无对应，已新建 xxx/ 目录」
```

**常见动态目录示例**:

| 内容类型 | 自动创建的目录 |
|----------|---------------|
| API 接口文档、端点定义 | `api/` |
| 安全策略、漏洞记录 | `security/` |
| 团队流程、协作规范 | `team/` |
| 运营数据、业务指标 | `metrics/` |
| 用户研究、访谈记录 | `research/` |
| 部署配置、环境说明 | `devops/` |
| 外部服务、第三方集成 | `integrations/` |

---

## 零、初始化流程 (INIT)

**触发时机**: 检测到 Memory Bank 目录不存在，或用户首次使用任何保存指令时。

```
初始化步骤:
1. 检查 ~/.claude/cc-memory-bank/ 是否存在
   └─ 不存在 → 创建全局预置目录树（见下方）
2. 检查 <project-root>/.cc-memory-bank/ 是否存在
   └─ 不存在 → 创建项目预置目录树
3. 生成 index.md（模板见第四节）
4. 生成空的 log.md
5. 确认输出：
   > 🗂️ Memory Bank 已初始化
   > - 全局: ~/.claude/cc-memory-bank/（preferences/ conventions/ decisions/）
   > - 项目: .cc-memory-bank/（overview/ engineering/ progress/ insights/ raw/）
```

**全局预置目录初始化命令**（AI 通过 Bash 工具执行）:
```bash
mkdir -p ~/.claude/cc-memory-bank/{preferences,conventions,decisions}
```

**项目预置目录初始化命令**:
```bash
mkdir -p .cc-memory-bank/{overview,engineering,progress,insights,raw}
```

---

## 二、检索优先原则 (CRITICAL)

**每次 recall 必须先读 index.md，再按需精准读取，禁止盲目读取全部文件。**

```
recall 流程:
1. Read index.md (全局 + 项目)          ← 了解有什么、在哪里
2. 根据当前任务判断哪些目录/文件相关    ← 按需定向
3. Read 相关文件 (通常 2-4 个)           ← 精准加载
4. 默默吸收，在后续行动中体现           ← 不复述，直接用
```

index.md 是检索的入口，它的质量直接决定记忆能否被找到。每次写入内容文件后，**必须同步更新 index.md**。

---

## 三、核心行为与指令触发

### 1. 快捷保存 (`/cc` 或 `cc`)
- **触发**: 发送单独的快捷输入 `/cc`、`cc`，或触发 `cc-memory-bank` Skill 时。
- **流程**:
  1. 若 Memory Bank 目录不存在，先执行**第零节初始化流程**。
  2. 审视会话历史，识别内容类型，映射到对应目录：
     - 架构/技术决策 → `engineering/architecture.md`
     - 技术栈/依赖变更 → `engineering/tech.md`
     - 已完成工作/新待办 → `progress/progress.md` / `progress/active.md`
     - 用户偏好 → 全局 `preferences/`
     - 项目概览/愿景 → `overview/`
     - **不属于任何预置目录** → 执行动态目录创建（见一·3节）
     - **原始对话存档** → `raw/`（见第三节·9）
  3. `Read` 目标文件（如存在） → `Edit` 增量写入（禁止整体覆盖）；文件不存在时用 `Write` 以模板初始化。
  4. 更新 `index.md`（若目录或文件摘要有变化）。
  5. 向 `log.md` 追加：`## [YYYY-MM-DD] save | <本次保存内容的一句话摘要>`。
  6. 输出简洁确认：
     > ✅ **记忆已保存**
     > - `engineering/architecture.md` — 记录了 xxx
     > - `log.md` — 已追加操作记录

### 2. 完整保存 (`update memory bank` / `保存记忆` / `save memory`)
- **触发**: 输入上述短语。
- **流程**: 与 `/cc` 相同，但额外主动询问用户是否有补充，并对全套文件做一次完整同步与整理，重建 `index.md`。

### 3. 全局保存 (`save to global` / `保存全局记忆`)
- **触发**: 用户明确要求保存到全局时。
- **流程**: 将当前会话中的通用偏好、规范、决策写入全局 `~/.claude/cc-memory-bank/` 对应目录，并更新全局 `index.md` 和 `log.md`。

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
     - `overview/` → 全局 `decisions/`（以项目名为标题追加一节）
     - `engineering/` → 全局 `decisions/`（追加技术决策小节）+ 全局 `conventions/`
  4. 每条内容标注来源：`<!-- 导入自: <路径> | YYYY-MM-DD -->`
  5. 更新全局 `index.md`，追加全局 `log.md`：`## [YYYY-MM-DD] import | <项目路径>`。
  6. 输出合并汇总。

### 7. 健康检查 (`/lint-memory`)
- **触发**: 用户输入 `/lint-memory`。
- **流程**:
  1. 读取 `index.md`，遍历所有已登记的目录和文件。
  2. 逐一检查并报告以下问题：
     - **矛盾**: 不同文件中存在相互冲突的描述或决策。
     - **过期**: 有明确时间戳的条目与近期记录产生了逻辑矛盾。
     - **孤岛**: `index.md` 中登记但实际文件/目录不存在，或存在但未登记。
     - **空洞**: `index.md` 中某目录/文件摘要为空或意义不明。
     - **待补充**: 提示哪些主题值得深入记录。
  3. 对可自动修复的问题（如 `index.md` 补漏）直接修复。
  4. 对需要用户判断的矛盾，以列表形式呈现，等待用户指示。
  5. 追加 `log.md`：`## [YYYY-MM-DD] lint | 发现 <N> 个问题，修复 <M> 个`。

### 8. 任务完成自动归档 (自动)
- **触发**: 当前会话完成了复杂开发任务，或 task.md 中任务全部标记 `[x]`。
- **流程**:
  1. `Read` 项目 `progress/progress.md` 和 `progress/active.md`。
  2. 将完成的任务从"待办"移至"已完成"，更新 `active.md` 为下一步建议。
  3. 更新 `index.md`，追加 `log.md`。
  4. 在最终总结中轻描一句："已自动将任务进度归档至 Memory Bank。"

### 9. 原始对话存档 (`/raw-save: <标题>` 或 `保存原始对话`)
- **触发**: 用户输入 `/raw-save: <标题>` 或 `保存原始对话`，或用户明确要求保留完整对话语料。
- **设计原则**: `raw/` 存储的是**原始语料**，必须做到字字不漏、原文照录。绝对禁止总结、改写、压缩、省略任何内容——哪怕是一句"好的"、一行代码、一条工具调用输出。这与 `insights/`（精炼结论）、`/cc`（要点摘取）的定位完全不同。
- **流程**:
  1. 若 `raw/` 目录不存在，先创建：`mkdir -p .cc-memory-bank/raw`。
  2. 文件命名规则：`YYYY-MM-DD-HH-MM-<标题>.md`（精确到分钟，避免同日文件冲突）。
  3. 按以下格式将**完整对话逐字写入**文件，每一轮 User / Assistant 消息均完整保留：
     ```markdown
     # <标题>

     > 存档时间: YYYY-MM-DD HH:MM
     > 来源: <项目名 / 会话描述>

     ---

     **[User]**

     <用户消息原文，一字不改，包括标点、换行、代码块>

     ---

     **[Assistant]**

     <助手回复原文，一字不改，包括代码块、工具调用输入输出、分析过程、每一行输出>

     ---

     **[User]**

     <下一条用户消息原文>

     ---

     **[Assistant]**

     <下一条助手回复原文>

     ---

     （以上 User / Assistant 交替格式循环，直到会话结束，不得跳过任何一轮）
     ```
  4. 在 `index.md` 的「文件清单」表新增一行，摘要注明：原始语料 + 主题关键词。
  5. 追加 `log.md`：`## [YYYY-MM-DD] raw | 存档「<标题>」→ raw/YYYY-MM-DD-HH-MM-<标题>.md`。
  6. 确认反馈：
     > 📄 **原始对话已存档**
     > - `raw/YYYY-MM-DD-HH-MM-<标题>.md` 已创建（完整原文，未作任何提炼）

- **注意事项**:
  - `raw/` 文件只写入、不编辑（历史语料不可篡改）；若需补充，新建文件。
  - 文件大小可能较大，`recall` 流程不主动加载 `raw/`，除非用户明确要求。
  - 同一主题可多次存档，文件名含时间戳可自然区分版本。
  - **严禁对 `raw/` 内任何文件执行整合、合并、摘要操作**，`ss` / `update memory bank` 等保存指令遇到 `raw/` 时跳过，不做任何处理。

---

## 四、index.md 规范

`index.md` 是检索入口，以**目录为单位**组织，格式必须严格统一：

```markdown
# Memory Bank 索引

> 最后更新: YYYY-MM-DD

## 目录结构

| 目录 | 用途 | 最后更新 |
|------|------|----------|
| [preferences/](preferences/) | 用户偏好：编码风格、审美倾向 | 2026-05-27 |
| [conventions/](conventions/) | 通用规范：命名约定、Git 提交格式 | 2026-05-27 |
| [decisions/](decisions/) | 跨项目决策：选择零依赖记忆方案 | 2026-05-27 |
| [raw/](raw/) | 原始对话语料：完整未提炼的会话存档 | 2026-05-27 |

## 文件清单

| 文件 | 摘要 | 最后更新 |
|------|------|----------|
| [preferences/coding.md](preferences/coding.md) | 首选 Vanilla CSS、模块化 JS | 2026-05-27 |
| [decisions/tech.md](decisions/tech.md) | 选择零依赖 Markdown 方案 | 2026-05-27 |
| [insights/2026-05-27-xxx.md](insights/2026-05-27-xxx.md) | xxx 对比分析结论 | 2026-05-27 |
| [raw/2026-05-27-14-30-xxx.md](raw/2026-05-27-14-30-xxx.md) | 原始语料：xxx 功能讨论完整对话 | 2026-05-27 |
```

**更新规则**:
- 新建目录时，在「目录结构」表新增一行。
- 新建或修改文件时，在「文件清单」表新增或更新对应行。
- 摘要控制在 20 字以内，突出最核心的关键词。

---

## 五、log.md 规范

`log.md` 是追加式时间线，**只追加，不修改历史记录**：

```markdown
# 操作日志

## [2026-05-27] init | 初始化 Memory Bank，创建 overview/ engineering/ progress/ insights/ raw/
## [2026-05-27] save | 记录 React 状态管理决策 → engineering/architecture.md
## [2026-05-27] mkdir | 新建动态目录 api/，保存接口文档
## [2026-05-27] recall | 加载了 index.md、engineering/architecture.md
## [2026-05-27] insight | 归档「Redux vs Zustand 对比分析」→ insights/
## [2026-05-27] import | ~/Desktop/old-project/.cc-memory-bank/
## [2026-05-27] lint | 发现 2 个问题，修复 1 个（index.md 补漏）
## [2026-05-27] raw | 存档「Memory Bank 功能讨论」→ raw/2026-05-27-14-30-memory-bank.md
```

格式固定为 `## [YYYY-MM-DD] <操作类型> | <一句话描述>`。操作类型包括：`init` / `save` / `mkdir` / `recall` / `insight` / `import` / `lint` / `raw`。

---

## 六、Markdown 文件模版

### 项目级 `overview/brief.md`
```markdown
# 项目概览 (Project Brief)

## 核心需求
- 简述项目的核心功能和解决的问题。

## 目标与愿景
- 产品的终极目标是什么。

## 相关文件
- [[engineering/architecture.md]] — 技术选型决策
- [[overview/product.md]] — 产品目标详述
```

### 项目级 `engineering/architecture.md`
```markdown
# 架构决策 (Architecture Decisions)

## YYYY-MM-DD: <决策标题>
- **背景**: 为什么需要做这个决定。
- **决定**: 最终选择了什么方案。
- **影响**: 对后续开发的影响。

## 相关文件
- [[engineering/tech.md]] — 具体技术栈
- [[overview/brief.md]] — 项目背景
```

### 项目级 `progress/progress.md`
```markdown
# 进度追踪 (Progress)

## 已完成 (Completed)
- [x] YYYY-MM-DD: <已完成的任务>

## 待做 (Todo)
- [ ] <待完成的任务>

## 相关文件
- [[progress/active.md]] — 当前工作焦点
```

### 全局 `preferences/coding.md`
```markdown
# 编码偏好 (Coding Preferences)

## 样式
- 首选 Vanilla CSS，除非明确要求使用 TailwindCSS。

## 代码风格
- JavaScript 保持模块化，避免过度封装。

## 交互习惯
- 保持回答简练，任务完成时自动归档进度。

## 相关文件
- [[conventions/general.md]] — 通用规范
```

---

## 七、增量更新策略 (CRITICAL)

**严禁整体覆盖文件。** 每次更新必须：
1. `Read` 目标文件，识别现有层级结构。
2. `Edit` 在精确位置插入新内容，保留所有历史记录。
3. 新建文件时用 `Write`，以对应模版初始化后再写入内容。
4. 每条新记录标注时间戳 `YYYY-MM-DD`。
5. 写完内容文件后，立即更新 `index.md` 对应行。
6. 新建目录后，在 `log.md` 追加 `mkdir` 操作记录。
