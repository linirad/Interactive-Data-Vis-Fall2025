---
title: "Lab 2: Subway Staffing"
toc: false
---

# Lab 2: Subway Staffing
<!-- Import Data -->
```js
const incidents = await FileAttachment("./data/incidents.csv").csv({ typed: true })
const local_events = await FileAttachment("./data/local_events.csv").csv({ typed: true })
const upcoming_events = await FileAttachment("./data/upcoming_events.csv").csv({ typed: true })
const ridership = await FileAttachment("./data/ridership.csv").csv({ typed: true })
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

```js
// const allStations = Array.from(new Set(ridership.map(d => d.station)))
const allStations = ["All Stations", ...Array.from(new Set(ridership.map(d => d.station)))]
// const selectedStation = view(Inputs.select(allStations))
const fareChange = new Date("2025-07-15")
```

<!-- ```js
// First attempt at plot for Q1
Plot.plot({
  height: 200, 
  marks: [
    Plot.line(ridership. filter(d => d.station === selectedStation), {
      x: "date",
      y: "entrances",
      // z: "station",
      // stroke: "station",
    }),
    Plot.ruleX(local_events.filter(d => d.nearby_station === selectedStation), {
    // Plot.ruleX(local_events, {
      x: "date", 
      tip: true,
      channels: {
        "Event": "event_name", 
        "Station": "nearby_station"
      }
    }),
    Plot.ruleX([fareChange], { stroke: "red", tip: true }),
    Plot.text([fareChange], {
      x: d => d,
      dy: -70,
      dx: 37, 
      text: () => "Fare increase",
      anchor: "left"
    })
  ]
})
``` -->

<br>
<h2 style="white-space: nowrap;">1. Impact of summer 2025 local events and July 15th fare increase on ridership</h3>

```js
const anotherSelectedStation = view(Inputs.select(allStations))
```

```js
// Aggregate or filter data based on selection
const selectedStationData = (anotherSelectedStation === "All Stations" 
  ? Array.from(
      ridership.reduce((acc, d) => {
        const dateKey = d.date.toDateString();
        if (!acc.has(dateKey)) {
          acc.set(dateKey, { date: d.date, entrances: 0, exits: 0, station: "All Stations" });
        }
        const current = acc.get(dateKey);
        current.entrances += d.entrances || 0;
        current.exits += d.exits || 0;
        return acc;
      }, new Map()).values()
    )
  : ridership.filter(d => d.station === anotherSelectedStation)
).map(d => ({
  ...d,
  total_traffic: (d.entrances || 0) + (d.exits || 0)
})).sort((a, b) => a.date - b.date);

// Filter events based on selection
const selectedEvents = anotherSelectedStation === "All Stations"
  ? local_events
  : local_events.filter(d => d["nearby_station"].includes(anotherSelectedStation));

// const selectedStationData = ridership.filter(d => d.station === anotherSelectedStation).map(d => ({
//     ...d,  // Keep all existing fields
//     total_traffic: (d.entrances || 0) + (d.exits || 0)
//   })) 
// const selectedEvents = local_events.filter(d => d["nearby_station"].includes(anotherSelectedStation))
// display(selectedStationData[0])

// Find the date with highest traffic
const peakTrafficDay = selectedStationData.reduce((max, d) => 
  d.total_traffic > max.total_traffic ? d : max
, { total_traffic: 0 });

// Find event(s) on that date
const popularEvent = selectedEvents.find(event => 
  event.date.toDateString() === peakTrafficDay.date.toDateString()
);
// Combine into one object
const popularEventWithTraffic = popularEvent ? {
  ...popularEvent,
  ...peakTrafficDay  // Spreads all traffic fields
} : null;
// display(popularEventWithTraffic)
```

```js
// Split data into before/after fare change
const beforeData = selectedStationData.filter(d => d.date < fareChange);
const afterData = selectedStationData.filter(d => d.date >= fareChange);

