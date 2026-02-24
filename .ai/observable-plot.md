# Observable Plot — AI Reference

Observable Plot is a declarative JavaScript library for exploratory data visualization.
Charts are composed from marks — geometric shapes bound to data — not assembled from chart-type templates.

Source: https://observablehq.com/plot

---

## Core Concept: Plot.plot()

`Plot.plot(options)` renders an SVG or HTML figure element. In Observable Framework `.md`
pages, Plot is available globally — do NOT import it.

```js
// In Observable Framework .md pages:
Plot.plot({ marks: [...] })

// In plain JS (outside Framework):
import * as Plot from "@observablehq/plot"
Plot.plot({ marks: [...] })
```

### Top-level options

```js
Plot.plot({
  width: 640,           // default; use `width` reactive variable in Framework for responsive
  height: 400,          // default: auto-computed from scales
  marginTop: 20,
  marginRight: 20,
  marginBottom: 30,
  marginLeft: 40,
  grid: true,           // show axis grid lines
  inset: 6,             // padding around the frame
  aspectRatio: 1,       // lock x/y scale ratio (useful for maps, scatter)
  style: "overflow: visible;",  // CSS applied to the SVG
  caption: "Source: ...", // figure caption
  marks: [...]          // array of marks, drawn in order (last on top)
})
```

**Responsive width in Observable Framework:**
```js
Plot.plot({
  width,   // `width` is a built-in reactive variable in Framework
  marks: [...]
})
```

---

## Marks Overview

Plot has marks, not chart types. Compose multiple marks in one plot.

| Mark | Chart type | Constructor |
|---|---|---|
| `dot` | Scatterplot | `Plot.dot(data, options)` |
| `line` / `lineY` / `lineX` | Line chart | `Plot.lineY(data, options)` |
| `barY` / `barX` | Bar chart | `Plot.barY(data, options)` |
| `areaY` / `areaX` | Area chart | `Plot.areaY(data, options)` |
| `rect` / `rectY` / `rectX` | Heatmap / histogram | `Plot.rectY(data, Plot.binX({y: "count"}, {x: "value"}))` |
| `text` | Labels / annotations | `Plot.text(data, options)` |
| `ruleY` / `ruleX` | Reference lines | `Plot.ruleY([0])` |
| `tickY` / `tickX` | Tick marks | `Plot.tickX(data, options)` |
| `cell` | Grid / calendar heatmap | `Plot.cell(data, options)` |
| `image` | Image marks | `Plot.image(data, options)` |
| `vector` | Arrow/vector field | `Plot.vector(data, options)` |
| `geo` | Geographic shapes | `Plot.geo(data, options)` |
| `frame` | Border around plot | `Plot.frame()` |
| `crosshair` | Interactive crosshair | `Plot.crosshair(data, options)` |

---

## Dot Mark — Scatterplot

```js
Plot.plot({
  marks: [
    Plot.dot(data, {
      x: "weight",        // field name string, or accessor function d => d.weight
      y: "height",
      fill: "species",    // color encoding (categorical)
      r: 4,               // radius in pixels (constant), or channel: r: "value"
      opacity: 0.7,
      symbol: "species",  // shape encoding: "circle","square","triangle","star",etc.
      tip: true           // show tooltip on hover
    })
  ]
})
```

**With color and symbol legends:**
```js
Plot.plot({
  color: { legend: true, scheme: "Dark2" },
  symbol: { legend: true },
  marks: [
    Plot.dot(penguins, {
      x: "body_mass_g",
      y: "flipper_length_mm",
      fill: "species",
      symbol: "species"
    })
  ]
})
```

---

## Line Mark — Line / Time Series Chart

Use `lineY` when x is the domain axis (most common).

```js
Plot.plot({
  marks: [
    Plot.lineY(aapl, {
      x: "Date",
      y: "Close",
      stroke: "steelblue",
      curve: "linear",       // or "step", "natural", "monotone-x", "basis"
      marker: "circle"       // add marker dots: "circle","dot","arrow","tick"
    })
  ]
})
```

**Multi-series line chart:**
```js
Plot.plot({
  color: { legend: true },
  marks: [
    Plot.ruleY([0]),
    Plot.lineY(data, {
      x: "date",
      y: "value",
      stroke: "series"   // one line per unique value of "series"
    })
  ]
})
```

**If data is NaN/null, the line breaks automatically at gaps** — no special handling needed.

