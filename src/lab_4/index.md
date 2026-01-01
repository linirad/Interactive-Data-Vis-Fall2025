---
title: "Lab 4: Clearwater Crisis"
toc: false
---

<!-- Import Data -->
```js
const surveys = await FileAttachment("data/fish_surveys.csv").csv({ typed: true });
const stations = await FileAttachment("data/monitoring_stations.csv").csv({ typed: true });
const suspects = await FileAttachment("data/suspect_activities.csv").csv({ typed: true });
const quality = await FileAttachment("data/water_quality.csv").csv({ typed: true });


// display(surveys.slice(0,10));
// display(stations.slice(0,10));
// display(suspects.slice(0,10));
// display(quality.slice(0,10));

const threshold = {
  heavy_metals: { concern: 20, limit: 30, label: "Heavy metals (ppb)" },
  nitrogen: { concern: 1.5, limit: 2.0, label: "Nitrogen (mg/L)" },
  phosphorus: { concern: 0.05, limit: 0.1, label: "Phosphorus (mg/L)" },
  dissolved_oxygen: { concern: 7, limit: 5, label: "Dissolved Oxygen (mg/L)" },
  pH: { 
    concernMin: 6.5, 
    concernMax: 8.5, 
    limitMin: 6.0, 
    limitMax: 9.0, 
    label: "pH" 
  },
  turbidity: { concern: 10, limit: 26, label: "turbidity NTU"}
};
```

<h1 style="display: block; width: 100%; max-width: 100%; box-sizing: border-box; text-align: left; margin-bottom: 10px;">
  Lab 4: Clearwater Crisis
</h1> 

Lake Clearwater was a thriving environment housing healthy fish populations. The ecological collapse over the past two years at the lake has witnessed a drastic fall in the fish populations and a disturbing rise in contamination levels. One of the establishments operating around the river i.e. either Riverside Farm, Clearwater Fishing Lodge, Lakeside resort or ChemTech Manufacturing is likely responsible for the crisis. Let us evaluate the data to find the culprit. 

<br>

<h2>
  <i>What does the decline in fish population tell us about contamination levels?</i>
</h2>
<br>

```js
const sortedSurvey = surveys.sort((a, b) => 
  new Date(a.date) - new Date(b.date)
);
// Pre-process the data to add change values
const groupedData = d3.group(sortedSurvey, 
  d => `${d.species}-${d.station_id}-${d.pollution_sensitivity}`
);

const processedData = [];
for (const [key, values] of groupedData) {
  const first = values[0].count;
  const last = values[values.length - 1].count;
  processedData.push({
    ...values[0],
    firstCount: first,
    lastCount: last,
    change: last - first
  });
};

// display(processedData)
// largest decline for static annotation
const biggestDecline = processedData.reduce((min, d) => 
  d.change < min.change ? d : min
);
```

```js
Plot.plot({
  title: "Fish Population Trend at the Four Monitoring Stations",
  height: 500,
  marginLeft: 80,
  width: 600,
  grid: false,
  x: {nice: true, label: "Species count"},
  y: {inset: 5, label: "Species"},
  fx: {label: "Pollution Sensitivity",
       domain: ["High", "Medium", "Low"]
  },
  fy: {label: "Station Id"},
  color: {
    scheme: "reds",
    reverse: true,
    label: "Change in count", 
    tickFormat: "+f", 
    legend: true
  },
  facet: {marginRight: 90},
  marks: [
    Plot.frame({fill: "grey"}),
    Plot.arrow(processedData, {
      x1: "firstCount",
      x2: "lastCount",
      y: "species",
      fy: "station_id",
      fx: "pollution_sensitivity",
      stroke: "change",
      strokeWidth: 2,
      tip: true
    }),
    // Annotation with pointer
    Plot.text([biggestDecline], {
      x: "lastCount",
      y: "species",
      fy: "station_id",
      fx: "pollution_sensitivity",
      text: d => `⚠️ Largest decline ${d.species}: ${d.change}`,
      dy: -15,
      textAnchor: "start",
      fill: "white",
      fontSize: 9,
      fontWeight: "bold",
      lineWidth: 15
    })
  ]
})
```