// Compute averages
const avgBefore = beforeData.reduce((sum, d) => sum + d.total_traffic, 0) / beforeData.length;
const avgAfter = afterData.reduce((sum, d) => sum + d.total_traffic, 0) / afterData.length;

// Find representative x positions (midpoints)
const xBefore = new Date((beforeData[0].date.getTime() + beforeData.at(-1).date.getTime()) / 2);
const xAfter = new Date((afterData[0].date.getTime() + afterData.at(-1).date.getTime()) / 2);

// Prepare objects for plotting
const avgPoints = [
  { x: xBefore, y: avgBefore, label: `Avg ridership before fare rise: ${avgBefore.toFixed(0)}` },
  { x: xAfter,  y: avgAfter,  label: `Avg ridership after fare rise: ${avgAfter.toFixed(0)}` }
];

// Compute change rate (percentage)
const changeRate = ((avgAfter - avgBefore) / avgBefore) * 100;
// console.log(`Ridership change rate: ${changeRate.toFixed(2)}%`);
const minDate = new Date(Math.min(...selectedStationData.map(d => d.date)));
const maxDate = new Date(Math.max(...selectedStationData.map(d => d.date)));
```

```html
<div class="card">
${resize((width) => Plot.plot({
  height: 300, 
  marginLeft: 50,
  width,
  grid: true,
  title: anotherSelectedStation === "All Stations" 
    ? "Overall System Total Traffic" 
    : `${anotherSelectedStation} Total Traffic`,
  y: {   // <-- add this
    label: "Total Traffic"
  },
  marks: [
    Plot.frame(),
    // shading before and after fareChange
    Plot.rectY([
      { x1: minDate, x2: fareChange, fill: "#f0f7ff" },
      { x1: fareChange, x2: maxDate, fill: "#f6cfcfff" }
    ], {
      x1: "x1",
      x2: "x2",
      y1: null,   // span full vertical range
      y2: null,   // span full vertical range
      fill: "fill",
      opacity: 0.4
    }),
    // the line based on the filtered (selected) station data
    Plot.line(selectedStationData, {
      x: "date",
      y: "total_traffic",
      tip: true
    }),
    // Trend line
    Plot.linearRegressionY(selectedStationData, {
      x: "date",
      y: "total_traffic",
      stroke: "red",
      strokeWidth: 1,
      strokeDasharray: "5,5"  // Dashed line
    }),
    // dots for the local events that correspond to the selected station
    Plot.dot(selectedEvents, {
      x: "date",
      y: eventDataObj => {
        const stationData = selectedStationData.find(stationDataObj => { 
          return eventDataObj.date.toDateString() === stationDataObj.date.toDateString()
        })
        return stationData ? stationData.total_traffic : null;
      },
      stroke: "blue", 
      fill: "white",
      tip: {
        format: {
          x: false,
          y: false
        },
      channels: {
        "Event": "event_name",
        ...(anotherSelectedStation === "All Stations" ? { "Station": "nearby_station" } : {}),
        "Total traffic": eventDataObj => {
          const stationData = selectedStationData.find(stationDataObj => 
            eventDataObj.date.toDateString() === stationDataObj.date.toDateString()
          );
          return stationData ? stationData.total_traffic : null;
        }
      }
    }
    }),
    // Dot and Annotation for most popular event
    ...(popularEventWithTraffic && anotherSelectedStation !== "All Stations" ? [
      Plot.dot([popularEventWithTraffic], {
        x: "date",
        y: "total_traffic",
        stroke: "red",
        fill: "gold",
        r: 2,
        strokeWidth: 3
      }),
      Plot.tip([popularEventWithTraffic], {
        x: "date",
        y: "total_traffic",
        format: {
          x: false,
          y: false
        },
        channels: {
          "Popular Event 🏆": "event_name",
          ...(anotherSelectedStation === "All Stations" ? { "Station": "nearby_station" } : {}),
          "Total traffic": "total_traffic"
        }
      })
    ] : []),
    Plot.ruleX([fareChange], { stroke: "red"}),
    Plot.text([fareChange], {
      x: d => d,
      y: 0,
      text: () => "Fare increase",
      textAnchor: "end",
      dx: 0,
      dy: -10,
      fill: "red",
      fontWeight: "bold"
    }),
    // Vertical average bars
    Plot.ruleX(avgPoints, {
      x: "x",
      y1: 0,
      y2: "y",
      stroke: d => d.x < fareChange ? "blue" : "green",
      strokeWidth: 3,
      opacity: 0.7
    }),
    // Average labels
    Plot.text(avgPoints, {
      x: "x",
      y: "y",
      text: d => d.label,
      dy: 100,
      fill: d => d.x < fareChange ? "blue" : "green",
      textAnchor: "middle",
      fontWeight: "bold",
      fontSize: 11
    }),
    // Baseline (y = 0)
    Plot.ruleY([0])
  ]
}))}
</div>
```

<b>FINDING</b>

```js
// Compute change rate
const changeRate = ((avgAfter - avgBefore) / avgBefore) * 100;

