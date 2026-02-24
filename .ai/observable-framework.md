# Observable Framework — AI Reference

Observable Framework is a static site generator for data apps and dashboards.
Pages are `.md` files with embedded reactive JavaScript. Build output is a static site.

Source: https://observablehq.com/framework

---

## Project Structure

```
.
├─ src/
│  ├─ .observablehq/
│  │  ├─ cache/           # data loader output cache (do not edit)
│  │  └─ deploy.json
│  ├─ components/
│  │  └─ myChart.js       # shared JS modules imported across pages
│  ├─ data/
│  │  └─ sales.csv.py     # data loader → generates sales.csv at build time
│  ├─ index.md            # home page → served at /
│  └─ report.md           # page → served at /report
├─ observablehq.config.js
└─ package.json
```

File-based routing: `src/foo/bar.md` → `/foo/bar`

---

## Markdown Pages

Pages are `.md` files. They support:
- Standard CommonMark Markdown
- YAML front matter
- Fenced code blocks (JS, SQL, HTML, etc.)
- Inline JavaScript expressions
- Grid/card layout helpers
- HTML blocks

### Front Matter

```yaml
---
title: My Dashboard
toc: false
---
```

---

## JavaScript Fenced Code Blocks

Use ` ```js ` for reactive JavaScript. Code runs in topological (dataflow) order,
not top-to-bottom order. All top-level `const` declarations are reactive variables.

```js
const data = FileAttachment("sales.csv").csv({typed: true})
```

```js
Plot.plot({
  marks: [Plot.barY(data, {x: "category", y: "revenue"})]
})
```

**Implicit display:** Expression blocks (no semicolon) automatically display their value.
Add a semicolon to suppress display.

```js
// displays the plot:
Plot.plot({ marks: [...] })

// does NOT display (semicolon suppresses):
const total = data.reduce((s, d) => s + d.value, 0);
```

**Echo (show source code):**
```js echo
Plot.plot({ marks: [...] })
```

---

## Inline Expressions

Use `${...}` to interpolate values into Markdown text:

```md
The total revenue is **${d3.format("$,.0f")(total)}**.

There are ${data.length} records in this dataset.
```

Inline expressions cannot declare top-level variables. Use a code block for that.

---

## Reactivity

Variables defined in one code block are automatically available to all other blocks.
Blocks re-run automatically when their upstream dependencies change.

```js
// Block 1: declares a reactive variable
const selectedYear = view(Inputs.select([2022, 2023, 2024], {label: "Year"}))
```

```js
// Block 2: automatically re-runs when selectedYear changes
const filtered = data.filter(d => d.year === selectedYear)
```

---

## view() and Inputs

`view()` renders an input widget AND exposes its value as a reactive variable.

```js
const category = view(Inputs.select(["A", "B", "C"], {label: "Category"}))
const threshold = view(Inputs.range([0, 100], {step: 1, label: "Threshold", value: 50}))
const showGrid = view(Inputs.toggle({label: "Show grid", value: true}))
const search = view(Inputs.search(data, {label: "Search"}))
```

**Placing inputs inside cards/layouts:**
When you need an input inside a layout element (e.g., a card), declare it separately
and use `Generators.input` to get the reactive value:

```js
const yearInput = Inputs.select([2022, 2023, 2024], {label: "Year"});
const year = Generators.input(yearInput);
```

Then place `yearInput` in your HTML layout and use `year` in downstream code.

### Available Inputs

| Input | Description |
|---|---|
| `Inputs.select(options, opts)` | Dropdown select |
| `Inputs.range([min, max], opts)` | Slider |
| `Inputs.toggle(opts)` | Boolean checkbox |
| `Inputs.checkbox(options, opts)` | Multi-select checkboxes |
| `Inputs.radio(options, opts)` | Radio buttons |
| `Inputs.text(opts)` | Text field |
| `Inputs.search(data, opts)` | Search/filter input |
| `Inputs.table(data, opts)` | Interactive data table |

---

## FileAttachment

Load static files or data loader output. Available globally in `.md` pages.

```js
// CSV with automatic type inference
const sales = FileAttachment("data/sales.csv").csv({typed: true})

// JSON
const config = FileAttachment("data/config.json").json()

// Parquet (via Arrow)
const df = FileAttachment("data/events.parquet").parquet()

// Plain text
const notes = FileAttachment("notes.txt").text()

