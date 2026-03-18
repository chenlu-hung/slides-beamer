---
name: beamer-slides
description: Create LaTeX Beamer presentations using the keynote theme. Handles two phases: (1) discuss topic and save structured markdown outline, (2) convert outline to compilable XeLaTeX .tex file. Trigger on phrases like "create slides", "make presentation", "generate beamer", "outline a talk", "投影片", "簡報", or "convert outline to slides".
triggers:
  - create slides
  - make presentation
  - generate beamer
  - beamer slides
  - outline a talk
  - convert outline to slides
  - 投影片
  - 簡報
  - 製作簡報
  - 幫我做投影片
---

# Beamer Slides Skill

You are helping the user create a LaTeX Beamer presentation using the **keynote theme (Version 1)**. Follow the two-phase workflow below. Determine which phase applies from context: if there is no outline file yet, start with Phase 1; if the user references an existing outline or says "convert" / "generate .tex", go directly to Phase 2.

---

## Phase 1 — Discuss Topic and Save Outline

### 1.1 Gather Information

Ask the user (or infer from their message) the following. Do not ask all at once — ask only what is missing:

- **Topic**: What is the presentation about?
- **Audience**: Who will watch it? (e.g., conference attendees, students, colleagues)
- **Length**: How many slides / minutes?
- **Structure preference**: Does the user have a preferred section breakdown, or should you suggest one?
- **Language**: English, Traditional Chinese (繁體中文), or mixed?
- **Output filename**: What should the `.tex` file be called? (default: `slides.tex`)

### 1.2 Propose Outline

Draft a structured outline in the format defined in `references/outline-format.md`. The outline must include:

- YAML frontmatter with `title`, `subtitle` (if applicable), `author`, `institute`, `date`, and `output`
- `## Section` headings for each major section
- `### Slide Title` headings for each slide
- Bullet points, numbered lists, or other content types as appropriate
- Rough speaker notes where helpful (`<!-- notes --> … <!-- end notes -->`)

Present the outline to the user in a fenced code block. Ask: "Does this look right? Any sections or slides you'd like to add, remove, or rearrange?"

### 1.3 Iterate

Revise the outline until the user approves it. Keep changes minimal and targeted — do not redesign sections the user did not mention.

### 1.4 Save Outline

Once approved, determine the save path:
- If the current working directory is a project directory (has `.git` or LaTeX files), save there.
- Otherwise, save in the current working directory.
- Default filename: `outline.md`

Write the final outline using the Write tool. Confirm to the user: "Outline saved to `outline.md`. Run Phase 2 when you're ready to generate the `.tex` file."

---

## Phase 2 — Generate `.tex` File

### 2.1 Read Inputs

Read both of:
1. The outline file (path provided by user, or `outline.md` in current directory)
2. `${CLAUDE_PLUGIN_ROOT}/skills/beamer-slides/assets/template.tex`

### 2.2 Parse the Outline

Parse the YAML frontmatter for metadata. Then walk the markdown body and convert each element according to the mapping in `references/outline-format.md`:

| Outline element | LaTeX |
|---|---|
| `## Section` | `\section{…}` (outside any frame) |
| `### Slide Title` | `\begin{frame}{…} … \end{frame}` |
| `- item` | `\item` in `itemize` |
| `1. item` | `\item` in `enumerate` |
| ` ```lang … ``` ` | `\begin{lstlisting}[language=lang] … \end{lstlisting}` |
| `$…$` / `$$…$$` | math pass-through |
| `![alt](path)` | `\centering\includegraphics[width=0.8\textwidth]{path}` |
| GFM table | `booktabs` tabular |
| `<!-- columns --> … <!-- split --> … <!-- end columns -->` | `\begin{columns}[T]` … `\end{columns}` |
| `> [!block T]` | `\begin{block}{T} … \end{block}` |
| `> [!alertblock T]` | `\begin{alertblock}{T} … \end{alertblock}` |
| `> [!exampleblock T]` | `\begin{exampleblock}{T} … \end{exampleblock}` |
| `<!-- notes --> … <!-- end notes -->` | `\note{…}` at end of frame |
| `<!-- pause -->` | `\pause` |

### 2.3 Apply Template

Start from `template.tex`. Replace the metadata placeholders with values from the frontmatter. Insert the generated body between `\begin{document}` … `\end{document}`, replacing the skeleton frames.

Keep the preamble intact. Do **not** add any font packages — the keynote theme handles `fontspec`, `xeCJK`, Helvetica Neue, PingFang TC, and `unicode-math` internally.

### 2.4 Slide Design Guidelines

Apply these rules when generating frames:

- **One idea per slide.** If a slide has more than 6–7 bullet points, split it.
- **Short bullets.** Each bullet should be a phrase, not a full sentence. Cut filler words.
- **Avoid walls of text.** If the user's outline contains paragraphs, convert them to bullets or put long text in `\note{}`.
- **Section intro frames.** The keynote `\AtBeginSection` hook auto-generates section intro frames (number + title). Do not add manual title frames for sections.
- **Code slides.** Use `lstlisting` for all code. Set `language=` to the appropriate identifier. For long code, add `firstline=` and `lastline=` or split across two slides.
- **Two-column.** Use two-column layout when comparing two things or pairing a figure with bullet points. Equal column widths (`0.5\textwidth` each) are the default.
- **Math.** Wrap display math in `equation` or `align` environments rather than bare `$$` when inside a frame.
- **Figures.** Always add `\caption{}` and `\label{}` if the image has a meaningful caption. Center figures with `\centering`.
- **Tables.** Use `\toprule`, `\midrule`, `\bottomrule` from `booktabs`. Keep tables to at most 5–6 columns.
- **Chinese text.** If the presentation is in Chinese or mixed, ensure all Chinese content is encoded UTF-8. The theme's `xeCJK` configuration handles rendering.

### 2.5 Write Output

Determine the output path:
- Use the `output` field from the YAML frontmatter if present.
- Otherwise, derive from the outline filename (e.g., `outline.md` → `slides.tex`).
- Write to the same directory as the outline file.

Write the `.tex` file using the Write tool.

### 2.6 Confirm and Compile Instructions

After writing, tell the user:

```
Generated: <output path>

To compile:
  xelatex <filename>.tex
  xelatex <filename>.tex   # run twice for cross-references

If xelatex is not on your PATH, open the file in TeXShop or VS Code with LaTeX Workshop and set the compiler to XeLaTeX.
```

If you notice anything in the outline that could not be faithfully converted (e.g., a directive you don't recognise, a missing figure file), list those items as warnings after the compile instructions.

---

## Resources

- **Outline format spec**: `${CLAUDE_PLUGIN_ROOT}/skills/beamer-slides/references/outline-format.md`
- **Template**: `${CLAUDE_PLUGIN_ROOT}/skills/beamer-slides/assets/template.tex`
- **Theme source** (read-only, do not modify):
  - `~/Library/texmf/tex/latex/beamerthemekeynote/beamerthemekeynote.sty`
  - `~/Library/texmf/tex/latex/beamerthemekeynote/beamerfontthemekeynote.sty`
  - `~/Library/texmf/tex/latex/beamerthemekeynote/beamerinnerthemekeynote.sty`

---

## Error Handling

- **Missing outline file**: Ask the user for the correct path. Do not guess.
- **Malformed YAML frontmatter**: Report the parse error and ask the user to fix it before proceeding.
- **Unknown content directive**: Skip it, emit a `% TODO: unrecognised directive` comment in the .tex output, and warn the user.
- **Very long outline (>40 slides)**: Warn the user that the presentation may be too long and ask whether to proceed or trim.
