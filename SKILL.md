---
name: claw-markdown-preview
description: 本工具是一个 Agent 技能（Skill），可被 AI 助手在对话中直接调用。让 AI 生成的 Markdown 立刻「看得见」——在 AI 助手对话里说一句"预览这个 md""看看渲染效果"，即可把生成的文档在浏览器渲染成精美网页，不切应用、不手动开文件。四大能力：① 即时预览（支持 WorkBuddy / QClaw / OpenClaw / Hermes 等平台）：AI 写完文档当场看排版、代码高亮、表格效果；② 公众号排版：22 种主题 + 4 种视图（编辑/预览/双栏/手机）随意切换，排版效果所见即所得，告别手动调格式；③ 富文本复制（含图片）：一键拷贝带样式正文，图片自动 base64 内联，直粘公众号/知乎后台，零二次排版；④ 实时编辑 + 图片粘贴：双栏边写边看，编辑器内 Ctrl+V 即贴本地图片，自动内联进文档，无需手动存图传图。写技术文档、做公众号、整教程，一个技能全包。
version: 1.4.0
agent_created: true
metadata:
  openclaw:
    emoji: "📄"
    os: ["darwin", "linux", "win32"]
    requires:
      bins: ["python3"]
    install: []
  workbuddy:
    emoji: "📄"
    os: ["darwin", "linux", "win32"]
    requires:
      bins: ["python3"]
    install: []
  qclaw:
    emoji: "📄"
    os: ["darwin", "linux", "win32"]
    requires:
      bins: ["python3"]
    install: []
  hermes:
    emoji: "📄"
    os: ["darwin", "linux", "win32"]
    requires:
      bins: ["python3"]
    install: []
---

# Markdown 预览 + 富文本复制

启动本地 HTTP 服务，在浏览器中打开 Markdown 预览页面。支持四种视图模式和多风格主题切换。

## 执行入口

**前提检查：**
- 确认目标文件路径存在且为 `.md` 文件
- 确认 `scripts/preview_server.py` 存在且可用（路径相对于技能目录）

**第一步：进程清理（避免端口冲突）**

```bash
# 1) 杀掉上次残留的预览服务进程（通过 PID 文件）
PID_FILE=/tmp/claw-markdown-preview.pid
if [ -f "$PID_FILE" ]; then
  OLD_PID=$(cat "$PID_FILE")
  kill "$OLD_PID" 2>/dev/null || true
  rm -f "$PID_FILE"
fi

# 2) 兜底：杀掉所有残留的 preview_server.py 进程（可能来自其他技能目录）
pkill -f "preview_server.py" 2>/dev/null || true
```

**第二步：启动预览服务（后台运行，不要阻塞等待）**

```bash
PYTHONUNBUFFERED=1 python3 \
  scripts/preview_server.py \
  --file "<MD_FILE_PATH>" \
  --no-open \
  --heartbeat-timeout 60
```

关键说明：
- **`PYTHONUNBUFFERED=1`**：禁用 stdout 缓冲，确保启动日志立即可读（后台运行时 Python 默认缓冲 stdout，不加此变量会拿不到端口输出）
- **`--heartbeat-timeout 60`**：后台/Agent 场景必加，默认 10 秒太短 —— 后台启动后 agent 需要读输出、检测端口、开浏览器，整个流程超过 10 秒会导致服务还没打开浏览器就自动退出
- **`--no-open`**：禁用浏览器自动打开（由后续系统命令管理预览）
- 上述命令需在技能目录下执行（`cd` 到技能目录）；若在其他目录，需使用 preview_server.py 的绝对路径
- **必须在后台运行**（不要阻塞等待命令返回），否则会卡住

**第三步：获取端口（两种方式，优先方式 A）**

方式 A — 从后台任务输出提取（PYTHONUNBUFFERED=1 后通常 2-3 秒内可见）：
```
== Markdown 预览服务已启动: http://127.0.0.1:<PORT> ==
```

方式 B — 如果输出未及时出现，用 lsof 直接检测监听端口：
```bash
lsof -iTCP -sTCP:LISTEN -P -n 2>/dev/null | grep -i python
# 或直接测试默认端口 8765
curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:8765/
# 返回 200 即为服务端口
```

**第四步：记录 PID 并用浏览器打开预览**