// Create a Markdown-formatted string
const markdownOutput = `
🚆 Ridership Change Summary

- Average before fare rise: ${avgBefore.toFixed(0)}
- Average after fare rise: ${avgAfter.toFixed(0)}
- Change rate: ${changeRate.toFixed(2)}%

${
  changeRate >= 0 
    ? `📈 Ridership increased by ${changeRate.toFixed(2)}% after the fare increase.` 
    : `📉 Ridership decreased by ${Math.abs(changeRate).toFixed(2)}% after the fare increase.`
}
\n The chart shows that ridership consistently goes up on event days and most popular event for each station is flagged.
`;

display(markdownOutput);
```

<br>

<h2 style="white-space: nowrap;">2. Compare response times across stations</h3>

<div class="card">
${Plot.plot({
  height: 300,
  width,
  title: "Response time for incidents across all stations",
  x: {
    label: "Response Time (minutes)",
    grid: true
  },
  color: {
    legend: true,
    label: "Severity"
  },
  marks: [
    Plot.frame(),
    Plot.dot(incidents, Plot.dodgeY("bottom", {
      x: "response_time_minutes",
      y: "count",
      fill: "severity",
      // fy: "station",
      r: 3,
      fillOpacity: 0.6,
      tip: true,
      channels: {
        Station: "station"
      }
    }))
  ]
})
}
</div>

<br>

```js
const grouped = incidents.reduce((acc, d) => {
  // Create a unique key combining station and severity
  const key = `${d.station}|${d.severity}`;
  // If this combination doesn't exist yet, create an empty array
  if (!acc[key]) acc[key] = [];
  // Add the current incident to this group
  acc[key].push(d);
  return acc;
}, {});

const avgByStationSeverity = Object.entries(grouped)
   // Convert object to array of [key, value] pairs
  .map(([key, rows]) => ({
    // For each group, extract station and severity from first row
    station: rows[0].station,
    severity: rows[0].severity,
    // Calculate average response time
    avg: Number((rows.reduce((sum, r) => sum + (+r.response_time_minutes || 0), 0) / rows.length).toFixed(2)) // Divide total by count
  }))
  .sort((a, b) => a.station.localeCompare(b.station) || a.avg - b.avg); // Sort by station name first then avg;

// display(avgByStationSeverity);
```


<div class="card">
${Plot.plot({
  marginLeft: 110,
  height: 300,
  width,
  title: "Station-wise average response time",
  x: {
    label: "Average Response Time (minutes)",
    grid: true
  },
  y: {
    label: "Station"
  },
  marks: [
    Plot.frame(),
    Plot.barX(avgByStationSeverity, {
      x: "avg",
      y: "station",
      fill: "severity",
      order: ["high", "medium", "low"],
      tip: true,
      sort: {y: "x"}  // Sort by the x value (avg)
    })
  ]
})
}
</div>



<br>
<b>FINDING:</b> The plot shows that <b>59 St. Columbus Circle</b> has the <b>worst</b> average response time and <b>Fulton St.</b> has the <b>best</b>. 
<br>
<br>

<h2 style="white-space: nowrap;">3. Three Stations that need most staffing help for upcoming events</h3>


```js
// Add staff count to events
const eventsWithStaff = upcoming_events.map(d => ({
  ...d,
  staff_count: currentStaffing[d.nearby_station] || 0
}));