<br>
<!-- <p style="max-width: none; width: 100%; word-wrap: break-word; overflow-wrap: break-word;"> -->
<p>
The sharpest decline is seen in the trout population in the West station with a drop of 30 in the trout count over the two year period. Recent studies have shown that trout exhibit the highest sensitivity to heavy metal contamination. The Bass population has also declined significantly by 23. Both species are susceptible to heavy metal contamination and the next step would be to interrogate the water quality recorded at the four monitoring stations.
</p>
<br>

<h2>
  <i> Which monitoring station exhibits the highest pollution spike?</i>
</h2>
<br>

```js
const variables = [
  "nitrogen_mg_per_L",
  "phosphorus_mg_per_L",
  "heavy_metals_ppb",
  "turbidity_ntu",
  "ph",
  "dissolved_oxygen_mg_per_L"
];

const long = quality.flatMap(d =>
  variables.map(v => ({
    date: d.date,
    station_id: d.station_id,
    variable: v,
    value: d[v]
  }))
);

// Find the highest point across all data
const maxPoint = long.reduce((max, d) => 
  d.value > max.value ? d : max
);

// Get unique stations
const stations1 = Array.from(new Set(long.map(d => d.station_id)));

// Create threshold lines for heavy metals only - they'll appear on all facets
const heavyMetalThresholdRows = stations1.flatMap(station => [
  {
    station_id: station,
    type: "concern",
    threshold: threshold.heavy_metals.concern,
    label: `Heavy Metal Concern Level: ${threshold.heavy_metals.concern} ppb`
  },
  {
    station_id: station,
    type: "limit",
    threshold: threshold.heavy_metals.limit,
    label: `Heavy Metal Threshold Limit: ${threshold.heavy_metals.limit} ppb`
  }
]);
```

```js
Plot.plot({
  title: "Pollution Levels Recorded at the Monitoring Stations",
  width: 600,
  height: 500,
  facet: {
    data: long,
    x: "station_id"
  },
  marks: [
    Plot.frame(),
    Plot.line(long, {
      x: "date",
      y: "value",
      stroke: "variable",    // color by variable
      tip: true
    }),
    Plot.text(
      long.filter(d => d.station_id === maxPoint.station_id && 
                       d.date.getTime() === maxPoint.date.getTime() && 
                       d.variable === maxPoint.variable),
      {
        x: "date",
        y: d => d.value - 1,
        fx: "station_id",
        text: d => `Max: ${d.value.toFixed(2)}\n${d.variable.replace(/_/g, " ")}`,
        fill: "red",
        fontWeight: "bold",
        textAnchor: "middle"
      }),
    // Threshold lines - only show for heavy metals data
    Plot.ruleY(
      heavyMetalThresholdRows,
      {
        y: "threshold",
        fx: "station_id",
        stroke: d => d.type === "limit" ? "red" : "orange",
        strokeDash: d => d.type === "limit" ? [6, 3] : [3, 3],
        strokeWidth: 1,
        tip: {
          format: {
            y: false,  // Don't show the default y value
            fx: false  // Don't show station_id
          }
        },
        title: d => d.label  // Use the label field for tooltip
      }
    ),
    // Optional: Add text labels for thresholds
    Plot.text(
      heavyMetalThresholdRows.filter(d => d.type === "concern"),
      {
        x: long[0].date,
        y: "threshold",
        fx: "station_id",
        text: "Threshold Concern Level",  // Static label
        fill: "orange",
        fontSize: 10,
        dx: 5,
        dy: -5,
        textAnchor: "start"
      }
    ),
    Plot.text(
      heavyMetalThresholdRows.filter(d => d.type === "limit"),
      {
        x: long[0].date,
        y: "threshold",
        fx: "station_id",
        text: "Threshold Limit",  // Static label
        fill: "red",
        fontSize: 10,
        dx: 5,
        dy: 5,
        textAnchor: "start"
      }
    )
  ],
  x: { type: "utc", label: "Date" },
  // y: { label: "Value" },
  color: { legend: true }
})
```

<p>
Once again, the West station exhibits the most spikes in heavy metal contamination with scores ranging from 22.5 to a max of 48.8 ppb, way over the concern threshold of 20 ppb and regulatory limit of 30 ppb. Given the decline of trout and bass populations in the West station in addition to these high contamination levels, clearly the West station warrants a more focused examination. Next, let us check the suspects operating closest to the monitoring stations to find the possible source of the contamination.
</p>

