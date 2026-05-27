# Memory Bank Plugin for Antigravity IDE

一个类 Cline 风格的零依赖、纯 Markdown 文件的跨会话记忆系统。

## ✨ 特性

- **零依赖**: 无需外部 MCP 服务器或数据库，直接使用 AI 代理内置的本地文件读写工具。
- **项目级隔离**: 每个项目独立的 `.memory-bank/` 文件夹，包含项目背景、架构决策、开发进度等。
- **全局同步**: 通用的 `~/.claude/cc-memory-bank/` 文件夹，跨项目沉淀你的编码偏好与通用规范。
- **快捷保存**: 聊天发送 `cc` 回车即可自动提炼近期对话要点并增量存入对应的记忆文件。
- **自动归档**: 在复杂任务完成或 walkthrough.md/task.md 标记完成后，AI 自动归档并整理进度。
- **Git 友好**: 项目记忆是纯文本的 Markdown 文件，可以直接随代码提交（便于团队共享）或加入 `.gitignore`（仅本地保存）。

---

## 📂 存储目录

### 1. 全局记忆 (跨项目通用)
存储在 `~/.claude/cc-memory-bank/`
- `preferences.md`: 你的开发偏好、UI 审美追求、代码注释偏好等。
- `conventions.md`: 通用规范、接口定义标准、安全规则等。
- `decisions.md`: 跨项目的重要大方向思考沉淀。

### 2. 项目记忆 (当前项目独立)
存储在 `<project-root>/.memory-bank/`
- `brief.md`: 项目核心诉求与概览。
- `product.md`: 产品目标与用户体验追求。
- `architecture.md`: 选型及架构设计决策。
- `tech.md`: 技术栈、核心依赖与编译部署指令。
- `progress.md`: 整体任务进度追踪（Completed & Todo）。
- `active.md`: 当前工作焦点与近期变更。

---

## 🚀 触发指令

| 指令 (聊天框输入) | 对应行为 |
|------------------|----------|
| **`cc`** | **快捷保存**: 自动审视最近对话，提炼要点增量写入文件，并给出简短反馈。 |
| **`update memory bank`** / **`保存记忆`** | **完整保存**: 整理全套记忆文件，支持更长、更深入的记忆重构。 |
| **`save to global`** / **`保存全局记忆`** | **强制全局**: 将当前偏好或规范保存到全局记忆文件中。 |
| **`recall`** / **`回忆`** | **加载记忆**: AI 重新读取项目与全局记忆，隐式恢复上下文。 |

---

## ⌨️ 进阶：配置真正的 VS Code 快捷键

虽然插件无法直接绑定系统级键盘快捷键，但你可以通过修改 VS Code 的 `keybindings.json`，实现“按一下快捷键，自动向聊天发送 `cc`”的丝滑体验！

1. 打开 VS Code，按 `Cmd+Shift+P` (Mac) 输入 `Open Keyboard Shortcuts (JSON)`。
2. 在数组中加入以下配置：

```json
{
  "key": "cmd+shift+m",
  "command": "workbench.action.chat.send",
  "args": {
    "text": "cc"
  }
}
```

配置后，你只需在聊天框内或编辑器中按下 `Cmd+Shift+M`，即可快捷触发记忆保存！
