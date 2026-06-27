# Claude Code Sessions 管理器

一个**零依赖、零安装**的单文件网页工具，用来浏览、阅读、重命名和删除 Claude Code 在本地保存的会话历史（`~/.claude/projects/` 下的 JSONL 文件）。

打开 `ClaudeSessionViewer` 即可使用，所有数据在本地读写，不联网，不上传。


---

## 快速开始

1. 用 **Chrome 或 Edge**（86+）打开 [`ClaudeSessionViewer`](ClaudeSessionViewer)
   - 双击文件，或拖进浏览器窗口
2. 欢迎页会提示操作路径，点击右上角 **「选择 .claude 目录」**
3. 在弹出的目录选择框里找到并选择你的 `.claude` 文件夹
   - Windows：`C:\Users\<用户名>\.claude`
   - macOS / Linux：`~/.claude`
4. 浏览器询问读写权限时点击「允许」即可

> ⚠️ 仅支持 **Chromium 内核浏览器**（Chrome / Edge / Arc 等），依赖 [File System Access API](https://developer.mozilla.org/docs/Web/API/File_System_Access_API)。Firefox / Safari 暂不支持。

---

## 功能

| 功能 | 说明 |
|------|------|
| **项目列表** | 左栏自动列出所有项目，名称从会话内的 `cwd` 字段解析为可读目录名 |
| **会话列表** | 中栏显示会话标题、最后修改时间、对话轮数、文件大小 |
| **对话查看** | 右栏以气泡形式还原完整对话，支持 Markdown 渲染和代码高亮 |
| **思考 / 工具折叠** | `thinking` 块、工具调用、工具结果默认折叠，点击展开，保持对话界面干净 |
| **Token 统计** | 每个会话底部显示输入 / 输出 / 缓存 token 用量，一眼看清消耗 |
| **全文搜索** | 顶部搜索框跨所有项目搜索对话内容，命中关键词高亮显示 |
| **重命名** | 修改 JSONL 文件名（不改动文件内容，`sessionId` 不受影响） |
| **回收站删除** | 删除先进回收站，可随时**还原**或**彻底删除** |
| **导出 Markdown** | 一键把当前会话导出为 `.md` 文件，便于存档或分享 |
| **活跃标记** | 正在运行的会话标 `● 活跃`，子代理会话标 `Agent` |

---

## 回收站机制

浏览器沙箱**无法访问操作系统的回收站**，本工具内置软删除机制：

- 删除时，文件被移动到 `~/.claude/projects/.session-manager-trash/`
- `_manifest.json` 记录原项目、原文件名、删除时间
- 在右上角「回收站」里可以：
  - **还原** → 文件放回原项目（若原位置已有同名文件，自动加 `-restored` 后缀）
  - **彻底删除** / **清空回收站** → 从磁盘移除，不可恢复

删除是两步操作，由你最终决定是否真正清除。

---

## 数据说明

工具读取的是 Claude Code 自己生成的文件，结构如下：

```
~/.claude/
├── projects/
│   ├── <编码后的项目路径>/
│   │   ├── <sessionId>.jsonl      # 一次会话的完整记录（每行一个 JSON）
│   │   └── agent-<hash>.jsonl     # 子代理会话
│   └── .session-manager-trash/    # 本工具回收站（已删除文件 + manifest）
└── sessions/
    └── <pid>.json                 # 活跃会话元数据（用于标记「活跃」）
```

JSONL 每行是一条记录，常见 `type`：`user` / `assistant` / `ai-title` / `tool_use` / `tool_result` / `thinking` 等。本工具只渲染对话相关记录，内部队列、快照、模式切换等记录会被跳过。

---

## 隐私

- 纯前端，**无后端、无网络请求**，不收集任何数据
- 所有读写通过浏览器本地文件授权完成，关闭页面即结束授权
- 源码只有一个 `ClaudeSessionViewer`，可自行审阅

---

## 常见问题

**Q：点了「选择 .claude 目录」没反应？**
A：确认使用的是 Chrome / Edge，且文件通过 `file://` 或 `http(s)://` 打开。部分企业安全策略会禁用 File System Access API。

**Q：会话标题显示成一长串 UUID？**
A：该会话未生成 `ai-title`，且无可用的首条用户消息，回退到文件名显示。可用「重命名」功能手动命名。

**Q：重命名会影响 Claude Code 读取吗？**
A：不会。Claude Code 用文件内部的 `sessionId` 识别会话，文件名只是容器，改名无影响。

**Q：删除的文件去哪了？**
A：移入 `~/.claude/projects/.session-manager-trash/`，可在右上角「回收站」还原或彻底删除。

**Q：支持同时打开多个项目的会话吗？**
A：全文搜索可跨所有项目检索，但对话查看每次只显示一条会话。

---

## 技术栈

- 单文件 `ClaudeSessionViewer`：内联 HTML + CSS + 原生 JS，无任何框架或依赖
- [File System Access API](https://developer.mozilla.org/docs/Web/API/File_System_Access_API) 实现本地文件读写
- 配色沿用 Claude 设计语言（暖米色底 + 陶土橙点缀），欢迎页采用渐变背景与卡片式引导布局
