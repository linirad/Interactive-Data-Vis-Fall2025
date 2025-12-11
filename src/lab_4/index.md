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


display(surveys.slice(0,10));
display(stations.slice(0,10));
display(suspects.slice(0,10));
display(quality.slice(0,10));

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
  height: 500,
  marginLeft: 80,
  width: 800,
  grid: false,
  x: {nice: true, label: "Species count"},
  y: {inset: 5, label: "Species"},
  fx: {label: "Pollution Sensitivity"},
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
    Plot.frame({fill: "#b3c7eeff"}),
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
      fill: "red",
      fontSize: 9,
      fontWeight: "bold",
      lineWidth: 15
    })
  ]
})
```

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

// Create threshold rows only for heavy_metals
// const stations = Array.from(new Set(long.map(d => d.station_id)));

// const heavyMetalThresholdRows = [
//   ...stations.map(station => ({
//     station_id: station,
//     type: "concern",
//     threshold: threshold.heavy_metals.concern,
//     label: "threshold concern"
//   })),
//   ...stations.map(station => ({
//     station_id: station,
//     type: "limit",
//     threshold: threshold.heavy_metals.limit,
//     label: "threshold limit"
//   }))
// ];
```

```js
Plot.plot({
  width: 800,
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
    // Arrow - only render for the station with max point
    // Plot.arrow(
    //   long.filter(d => d.station_id === maxPoint.station_id && 
    //                    d.date.getTime() === maxPoint.date.getTime() && 
    //                    d.variable === maxPoint.variable),
    //   {
    //     x1: d => d.date,
    //     y1: d => d.value + 5,
    //     x2: d => d.date,
    //     y2: d => d.value + 0.5,
    //     fx: "station_id",
    //     stroke: "red",
    //     strokeWidth: 2,
    //     headLength: 8
    //   }
    // ),
    // Text annotation - only render for the station with max point
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
      })
    // Plot.ruleY(heavyMetalThresholdRows, {
    //   y: "threshold",
    //   stroke: d => d.type === "limit" ? "red" : "orange",
    //   strokeDash: d => d.type === "limit" ? [6, 3] : [3, 3],
    //   strokeWidth: 2,
    //   filter: d => d.station_id === Plot.currentFacet, 
    //   tip: d => `${d.label}: ${d.threshold}`
    // })
  ],
  x: { type: "utc", label: "Date" },
  // y: { label: "Value" },
  color: { legend: true }
})
```

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
```

```js
Plot.plot({
  width: 800,
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
  x: { label: null, tickFormat: () => "" },
  y: { label: "Distance / Depth (m)", grid: true },
  color: { legend: true }
})
```

```js
// Filter and parse dates
const chemtechActivities = suspects.filter(d => d.suspect === "ChemTech Manufacturing");
chemtechActivities.forEach(d => d.date = new Date(d.date));

const westTrout = surveys.filter(d => d.station_id === "West" && d.species === "Trout");
westTrout.forEach(d => d.date = new Date(d.date));

const westHeavyMetals = quality.filter(d => d.station_id === "West");
westHeavyMetals.forEach(d => d.date = new Date(d.date));
```

```js
Plot.plot({
  width: 800,
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
      "Heavy Metals (ppb × 2)", 
      ...new Set(chemtechActivities.map(d => d.activity_type))
    ],
    range: ["#76b7b2", "#9467bd", "#e15759"]
  }
})
```

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