<br>
<h2>
  <i> Which of the suspects operate closest to the West station?</i>
</h2>
<br>

```js
const distanceFields = [
  "distance_to_chemtech_m",
  "distance_to_farm_m",
  "distance_to_resort_m",
  "distance_to_lodge_m"
];

// Convert to long format
const longDistances = stations.flatMap(d =>
  distanceFields.map(f => ({
    station_id: d.station_id,
    variable: f,
    value: d[f]
  }))
);
// Find the closest distance in West station (excluding water_depth_m)
const minPointWest = longDistances
  .filter(d => d.station_id === "West" && d.variable !== "water_depth_m")
  .reduce((min, d) => d.value < min.value ? d : min);

const labelMap = {
  "distance_to_chemtech_m": "ChemTech",
  "distance_to_farm_m": "Riverside Farm",
  "distance_to_lodge_m": "Fishing Lodge",
  "distance_to_resort_m": "Lakeside Resort"
};
```

```js
Plot.plot({
  title: "Proximity of the Suspects to the Monitoring Stations",
  width: 600,
  height: 500,
  marginBottom: 80, 
  fx: { label: null},
  marks: [
    Plot.frame(),
    Plot.barY(longDistances, {
      x: "variable",
      y: "value",
      fill: "variable",
      fx: "station_id",
      tip: true
    }),
    Plot.ruleY([0]),
    // Arrow pointing to closest distance - filter data to match West
    Plot.arrow(
      longDistances.filter(d => 
        d.station_id === "West" && 
        d.variable === minPointWest.variable
      ),
      {
        x1: "variable",
        y1: d => d.value + 200,
        x2: "variable",
        y2: d => d.value + 30,
        fx: "station_id",
        stroke: "red",
        strokeWidth: 2,
        headLength: 12
      }
    ),
    // Text annotation - filter data to match West
    Plot.text(
      longDistances.filter(d => 
        d.station_id === "West" && 
        d.variable === minPointWest.variable
      ),
      {
        x: "variable",
        y: d => d.value + 300,
        fx: "station_id",
        text: d => `Closest: ${d.value}m`,
        fill: "red",
        fontWeight: "bold",
        fontSize: 11,
        textAnchor: "middle"
      }
    )
  ],
  // x: { label: null, tickFormat: () => "" },
  x: { 
    label: "Suspects",  
    tickFormat: d => labelMap[d] || d,  // Use custom labels or fallback to original
    tickRotate: -45
  },
  y: { label: "Distance / Depth (m)", grid: true },
  color: { legend: true }
})
```
<p>
ChemTech Manufacturing is closest to the West station, operating at a distance of 800m. Now that we have biological indicators connecting the high trout and bass mortality to heavy metal contamination in the West station and spatial factors finding ChemTech in closest proximity to the West station, we should evaluate if the contamination spike and high mortality coincide with ChemTech’s activities.
</p>

<br>
<h2>
  <i> Are ChemTech’s activities related to the contamination and fish mortality spikes?</i>
</h2>
<br>

```js
// Filter and parse dates
const chemtechActivities = suspects.filter(d => d.suspect === "ChemTech Manufacturing");
chemtechActivities.forEach(d => d.date = new Date(d.date));

const westTrout = surveys.filter(d => d.station_id === "West" && d.species === "Trout");
westTrout.forEach(d => d.date = new Date(d.date));

const westBass = surveys.filter(d => d.station_id === "West" && d.species === "Bass");
westBass.forEach(d => d.date = new Date(d.date));

const westHeavyMetals = quality.filter(d => d.station_id === "West");
westHeavyMetals.forEach(d => d.date = new Date(d.date));
```

