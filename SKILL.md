---
name: claw-markdown-preview
description: Markdown 预览——用户说出"预览这个 md"、"看看渲染效果"、"打开预览"、"复制富文本到公众号"、"markdown 预览"等时触发技能。
version: 1.4.4
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

# Markdown 编辑、多风格预览

通过浏览器编辑、预览AI生成的 Markdown 内容、用户本地 MarkDown 文件。支持多主题风格切换、实时编辑、富文本复制（可粘贴到公众号/知乎）、图片粘贴

## 触发方式

当用户说出"预览这个 md""看看渲染效果""打开预览""markdown 预览""预览一下"等时触发。

## 执行入口

**前提检查：**
- 确认目标文件路径存在且为 `.md` 文件
- 确认 `scripts/preview_server.py` 存在且可用（路径相对于技能目录）

**执行（一条命令，替换 `<MD_FILE_PATH>` 后直接运行，无需分步或手动提取端口）：**

```bash
pkill -f preview_server.py 2>/dev/null; rm -f /tmp/claw-markdown-preview.pid
nohup python3 scripts/preview_server.py --file "<MD_FILE_PATH>" --no-open --heartbeat-timeout 30 > /tmp/claw-md-preview.log 2>&1 &
sleep 2
if curl -s -o /dev/null -w '%{http_code}' --max-time 2 http://127.0.0.1:8765/ | grep -q 200; then
  PORT=8765
else
  PORT=$(grep -oE '127\.0\.0\.1:[0-9]+' /tmp/claw-md-preview.log | head -1 | sed 's/.*://')
fi
lsof -iTCP:$PORT -sTCP:LISTEN -t | head -1 > /tmp/claw-markdown-preview.pid
open "http://127.0.0.1:$PORT"
```

**说明：**
- 这条命令自动完成：清理残留进程 → 启动服务（nohup 后台常驻，心跳 30 秒纯为兜底）→ 等待 2 秒就绪 → `curl` 快速探测默认端口 8765 → 被占用时从日志精确提取端口 → 记录 PID → 打开浏览器。
- Agent 无需读输出、无需手动提取端口、无需分步返回。一条命令 5 秒走完。
- 首次运行用默认端口 8765，`curl` 直通不进正则分支；仅端口冲突时才走日志提取降级路径。
- 心跳 30 秒纯为兜底——正常流程碰不到超时。

> **不要用 `present_files`**，内置浏览器窗口太挤，用系统命令 `open` 打开外部浏览器。

## 可选参数

| 参数 | 说明 |
|------|------|
| `--file <路径>` | 预览指定 markdown 文件 |
| `--port <端口>` | 指定端口，默认 `8765` |
| `--stdin` | 从标准输入读取 markdown 内容 |
| `--no-open` | 不自动打开系统浏览器（后台/Agent 场景必加） |
| `--verbose` | 输出访问日志，便于调试 |
| `--heartbeat-timeout <秒>` | 心跳超时秒数，页面关闭后超时自动停止服务（默认 `10`，Agent 场景建议 `30`） |

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

## 后台运行说明

执行入口的一条命令已自动处理心跳超时（30s 兜底）/ 输出缓冲（nohup 重定向）/ PID 获取（lsof）/ 残留清理（pkill）。若需手动启动，参考上述命令结构，务必使用 `python3`（与 frontmatter `requires: python3` 一致）而非绝对路径。
