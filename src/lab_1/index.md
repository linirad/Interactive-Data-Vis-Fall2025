---
# title: "Lab 1: Passing Pollinators "
toc: true
---

# Lab 1: Report on Pollinator Trends &#x1F41D; 

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
      stroke: "pollinator_species", 
      tip: true
    })
  ],
   height: 300,
   x: {
        domain: [0, 0.60],
        label: "Avg Body Mass (g)",
        },
    y: {
        domain: [10, 60],
        label: "Avg Wingspan (mm)",
    },   
    title: "Species-wise Body Mass and Wing Span Distribution",
    color: {legend: true},
})
```
<br>


## Section 2: Ideal Weather for Pollinating

<br>

<!-- ```js
//Bar chart: Visit Count by Weather Condition
Plot.plot({
  marks: [
    Plot.frame(),
    Plot.barY(pollinators, {
      x: "weather_condition", 
      y: "visit_count", 
      aggregate: "mean", 
      fill: "weather_condition"})
  ],
  color: { legend: true },
  height: 300, // Set the height of the chart
  y: { label: "Average Visit Count", grid: true},
  x: { label: "Weather Condition" },
  title: "2.1 Pollinator Visits by Weather Condition"
})
``` -->

```js
Plot.plot({
  marks: [
    Plot.frame(),
    Plot.barY(pollinators, 
     Plot.groupX(
      { y: "mean" },
      {
        x: "weather_condition", 
        y: "visit_count", 
        fill: "weather_condition",
        tip: true
      }
      ))
  ],
  color: { legend: true },
  height: 300, // Set the height of the chart
  y: { label: "Average Visit Count", grid: true},
  x: { label: "Weather Condition" },
  title: "2.1 Pollinator Visits by Weather Condition"
})
```

```js
// Temperature plot
Plot.plot({
  marks: [
    Plot.frame(),
    Plot.rectY(pollinators, 
    Plot.binX(
      {y: "sum"}, 
      {x: "temperature", y: "visit_count", 
       fill: "weather_condition", 
       tip: true})),
  ],
  color: { legend: true },
  height: 300,
  y: {domain: [0, 400], label: "Visit Count (sum)", grid: true},
  x: {domain: [0, 50], label: "Temperature (°C)"},
  title: "2.2 Temperature"
})
```

```js
// Humidity plot
Plot.plot({
  marks: [
    Plot.frame(),
    Plot.rectY(pollinators, 
    Plot.binX(
      {y: "sum"}, 
      {x: "humidity", y: "visit_count", fill: "weather_condition", tip: true})),
  ],
  color: { legend: true },
  height: 300,
  y: {domain: [0, 400], label: "Visit Count (sum)", grid: true},
  x: {domain: [0, 100], label: "Humidity (%)"},
  title: "2.3 Humidity"
})
```

```js
// Wind speed plot
Plot.plot({
  marks: [
    Plot.frame(),
    Plot.rectY(pollinators, 
    Plot.binX(
      {y: "sum"}, 
      {x: "wind_speed", y: "visit_count", fill: "weather_condition", tip: true})),
  ],
  color: { legend: true },
  height: 300,
  y: {domain: [0, 500], label: "Visit Count (sum)", grid: true},
  x: {domain: [0, 50], label: "Wind Speed (km/h)"},
  title: "2.4 Wind Speed"
})
```

```js
// Observation hour plot
Plot.plot({
  marks: [
    Plot.frame(),
    Plot.rectY(pollinators, 
    Plot.binX(
      {y: "count"}, 
      {x: "observation_hour", y: "visit_count", fill: "weather_condition", tip: true})),
  ],
  color: { legend: true },
  height: 300,
  y: {domain: [0, 70], label: "Visit Count (sum)", grid: true},
  x: {domain: [0, 50], label: "Observation hour"},
  title: "2.5 Observation Hour"
})
```

<br>

## Section 3: Flower with Most Nectar Production


```js
// Flower species vs nectar production plot
Plot.plot({
  marks: [
    Plot.frame(),
    Plot.barY(pollinators, 
     Plot.groupX(
      { y: "mean" },
      {
        x: "flower_species", 
        y: "nectar_production", 
        fill: "flower_species",
        sort: { x: "-y"},
        tip: true
      }
      ))
  ],
  color: { legend: true },
  height: 300, // Set the height of the chart
  y: { label: "Nectar Production (μL)", grid: true},
  x: { label: "Flower Species" },
  title: "Flower with Most Nectar Production"
})
```

<!-- ```js
// Flower species vs nectar production plot
Plot.plot({
  marks: [
    Plot.frame(),
    Plot.barY(pollinators, {
      x: "flower_species", 
      y: "nectar_production", 
      aggregate: "mean", 
      fill: "#69b3a2", 
      sort: { x: "-y"}, 
      tip: true}) 
  ],
  height: 300, // Set the height of the chart
  x: { label: "Flower Species" },
  y: { label: "Nectar Production (μL)", domain: [0, 100],
    grid: true},
  title: "Flower with Most Nectar Production"
})
``` -->

Based on the dashboard, listed below are the section wise results: 
<ul>
<li> <b>Section 1:</b> The highest variation in the average body mass and wing span distribution is found in the carpenter bee followed by the bumblebee and the honeybee.</li>
<li><b>Section 2:</b> The ideal weather conditions for pollination are listed below and the plots show that bees prefer:</li>
<ul>
  <li> cloudy or partly cloudy days, </li>
  <li> warmer temperatures in the range of 16-29 degree celcius, </li>
  <li> humidity in the range of 60 to 90%,</li>
  <li> lower wind speeds and</li>
  <li> working during the day (8 am to 8 pm) </li>
</ul>
<li><b>Section 3:</b> The flower with the most nectar production is the sunflower followed by the coneflower and Lavender.
</li>
</ul>


```js
// Alternate Section 2 plot with dropdown for weather conditions plots. Could not create the distinct plot especially of wind speed that is achieved here in my attempts with binX
// Plot.plot({
//   grid: true,
//   marginRight: 60,
//   facet: {label: null}, 
//       y: {
//           domain: [0, 30],
//           label: "Number of Visits"
//          },
//       x: {
//           domain: [0, 100],
//          },  
//     color: {legend: true}, 
//     title: "2.1 Plot-wise Pollinator Visits by Weather Conditions",
//   marks: [
//     Plot.frame(),
//     Plot.ruleX([0]),
//     Plot.dot(pollinators, {
//       x: xvariable,
//       y: "visit_count",
//       fy: "location",
//       stroke: "weather_condition",
//       sort: { x: "x", reverse: false, reduce: "median", order: "descending" },
//       sort: { y: "y", reverse: true, order: "ascending" },
//       tip: true,
//     })
//   ]
// })
```

```js
// const xvariable = view(Inputs.select(
//   ["temperature", "humidity", "wind_speed", "observation_hour"],
//   {label: "x-axis", value: "temperature"}
// ));
```


```js
// Initial experiments with scatter plots. Difficult to read and reach findings
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

```js
// Attempt with groupX - not readable to reach a conclusion
// Plot.plot({
//   marks: [
//     Plot.barY(pollinators, 
//       Plot.groupX(
//         { y: "count" },
//         { 
//           x: "temperature", 
//           fill: "weather_condition", 
//         }
//       )
//     )
//   ],
//   color: { legend: true }
// })
```