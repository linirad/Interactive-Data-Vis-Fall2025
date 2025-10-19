---
title: "Lab 1: Passing Pollinators"
toc: true
---

# Lab 1: Report on Pollinator Trends
<!--  Fetch pollinator data -->

```js
const pollinator = FileAttachment("./data/pollinator_activity_data.csv").csv({ typed: true })
```

```js
//Inputs.table(pollinator)
```
<br>

## Section 1: Body Mass and Wing Span Distribution

```js
Plot.plot({
  marks: [
    Plot.dot(pollinator, {
      x: "avg_wing_span_mm",
      y: "avg_body_mass_g",
      stroke: "pollinator_species", tip: true
    })
  ],
  y: { label: "Average Body Mass" },
  x: { label: "Average Wing Span" },
  title: "Species-wise Body Mass and Wing Span Distribution"
})
```
<br>
Based on the plot, the average body mass and wing span distribution for the different species are listed below:
<ul>
  <li> For the honeybee, around 19 to 0.096; </li>
  <li> For the bumblebee, around 34 to 0.26; </li>
  <li> For the carpenter bee, around 41 to 0.44 </li>
</ul> 

<br>

## Section 2: Ideal Weather for Pollinating

```js
// 1. Bar chart: Visit Count by Weather Condition
Plot.plot({
  height: 200,
  y: {
    grid: true
  },
  marks: [
    Plot.barY(pollinator, {x: "weather_condition", y: "visit_count", aggregate: "mean", fill: "#69b3a2"})
  ],
  width: 600, // Set the width of the chart
  height: 400, // Set the height of the chart
  y: { label: "Average Visit Count" },
  x: { label: "Weather Condition" },
  title: "2.1 Pollinator Visits by Weather Condition"
})
```

```js
// 2. Scatter plot: Temperature vs Visit Count
Plot.plot({
  marks: [
    Plot.dot(pollinator, {
      x: "temperature",
      y: "visit_count",
      tip: true
    })

  ],
  x: { label: "Temperature (°C)" },
  y: { label: "Visit Count" },
  title: "2.2 Temperature vs Pollinator Visits"
})
```

```js
// 3. Scatter plot: Wind Speed vs Visit Count
Plot.plot({
  marks: [
      Plot.dot(pollinator, {
      x: "wind_speed",
      y: "visit_count",
      tip: true
    })

  ],
  x: { label: "Wind Speed (km/h)" },
  y: { label: "Visit Count" },
  title: "2.3 Wind Speed vs Pollinator Visits"
})
```

```js
// 4. Scatter plot: Humidity vs Visit Count
Plot.plot({
  marks: [
    Plot.dot(pollinator, { 
      x: "humidity",
      y: "visit_count", 
      tip: true
    })
  ],
  x: { label: "Humidity (%)" },
  y: { label: "Visit Count" },
  title: "2.4 Humidity vs Pollinator Visits"
})
```

Based on the plots 2.1 to 2.4, the ideal weather conditions for pollination are listed below:
<ul>
  <li> the most visits happen on cloudy or partly cloudy days </li>
  <li> relatively more visit when the temperature is over 21 degree celcius </li>
  <li> more visits when the wind speed is less than 5 km/h </li>
  <li> humidity does not appear to be a factor and the visits are evenly distributed across the range</li>
</ul>

<br>

## Section 3: Flower with Most Nectar Production

```js
Plot.plot({
  height: 200,
  y: {
    grid: true
  },
  marks: [
    Plot.barY(pollinator, {x: "flower_species", y: "nectar_production", aggregate: "mean", fill: "#69b3a2"})
  ],
  width: 600, // Set the width of the chart
  height: 400, // Set the height of the chart
  title: "Flower with Most Nectar Production"
})
```

Based on the plot, the flower with the most nectar production is the sunflower followed by the coneflower and Lavender.