// Aggregate total attendance per station
const stationAggregates = Object.entries(
  eventsWithStaff.reduce((acc, d) => {
    const total = acc[d.nearby_station]?.totalAttendance || 0;
    const staffCount = d.staff_count;
    acc[d.nearby_station] = {
      totalAttendance: total + (d.expected_attendance || 0),
      staffCount: staffCount
    };
    return acc;
  }, {})
).map(([station, { totalAttendance, staffCount }]) => ({
  station,
  staffCount,
  totalAttendance,
  perStaffLoad: staffCount > 0 ? Math.ceil(totalAttendance / staffCount) : 0
}));

// display(stationAggregates);
// Sort stations by per-staff load descending
const sortedStations = stationAggregates
  .sort((a, b) => b.perStaffLoad - a.perStaffLoad)
  .map(d => d.station);

// Top 3 stations
const top3Stations = stationAggregates
  .sort((a, b) => b.perStaffLoad - a.perStaffLoad)
  .slice(0, 3)
  .map(d => d.station);

```


<div class="grid grid-cols-2">
  <!-- first column -->
  <div class="card">
  ${Plot.plot({
    marginLeft: 80,
    marginBottom: 100,
    title: "Projected Staffing Load for Upcoming Events in 2026",
    x: { label: "Station", tickRotate: 90, grid: true, domain: sortedStations },
    y: { label: "Projected Load Per Staff" },
    marks: [
      Plot.frame(),
      Plot.barY(
        stationAggregates,
        {
          x: "station",
          y: "perStaffLoad",
          fill: d => top3Stations.includes(d.station) ? "crimson" : "#bbb",
          tip: d => `${d.station}: ${d.perStaffLoad} per staff (Staff: ${d.staffCount}, Attendance: ${d.totalAttendance})`
        }
      )
    ]
  })
  }
</div>
<!-- second column -->
<div class="card">
${Plot.plot({
  marginLeft: 80,
  x: {
    label: "Response Time (minutes)",
    grid: true
  },
  y: {
    label: "Staffing Count"
  },
  marks: [
    Plot.dot(incidents, Plot.hexbin(
      {r: "count"}, {x: "staffing_count", y: "response_time_minutes", tip: true, fill: "severity", sort: {y: "x"}}
      ))
  ]
})}
</div>
</div>


<b>FINDING</b> Based on the expected attendance for upcoming events in 2026, the plot shows the projected staffing load per staff using current staffing levels. The plot shows that the top 3 stations that will require staffing help are <b>Canal St, 34th St. Penn Station and 23 St</b>.

<br>

<h2 style="white-space: nowrap;">4. Prioritize one station for increased staffing</h3>

```js
// Compute first factor for prioritization of one station - Average ridership 
const stationAverages = ridership.reduce((acc, d) => {
  const station = d.station;
  const total = (d.entrances || 0) + (d.exits || 0);

  if (!acc[station]) {
    acc[station] = { station, sum: 0, count: 0 };
  }

  acc[station].sum += total;
  acc[station].count += 1;

  return acc;
}, {});

const averageRidership = Object.values(stationAverages).map(s => ({
  station: s.station,
  avg_total: Math.ceil(s.sum / s.count)
}));
// display(averagesArray);

// second factor - upcoming events stationAggregates
// display(stationAggregates)