---

## Bar Mark — Bar Chart

Use `barY` for vertical bars, `barX` for horizontal bars.

```js
// Vertical bar chart
Plot.plot({
  marks: [
    Plot.barY(alphabet, {
      x: "letter",
      y: "frequency",
      sort: {x: "-y"},       // sort bars by descending y value
      fill: "steelblue",
      tip: true
    })
  ]
})
```

```js
// Horizontal bar chart
Plot.plot({
  marks: [
    Plot.barX(data, {
      x: "value",
      y: "category",
      sort: {y: "-x"},
      fill: "category"
    })
  ]
})
```

**Stacked bar chart:**
```js
Plot.plot({
  marks: [
    Plot.barY(data, {
      x: "year",
      y: "revenue",
      fill: "segment"    // stacks automatically when fill is a channel
    })
  ]
})
```

---

## Area Mark — Area Chart

```js
Plot.plot({
  marks: [
    Plot.ruleY([0]),
    Plot.areaY(aapl, {
      x: "Date",
      y: "Close",
      fill: "steelblue",
      fillOpacity: 0.2,
      curve: "linear"
    }),
    Plot.lineY(aapl, {x: "Date", y: "Close"})
  ]
})
```

**Stacked area / streamgraph:**
```js
Plot.plot({
  marks: [
    Plot.areaY(data, {
      x: "date",
      y: "value",
      fill: "series",
      offset: "wiggle"   // for streamgraph; or "center", "normalize"
    })
  ]
})
```

---

## Rect Mark — Histogram / Heatmap

**Histogram (use with bin transform):**
```js
Plot.plot({
  marks: [
    Plot.rectY(data, Plot.binX({y: "count"}, {x: "value", thresholds: 20}))
  ]
})
```

**Heatmap / calendar:**
```js
Plot.plot({
  marks: [
    Plot.cell(data, {
      x: "weekday",
      y: "hour",
      fill: "count",
      inset: 0.5
    })
  ]
})
```

---

## Rule Mark — Reference Lines

```js
Plot.ruleY([0])                  // horizontal line at y = 0
Plot.ruleX([0])                  // vertical line at x = 0
Plot.ruleY([mean], {stroke: "red", strokeDasharray: "4 2"})
Plot.ruleX(data, {x: "date", stroke: "gray"})  // one rule per data point
```

---

## Text Mark — Labels and Annotations

```js
Plot.plot({
  marks: [
    Plot.dot(data, {x: "x", y: "y"}),
    Plot.text(data, {
      x: "x",
      y: "y",
      text: "label",             // field name or function
      dy: -8,                    // vertical offset in pixels
      fontSize: 12,
      fill: "black",
      textAnchor: "middle"       // "start", "middle", "end"
    })
  ]
})
```

---

## Crosshair Mark — Interactive Tooltip

```js
Plot.plot({
  marks: [
    Plot.dot(data, {x: "x", y: "y"}),
    Plot.crosshair(data, {x: "x", y: "y"})
  ]
})
```

---

## Transforms

Transforms compute derived values before rendering. Pass transforms as additional
arguments to the mark constructor, wrapping the options.

### binX / binY — Histogram bins

```js
// Histogram: count values in bins
Plot.rectY(data, Plot.binX({y: "count"}, {x: "value"}))
Plot.rectY(data, Plot.binX({y: "count"}, {x: "value", thresholds: 20}))
Plot.rectY(data, Plot.binX({y: "count"}, {x: "date", thresholds: "month"}))
```

### groupX / groupY / group — Aggregate by category

```js
// Bar chart with automatic count
Plot.barY(data, Plot.groupX({y: "count"}, {x: "category"}))

// Bar chart with sum
Plot.barY(data, Plot.groupX({y: "sum"}, {x: "category", y: "value"}))

// Group by two dimensions
Plot.cell(data, Plot.group({fill: "count"}, {x: "weekday", y: "hour"}))
```

Available reducers: `"count"`, `"sum"`, `"mean"`, `"median"`, `"min"`, `"max"`,
`"deviation"`, `"variance"`, `"proportion"`, `"proportion-facet"`, `"first"`, `"last"`

### stack — Stacked charts

Stacking happens automatically when `fill` is a categorical channel on `barY`/`areaY`.
Override with explicit stack options:

