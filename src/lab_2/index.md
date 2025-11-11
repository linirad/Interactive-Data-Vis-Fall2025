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
const allStations = Array.from(new Set(ridership.map(d => d.station)))
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
const selectedStationData = ridership.filter(d => d.station === anotherSelectedStation).map(d => ({
    ...d,  // Keep all existing fields
    total_traffic: (d.entrances || 0) + (d.exits || 0)
  })) 
const selectedEvents = local_events.filter(d => d["nearby_station"].includes(anotherSelectedStation))
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
Plot.plot({
  height: 300, 
  width,
  title: "Station-wise Total Traffic",
  marks: [
    Plot.frame(),
    // the line based on the filtered (selected) station data
    Plot.line(selectedStationData, {
      x: "date",
      y: "total_traffic",
      // tip: true
    }),
    // dots for the local events that correspond to the selected station
    Plot.dot(selectedEvents, {
      x: "date", // position at the event date
      y: eventDataObj => {
        // console.log("event data:", eventDataObj)
        // find the station data for this date
        const stationData = selectedStationData.find(stationDataObj => { 
          // console.log("station data:", stationDataObj)
          // which station data matches this event data date?
          return eventDataObj.date.toDateString() === stationDataObj.date.toDateString()
        })
        return stationData ? stationData.total_traffic : null;
      },
      stroke: "blue", 
      fill: "white",
      tip: true,
      channels: {
        "Event": "event_name", 
        "Total traffic": "total_traffic"
      }
    }),
    Plot.tip([popularEventWithTraffic], {
      x: popularEventWithTraffic.date,
      y: popularEventWithTraffic.total_traffic,
      channels: {
        "Most Popular Event": "event_name", 
      }
    }),
    Plot.ruleY([0]),
     Plot.ruleX([fareChange], { stroke: "red"}),
     Plot.text([fareChange], {
      x: d => d,
      y: 0,
      // dy: -100,
      // dx: 37, 
      text: () => "Fare increase",
      // rotate: -90,        // rotate text vertically
      textAnchor: "end",  // aligns bottom of text to y=0
      dx: 0,            // move left/right as needed
      dy: -10,              // adjust vertical position if needed
      fill: "red",
      fontWeight: "bold"
      // anchor: "bottom"
    })
  ]
})
```

<br>

<h2 style="white-space: nowrap;">2. Compare response times across stations</h3>

```js
Plot.plot({
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
      tip: true
    }))
  ]
})
```

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
    avg: rows.reduce((sum, r) => sum + (+r.response_time_minutes || 0), 0) / rows.length // Divide total by count
  }))
  .sort((a, b) => a.station.localeCompare(b.station) || a.avg - b.avg); // Sort by station name first then avg
```

```js
Plot.plot({
  marginLeft: 80,
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
```



```js
Plot.plot({
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
})
  ```
