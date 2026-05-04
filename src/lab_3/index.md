---
title: "Lab 3: Mayoral Mystery"
toc: false
---

<!-- Import Data -->
```js
const nyc = await FileAttachment("data/nyc.json").json();
const results = await FileAttachment("data/election_results.csv").csv({ typed: true });
const survey = await FileAttachment("data/survey_responses.csv").csv({ typed: true });
const events = await FileAttachment("data/campaign_events.csv").csv({ typed: true });

// Note: you don't have to keep this, but some helpful data exposure to see what we've loaded. 
// NYC geoJSON data
display(nyc)
// Campaign data (first 10 objects)
display(results.slice(0,10))
display(survey.slice(0,10))
display(events.slice(0,10))
```


```js
// The nyc file is saved in data as a topoJSON instead of a geoJSON. Thats primarily for size reasons -- it saves us 3MB of data. For Plot to render it, we have to convert it back to its geoJSON feature collection. 
const districts = topojson.feature(nyc, nyc.objects.districts)
display(districts)
```

## Election Results by District

```js
// Build a quick lookup so each district can be colored by candidate votes.
const votesByDistrict = new Map(results.map(d => [d.boro_cd, d.votes_candidate]));

display(Plot.plot({
    color: {
    scheme: "blues",
    legend: true,
    label: "Votes for candidate"
  },
  projection: {
    domain: districts,
    type: "mercator",
  },
  marks: [
    Plot.geo(districts, {
      fill: d => votesByDistrict.get(d.properties.BoroCD) ?? null,
      stroke: "white",
      tip: true,
      title: d => {
        const district = d.properties.BoroCD;
        const votes = votesByDistrict.get(district);
        return `District ${district}\nVotes: ${votes ?? "No data"}`;
      }
    })
  ]
}))
```
<p>
The map above represents the number of votes by district for your candidate in the recent 2024 NYC election. The choropleth map uses the color blue to represent the number of votes across geographic districts in NYC, where light blue indicates less votes and dark blue indicates more votes. On its face, the map indicates that your candidate prevailed in two boroughs: the Bronx and Brooklyn. More specifically for instance, District's 312, 313, and 315 in the southern part of Brooklyn recorded over 16,000 votes <i>each</i> for your candidate. The Bronx, it was District's 203, 209, and 212 that folks favored your candidate. This is a strong area of success for your campaign, because coupled with hitting both boroughs heavily with campaign events (which we will discuss further on), you saw high outputs at the polls. However, your candidate did not bode well in Manhattan, Staten Island (except District 503), and Queens. 

As you'll see in the map below, your campaign appears to have galvanized the NYC districts comprised of lower income voters.
</p>

## Were low income districts more likely to vote for your candidate? (Possibly!)

```js
const incomeByDistrict = new Map(results.map(d => [d.boro_cd, d.median_household_income]));

display(Plot.plot({
    color: {
    scheme: "greens",
    legend: true,
    label: "Median Household Income by NYC Voting District"
  },
  projection: {
    domain: districts,
    type: "mercator",
  },
  marks: [
    Plot.geo(districts, {
      fill: d => incomeByDistrict.get(d.properties.BoroCD) ?? null,
      stroke: "white",
      tip: true,
      title: d => {
        const district = d.properties.BoroCD;
        const income = incomeByDistrict.get(district);
        return `District ${district}\nMedian income: ${income ?? "No data"}`;
      }
    })
  ]
}))
```
<p>
Taking a look at the same NYC voting district boundary map, here, we see the socio-economic makeup of New York City. Lighter green indicates districts with a lower Median Household Income and darker green indicates higher Median Household Income. In tandem with the previous map, a noticeable trend emerges -- low median household income districts voted at a higher rate for your candidate. 

For consistency and accuracy, we will continue examining the six districts from the previous map. The following Brooklyn district's have the corresponding median household incomes: 312 ($41,230), 313 ($37,560), and 315 ($43,980). In the Bronx, District's 212 median income is $37,120, where 203 and 209 are both roughly $41,000. Each of these district's are in a <b>low</b> rated income category per the dataset, as their median incomes are below $50,000. This map cannot explain a reason <i>why</i> this trend exists, however, the next map provides us with one potential answer.

Surprisingly, and contrary to the rest of the borough (and city), Staten Island's District 503 cast roughly 11,000 votes for your candidate albeit their median household income is relatively high (~$103,000) compared to the districts that primarily voted for the candidate. This could be due to a number of reasons, such as discontent with the opponent, political party outliers, or even deep support/belief in your candidates beliefs or initiatives.
</p>

<br>
</br>

## Where were campaign events held?

```js
display(Plot.plot({
  title: "Campaign Events Across NYC",
  projection: {
    domain: districts,
    type: "mercator"
  },
  color: { legend: true, label: "Event type" },
  r: { range: [3, 14], label: "Estimated attendance" },
  marks: [
    Plot.geo(districts, {
      fill: "#f6f6f6",
      stroke: "white"
    }),
    Plot.dot(
      events,
      {
        x: "longitude",
        y: "latitude",
        fill: "event_type",
        r: "estimated_attendance",
        opacity: 0.8,
        tip: true,
        title: d =>
          `${d.event_type}\nDistrict: ${d.boro_cd}\nAttendance: ${d.estimated_attendance}\nDate: ${d.event_date}`
      }
    ),
  ]
}))
```
<p>
Campaigns spend millions of dollars and host dozens, if not hundreds, of events to maximize their candidates chance of electoral victory. Your campaign hosted a slew of events ranging from canvassing kickoffs to town halls to rountables. If we take a look at a map of where the events were held across NYC, we begin to understand the aforementioned trend. A majority of the campaign events occurred in Brooklyn and the Bronx (as well as Manhattan), where your candidate received a considerable number of votes. 

Although the campaign had great success in Brooklyn and the Bronx, your candidate struggled to garner moderate to high numbers of votes in the remaining three boroughs. These are places the campaign did not heavily hit with events; therefore, either the residents might not know who your candidate is or felt they did not know them well enough to vote for them.

Overall, from my analysis, we uncovered that a potential reason your candidate did not get elected is that the campaign events did not truly reach the Manhattan, Staten Island, or Queens communities. If having done so, you could have potentially garnered a substantial amount of support. Additionally, socio-economically, although the campaign fared well with low income districts, you did not fare well with middle-class districts. 

In summation, your future campaign should focus on middle-class districts in addition to lower-class districts and host more events in each borough so all New Yorkers can feel heard and truly get to know your candidate.
<br>
</br>

Some questions to examine in the future with more data/analysis may be:
<ul>
<li>Why didn't Manhattan voters cast votes at high rates for your candidate even though the campaign held over a dozen events there?</li>
<li>Why is Staten Island's District 503 an outlier? Is there something we can learn from their voters that can inform our next campaign?</li>

</ul>

</p>