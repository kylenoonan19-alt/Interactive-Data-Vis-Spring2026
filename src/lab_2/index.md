---
title: "Lab 2: Subway Staffing"
toc: true
---

<!-- Import Data -->
```js
const incidents = FileAttachment("./data/incidents.csv").csv({ typed: true })
const local_events = FileAttachment("./data/local_events.csv").csv({ typed: true })
const upcoming_events = FileAttachment("./data/upcoming_events.csv").csv({ typed: true })
const ridership = FileAttachment("./data/ridership.csv").csv({ typed: true })
```

<!-- Include current staffing counts from the prompt -->
```js
const currentStaffing = {
  "Times Sq-42 St": 19,
  "Grand Central-42 St": 18,
  "34 St-Penn Station": 15,
  "14 St-Union Sq": 4,
  "Fulton St": 17,
  "42 St-Port Authority": 14,
  "Herald Sq-34 St": 15,
  "Canal St": 4,
  "59 St-Columbus Circle": 6,
  "125 St": 7,
  "96 St": 19,
  "86 St": 19,
  "72 St": 10,
  "66 St-Lincoln Center": 15,
  "50 St": 20,
  "28 St": 13,
  "23 St": 8,
  "Christopher St": 15,
  "Houston St": 18,
  "Spring St": 12,
  "Chambers St": 18,
  "Wall St": 9,
  "Bowling Green": 6,
  "West 4 St-Wash Sq": 4,
  "Astor Pl": 7
}
```

<h1>Subway Staffing</h1>

<br>

## 1. How did local events impact ridership in summer 2025? What effect did the July 15th fare increase have? ##

```js
const eventsByDate = new Set(local_events.map((d) => +d.date))

const dailyRidershipMap = new Map()
for (const row of ridership) {
  const key = +row.date
  const total = row.entrances + row.exits
  dailyRidershipMap.set(key, (dailyRidershipMap.get(key) ?? 0) + total)
}

const dailyRidership = Array.from(dailyRidershipMap, ([dateValue, total_riders]) => {
  const date = new Date(Number(dateValue))
  return {
    date,
    total_riders,
    event_day: eventsByDate.has(Number(dateValue)) ? "Event day" : "Non-event day",
    fare_period: date < new Date("2025-07-15") ? "Before fare increase" : "After fare increase"
  }
})

const avgByEventDay = Array.from(
  dailyRidership.reduce((acc, d) => {
    const key = d.event_day
    if (!acc.has(key)) acc.set(key, { label: key, total: 0, count: 0 })
    const row = acc.get(key)
    row.total += d.total_riders
    row.count += 1
    return acc
  }, new Map()).values()
).map((d) => ({ ...d, avg_riders: d.total / d.count }))

const eventAvg = avgByEventDay.find((d) => d.label === "Event day")?.avg_riders ?? 0
const nonEventAvg = avgByEventDay.find((d) => d.label === "Non-event day")?.avg_riders ?? 1
const eventLiftPct = ((eventAvg - nonEventAvg) / nonEventAvg) * 100
const fareChangeDate = new Date("2025-07-15")

const avgByFarePeriod = Array.from(
  dailyRidership.reduce((acc, d) => {
    const key = d.fare_period
    if (!acc.has(key)) acc.set(key, { label: key, total: 0, count: 0 })
    const row = acc.get(key)
    row.total += d.total_riders
    row.count += 1
    return acc
  }, new Map()).values()
).map((d) => ({ ...d, avg_riders: d.total / d.count }))

const beforeFareAvg = avgByFarePeriod.find((d) => d.label === "Before fare increase")?.avg_riders ?? 0
const afterFareAvg = avgByFarePeriod.find((d) => d.label === "After fare increase")?.avg_riders ?? 1
const fareChangePct = ((afterFareAvg - beforeFareAvg) / beforeFareAvg) * 100

display(
  Plot.plot({
    marginLeft: 70,
    marginRight: 100,
    title: "Average NYC Subway Ridership: Event days vs Non-event days",
    y: { label: "Average riders", tickFormat: d3.format(",~s"), grid: true },
    x: { label: null },
    marginBottom: 45,
    marks: [
      Plot.barY(
        dailyRidership,
        Plot.groupX(
          { y: "mean" },
          { x: "event_day", y: "total_riders", fill: "event_day", tip: true }
        )
      ),
    ],
    color: { legend: true, domain: ["Event day", "Non-event day"]}
  })
)
```

<p> The bar graph above represents the average subway ridership on days where an event took place in New York City and non-event days in summer 2025. By aggregating the data from the "ridership" and local_events" datasets, we can understand how local events impacted ridership in NYC. The results show that the average citywide ridership was <b>614,962</b> on event days vs <b>593,477</b> on non-event days (+3.6%). Overall, ridership on event days was, on average, 3.6% higher than ridership on non-event days.</p>


