---
# 作者：算个文科生吧
# 联系方式(wx):RabbitRobot2025
# Github:https://github.com/lijinghai
name: ljh-processon-mindmap-safe
description: Generate ProcessOn online mind maps from Markdown or natural-language content, especially on Windows or Chinese-language machines where PowerShell/stdin encoding can garble Chinese text. Use when the user asks Codex to create, redraw, modify, or test a ProcessOn mind map/brain map/思维导图/脑图 and wants complete image/edit links without乱码.
---

# LJH ProcessOn Mindmap Safe

## Purpose

Create ProcessOn mind maps while preserving Chinese text. The safe path is: build Markdown, save it as a UTF-8 file, then call the bundled ProcessOn client with `--markdown-file`. Avoid piping Chinese Markdown through Windows PowerShell stdin.

## Non-Negotiable Encoding Rule

On Windows, do not use this pattern for Chinese content:

```powershell
@'
# 中文标题
'@ | python scripts/processon_mindmap_client.py --markdown -
```

PowerShell may encode the pipe in a legacy code page, and ProcessOn may receive `????` instead of Chinese. Always write a UTF-8 Markdown file first.

## Workflow

1. Draft the full mind-map content as clean Markdown.
2. Save it to a temporary UTF-8 `.md` file under `.agents/cache/`, `work/`, or the system temp directory.
3. Run `scripts/processon_mindmap_client.py` with `--markdown-file <file>`.
4. If the temp file is disposable, add `--cleanup-markdown-file`.
5. Return the Markdown plus the complete raw image link and complete edit/view link from `data.copyBlock`.

## UTF-8 File Creation

Use a file-writing method that explicitly writes UTF-8. In Codex desktop, Node REPL is reliable:

```javascript
const fs = await import('node:fs');
fs.writeFileSync('work/mindmap.md', markdown, { encoding: 'utf8' });
```

Python is also fine when available:

```python
from pathlib import Path
Path('work/mindmap.md').write_text(markdown, encoding='utf-8')
```

## Command

From the skill folder:

```powershell
python scripts/processon_mindmap_client.py --title "VLN 是什么" --theme "现代活力" --structure "mind_free" --markdown-file "work/mindmap.md"
```

If Python is not on PATH, use the bundled Python executable available in the workspace or any Python 3.8+ interpreter.

## Structure Selection

Use these ProcessOn structures exactly:

- `mind_free`: default mind map for concepts, knowledge frameworks, summaries, brainstorming.
- `mind_right`: rightward logic chart for outlines, steps, reports, execution plans.
- `mind_org`: organization chart for hierarchy, departments, roles, reporting lines.
- `mind_ishikawa_left`: fishbone chart for root-cause analysis.
- `mind_timeline_h`: timeline for stages, milestones, event evolution.
- `mind_tree_free`: tree chart for WBS/task breakdown.
- `mind_treeTable_left_title`: tree table for comparisons, parameters, grouped data.

If unsure, use `mind_free`.

## Theme Selection

Use one of these theme names. The script maps names to ProcessOn theme JSON:

- `现代活力`: colorful, clear, energetic.
- `极简黑白`: professional, restrained.
- `柔和雅韵`: soft and readable.
- `暗夜极光`: dark, high-contrast.
- `浪漫治愈`: pink and gentle.
- `复古单色`: subdued purple monochrome.

## Output Requirements

After the script returns JSON:

- Show the Markdown in a fenced `markdown` code block.
- Output `data.copyBlock` exactly when present.
- Do not shorten, wrap, or relabel the raw URLs; keep all query parameters.
- If generation fails because of network sandboxing, rerun the same command with approval/escalation rather than changing the encoding path.

## Script Notes

The bundled `scripts/processon_mindmap_client.py` is self-contained and uses only Python standard-library modules. It calls ProcessOn's Markdown transform API and reads Markdown files with `encoding='utf-8'`.