```js
Plot.barY(data, Plot.stackY({x: "year", y: "value", fill: "segment", offset: "normalize"}))
```

Stack offsets: `null` (zero baseline), `"normalize"` (0–1), `"center"` (symmetric),
`"wiggle"` (streamgraph)

### normalize — Index / ratio transform

```js
// Index to 100 at first date
Plot.lineY(data, Plot.normalizeY("first", {x: "date", y: "value", stroke: "series"}))

// Normalize by sum
Plot.lineY(data, Plot.normalizeY("sum", {x: "date", y: "value", stroke: "series"}))
```

Basis: `"first"`, `"last"`, `"min"`, `"max"`, `"mean"`, `"median"`, `"sum"`, `"deviation"`

### windowY — Rolling average / moving window

```js
Plot.lineY(data, Plot.windowY({k: 7}, {x: "date", y: "value"}))  // 7-day rolling mean
Plot.lineY(data, Plot.windowY({k: 30, reduce: "mean"}, {x: "date", y: "value"}))
```

Reducers: `"mean"`, `"sum"`, `"min"`, `"max"`, `"median"`, `"variance"`, `"deviation"`

### dodge — Beeswarm

```js
Plot.plot({
  marks: [
    Plot.dot(data, Plot.dodgeY({x: "value", r: 3}))
  ]
})
```

### select — Filter to specific rows per series

```js
// Label only the last point per series
Plot.text(data, Plot.selectLast({x: "date", y: "value", text: "series", stroke: "series"}))
Plot.text(data, Plot.selectFirst({x: "date", y: "value", text: "series"}))
Plot.text(data, Plot.selectMaxY({x: "date", y: "value", text: d => d.value.toFixed(1)}))
```

### sort / filter / map / reverse

```js
// Sort bars
Plot.barY(data, {x: "letter", y: "freq", sort: {x: "-y"}})

// Filter inline
Plot.dot(data.filter(d => d.year >= 2020), {x: "date", y: "value"})
```

---

## Scales

Scales map data values to visual values (position, color, size).

### x / y scale options

```js
Plot.plot({
  x: {
    type: "linear",       // "linear","log","pow","sqrt","symlog","ordinal","band","point","time","utc"
    domain: [0, 100],     // explicit domain
    range: [0, 640],      // explicit range (usually auto)
    label: "Revenue ($)", // axis label
    tickFormat: "$.2s",   // d3-format string or function
    ticks: 5,             // number of ticks
    grid: true,           // grid lines for this axis
    reverse: false,       // flip the axis
    nice: true,           // extend domain to nice round numbers
    zero: true,           // include zero in domain
    inset: 6,             // padding inside the frame
    percent: true,        // multiply values by 100 for display
    clamp: false          // clamp out-of-range values to domain edges
  },
  marks: [...]
})
```

### Color scale options

```js
Plot.plot({
  color: {
    type: "linear",          // or "ordinal","categorical","diverging","threshold","quantile","quantize"
    scheme: "Blues",         // named d3/ColorBrewer scheme
    domain: [0, 100],
    range: ["white", "red"],
    legend: true,            // show a color legend
    label: "Temperature",
    reverse: false,
    unknown: "#ccc",         // color for null/undefined values
    pivot: 0                 // for diverging: center value
  },
  marks: [...]
})
```

**Color schemes:** Categorical: `"observable10"` (default), `"tableau10"`, `"Dark2"`,
`"Set2"`, `"Paired"`. Sequential: `"Blues"`, `"Greens"`, `"Reds"`, `"Oranges"`,
`"Purples"`, `"YlOrRd"`, `"Viridis"`, `"Plasma"`, `"Turbo"`, `"Magma"`. Diverging:
`"RdBu"`, `"RdYlGn"`, `"BrBG"`, `"PiYG"`, `"PRGn"`.

### Size (r) scale options

```js
Plot.plot({
  r: {domain: [0, 100], range: [2, 20]},
  marks: [Plot.dot(data, {x: "x", y: "y", r: "size"})]
})
```

---

## Faceting

Split a chart into small multiples with `fx` (facet by column) or `fy` (facet by row).

```js
Plot.plot({
  facet: {marginRight: 80},
  marks: [
    Plot.barY(data, {
      x: "category",
      y: "value",
      fy: "year"          // one row of facets per year
    })
  ]
})
```

```js
// 2D facet grid
Plot.dot(data, {x: "x", y: "y", fx: "col_facet", fy: "row_facet"})
```