// Image (returns URL string)
const logo = FileAttachment("logo.png").href()
```

**Critical rule:** `FileAttachment` only accepts a **static string literal**.
Dynamic paths like `` FileAttachment(`data/${name}.csv`) `` are INVALID.

The returned value is a Promise. In downstream code blocks it is automatically awaited.

---

## SQL Fenced Code Blocks

Framework includes built-in DuckDB for client-side SQL.

First, register your data sources in `observablehq.config.js`:

```js
// observablehq.config.js
export default {
  sql: {
    sales: "data/sales.parquet",
    customers: "data/customers.csv"
  }
}
```

Then query in `.md` pages:

```sql
SELECT category, SUM(revenue) AS total
FROM sales
GROUP BY category
ORDER BY total DESC
LIMIT 10
```

**Capture results as a named variable:**

```sql id=topCategories
SELECT category, SUM(revenue) AS total
FROM sales
GROUP BY category
ORDER BY total DESC
LIMIT 10
```

Now `topCategories` is available as a reactive variable in subsequent JS blocks.

**Display + capture:**

```sql id=results display
SELECT * FROM sales WHERE year = ${selectedYear}
```

**Inline SQL in JavaScript:**

```js
const [summary] = await sql`SELECT COUNT(*) AS n FROM sales`
```

---

## Data Loaders

Data loaders are files in `src/data/` (or anywhere in `src/`) whose filename ends with
a double extension like `.csv.py`, `.json.sh`, `.parquet.js`, etc.
They run at build time (and during dev preview) and output to stdout.

Framework caches loader output in `.observablehq/cache/`.

### Naming convention

| Loader file | Generated file | Access via |
|---|---|---|
| `src/data/sales.csv.py` | `src/data/sales.csv` | `FileAttachment("data/sales.csv")` |
| `src/data/metrics.json.js` | `src/data/metrics.json` | `FileAttachment("data/metrics.json")` |
| `src/data/events.parquet.py` | `src/data/events.parquet` | `FileAttachment("data/events.parquet")` |

### Example Python data loader

```python
# src/data/sales.csv.py
import sys
import duckdb

conn = duckdb.connect()
result = conn.execute("SELECT * FROM 'source.parquet'").df()
result.to_csv(sys.stdout, index=False)
```

### Example JS data loader

```js
// src/data/metrics.json.js
import * as fs from "node:fs"
const data = JSON.parse(fs.readFileSync("source.json", "utf8"))
process.stdout.write(JSON.stringify(data))
```

---

## Layout Helpers

Observable Framework provides CSS grid utilities for dashboard layouts.

### Grid

```html
<div class="grid grid-cols-2">
  <div class="card">
    ${Plot.plot({...})}
  </div>
  <div class="card">
    ${Plot.plot({...})}
  </div>
</div>
```

Grid column classes: `grid-cols-1`, `grid-cols-2`, `grid-cols-3`, `grid-cols-4`

### Cards

```html
<div class="card">
  <h2>Total Revenue</h2>
  <p class="big-number">${d3.format("$,.0f")(totalRevenue)}</p>
</div>
```

### Notes / callouts

```html
<div class="note">This is an informational callout.</div>
<div class="warning">This is a warning callout.</div>
<div class="tip">This is a tip callout.</div>
```

---

## Importing Modules

**From npm:**
```js
import * as d3 from "npm:d3"
import confetti from "npm:canvas-confetti"
```

**Local shared modules (from src/components/):**
```js
import {renderChart} from "./components/myChart.js"
```

**Observable stdlib (available globally, but can be imported explicitly):**
```js
import {FileAttachment, Generators, Mutable, resize} from "observablehq:stdlib"
```

**Do NOT use bare npm-style imports** (`import Plot from "plot"`) in `.md` pages.
Observable Plot, D3, and stdlib are available globally without importing.

---

## display() Function

`display()` explicitly renders a value on the page. Use when you need to show output
from within a function or conditional:

```js
const chart = Plot.plot({...})
display(chart)
```

Without `display()`, only the final expression in a block is shown (implicit display).

---

## Common Patterns

### Filtered chart reacting to an input

```js
const species = view(Inputs.select(["Adelie", "Chinstrap", "Gentoo"], {label: "Species"}))
```

```js
Plot.plot({
  marks: [
    Plot.dot(
      penguins.filter(d => d.species === species),
      {x: "body_mass_g", y: "flipper_length_mm", fill: "species"}
    )
  ]
})
```

### Async data loading with a loader

```js
const sales = FileAttachment("data/sales.csv").csv({typed: true})
// `sales` is automatically awaited in downstream blocks
```

### Conditional display

```js
{
  if (data.length === 0) {
    display(html`<p class="warning">No data found.</p>`)
  } else {
    display(Plot.plot({...}))
  }
}
```

---

## What NOT to Do

- ❌ Don't use `fetch()` to load data — use `FileAttachment` or SQL blocks
- ❌ Don't use React, Vue, or Svelte state — use `view()` + `Inputs`
- ❌ Don't write `.html` pages — write `.md` pages
- ❌ Don't use dynamic paths in `FileAttachment` — always static string literals
- ❌ Don't `import Plot from "npm:@observablehq/plot"` in `.md` — it's already global
- ❌ Don't use D3 for charts when Observable Plot can do it — prefer Plot
- ❌ Don't add semicolons to expression blocks if you want them to display

---

## Reference Links

- Framework docs: https://observablehq.com/framework
- Markdown: https://observablehq.com/framework/markdown
- Reactivity: https://observablehq.com/framework/reactivity
- Data loaders: https://observablehq.com/framework/data-loaders
- SQL: https://observablehq.com/framework/sql
- FileAttachment: https://observablehq.com/framework/files
- Project structure: https://observablehq.com/framework/project-structure
