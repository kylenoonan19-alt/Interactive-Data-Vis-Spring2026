---
title: "Lab 1: Passing Pollinators"
toc: true
---

```js
const bees = FileAttachment("data/pollinator_activity_data.csv").csv({ typed: true });
```

```js
display(bees)
```

<h1>Passing Pollinators</h1>

<br>

<h2> 1. What is the body mass and wing span distribution of each pollinator species observed?</h2>

<br>

```js
Plot.plot({
  title: "Pollinators by Average Body Mass",
  x: { label: "Pollinator species", domain: ["Carpenter Bee", "Bumblebee", "Honeybee"] },
  y: { label: "Average body mass (g)", grid: true, domain: [0, 0.50] },
  marks: [
    Plot.frame(),
    Plot.barY(
      bees,
      Plot.groupX(
        { y: "mean" },
        {
          x: "pollinator_species",
          y: "avg_body_mass_g",
          fill: "green",
          tip: true
        }
      )
    )
  ]
})
```

<p>The pollinator, or bee, with the largest average body mass is the Carpenter Bee at .49 grams. In second is the Bumblee at .251 grams. And third, the Honeybee weighing in at a mere .1 grams.</p>

```js
Plot.plot({
  title: "Pollinators by Average Wing Span",
  x: { label: "Pollinator Species", grid: true, domain: ["Carpenter Bee", "Bumblebee", "Honeybee"] },
  y: { label: "Average Wing Span (mm)", grid: true, domain: [0, 55] },
  marks: [
    Plot.frame(),
    Plot.dot(
      bees,
      Plot.groupX(
        { y: "mean" },
        {
          x: "pollinator_species",
          y: "avg_wing_span_mm",
          r: 8,
          fill: "gold",
          tip: true
        }
      )
    )
  ]
})
```

<p>Unsurprisngly due to its large body mass, the Carptener Bee has the largest average wing span at 42.3 millimeters. The Bumblebee reigns in second at 34.8mm. And in third place (again) the Honeybee's wingspan, on average, is 19mm.</p>

<br>

<h2> 2. What is the ideal weather (conditions, temperature, etc) for pollinating? </h2>

<br>

```js
Plot.plot({
  title: "Optimal Weather Conditions for Pollinating",
  x: { label: "Weather Condition", domain: ["Partly Cloudy", "Cloudy", "Sunny"] },
  y: { label: "Nectar Production (μL)", grid: true, domain: [0, 1] },
  marks: [
    Plot.frame(),
    Plot.dot(
      bees,
      Plot.groupX(
        { y: "mean" },
        {
          x: "weather_condition",
          y: "nectar_production",
          fill: "weather_condition",
          r: 8
        }
      )
    )
  ]
})
```

```js
Plot.plot({
  title: "Average Nectar Production by Temperature",
  x: { label: "Temperature (°C)" },
  y: { label: "Nectar Production (μL)", grid: true, domain: [0, 0.8] },
  marks: [
    Plot.ruleY([0]),
    Plot.frame(),
    Plot.rectY(
      bees,
      Plot.binX(
        { y: "mean" },
        {
          x: "temperature",
          y: "nectar_production",
          fill: "gold",
          fillOpacity: 0.85,
          thresholds: 15,
          tip: true
        }
      )
    )
  ]
})
```

<p>For this question, I chose the "nectar_production" variable to represent the construct, pollinating. The nectar production variable consists of the nectar produced per flower in microliters during the observation period. Pollination is "the transfer of pollen from an anther of a plant to the stigma of a plant, later enabling fertilization and the production of seeds." The first plot, a dot plot, represents the otpimal weather conditions for pollinating. The most optimal weather condition for pollinating, through nectar production, is cloudy. The second chart analyzes the ideal temperature for pollinating through binned temperature values. The highest nectar production, .76 μL occurred between 17-18 °C, whereas, the lowest production, .49 μL occurred between 14-15 °C.</p>

<br>

<h2> 3. Which flower has the most nectar production? </h2>

<br>

```js
Plot.plot({
  title: "Nectar Production",
  x: { label: "Flower"},
  y: { label: "Nectar Production (μL)", grid: true, domain: [0, 1.0] },
  marks: [
    Plot.frame(),
    Plot.barY(
      bees,
      Plot.groupX(
        { y: "mean" },
        {
          x: "flower_species",
          y: "nectar_production",
          fill: "maroon",
          tip: true
        }
      )
    )
  ]
})
```

<p>A bar chart was constructed to answer the question of which flower produces the most nectar amongst three flower species. The Sunflower produces the most nectar at .94 microliters (μL). Sunflowers are a high-energy, carbohydrate-loaded food source, especially popular amongst honeybees and bumblebees; two of the smaller bees in our study. The Coneflower produces .64 microliters of nectar, and the Lavendar flower produces .54 microliters.</p>
