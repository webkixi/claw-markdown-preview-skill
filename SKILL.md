---
name: claw-markdown-preview
description: 预览 Markdown 文件 / 查看渲染效果 / 复制富文本到公众号。当用户说"预览这个 md"、"看看渲染效果"、"复制富文本到公众号"、"markdown 预览"、"我想看看这个文档长什么样"时自动触发。支持 22 种主题风格切换，可编辑、可双栏、可手机预览，复制按钮可将当前主题样式内联后写入剪贴板，直接粘贴到公众号编辑器即可。
version: 1.3.0
agent_created: true
metadata:
  openclaw:
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
# 杀掉上次残留的预览服务进程
PID_FILE=/tmp/claw-markdown-preview.pid
if [ -f "$PID_FILE" ]; then
  OLD_PID=$(cat "$PID_FILE")
  kill "$OLD_PID" 2>/dev/null || true
  rm -f "$PID_FILE"
fi
```

**第二步：启动预览服务（后台）**

```bash
python3 scripts/preview_server.py \
  --file "<MD_FILE_PATH>" \
  --no-open
```

> 使用 `--no-open` 禁用浏览器自动打开（由后续系统命令管理预览）。
> 上述命令在技能目录下执行；若在其他目录，需使用 preview_server.py 的绝对路径。

**第三步：记录 PID 并获取端口**

从打印日志中提取端口号：
```
== Markdown 预览服务已启动: http://127.0.0.1:<PORT> ==
```

然后记录 PID：
```bash
echo $! > /tmp/claw-markdown-preview.pid
```

**第四步：用系统默认浏览器打开预览**

直接用系统命令打开 URL（不要用 `present_files`，内置浏览器窗口太挤）：

```bash
# macOS
open "http://127.0.0.1:<PORT>"

# Linux
xdg-open "http://127.0.0.1:<PORT>"

# Windows
start "" "http://127.0.0.1:<PORT>"
```

## 可选参数

| 参数 | 说明 |
|------|------|
| `--file <路径>` | 预览指定 markdown 文件 |
| `--port <端口>` | 指定端口，默认 `8765` |
| `--stdin` | 从标准输入读取 markdown 内容 |
| `--no-open` | 不自动打开系统浏览器（WorkBuddy 场景必加） |
| `--verbose` | 输出访问日志，便于调试 |
| `--heartbeat-timeout <秒>` | 心跳超时秒数，页面关闭后超时自动停止服务（默认 `10`） |

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
- **热更新**：使用 `--file` 模式时，页面每 3 秒轮询文件内容变化，WorkBuddy 中修改文件后页面自动刷新（弹出"外部内容已更新"提示）
- **心跳自停**：页面每 5 秒发送心跳，关闭页面后服务自动停止（正常关闭立即停止，异常退出最多等待 `--heartbeat-timeout` 秒后停止）
- **关闭保存**：关闭页面时自动将编辑器内容写回 md 文件
- `--stdin` 模式不支持热更新和粘贴图片（内容固定）
- 服务终止方式：`kill $(cat /tmp/claw-markdown-preview.pid)`
