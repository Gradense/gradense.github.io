+++
title = '{{ replace .File.ContentBaseName "-" " " | title }}'
date = {{ .Date }}
draft = true
description = ""
summary = ""
tags = []
categories = []
series = []

# Reading experience toggles (all optional — remove any line to fall back to the site default)
math = false          # set true if this post uses \( \) or $$ $$ LaTeX equations
ShowToc = true
TocOpen = false
ShowCodeCopyButtons = true
ShowReadingTime = true

# cover.image = "images/cover.png"
# cover.alt = "Descriptive alt text for the cover image"
+++

<!--
Writing checklist:
- Lead with the concrete problem or result before the theory.
- Fence code blocks with a language tag, e.g. ```python or ```bash, for syntax highlighting.
- Standalone equations: wrap in $$ ... $$ on their own line — no special escaping needed.
- Inline math: type it as \\( ... \\) — TWO backslashes, not one. Markdown's own escaping
  rules eat a single backslash before a parenthesis, which silently breaks KaTeX. Typing
  \\( survives Markdown and renders correctly as \( in the page for KaTeX to find.
- Bare $...$ is intentionally NOT treated as math, so dollar amounts like $20/month in your
  prose render as plain text instead of breaking as an equation.
- Keep the first ~150 characters strong — it's used as the summary/description if you leave those blank.
-->
