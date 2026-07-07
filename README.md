# claw-markdown-preview

> 一个 **Agent 技能（Skill）**：在 AI 对话里即时预览 Markdown，让 AI 生成的文档立刻「看得见」。

## 这是什么

`claw-markdown-preview` 是一个可被 AI 助手（Agent）在对话中**直接调用**的技能。当你让 Agent 生成 Markdown 文档后，只需说一句"预览这个 md""看看渲染效果"，它就会在浏览器里把文档渲染成精美网页——支持 22 种主题、4 种视图，不切应用、不手动开文件。

⚠️ **声明**：本工具为社区开源作品，非任何 AI 平台的官方出品，按原样提供、风险自负。

## 支持的 Agent 平台

- WorkBuddy
- QClaw
- OpenClaw
- Hermes
- 任何能提供「执行 `python3` 命令 + 访问 localhost 浏览器预览」能力的 Agent

## 四大能力

1. **即时预览**：AI 写完文档当场看排版、代码高亮、表格效果。
2. **公众号排版**：22 种主题 + 4 种视图（编辑 / 预览 / 双栏 / 手机）随意切换，排版效果所见即所得，告别手动调格式。
3. **富文本复制（含图片）**：一键拷贝带样式正文，图片自动 base64 内联，直粘公众号 / 知乎后台，零二次排版。
4. **实时编辑 + 图片粘贴**：双栏边写边看，编辑器内 `Ctrl+V` 即贴本地图片，自动内联进文档，无需手动存图传图。

## 安装

将本仓库克隆 / 下载到你的 Agent 技能目录即可（如 WorkBuddy 的 `~/.workbuddy/skills/`），或直接从 SkillHub / ClawHub 安装。

```bash
git clone https://github.com/webkixi/claw-markdown-preview-skill.git
```

## 用法

在 Agent 对话中：

- "预览这个 md" → 启动本地预览服务并在浏览器打开
- "看看渲染效果" → 同上
- "打开预览" → 同上

技能会自动完成：清理残留进程 → 后台启动预览服务（默认端口 `8765`）→ 用系统浏览器打开预览页。

## 技术说明

- 预览服务 `scripts/preview_server.py` 仅依赖 Python 标准库，跨平台（macOS / Linux / Windows）。
- 前端 `assets/` 为纯静态资源，提供 22 种预览主题 + 9 种代码高亮主题。
- 所有文件均为本地处理，不上传任何内容。

## 许可

MIT
