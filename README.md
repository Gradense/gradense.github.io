# AI/ML Blog — Hugo + PaperMod starter (Phase 2 output)

This is the code from Phase 2: core config, layouts, and styling. It's tested
and builds cleanly, but it's not a full repo yet — a few things still need to
happen in Phase 4:

- The PaperMod theme itself isn't in here. It's added as a **git submodule**
  (not vendored copy) — Phase 4 gives the exact command.
- `content/posts/` is empty. Phase 3 adds a full sample AI/ML post.
- `hugo.toml`'s `baseURL` is a placeholder (`https://www.yourdomain.com/`) —
  update it once your domain is connected in Phase 4. The GitHub Actions
  workflow overrides this automatically at deploy time either way.

## What's here

| File | Purpose |
|---|---|
| `hugo.toml` | Site config: theme params, dark-mode default, syntax highlighting, KaTeX toggle, menu |
| `archetypes/default.md` | Front-matter template used every time you run `hugo new posts/my-post.md` |
| `layouts/partials/extend_head.html` | Loads JetBrains Mono + KaTeX (only on pages with `math = true`) without touching the theme |
| `assets/css/extended/custom.css` | Code-block font/border polish, KaTeX mobile overflow fix — auto-bundled by PaperMod, no theme edits |
| `content/search.md` | Enables the `/search/` page (Fuse.js, built into the theme) |
| `.github/workflows/hugo.yml` | Builds and deploys to GitHub Pages on every push to `main` |

## Verified locally

Built successfully with Hugo (extended) against PaperMod — Chroma syntax
highlighting emits proper CSS classes, the KaTeX partial loads conditionally,
custom CSS bundles correctly, and dollar amounts like `$20/month` in prose are
confirmed *not* to get swallowed as math.
