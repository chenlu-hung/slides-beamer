# Beamer Outline Format Reference

This document defines the markdown outline format used by the `beamer-slides` skill. Phase 1 produces a file in this format; Phase 2 consumes it.

---

## YAML Frontmatter

Every outline begins with a YAML block:

```yaml
---
title: "Presentation Title"
subtitle: "Optional Subtitle"
author: "Your Name"
institute: "Your Affiliation"
date: "2026-03-18"        # or \today
output: "slides.tex"      # target .tex filename
---
```

All fields except `title` are optional.

---

## Structure Mapping

| Outline Markdown          | LaTeX Output                          |
|---------------------------|---------------------------------------|
| `## Section Name`         | `\section{Section Name}`              |
| `### Slide Title`         | `\begin{frame}{Slide Title} … \end{frame}` |
| `- bullet`                | `\item` inside `itemize`              |
| `1. item`                 | `\item` inside `enumerate`            |
| ` ```lang … ``` `         | `\begin{lstlisting}[language=lang]`   |
| `$...$` / `$$...$$`       | inline / display math (pass-through)  |
| `![alt](path)`            | `\includegraphics`                    |
| table (GFM)               | `tabular` wrapped in `table`          |

---

## Content Types

### Bullets and Numbered Lists

```markdown
### Slide Title
- First point
- Second point
  - Nested point
```

Nested bullets become nested `itemize` environments (up to two levels).

### Code Blocks

````markdown
### Code Example

```python
def hello():
    print("Hello, World!")
```
````

Specify the language identifier after the opening fence. Supported identifiers map directly to `lstlisting` language names (`python`, `java`, `c`, `cpp`, `bash`, `sql`, `javascript`, `typescript`, `latex`). Unknown identifiers are passed through as-is.

### Math

```markdown
### Key Formula

The loss function is:

$$L = -\sum_{i} y_i \log \hat{y}_i$$

Inline math also works: $f(x) = x^2$.
```

### Images

```markdown
### Figure

![Network Architecture](figures/arch.png)
```

Images are centered with `\centering` and scaled to `0.8\textwidth` by default. You can override width with an HTML-style attribute:

```markdown
![caption](path){width=0.5\textwidth}
```

### Tables

```markdown
### Comparison

| Method | Accuracy | Speed |
|--------|----------|-------|
| A      | 92%      | Fast  |
| B      | 95%      | Slow  |
```

Use `booktabs` rules (`\toprule`, `\midrule`, `\bottomrule`).

---

## Advanced Directives

### Two-Column Layout

Wrap content between `<!-- columns -->` and `<!-- end columns -->`. Separate the two columns with `<!-- split -->`:

```markdown
### Results

<!-- columns -->
- Left column bullet A
- Left column bullet B

<!-- split -->

![Figure](figures/result.png)
<!-- end columns -->
```

Produces equal-width `\begin{columns}[T]` layout.

### Beamer Blocks

Use blockquote syntax with a type tag:

```markdown
> [!block Title]
> This is a plain block.

> [!alertblock Warning]
> This will be highlighted in red.

> [!exampleblock Example]
> This will be highlighted in green.
```

Maps to `\begin{block}`, `\begin{alertblock}`, `\begin{exampleblock}`.

### Speaker Notes

```markdown
### Slide Title
- Visible content

<!-- notes -->
These are speaker notes. They appear in \note{} and are visible in presenter mode.
<!-- end notes -->
```

### Overlay / Pause

Insert `<!-- pause -->` between items to emit `\pause`:

```markdown
### Incremental Reveal
- First item
<!-- pause -->
- Second item (appears after click)
```

---

## Complete Example Outline

```markdown
---
title: "Deep Learning for NLP"
subtitle: "A Practical Introduction"
author: "Jane Smith"
institute: "Example University"
date: "2026-03-18"
output: "dl-nlp.tex"
---

## Introduction

### Why NLP Matters
- Language is the primary medium of human knowledge
- Billions of documents available for analysis
- Practical applications: search, translation, assistants

### Scope of This Talk
1. Tokenisation and embeddings
2. Transformer architecture
3. Fine-tuning pre-trained models

## Embeddings

### Word Vectors

Words are mapped to dense vectors:

$$\text{similarity}(u, v) = \frac{u \cdot v}{\|u\|\|v\|}$$

### Code: Loading Embeddings

```python
from gensim.models import Word2Vec
model = Word2Vec.load("model.bin")
vec = model.wv["language"]
```

## Transformer

### Architecture Overview

<!-- columns -->
- Encoder stack
- Multi-head self-attention
- Positional encoding

<!-- split -->

![Transformer](figures/transformer.png){width=0.45\textwidth}
<!-- end columns -->

> [!block Key Insight]
> Attention allows the model to relate any two positions in the sequence directly.

## Results

### Benchmark Performance

| Model    | BLEU | ROUGE-L |
|----------|------|---------|
| Baseline | 28.3 | 41.2    |
| Ours     | 34.7 | 47.8    |

## Conclusion

### Summary
- Transformers dominate modern NLP
- Pre-trained models reduce labelled-data requirements
- Future: multimodal and longer context

### Thank You
Questions?

<!-- notes -->
Mention upcoming workshop and point to GitHub repo.
<!-- end notes -->
```