---

## Legends

```js
Plot.plot({
  color: { legend: true, label: "Species" },  // color swatch legend
  r: { legend: true },                          // size legend
  symbol: { legend: true },                     // symbol legend
  marks: [...]
})
```

Or render a standalone legend:
```js
Plot.legend({ color: { domain: ["A","B","C"], scheme: "Dark2" } })
```

---

## Tooltip / tip option

Add `tip: true` to any mark for a built-in hover tooltip:

```js
Plot.dot(data, {x: "x", y: "y", tip: true})
Plot.barY(data, {x: "cat", y: "val", tip: true})
```

Customize which fields appear:
```js
Plot.dot(data, {
  x: "x",
  y: "y",
  tip: {
    format: {
      x: d => d3.format(".2f")(d),
      y: d => d3.format("$,.0f")(d)
    }
  }
})
```

---

## Common Composition Patterns

### Scatter + trend line

```js
Plot.plot({
  marks: [
    Plot.dot(data, {x: "x", y: "y", opacity: 0.5}),
    Plot.linearRegressionY(data, {x: "x", y: "y", stroke: "red"})
  ]
})
```

### Line + confidence band

```js
Plot.plot({
  marks: [
    Plot.areaY(data, {x: "date", y1: "lower", y2: "upper", fillOpacity: 0.2}),
    Plot.lineY(data, {x: "date", y: "value"})
  ]
})
```

### Bar + annotation rule

```js
Plot.plot({
  marks: [
    Plot.barY(data, {x: "category", y: "value"}),
    Plot.ruleY([target], {stroke: "red", strokeWidth: 2, strokeDasharray: "4 2"}),
    Plot.text([{category: "A", value: target}], {
      x: "category", y: "value",
      text: d => `Target: ${d.value}`,
      dy: -8, fill: "red"
    })
  ]
})
```

### Histogram with density

```js
Plot.plot({
  marks: [
    Plot.rectY(data, Plot.binX({y: "count"}, {x: "value", thresholds: 30})),
    Plot.ruleY([0])
  ]
})
```

### Grouped / faceted bar

```js
Plot.plot({
  marks: [
    Plot.barY(data, Plot.groupX({y: "sum"}, {
      x: "quarter",
      y: "revenue",
      fill: "region",
      fx: "year"        // facet by year
    }))
  ]
})
```

---

## Accessor Functions

Any channel option can be a field name string OR an accessor function:

```js
// Equivalent:
Plot.dot(data, {x: "date", y: "value"})
Plot.dot(data, {x: d => d.date, y: d => d.value})

// Only possible with function:
Plot.dot(data, {
  x: d => new Date(d.year, d.month - 1),
  y: d => d.revenue / 1_000_000,
  r: d => Math.sqrt(d.count),
  fill: d => d.value > 0 ? "positive" : "negative"
})
```

---

## What NOT to Do

- ❌ Don't build bar charts with D3 selections — use `Plot.barY`
- ❌ Don't manually compute bins for histograms — use `Plot.binX`
- ❌ Don't manually compute group counts — use `Plot.groupX({y: "count"}, ...)`
- ❌ Don't use `npm:@observablehq/plot` import in Framework `.md` pages — Plot is global
- ❌ Don't use Chart.js, Recharts, Vega, or D3 for charts — use Observable Plot
- ❌ Don't add `.plot()` to individual marks — call `Plot.plot({marks: [...]})` at the top
- ❌ Don't forget `Plot.ruleY([0])` on bar and area charts — it anchors the baseline

---

## Reference Links

- Plot home: https://observablehq.com/plot
- Marks: https://observablehq.com/plot/features/marks
- Plot.plot() options: https://observablehq.com/plot/features/plots
- Transforms: https://observablehq.com/plot/features/transforms
- Scales: https://observablehq.com/plot/features/scales
- Dot mark: https://observablehq.com/plot/marks/dot
- Line mark: https://observablehq.com/plot/marks/line
- Bar mark: https://observablehq.com/plot/marks/bar
- Area mark: https://observablehq.com/plot/marks/area
- Bin transform: https://observablehq.com/plot/transforms/bin
- Group transform: https://observablehq.com/plot/transforms/group
- Stack transform: https://observablehq.com/plot/transforms/stack
- Normalize transform: https://observablehq.com/plot/transforms/normalize