```js
Plot.plot({
  title: "Effect of ChemTech Activities on Heavy Metals contamination and Fish Mortality",
  width: 600,
  height: 500,
  marks: [
    // Trout count line
    Plot.line(westTrout, {
      x: "date",
      y: "count",
      stroke: () => "Trout Count",
      strokeWidth: 2.5
    }),
    Plot.dot(westTrout, {
      x: "date",
      y: "count",
      fill: () => "Trout Count",
      r: 5,
      title: d => `Date: ${d.date.toLocaleDateString()}\nTrout Count: ${d.count} fish`,
      tip: true
    }),
    Plot.line(westBass, {
      x: "date",
      y: "count",
      stroke: () => "Bass Count",
      strokeWidth: 2.5
    }),
    Plot.dot(westBass, {
      x: "date",
      y: "count",
      fill: () => "Bass Count",
      r: 5,
      title: d => `Date: ${d.date.toLocaleDateString()}\nBass Count: ${d.count} fish`,
      tip: true
    }),
    // Heavy metals line (scaled to match trout count range)
    Plot.line(westHeavyMetals, {
      x: "date",
      y: d => d.heavy_metals_ppb * 2,  // Scale up for visibility
      stroke: () => "Heavy Metals (ppb × 2)",
      strokeWidth: 2.5,
      strokeDasharray: "5,5"
    }),
    Plot.dot(westHeavyMetals, {
      x: "date",
      y: d => d.heavy_metals_ppb * 2,
      fill: () => "Heavy Metals (ppb × 2)",
      r: 5,
      title: d => `Date: ${d.date.toLocaleDateString()}\nHeavy Metals: ${d.heavy_metals_ppb} ppb`,
      tip: true
    }),
    // ChemTech activities as vertical lines
    Plot.ruleX(chemtechActivities, {
      x: "date",
      stroke: "activity_type",
      strokeWidth: 3,
      strokeOpacity: 0.7,
      title: d => `Date: ${d.date.toLocaleDateString()}\nActivity: ${d.activity_type}\nIntensity: ${d.intensity}\nNotes: ${d.notes}`,
      tip: true
    }),
  ],
  x: { type: "utc", label: "Date", grid: true },
  y: { label: "Count / Scaled Values", grid: true },
  color: { 
    legend: true,
    domain: [
      "Trout Count", 
      "Bass Count",
      "Heavy Metals (ppb × 2)", 
      ...new Set(chemtechActivities.map(d => d.activity_type))
    ],
    range: ["#76b7b2", "#ff7f0e", "#9467bd", "#e15759"]
  }
})
```

<br>
<p>
Almost every ChemTech maintenance shutdown coincides with a spike in the heavy metals contamination level and a drop in both the trout and bass count. The year-end maintenance shutdown in December 2023 exhibits the highest heavy metal level of 48.8 and a sharp drop in the fish population. <br>
<b>The data categorically shows that ChemTech Manufacturing and their quarterly maintenance shutdowns are the primary culprits responsible for the Clearwater Crisis.</b>
</p>

<!-- ```js
// Line chart showing pollutant levels chosen over dropdown selection due to clarity
const pollutant = view(Inputs.select(
  [
    "nitrogen_mg_per_L",
    "phosphorus_mg_per_L",
    "heavy_metals_ppb",
    "turbidity_ntu",
    "ph",
    "dissolved_oxygen_mg_per_L"
  ],
  {label: "Select Pollutant", value: "nitrogen_mg_per_L"}
));
```

```js
Plot.plot({
  width: 900,
  height: 500,
  facet: {
    data: quality,
    x: "station_id"
  },
  marks: [
    Plot.line(quality, {
      x: "date",
      y: pollutant,
      stroke: "station_id",
      tip: true,
      title: d => `${d.station_id}: ${d[pollutant]}`
    })
  ],
  x: {label: "Date", type: "utc"},
  y: {label: pollutant},
  color: {legend: true}
})
``` -->

<!-- 
// Arrow plot chosen over faceted line plot to show trout decline to showcase a different kind of plot
```js
Plot.plot({
  height: 600,
  marginLeft: 110,
  marginRight: 90,
  grid: true,
  x: {
    label: "Date",
    type: "time",
    tickFormat: d3.utcFormat("%b %Y")  // Format as "Jan 2023"
  },
  y: {
    label: "Species Count",
    nice: true
  },
  color: {
    legend: true,
    label: "Species"
  },
  fy: {
    label: "Station"
  },
  marks: [
    Plot.frame(),
    Plot.line(surveys, {
      x: "date",       
      y: "count",      
      stroke: "species",
      fy: "station_id",
      fx: "pollution_sensitivity",
      strokeWidth: 2,
      marker: "dot"
    }),
    Plot.dot(surveys, {
      x: "date",       
      y: "count",      
      fill: "species",
      fy: "station_id",
      fx: "pollution_sensitivity",
      r: 3,
      tip: true
    })
  ]
})
``` -->
