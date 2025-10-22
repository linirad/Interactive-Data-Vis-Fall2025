---
title: "Lab 1: Passing Pollinators"
toc: true
---

# Lab 1: Report on Pollinator Trends
<!--  Fetch pollinator data -->

```js
const pollinators = FileAttachment("./data/pollinator_activity_data.csv").csv({ typed: true })
```
<br>

## Dataset Used

```js
Inputs.table(pollinators)
```
<br>

## Section 1: Body Mass and Wing Span Distribution

```js
Plot.plot({
  marks: [
    Plot.frame(),
    Plot.dot(pollinators, {
      y: "avg_wing_span_mm", 
      x: "avg_body_mass_g", 
      stroke: "pollinator_species", tip: true
    })
  ],
   x: {ticks: d3.range(0, 0.6, 0.1),
        domain: [0, 0.60],
        label: "Avg Body Mass (g)",
        },
    y: {ticks: d3.range(10, 60, 5),
        domain: [10, 60],
        label: "Avg Wingspan (mm)",
    },   
    title: "Species-wise Body Mass and Wing Span Distribution",
    color: {legend: true},
})
```
<br>
Based on the plot, the highest variation in the average body mass and wing span distribution is found in the carpenter bee followed by the bumblebee and the honeybee.
<br>

## Section 2: Ideal Weather for Pollinating

#### Credit: Thanks to Joseph Borri. Studied and Adapted from Joseph's lab.
<br>

```js
Plot.plot({
  grid: true,
  marginRight: 60,
  facet: {label: null}, 
      y: {ticks: d3.range(0, 30, 2),
          domain: [0, 30],
          label: "Number of Visits"
         },
      x: {ticks: d3.range(0, 100, 10),
          domain: [0, 100],
         },  
    color: {legend: true}, 
    title: "2.1 Plot-wise Pollinator Visits by Weather Conditions",
  marks: [
    Plot.frame(),
    Plot.ruleX([0]),
    Plot.dot(pollinators, {
      x: xvariable,
      y: "visit_count",
      fy: "location",
      stroke: "weather_condition",
      sort: { x: "x", reverse: false, reduce: "median", order: "descending" },
      sort: { y: "y", reverse: true, order: "ascending" },
      tip: true,
    })
  ]
})
```

```js
const xvariable = view(Inputs.select(
  ["temperature", "humidity", "wind_speed", "observation_hour"],
  {label: "x-axis", value: "temperature"}
));
```
<br>

```js
//1. Bar chart: Visit Count by Weather Condition
Plot.plot({
  height: 200,
  y: {
    grid: true
  },
  marks: [
    Plot.frame(),
    Plot.barY(pollinators, {x: "weather_condition", y: "visit_count", aggregate: "mean", fill: "#69b3a2"})
  ],
  width: 600, // Set the width of the chart
  height: 400, // Set the height of the chart
  y: { label: "Average Visit Count" },
  x: { label: "Weather Condition" },
  title: "2.2 Pollinator Visits by Weather Condition"
})
```
<br>

```js

// 2. Scatter plot: Temperature vs Visit Count
// Plot.plot({
//   marks: [
//     Plot.dot(pollinators, {
//       x: "temperature",
//       y: "visit_count",
//       tip: true
//     })

//   ],
//   x: { label: "Temperature (°C)" },
//   y: { label: "Visit Count" },
//   title: "2.2 Temperature vs Pollinator Visits"
// })
```

```js
// 3. Scatter plot: Wind Speed vs Visit Count
// Plot.plot({
//   marks: [
//       Plot.dot(pollinators, {
//       x: "wind_speed",
//       y: "visit_count",
//       tip: true
//     })

//   ],
//   x: { label: "Wind Speed (km/h)" },
//   y: { label: "Visit Count" },
//   title: "2.3 Wind Speed vs Pollinator Visits"
// })
```

```js
// 4. Scatter plot: Humidity vs Visit Count
// Plot.plot({
//   marks: [
//     Plot.dot(pollinators, { 
//       x: "humidity",
//       y: "visit_count", 
//       tip: true
//     })
//   ],
//   x: { label: "Humidity (%)" },
//   y: { label: "Visit Count" },
//   title: "2.4 Humidity vs Pollinator Visits"
// })
```

The ideal weather conditions for pollination are listed below and the plots show that the bees prefer:
<ul>
  <li> cloudy or partly cloudy days, </li>
  <li> temperature in the range of 16-29 degree celcius, </li>
  <li> lower wind speed, less than 5 km/h and</li>
  <li> humidity in the range of 60 to 90%</li>
</ul>

<br>

## Section 3: Flower with Most Nectar Production

```js
Plot.plot({
  height: 200,
  y: {
    domain: [0, 100],
    grid: true
  },
  marks: [
    Plot.frame(),
    Plot.barY(pollinators, {x: "flower_species", y: "nectar_production", aggregate: "mean", fill: "#69b3a2"})
  ],
  width: 600, // Set the width of the chart
  height: 400, // Set the height of the chart
  title: "Flower with Most Nectar Production"
})
```

Based on the plot, the flower with the most nectar production is the sunflower followed by the coneflower and Lavender.