```js
display(
  Plot.plot({
    marginLeft: 70,
    title: "Daily ridership trend with July 15 marker",
    x: { label: null },
    y: { label: "Total daily riders", tickFormat: d3.format(",~s"), grid: true },
    marks: [
      Plot.lineY(dailyRidership, { x: "date", y: "total_riders", stroke: "#4c78a8" }),
      Plot.ruleX([fareChangeDate], { stroke: "red", strokeWidth: 2 }),
      Plot.text([fareChangeDate], {
        x: (d) => d,
        y: d3.max(dailyRidership, (d) => d.total_riders),
        text: () => "July 15",
        dy: -8,
        fill: "red",
        textAnchor: "start"
      })
    ]
  })
)

```
<p> The line graph above represents the daily average subway ridership in Summer 2025 with a line at July 15th. On July 15th, the MTA raised the cost of a single subway ride from $2.75 to $3. The results show that the average daily ridership was <b>632,348</b> before July 15th and <b>568,800</b> after July 15th (−10.0%). Average daily ridership decreased 10% after July 15th. </p>

<br>

## 2. How do the stations compare when it comes to response time? Which are the best, which are the worst? ##

```js
display(Plot.plot({
  marginLeft: 180,
  marginRight: 80,
  title: "Average Response Time by Subway Station",
  x: { label: "Response Time (mins)", domain: [0,20]},
  y: { label: "Subway Station", grid: true },
  color: { legend: true, scheme: "YlOrRd", label: "Avg response time (mins)" },
  marks: [
Plot.barX(
  incidents,
  Plot.groupY(
    { x: "mean", fill: "mean" },
    { y: "station", x: "response_time_minutes", fill: "response_time_minutes", sort: { y: "x" }, tip: true} ,
  )
)
  ]
}))

```

<p> The horizontal bar chart above represents the average response times by subway station between 2015 and 2025. The three subway stations with the worst (longest) response times were 59 St-Columbus Circle (19 minutes) West 4 St-Wash Sq (18.71 minutes) and 125 St (18.54 minutes). Whereas, the stations with the best (quickest) response times were Fulton St, Houston St, and Times Sq-42 St, in that order, which all had close to 5 minutes. </p>


<br>

## 3. Which three stations need the most staffing help for next summer based on the 2026 event calendar? ##

```js
const top3Stations = Object.entries(
  upcoming_events.reduce((acc, d) => {
    acc[d.nearby_station] = (acc[d.nearby_station] || 0) + d.expected_attendance;
    return acc;
  }, {})
)
  .map(([nearby_station, total_expected_attendance]) => ({
    nearby_station,
    total_expected_attendance
  }))
  .sort((a, b) => b.total_expected_attendance - a.total_expected_attendance)
  .slice(0, 3);

display(
  Plot.plot({
  marginLeft: 50,
  marginRight: 40,
    title: "Top 3 Stations Needing Staffing (Summer 2026)",
    x: { label: "Nearby Station"},
    y: { label: "Total Expected Attendance", grid: true },
    color: { legend: true, scheme: "YlGnBu", label: "Total Expected Attendance" },
      marks: [
       Plot.barY(top3Stations, {
        x: "nearby_station",
        y: "total_expected_attendance",
        sort: { x: "y" },
        fill: "total_expected_attendance",
        tip: true,
      })
    ]
  })
)
```
<p> From June 1 - August 15, NYC will be the place-to-be for any summer fun, as there will be citywide events taking place. The quickest, relatively least stressful mode of transportation will be the subway. Therefore, the MTA needed to know which stations to staff at higher capacities due to the anticipated influx of attendees at these local events. The bar graph above shows the three subway stations, nearest to the events, that the MTA should prioritize staffing at -- 34 St-Penn Station, Canal St, and Chambers St. Canal Street is nearest to most of the events occurring this summer in NYC, and they might see upwards of <b>70,000+</b> people using that station. Second, 34 St-Penn Station might see up to <b>59,378</b> people. Lastly, Chambers St. might see up to <b>49,593</b> attendees, <i>just</i> for these events. This graph does not account for the NYers that will be taking the subway during those hot summer days as well.</p>

<br>

## [BONUS] If you had to prioritize one station to get increased staffing, which would it be and why? ##

<p> I would prioritize staffing Canal Street for two reasons. Firstly, Canal Street is expecting over 70,000 people, who are just attending events, during Summer 2026. This influx of passengers will require more platform staff, train operators, conductors, and safety personnel. Secondly, Canal Street ranked near the worst in average response time at close to 18.5 minutes. Therefore, due to the influx in passengers and the subpar average response time, the MTA should increase staffing at Canal Street during Summer 2026 to ensure trains run efficently and passengers reach their intended destinations safely and on-time. </p>