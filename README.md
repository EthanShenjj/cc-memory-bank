# Memory Bank Plugin

一个类 Cline 风格的零依赖、纯 Markdown 文件的跨会话记忆系统。记忆是**持续累积的复利资产**，每次保存都让下一次检索更精准。

## ✨ 特性

- **零依赖**: 无需外部 MCP 服务器或数据库，直接使用 AI 代理内置的本地文件读写工具。
- **项目级隔离**: 每个项目独立的 `.cc-memory-bank/` 文件夹，包含项目背景、架构决策、开发进度等。
- **全局同步**: 通用的 `~/.claude/cc-memory-bank/` 文件夹，跨项目沉淀你的编码偏好与通用规范。
- **index 导航**: 每个记忆目录维护一个 `index.md` 目录，recall 时优先读 index 再按需精准加载，不盲目读全部文件。
- **log 时间线**: `log.md` 追加式记录每次保存、回忆、导入、lint 操作，可用 `grep` 快速过滤历史。
- **快捷保存**: 聊天发送 `ss` 即可自动提炼近期对话要点，增量写入对应文件并同步 index 和 log。
- **答案回写**: 输入 `/file-insight: <标题>` 将重要分析结论归档为独立文件，复利积累洞察。
- **健康检查**: 输入 `/lint-memory` 自动扫描矛盾、过期、孤岛、空洞条目并给出修复建议。
- **跨项目导入**: 输入 `/ss-memory-bank: <路径>` 将任意项目的记忆一键合并到全局 Memory Bank。
- **自动归档**: 复杂任务完成后，AI 自动将进度从"待办"移至"已完成"并更新 active.md。
- **Git 友好**: 纯文本 Markdown，可随代码提交共享或加入 `.gitignore` 仅本地保存。

---

## 📂 存储目录

### 1. 全局记忆 (跨项目通用)
存储在 `~/.claude/cc-memory-bank/`

| 文件 | 用途 |
|------|------|
| `preferences.md` | 开发偏好、UI 审美、代码注释习惯 |
| `conventions.md` | 通用规范、接口标准、安全规则 |
| `decisions.md` | 跨项目重要决策与方向沉淀 |
| `index.md` | **检索入口** — 所有文件的摘要目录 |
| `log.md` | **操作时间线** — 追加式操作记录 |

### 2. 项目记忆 (当前项目独立)
存储在 `<project-root>/.cc-memory-bank/`

| 文件 | 用途 |
|------|------|
| `brief.md` | 项目核心诉求与概览 |
| `product.md` | 产品目标与用户体验追求 |
| `architecture.md` | 选型及架构设计决策 |
| `tech.md` | 技术栈、核心依赖与编译部署指令 |
| `progress.md` | 整体任务进度（Completed & Todo） |
| `active.md` | 当前工作焦点与近期变更 |
| `insights/` | 归档的分析结论、对比报告 |
| `index.md` | **检索入口** — 所有文件的摘要目录 |
| `log.md` | **操作时间线** — 追加式操作记录 |

---

## 🚀 触发指令

| 指令 | 行为 |
|------|------|
| **`ss`** | **快捷保存** — 提炼近期对话要点，增量写入 + 更新 index + 追加 log |
| **`update memory bank`** / **`保存记忆`** | **完整保存** — 全套文件同步整理，重建 index |
| **`save to global`** / **`保存全局记忆`** | **强制全局** — 将偏好或规范写入全局记忆 |
| **`recall`** / **`回忆`** | **加载记忆** — 先读 index，再按需精准加载相关文件 |
| **`/file-insight: <标题>`** | **答案回写** — 将当前分析结论归档至 `insights/` |
| **`/lint-memory`** | **健康检查** — 扫描矛盾、过期、孤岛、空洞条目 |
| **`/ss-memory-bank: <项目路径>`** | **跨项目导入** — 合并指定项目记忆到全局 |

---

## ⌨️ 进阶：配置 VS Code 快捷键

通过修改 VS Code 的 `keybindings.json`，实现"按一下快捷键，自动向聊天发送 `ss`"的丝滑体验：

1. 按 `Cmd+Shift+P` (Mac) 输入 `Open Keyboard Shortcuts (JSON)`。
2. 加入以下配置：

```json
{
  "key": "cmd+shift+m",
  "command": "workbench.action.chat.send",
  "args": {
    "text": "ss"
  }
}
```

配置后，按下 `Cmd+Shift+M` 即可快捷触发记忆保存（发送 `ss`）！