```bash
# 用 lsof 精确获取 PID（不要用 echo $!，后台模式下拿不到正确值）
PID=$(lsof -iTCP:<PORT> -sTCP:LISTEN -P -n -t 2>/dev/null | head -1)
echo "$PID" > /tmp/claw-markdown-preview.pid

# macOS
open "http://127.0.0.1:<PORT>"

# Linux
xdg-open "http://127.0.0.1:<PORT>"

# Windows
start "" "http://127.0.0.1:<PORT>"
```

> **不要用 `present_files`**，内置浏览器窗口太挤，用系统命令 `open` 打开外部浏览器。

## 可选参数

| 参数 | 说明 |
|------|------|
| `--file <路径>` | 预览指定 markdown 文件 |
| `--port <端口>` | 指定端口，默认 `8765` |
| `--stdin` | 从标准输入读取 markdown 内容 |
| `--no-open` | 不自动打开系统浏览器（后台/Agent 场景必加） |
| `--verbose` | 输出访问日志，便于调试 |
| `--heartbeat-timeout <秒>` | 心跳超时秒数，页面关闭后超时自动停止服务（默认 `10`，后台/Agent 场景建议 `60`） |

如果没有指定 `--file` 或 `--stdin`，服务启动后页面显示空编辑器，用户可自行粘贴 markdown。

## 功能说明

1. **编辑模式**：CodeMirror 编辑器，编辑 Markdown 原文
2. **预览模式**：渲染后的富文本预览，支持主题切换
3. **双栏模式**：编辑器和预览左右并排，支持滚动同步
4. **手机预览**：以手机宽度（500px）居中渲染预览

## 主题

22 种预览主题 + 9 种代码高亮主题，可在页面顶部下拉选择。

## 富文本复制

点击预览区的复制按钮，将当前主题样式内联后的 HTML 写入剪贴板，可直接粘贴到公众号编辑器等平台。复制时本地图片会自动转换为 base64 内联，公众号粘贴后自动识别。

## 粘贴图片

在编辑模式或双栏模式下，可直接粘贴剪贴板中的图片（Ctrl+V / Cmd+V）。图片会自动保存到 md 文件同目录的 `images/` 子目录，并在编辑器中插入标准 Markdown 图片语法 `![](images/img-xxx.png)`。

- 单张图片大小限制 5MB
- 图片以文件形式存储，跟随 md 文件，换浏览器/电脑不丢失
- 关闭页面时自动保存编辑器内容回 md 文件

## 注意事项

- 预览服务运行在 `assets/` 子目录上，文件由 `preview_server.py` 自动定位
- 端口默认 8765，被占用时自动递增寻找空闲端口（+20 以内）
- **热更新**：使用 `--file` 模式时，页面每 3 秒轮询文件内容变化，在 Agent 中修改文件后页面自动刷新（弹出"外部内容已更新"提示）
- **心跳自停**：页面每 5 秒发送心跳，关闭页面后服务自动停止（正常关闭立即停止，异常退出最多等待 `--heartbeat-timeout` 秒后停止）
- **关闭保存**：关闭页面时自动将编辑器内容写回 md 文件
- `--stdin` 模式不支持热更新和粘贴图片（内容固定）
- 服务终止方式：`kill $(cat /tmp/claw-markdown-preview.pid)`

## 后台运行与跨平台注意事项

以下问题在实际使用中高频出现，务必注意：

1. **心跳超时必须调大**：默认 `--heartbeat-timeout 10` 在后台模式下太短。Agent 从启动服务到打开浏览器需要多步操作（读输出→检测端口→记录 PID→执行 open），全程超过 10 秒会导致服务自动退出。**必须加 `--heartbeat-timeout 60`**。
2. **stdout 缓冲**：后台运行时 Python 默认缓冲 stdout，导致启动日志（含端口号）不可读。**必须加 `PYTHONUNBUFFERED=1` 环境变量**。
3. **PID 获取**：`echo $!` 在后台运行模式下不可靠。用 `lsof -iTCP:<PORT> -sTCP:LISTEN -t` 获取准确 PID。
4. **残留进程**：清理时除 PID 文件外，还要 `pkill -f "preview_server.py"` 兜底，防止其他技能目录（如 `.trae/skills/`）的旧进程占用端口。
5. **Python 路径**：使用 `python3`（frontmatter 已声明 `requires: python3`）。若某 Agent 平台提供隔离的 managed Python，可替换为该平台对应的 python 路径。