// Third factor - average response time for incident per staff
// Group incidents by station only (no more severity grouping)
const score_grouped = incidents.reduce((acc, d) => {
  const station = d.station;

  if (!acc[station]) acc[station] = [];
  acc[station].push(d);

  return acc;
}, {});

// Compute station-wise average response_time_minutes per staffing_count
// Group incidents by station only (ignore severity)
const grouped1 = incidents.reduce((acc, d) => {
  const station = d.station;  // Only station, ignore severity

  if (!acc[station]) acc[station] = [];
  acc[station].push(d);

  return acc;
}, {});

// Compute station-wise avg_response_time_per_staff
const incidentAvgByStation = Object.entries(grouped1)
  .map(([station, rows]) => {
    const totalResponse1 = rows.reduce(
      (sum, r) => sum + (+r.response_time_minutes || 0),
      0
    );

    const totalStaff1 = rows.reduce(
      (sum, r) => sum + (+r.staffing_count || 0),
      0
    );

    const avgPerStaff1 = totalStaff1 > 0
      ? Number((totalResponse1 / totalStaff1).toFixed(2))
      : 0;

    return {
      station,
      avg_response_time_per_staff: avgPerStaff1
    };
  })
  .sort((a, b) => a.station.localeCompare(b.station));


// display(avgByStation1);

// 1. Collect all unique stations
const allStations1 = new Set([
  ...averageRidership.map(d => d.station),
  ...stationAggregates.map(d => d.station), //Aggregate total attendance per station for upcoming events in 2026
  ...incidentAvgByStation.map(d => d.station)
]);

// 2. Compute min/max for each metric
const d1Values = averageRidership.map(d => d.avg_total);
const d2Values = stationAggregates.map(d => d.perStaffLoad);
const d3Values = incidentAvgByStation.map(d => d.avg_response_time_per_staff);

const d1Min = Math.min(...d1Values), d1Max = Math.max(...d1Values);
const d2Min = Math.min(...d2Values), d2Max = Math.max(...d2Values);
const d3Min = Math.min(...d3Values), d3Max = Math.max(...d3Values);

// 3. Convert arrays to lookup maps
const map1 = Object.fromEntries(averageRidership.map(d => [d.station, d.avg_total]));
const map2 = Object.fromEntries(stationAggregates.map(d => [d.station, d.perStaffLoad]));
const map3 = Object.fromEntries(incidentAvgByStation.map(d => [d.station, d.avg_response_time_per_staff]));

// 4. Compute composite scores with 33.33% weightage each
const compositeScores = [];

allStations1.forEach(station => {
  const v1 = map1[station] ?? 0;  // avg_total
  const v2 = map2[station] ?? 0;  // perStaffLoad
  const v3 = map3[station] ?? 0;  // avg_response_time_per_staff

  // Normalize 0–1 scale
  const norm1 = d1Max !== d1Min ? (v1 - d1Min) / (d1Max - d1Min) : 0;
  const norm2 = d2Max !== d2Min ? (v2 - d2Min) / (d2Max - d2Min) : 0;
  const norm3 = d3Max !== d3Min ? 1 - (v3 - d3Min) / (d3Max - d3Min) : 0; // invert response time

  // Composite score with 33.33% weight each
  const composite_score = Number((norm1 * 0.20 + norm2 * 0.60 + norm3 * 0.20).toFixed(3));

  compositeScores.push({ station, composite_score });
});

// 5. Sort by composite_score descending
compositeScores.sort((a, b) => b.composite_score - a.composite_score);

// Sort ascending by composite_score
compositeScores.sort((a, b) => a.composite_score - b.composite_score);
// display(compositeScores);
```

```js
// Sort descending by composite_score for axis ordering
const sortedStations1 = compositeScores
  .slice()
  .sort((a, b) => b.composite_score - a.composite_score);
