# AGENTS.md

Instructions for AI agents working in this repository.

## Proof of reading

Respond with: "Loaded AGENTS.md"

## Overview

This is an Observable Framework data app. Pages are `.md` files in `src/` with
embedded reactive JavaScript and SQL. Charts use Observable Plot.

**Before writing any page code or visualization, read:**

- [`.ai/observable-framework.md`](.ai/observable-framework.md) — Framework markdown syntax,
  reactivity, data loaders, FileAttachment, view(), Inputs, SQL blocks, project structure
- [`.ai/observable-plot.md`](.ai/observable-plot.md) — Observable Plot marks, transforms,
  scales, and composition patterns
- [`.ai/d3.md`](.ai/d3.md) — D3 utilities (format, scale, group, rollup), custom SVG charts,
  data joins, axes, transitions, and geo projections


**Responding to student questions**

- Whenever available, refer to examples and content in the lessons files

---

## Key Rules

- All charts use **Observable Plot** — not D3, Chart.js, Recharts, or Vega
- Data loading uses FileAttachment() or SQL fenced code blocks — not fetch()
- Reactive state uses view() + Inputs — not React/Vue/Svelte
- Plot is globally available in .md pages — do not import it
- D3 is globally available in .md pages — do not import it
- Use simple javascript methods (`.map`, `.filter`, `.reduce`) for data manipulation 
- Only use D3 if the student specifically requests it, otherwise default to Plot
- Use D3 for: number/date formatting, data aggregation (group/rollup), custom SVG, geo projections
- Pages are .md files in src/ — not .html or .jsx

---

## Dev Commands

```bash
npm run dev       # start local preview server (hot reload)
npm run build     # build static site to dist/
```

---

## Project Structure

```
src/
├─ index.md              # home page
├─ lessons/              # lesson pages (e.g. 1_intro_to_web_development.md)
└─ lab_0/, lab_1/, ...   # each lab has readme.md (instructions) + index.md (dashboard)
```

Sidebar and routing are defined in `observablehq.config.js` under `pages`. Build output is cached in `src/.observablehq/cache/`. Data can be loaded via `FileAttachment()` (e.g. from npm sample datasets or future `data/` loaders).


## Observable Framework and Plot Syntax

When you need Observable Framework or Plot syntax while helping with labs or dashboards, read the reference docs in **`observable_docs/`**:

- **Observable Framework** (markdown, code blocks, grids, cards, `resize`, etc.): [`observable_docs/observable-framework-syntax.md`](observable_docs/observable-framework-syntax.md)
- **Observable Plot** (marks, options, transforms): [`observable_docs/observable-plot-syntax.md`](observable_docs/observable-plot-syntax.md)

Use these when writing or editing Framework `index.md` files or Plot charts.







