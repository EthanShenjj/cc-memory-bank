# Memory Bank Plugin

一个类 Cline 风格的零依赖、纯 Markdown 文件的跨会话记忆系统。记忆是**持续累积的复利资产**，每次保存都让下一次检索更精准。

## ✨ 特性

- **零依赖**: 无需外部 MCP 服务器或数据库，直接使用 AI 代理内置的本地文件读写工具。
- **项目级隔离**: 每个项目独立的 `.cc-memory-bank/` 文件夹，包含项目背景、架构决策、开发进度等。
- **全局同步**: 通用的 `~/.claude/cc-memory-bank/` 文件夹，跨项目沉淀你的编码偏好与通用规范。
- **index 导航**: 每个记忆目录维护一个 `index.md` 目录，recall 时优先读 index 再按需精准加载，不盲目读全部文件。
- **log 时间线**: `log.md` 追加式记录每次保存、回忆、导入、lint 操作，可用 `grep` 快速过滤历史。
- **快捷保存**: 聊天输入快捷输入 `/cc` 或 `cc` 即可自动提炼近期对话要点，增量写入对应文件并同步 index 和 log。
- **答案回写**: 输入 `/file-insight: <标题>` 将重要分析结论归档为独立文件，复利积累洞察。
- **健康检查**: 输入 `/lint-memory` 自动扫描矛盾、过期、孤岛、空洞条目并给出修复建议。
- **跨项目导入**: 输入 `/cc-memory-bank: <路径>` 将任意项目的记忆一键合并到全局 Memory Bank。
- **自动归档**: 复杂任务完成后，AI 自动将进度从"待办"移至"已完成"并更新 active.md。
- **Git 友好**: 纯文本 Markdown，可随代码提交共享或加入 `.gitignore` 仅本地保存。

---

## 📂 存储目录

记忆以**目录分类**组织。安装后首次使用时自动初始化预置目录；遇到不属于现有目录的内容时，自动创建新目录。

### 1. 全局记忆 (跨项目通用)
存储在 `~/.claude/cc-memory-bank/`

```
~/.claude/cc-memory-bank/
├── index.md          # 检索入口
├── log.md            # 操作时间线
├── preferences/      # 开发偏好、UI 审美、代码习惯
├── conventions/      # 通用规范、接口标准、安全规则
└── decisions/        # 跨项目重要决策与方向沉淀
```

### 2. 项目记忆 (当前项目独立)
存储在 `<project-root>/.cc-memory-bank/`

```
.cc-memory-bank/
├── index.md          # 检索入口
├── log.md            # 操作时间线
├── overview/         # 项目概览：brief、product 目标
├── engineering/      # 技术工程：architecture、tech 栈
├── progress/         # 进度追踪：progress、active 焦点
└── insights/         # 分析洞察：YYYY-MM-DD-<标题>.md
```

### 3. 动态目录扩展

当内容不属于任何预置目录时，AI 会自动推断并创建新目录：

| 内容类型 | 自动创建的目录 |
|----------|---------------|
| API 接口文档 | `api/` |
| 安全策略、漏洞记录 | `security/` |
| 团队流程、协作规范 | `team/` |
| 运营数据、业务指标 | `metrics/` |
| 用户研究、访谈记录 | `research/` |
| 部署配置、环境说明 | `devops/` |

---

## 🚀 触发指令

| 指令 | 行为 |
|------|------|
| **`/cc`** 或 **`cc`** | **快捷输入** — 提炼近期对话要点，增量写入 + 更新 index + 追加 log（注意：是快捷输入 `/cc` 或 `cc`，而非快捷键 `ss`） |
| **`update memory bank`** / **`保存记忆`** | **完整保存** — 全套文件同步整理，重建 index |
| **`save to global`** / **`保存全局记忆`** | **强制全局** — 将偏好或规范写入全局记忆 |
| **`recall`** / **`回忆`** | **加载记忆** — 先读 index，再按需精准加载相关文件 |
| **`/file-insight: <标题>`** | **答案回写** — 将当前分析结论归档至 `insights/` |
| **`/lint-memory`** | **健康检查** — 扫描矛盾、过期、孤岛、空洞条目 |
| **`/cc-memory-bank: <项目路径>`** | **跨项目导入** — 合并指定项目记忆到全局 |

---

## ⌨️ 进阶：配置 VS Code 快捷键

通过修改 VS Code 的 `keybindings.json`，实现"按一下快捷键，自动向聊天发送 `/cc` 或 `cc`"的丝滑体验（若你在终端命令行使用且输入 `/cc` 会触发插件自动补全，可直接使用不带斜杠的 `cc` 作为快捷输入）：

1. 按 `Cmd+Shift+P` (Mac) 输入 `Open Keyboard Shortcuts (JSON)`。
2. 加入以下配置（以发送 `/cc` 为例，也可配置为 `cc`）：

```json
{
  "key": "cmd+shift+m",
  "command": "workbench.action.chat.send",
  "args": {
    "text": "/cc"
  }
}
```

配置后，按下 `Cmd+Shift+M` 即可快捷触发记忆保存（发送快捷输入 `/cc` 或 `cc`）！