```

<div class="card">
${Plot.plot({
  marks: [
    Plot.barX(sortedStations1, {
      y: "station",
      x: "composite_score",
      fill: d => d === sortedStations1[0] ? "red" : "lightgrey", // top station red
      tip: true,
      title: d => `${d.station}: ${d.composite_score}` // tooltip
    })
  ],
  x: {
    grid: true,
    label: "Composite Score",
    tickFormat: ".2f",
    domain: [0, 1]
  },
  y: {
    label: "Station",
    domain: sortedStations1.slice(0,10).map(d => d.station) // set y-axis order
  },
  height: 400,
  width: 800,
  marginLeft: 150
})
}
</div>

<b>FINDING:</b> Based on the composite score computed for each station with a weightage of 60% for total expected attendance for upcoming events in 2026, 20% for average ridership and 20% for average incident response time, the station that can be prioritized for increased staffing is <b>Canal St</b>.

<!-- ```js
//Q3. - Earlier attempt with computation of per staff load in plot.plot. Above is simplified to precompute per staff load before plotting.
const eventsWithStaff = upcoming_events.map(d => ({
  ...d,  // Copy all original fields (date, event_name, nearby_station, estimated_attendance)
  staff_count: currentStaffing[d.nearby_station] || 0  // Add staff_count from lookup
}));
// display(eventsWithStaff)
```

```js
// Compute y-values per station
const stationValues = Object.entries(
  eventsWithStaff.reduce((acc, d) => {
    const total = acc[d.nearby_station]?.totalAttendance || 0;
    const staffCount = d.staff_count;
    acc[d.nearby_station] = {
      totalAttendance: total + (d.expected_attendance || 0),
      staffCount: staffCount // assuming same for all in group
    };
    return acc;
  }, {})
).map(([station, {totalAttendance, staffCount}]) => ({
  station,
  y: staffCount > 0 ? Math.ceil(totalAttendance / staffCount) : 0
}));
// display(stationValues)
```

```js
// Sort stations by y descending
const sortedStations = stationValues.sort((a, b) => b.y - a.y).map(d => d.station);
const top3Stations = sortedStations.slice(0, 3);
```

```js
// Create the plot
Plot.plot({
  marginLeft: 80,
  marginBottom: 100,
  title: "Projected Staffing Load for Upcoming Events in 2026",
  x: {
    label: "Station",
    tickRotate: 90,
    grid: true,
    domain: sortedStations
  },
  y: {
    label: "Projected Load Per Staff"
  },
  marks: [
    Plot.frame(),
    Plot.barY(eventsWithStaff, Plot.groupX(
      {
        y: values => {
          const totalAttendance = values.reduce((sum, d) => sum + (d.expected_attendance || 0), 0);
          const staffCount = values[0].staff_count;
          return staffCount > 0 ? Math.ceil(totalAttendance / staffCount) : 0;
        },
        fill: values => (top3Stations.includes(values[0].nearby_station) ? "crimson" : "#bbb"),
        tip: values => {
          const totalAttendance = values.reduce((sum, d) => sum + (d.expected_attendance || 0), 0);
          const staffCount = values[0].staff_count;
          const load = staffCount > 0 ? Math.ceil(totalAttendance / staffCount) : 0;
          return `${values[0].nearby_station}: ${load} per staff`;
        }
      },
      { x: "nearby_station" }
    ))
  ]
})
``` -->

<!-- ```js
// Q3 - First attempt
Plot.plot({
  marginLeft: 80,
  marginBottom: 100,
  x: {
    label: "Station",
    tickRotate: 90,
    grid: true
  },
  y: {
    label: "Current Staffing Level Load"
  },
  marks: [
    Plot.frame(),
    Plot.barY(eventsWithStaff, Plot.groupX(
      {
        y: (values) => {
          const totalAttendance = values.reduce((sum, d) => sum + (d.expected_attendance || 0), 0);
          const staffCount = values[0].staff_count;
          return staffCount > 0 ? totalAttendance / staffCount : 0;
        },
      },
      {
        x: "nearby_station",
        fill: "#4f46e5",
        tip: true
      }
    ))
      ]
})
``` -->