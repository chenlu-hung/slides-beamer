# slides-beamer

A Claude Code plugin for generating LaTeX Beamer presentations using the [**keynote theme**](https://github.com/chenlu-hung/keynote-theme). Supports a two-phase workflow: discuss and save a structured outline, then convert the outline to a compilable XeLaTeX `.tex` file.

## Requirements

- [Claude Code](https://claude.ai/claude-code)
- XeLaTeX (e.g. via [MacTeX](https://www.tug.org/mactex/))
- keynote Beamer theme installed at `~/Library/texmf/tex/latex/beamerthemekeynote/`

## Installation

Copy the plugin to your global Claude Code plugins directory:

```bash
cp -r . ~/.claude/plugins/slides-beamer
```

Restart Claude Code — the plugin is auto-discovered from `~/.claude/plugins/`.

## Usage

### Phase 1 — Create an Outline

Trigger with any of these phrases (English or Chinese):

> "create slides", "make presentation", "generate beamer", "outline a talk", "投影片", "簡報", "幫我做投影片"

Claude will ask about your topic, audience, and structure, then propose a markdown outline. After you approve it, the outline is saved as `outline.md`.

### Phase 2 — Generate the `.tex` File

> "根據 outline.md 生成投影片" or "convert outline to slides"

Claude reads `outline.md` and produces a compilable `.tex` file.

### Compile

```bash
xelatex slides.tex
xelatex slides.tex   # run twice for cross-references
```

Or open in TeXShop / VS Code (LaTeX Workshop) with the compiler set to **XeLaTeX**.

## Outline Format

Outlines use a markdown format with YAML frontmatter. See [`skills/beamer-slides/references/outline-format.md`](skills/beamer-slides/references/outline-format.md) for the full spec.

Quick reference:

```markdown
---
title: "My Talk"
author: "Jane Smith"
date: "2026-03-18"
output: "slides.tex"
---

## Section Name

### Slide Title
- Bullet point
- Another point

### Code Example

```python
print("hello")
```

### Two-Column Layout

<!-- columns -->
- Left side content

<!-- split -->

![Figure](figures/img.png)
<!-- end columns -->
```

Supported directives: bullets, numbered lists, code blocks, math (`$…$`, `$$…$$`), images, GFM tables, two-column layout, beamer blocks (`[!block]`, `[!alertblock]`, `[!exampleblock]`), speaker notes (`<!-- notes -->`), and pause overlays (`<!-- pause -->`).

## Plugin Structure

```
slides-beamer/
├── .claude-plugin/
│   └── plugin.json
└── skills/
    └── beamer-slides/
        ├── SKILL.md
        ├── references/
        │   └── outline-format.md
        └── assets/
            └── template.tex
```

## Theme Notes

The keynote theme (Version 1) is **XeLaTeX-native** and handles all font configuration internally (`fontspec`, `xeCJK`, Helvetica Neue, PingFang TC, `unicode-math`). Do not add font packages to the generated `.tex` file.
