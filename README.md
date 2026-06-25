<!--
作者：算个文科生吧
联系方式(wx):RabbitRobot2025
Github:https://github.com/lijinghai
-->

# ljh_processon-mindmap-safe

这是一个用于 Codex 的 ProcessOn 思维导图 Skill。当前版本已同步 `processon-mindmap-generator` 1.1.10 的生成规则、结构参数、主题配置和客户端脚本，同时保留 Windows / PowerShell 中文内容的安全输入路径：先写入 UTF-8 Markdown 文件，再通过 `--markdown-file` 提交，避免中文被管道编码破坏成 `????`。

Skill 内部名称是 `ljh-processon-mindmap-safe`，文件夹名保留为 `ljh_processon-mindmap-safe`。

## 功能

- 将自然语言、Markdown、长文本、文档整理结果转换为 ProcessOn 在线可编辑脑图。
- 支持思维导图、逻辑图、组织结构图、鱼骨图、时间轴、树形图、树形表格。
- 支持官方 6 套主题：现代活力、复古单色、极简黑白、柔和雅韵、暗夜极光、浪漫治愈。
- 返回 Markdown 源内容、图片原始链接和在线编辑查看链接。
- Windows 中文内容默认走 UTF-8 文件输入，降低乱码风险。

## 目录结构

```text
ljh_processon-mindmap-safe/
├── SKILL.md
├── README.md
├── LICENSE
├── .gitignore
├── scripts/
│   ├── processon_mindmap_client.py
│   └── theme_presets.json
└── version/
    ├── coze-version.json
    └── github-version.json
```

## 使用方式

在 Codex 中可以直接说：

```text
使用 ljh-processon-mindmap-safe，帮我生成一张关于“多模态大模型是什么”的中文思维导图，要包含定义、核心能力、应用场景、技术难点和一句话总结。
```

Codex 会根据 `SKILL.md` 先生成完整 Markdown，再调用 `scripts/processon_mindmap_client.py` 提交到 ProcessOn，最后返回 Markdown、图片链接和在线编辑链接。

## 手动测试

Windows / PowerShell 中文安全调用方式：

```powershell
$path = Join-Path $env:TEMP "mindmap-input.md"
@(
  "# VLN 是什么"
  "## 定义"
  "- VLN 是 Vision-and-Language Navigation，即视觉与语言导航。"
  "## 关键能力"
  "- 语言理解"
  "- 视觉感知"
  "- 跨模态对齐"
  "- 路径规划"
) | Set-Content -LiteralPath $path -Encoding UTF8
python scripts/processon_mindmap_client.py --title "VLN 是什么" --theme "现代活力" --structure "mind_free" --markdown-file $path --cleanup-markdown-file
```

macOS / Linux 或纯 ASCII 内容也可以使用 stdin：

```bash
python3 scripts/processon_mindmap_client.py --title "Title" --theme "极简黑白" --structure "mind_free" --markdown - <<'EOF'
# Title
## Node
EOF
```

## 支持的结构参数

| 参数 | 说明 | 适合场景 |
| --- | --- | --- |
| `mind_free` | 思维导图 / 中心放射 | 概念解释、知识框架、总结归纳、头脑风暴 |
| `mind_right` | 向右逻辑图 | 提纲、步骤、汇报框架、执行计划 |
| `mind_org` | 组织结构图 | 部门、岗位、层级、汇报关系 |
| `mind_ishikawa_left` | 鱼骨图 | 问题原因分析、复盘归因 |
| `mind_timeline_h` | 时间轴 | 阶段规划、里程碑、事件演进 |
| `mind_tree_free` | 树形图 | 项目任务拆解、WBS |
| `mind_treeTable_left_title` | 树形表格 | 参数清单、分类汇总、多方案对比 |

## 注意事项

- 本 Skill 会联网调用 ProcessOn 的 Markdown 转思维导图接口。
- 内容会提交到 ProcessOn 在线服务，请避免上传不适合外发的敏感资料。
- 生成的在线编辑链接由 ProcessOn 返回，请妥善保存。
- 在 Windows 上处理中文时，请始终优先使用 UTF-8 Markdown 文件和 `--markdown-file`。