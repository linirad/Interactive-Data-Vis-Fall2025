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

// NYC geoJSON data
// display(nyc)
// Campaign data (first 10 objects)
// display(results.slice(0,10))
// display(survey)
// display(events.slice(0,10))
```

<div class="card" style="background-color: #f9f7f5fc; padding: 15px; width: 100%; box-sizing: border-box; display: block;">
  <h1 style="display: block; width: 100%; max-width: 100%; box-sizing: border-box; text-align: left; margin-bottom: 10px;">
    Lab 3: Mayoral Mystery
  </h1> 
  The dashboard presents an analysis of the data from the recent election cycle collected by the political campaign that includes election results, survey responses to proposed policies and campaign event data. The analysis shows the vote distribution for the candidate vs the opponent, the net support for policies amongst the voters and the potential impact of the Get Out to Vote effort on the turnout rate, the candidate's attempts to connect with the communities and the campaign events. Included are some suggestions to improve the voter turnout rate and mobilize more support for the candidate.
</div>

```js
// The nyc file is saved in data as a topoJSON instead of a geoJSON. Thats primarily for size reasons -- it saves us 3MB of data. For Plot to render it, we have to convert it back to its geoJSON feature collection. 
const districts = topojson.feature(nyc, nyc.objects.districts)
// display(districts)
```

```js
// Compute margin for each district
const margins = results.map(d => d.votes_candidate - d.votes_opponent);

// Use d3.extent to get min and max
const [minMargin, maxMarginRaw] = d3.extent(margins);

// Symmetric max for diverging scale
const maxMargin = Math.max(Math.abs(minMargin), Math.abs(maxMarginRaw));

// Calculate overall totals across all rows
const overallTotalVotes = results.reduce((sum, d) => sum + d.votes_candidate + d.votes_opponent, 0);
const overallCandidateVotes = results.reduce((sum, d) => sum + d.votes_candidate, 0);
const overallOpponentVotes = results.reduce((sum, d) => sum + d.votes_opponent, 0);

// Calculate overall percentages
const overallCandidateShare = parseFloat(((overallCandidateVotes / overallTotalVotes) * 100).toFixed(2));
const overallOpponentShare = parseFloat(((overallOpponentVotes / overallTotalVotes) * 100).toFixed(2));
```


<!-- <div class="card" style="background-color: #e9e2dea4;">
  <h1 style="text-align: left; margin-bottom: 10px;">
  Vote Distribution by Income Category and GOTV Effort</h1><p>The candidate received majority votes from middle and low income neighborhoods, while the opponent received majority support from high income neighborhoods. The divide is glaringly apparent in the plot. The boros in red flagging majority votes for the opponent are consistently high income neighborhoods as shown by the labels. The circle is used to indicate the number of doors knocked and the increasing radius shows that lower income neighborhoods saw more solicitation in this form.</p>
  <hr style="border: none; border-top: 1px solid #ccc; margin: 15px 0;">
  <div style="font-size: 13px; margin: 10px 0;">
    <div style="margin-bottom: 5px;">
      <span style="display:inline-block; width:14px; height:14px; border-radius:50%; 
                  border:1px solid #e30e0eff; background:#e30e0e22; margin-right:6px;"></span>
      <b>Circle size = GOTV doors knocked</b> (more doors = larger circle)
    </div>
    <div>
      <span style="font-weight:bold;">Income Category Labels:</span>
      <span style="margin-left:8px;">"High", "Middle", "Low" text appears directly on the map</span>
    </div>
</div> -->

<div class="card" style="background-color: #f9f7f5fc; padding: 15px; width: 100%; box-sizing: border-box; display: block;">
  <h1 style="display: block; width: 100%; max-width: 100%; box-sizing: border-box; text-align: left; margin-bottom: 10px;">
    Vote Distribution by Income Category and GOTV Effort
  </h1>  
${Plot.plot({
  width: 800,
  height: 600,
  projection: { domain: districts, type: "mercator" },
  color: {
    type: "diverging",
    scheme: "RdBu",
    domain: [-maxMargin, maxMargin],
    label: "Vote Margin (Candidate − Opponent)",
    legend: true
  },
  marks: [
    Plot.geo(districts, {
      fill: d => {
        // Find result for this district
        const r = results.find(r => r.boro_cd === d.properties.BoroCD);
        return r ? r.votes_candidate - r.votes_opponent : 0;
      },
      strokeWidth: 1.5,
      tip: true,
      channels: {
        "Borough District": d => d.properties.BoroCD,
        "Median Household Income:": d => results.find(r => r.boro_cd === d.properties.BoroCD)?.median_household_income ?? "N/A",
        "Income Category:": d => results.find(r => r.boro_cd === d.properties.BoroCD)?.income_category ?? "N/A",
        "Candidate Votes": d => results.find(r => r.boro_cd === d.properties.BoroCD)?.votes_candidate ?? 0,
        "Opponent Votes": d => results.find(r => r.boro_cd === d.properties.BoroCD)?.votes_opponent ?? 0,
        "Margin": d => results.find(r => r.boro_cd === d.properties.BoroCD)?.votes_candidate - results.find(r => r.boro_cd === d.properties.BoroCD)?.votes_opponent ?? 0,
        "GOTV Doors Knocked": d => results.find(r => r.boro_cd === d.properties.BoroCD)?.gotv_doors_knocked ?? 0
      },
    }),
    // Add circles sized by doors knocked
    Plot.dot(districts.features, {
      x: d => d3.geoCentroid(d)[0],
      y: d => d3.geoCentroid(d)[1],
      r: d => {
        const r = results.find(r => r.boro_cd === d.properties.BoroCD);
        return r?.gotv_doors_knocked ?? 0;
      },
      //fill: "white",
      fillOpacity: 0.6,
      stroke: "#060505ff",
      strokeWidth: 1,
    }),
    Plot.text(districts.features, {
      x: d => d3.geoCentroid(d)[0],
      y: d => d3.geoCentroid(d)[1],
      text: d => results.find(r => r.boro_cd === d.properties.BoroCD)?.income_category ?? "",
      fontSize: 7,
      //fill: "black",
      fill: d => {
        const cat = results.find(r => r.boro_cd === d.properties.BoroCD)?.income_category;
        if (cat === "High") return "#070606ff";    // red
        if (cat === "Middle") return "#ec370fff";  // yellow
        if (cat === "Low") return "#410dedff";     // blue
        return "black";                          // fallback
      },
      stroke: "black",
      strokeWidth: 0.1,
      textAnchor: "middle"
    })
  ]
})}
  <div style="font-size: 13px; margin: 10px 0; width: 100%; max-width: 100%; box-sizing: border-box;">
    <div style="display: flex; align-items: center; margin-bottom: 5px; flex-wrap: wrap;">
      <span style="width:14px; height:14px; border-radius:50%; 
                   border:1px solid #090808ff; margin-right:6px; flex-shrink:0;"></span>
      <span style="flex: 1 1 auto; min-width: 0;">
        <b>Circle size = GOTV doors knocked</b> (more doors = larger circle)
      </span>
    </div>
    <div style="flex-wrap: wrap;">
      <span style="font-weight:bold;">Income Category Labels:</span>
      <span style="margin-left:8px;color:#070606ff">"High"</span>, "<span style="color:#ec370fff">Middle</span>", "<span style="color:#410dedff">Low</span>" appears directly on the map</span>
    </div>
    <hr style="border: none; border-top: 1px solid #ccc; margin: 15px 0;"> 
    <h2><b>FINDINGS</b></h2>
    <p style="margin-bottom: 15px; width: 100%; max-width: 100%; box-sizing: border-box;">
    The candidate got <b>${overallCandidateShare}%</b> of the vote (${overallCandidateVotes.toLocaleString()} votes), while the opponent received ${overallOpponentShare}% (${overallOpponentVotes.toLocaleString()} votes).
    Total votes cast: ${overallTotalVotes.toLocaleString()}
    The candidate received majority votes from middle and low income neighborhoods, while the opponent received majority support from high income neighborhoods. The divide is glaringly apparent in the plot. The boros in red flagging majority votes for the opponent are consistently high income neighborhoods as shown by the labels. The circle is used to indicate the number of doors knocked and the increasing radius shows that lower income neighborhoods saw more solicitation in this form. Judging by the vote distribution, there appears to be major ideological differences between the candidate's governance approach and expectations of the affluent classes. So let us look at the net support for the candidates and opponents policy stances. 
  </p>
  </div>
</div>


<!-- ```js
// Earlier attempt with unnecessary Maps modified in next iteration to directly reference results
const resultsCandidate = new Map(results.map(d => [d.boro_cd, d.votes_candidate]));
const resultsOpponent = new Map(results.map(d => [d.boro_cd, d.votes_opponent]));
const medianIncome = new Map(results.map(d => [d.boro_cd, d.median_household_income]));
const incomeCategoryMap = new Map(results.map(d => [d.boro_cd, d.income_category]));

// 1️⃣ Compute margin (candidate − opponent) for each district
const margin = new Map(
  results.map(d => [d.boro_cd, d.votes_candidate - d.votes_opponent])
);

// 2️⃣ Compute symmetric domain for diverging scale using d3.extent
const [minMargin, maxMarginRaw] = d3.extent(results, d => d.votes_candidate - d.votes_opponent);
const maxMargin = Math.max(Math.abs(minMargin), Math.abs(maxMarginRaw));
```


<div class="card" style="background-color: #9d806c2f;" >
The candidate received majority votes from middle and low income neightborhoods, while the opponent received majority support from high income neighborhoods. The divide is glaringly apparent in the plot.
<hr style="border: none; border-top: 1px solid #ccc; margin: 15px 0;">
${Plot.plot({
  width: 800,
  height: 600,
  projection: {
    domain: districts,
    type: "mercator",
  },
  color: {
    type: "diverging",
    scheme: "RdBu",
    domain: [-maxMargin, maxMargin], // symmetric around zero
    label: "Vote Margin (Candidate − Opponent)",
    legend: true
  },
  marks: [
    Plot.frame(),
    Plot.geo(districts, {
      fill: (d, i) => {
        const rate = margin.get(d.properties.BoroCD)
        return rate
      },
        // stroke: d => {
        // const cat = incomeCategoryMap.get(d.properties.BoroCD);
        // return cat === "Low" ? "#c6dbef" :
        //       cat === "Middle" ? "#6baed6" :
        //       cat === "High" ? "#000" : "#ccc";
      //},
      strokeWidth: 1.5,
      tip: true, 
      channels: { 
        "Borough District": d => d.properties.BoroCD,
        "Median Household Income:": d => medianIncome.get(d.properties.BoroCD),
        "Income Category:": d => incomeCategoryMap.get(d.properties.BoroCD),
        "Candidate Votes": d => resultsCandidate.get(d.properties.BoroCD),
        "Opponent Votes": d => resultsOpponent.get(d.properties.BoroCD),
        "Margin": d => margin.get(d.properties.BoroCD)
        } 
    }), 
    // Add income category labels at each district centroid
    Plot.text(districts.features, {
      x: d => d3.geoCentroid(d)[0],
      y: d => d3.geoCentroid(d)[1],
      text: d => incomeCategoryMap.get(d.properties.BoroCD),
      fontSize: 7,
      fill: "black",
      stroke: "black",
      strokeWidth: 0.1,
      textAnchor: "middle",
    })
  ]
})
}
</div> -->


```js
// Code to arrive at diverging policy net_support with guidance from Claude
const voted = survey.filter(d => d.voted === "Yes" && d.voted_for);

const policies = [
  { key: 'police_reform_alignment', label: 'Police Reform' },
  { key: 'small_business_tax_alignment', label: 'Small Business Tax' },
  { key: 'childcare_support_alignment', label: 'Childcare Support' },
  { key: 'public_transit_alignment', label: 'Public Transit' },
  { key: 'affordable_housing_alignment', label: 'Affordable Housing' }
];

const candidates = ["Candidate", "Opponent"];

const policyDiverging = policies.flatMap(policy =>
  candidates.map(candidate => {
    const group = voted.filter(d => d.voted_for === candidate);
    const responses = group.filter(d => Number.isFinite(d[policy.key]));
    const total = responses.length;
    const high = responses.filter(d => d[policy.key] >= 4).length;
    const low = responses.filter(d => d[policy.key] <= 2).length;
    const net = total ? (high / total) - (low / total) : 0;

    return {
      policy: policy.label,
      candidate,
      // net_support: candidate === "Candidate" ? net : -net,
      net_support: net,
      positive_pct: total ? high / total : 0,
      negative_pct: total ? low / total : 0,
      neutral_pct: total ? (total - high - low) / total : 0,
      count: total
    };
  })
);
```

<div class="card" style="background-color: #f9f7f5fc;" >
  <h1 style="display: block; width: 100%; max-width: 100%; box-sizing: border-box; text-align: left; margin-bottom: 10px;">
    Net Support for Policies by Voter Group
  </h1>
${Plot.plot({
  marginLeft: 180,
  marginRight: 20,
  height: 350,
  width,
  grid: true,
  facet: {
    data: policyDiverging,
    y: "candidate",
    marginRight: 80  // Space for candidate labels
  },
  fy: {
    label: null,
    domain: ["Candidate", "Opponent"]
  },
  x: {
    label: "Net Support (Support - Opposition)",
    labelAnchor: "center",
    tickFormat: d => d > 0 ? `+${d}` : d,
    domain: [-1.1, 1.1]
  },
  y: {
    label: null,
    domain: [
      'Police Reform',
      'Public Transit', 
      'Affordable Housing',
      'Childcare Support',
      'Small Business Tax'
    ]
  },
  color: {
    domain: ["Candidate", "Opponent"],
    range: ["#4682b4", "#ec4509ff"],
    legend: true
  },
  marks: [
    Plot.ruleX([0], {stroke: "black", strokeWidth: 1.5}),
    Plot.barX(policyDiverging, {
      y: "policy",
      x: "net_support",
      fill: "candidate",
      tip: true,
      title: d => `${d.policy}
${d.candidate} voters
Net Support: ${(d.net_support >= 0 ? '+' : '') + d.net_support.toFixed(2)}
━━━━━━━━━━━━━━━━━━━
Positive (4-5): ${(d.positive_pct * 100).toFixed(1)}%
Neutral (3): ${(d.neutral_pct * 100).toFixed(1)}%
Negative (1-2): ${(d.negative_pct * 100).toFixed(1)}%
━━━━━━━━━━━━━━━━━━━
Total Respondents: ${d.count}`
    })
  ]
})
}
    <hr style="border: none; border-top: 1px solid #ccc; margin: 15px 0;"> 
    <h2><b>FINDINGS</b></h2>
    <p style="margin-bottom: 15px; width: 100%; max-width: 100%; box-sizing: border-box;">
    The plot visualizes policy preferences among voters who supported the candidate versus the opponent. The net support on all the policies is higher in the candidate supporters as compared to the opponent supporters except for police reform which has no support in either camps.</p> 
    <p style="margin-bottom: 15px; width: 100%; max-width: 100%; box-sizing: border-box;">
    <b>RECOMMENDATION:</b> The campaign must pay attention to the <span style="color:red">survey responses*</span> as they reveal insightful details on voter disengagement especially the bipartisan opposition to the candidate's police reform stance, lack of campaign presence in their areas, lack of engagement with the candidate and the voting process in general. The police reform policy stance appears to be a deal breaker for quite a few of the survey participants, The candidate needs to restrategize their campaign efforts and <b>improve the campaign outreach programs to cover neglected neighborhoods and hold open discussions on candidate policy stances to solicit community feedback and find solutions to gain community buy-in.</b>
    <details>
      <summary><span style="color:red">*Open Responses from Survey Data</span></summary>
      <div style="padding: 20px; border: solid 1px; border-radius: 5px; font-size: 0.86rem;">
        ${display(
          Inputs.table(
            survey.filter(
              d => d.open_response != null 
            ),
            {
              columns: ["open_response", "voted_for"],
              header: {
                open_response: "Open Response Comments",
                voted_for: "Voted For: "
              }
            }
          )
        )}
      </div>
    </details>
  </p>
</div>

```js
  // Calculate overall turnout rate by income category (weighted average)
  const categoryGroups = {};
  
  results.forEach(r => {
    const cat = r.income_category;
    if (!categoryGroups[cat]) {
      categoryGroups[cat] = {
        totalRegistered: 0,
        totalVoted: 0
      };
    }
    
    const totalVoted = r.votes_candidate + r.votes_opponent;
    categoryGroups[cat].totalRegistered += r.total_registered_voters;
    categoryGroups[cat].totalVoted += totalVoted;
  });
  
  const overallTurnout = Object.entries(categoryGroups).map(([category, data]) => ({
    category,
    overallTurnoutRate: (data.totalVoted / data.totalRegistered) * 100
  }));
  // display(overallTurnout);
  const maxDoors = Math.max(...results.map(r => r.gotv_doors_knocked));
  const maxTurnout = Math.max(...results.map(r => r.turnout_rate));
```

<div class="card" style="background-color: #f9f7f5fc;" >
  <h1 style="display: block; width: 100%; max-width: 100%; box-sizing: border-box; text-align: left; margin-bottom: 10px;">
    Potential Impact of Get Out to Vote Effort and Candidate Hours Spent on Turnout Rate
  </h1>
${Plot.plot({
  grid: true,
  x: { label: "GOTV Doors Knocked →" },
  y: { label: "↑ Turnout Rate" },
    symbol: {legend: true},
    marks: [
      Plot.dot(results, {
        x: "gotv_doors_knocked", 
        y: "turnout_rate", 
        stroke: "candidate_hours_spent",  
        symbol: "income_category",
        tip: true
      }),
      // Add horizontal lines for overall turnout by income category
      Plot.ruleY(overallTurnout, {
        y: "overallTurnoutRate",
        stroke: "#070606ff",
        strokeWidth: 1,
        strokeDasharray: "4,4",
        opacity: 0.6
      }),
      // Add always-visible static tips
      Plot.tip(overallTurnout, {
        x: maxDoors * 0.5,
        y: "overallTurnoutRate",
        title: d => `${d.category} Income Category\nOverall Turnout Rate: ${d.overallTurnoutRate.toFixed(1)}%`,
        anchor: "bottom"
      })
  ]
})
}
    <hr style="border: none; border-top: 1px solid #ccc; margin: 15px 0;"> 
    <h2><b>FINDINGS</b></h2>
    <p style="margin-bottom: 15px; width: 100%; max-width: 100%; box-sizing: border-box;">
    Let us check if the GOTV effort and the candidate hours spent had a positive impact on the voter turnout rate. The income category wise analysis of the plot is given below: <ul> <li><b>HIGH INCOME NEIGHBORHOOD:</b> <b>Highest turnout rate</b> despite low numbers of Get Out To Vote (GOTV) doors knocked and although candidate spent highest number of hours in these neighborhoods that has not translated to support for the candidate as seen in earlier plot.</li> <li><b>MIDDLE INCOME NEIGHBORHOOD:</b> <b>Lowest turnout rate</b>, lowest GOTV doors knocked and the candidate has spent the least amount of hours in these neighborhoods that have still shown more support to the candidate as compared to the opponent.</li><li><b>LOW INCOME NEIGHBORHOOD:</b> The overall turnout rate is only around 60 percent which is way lower than high income neighborhoods, despite GOTV doors knocked being highest in low income areas and support for the candidate is highest although the candidate has spent considerably less time here as compared to high income areas.</li> </ul><b>RECOMMENDATION:</b> <ul><li>GOTV Effort: <b>Current approach is ineffective and has to be reconsidered to idenfify possible obstacles to voting booth access and/or disengagement from voting process to improve turnout rate.</b></li><li>CANDIDATE PRESENCE: <b>Restrategize presence in high income neighborhoods</b> to address policy concerns of the residents. Redistribute time spent to <b>increase presence in low and middle income areas</b> where there is more support for the candidate.</li> <li><b>In low and middle income neighborhoods, the candidate should also address the low voter turnout rate.</b></li></ul>  
  </p>
</div>

<div class="card" style="background-color: #f9f7f5fc;" >
  <h1 style="display: block; width: 100%; max-width: 100%; box-sizing: border-box; text-align: left; margin-bottom: 10px;">
    Income Category-wise Estimated Attendance of Events
  </h1>
${Plot.plot({
  x: {axis: null},
  y: {tickFormat: "s", grid: true},
  color: {scheme: "spectral", legend: true},
  marks: [
    Plot.barY(events, Plot.groupX(
      {y: "sum"},
      {
        x: "event_type",
        y: "estimated_attendance",
        fill: "event_type",
        tip: true,
        fx: "income_category",
        sort: {x: null, color: null, fx: {value: "-y", reduce: "sum"}}
      }
    )),
    Plot.ruleY([0])
  ]
})
}
  <hr style="border: none; border-top: 1px solid #ccc; margin: 15px 0;"> 
  <h2><b>FINDINGS</b></h2>
  <p style="margin-bottom: 15px; width: 100%; max-width: 100%; box-sizing: border-box;">
  Finally, let us see what the estimated attendance for different events across the different income neighborhoods reveal. The estimated attendance shows that the response to all the events is expected to be highest in low income neighborhoods and significantly lower in middle as well as high income areas.</p> 
  <p style="margin-bottom: 15px; width: 100%; max-width: 100%; box-sizing: border-box;">
  <b>RECOMMENDATION:</b> Considering the candidate has considerably less support in high income neighborhoods and there are similar concerns around policy stances across the map, the campaign should make a concerted effort to address the concerns in all the neighborhoods and focus on improving the event attendance numbers in middle and high income areas. <b> Informed volunteer training around known issues. Engage in policy specific concerns in community meetings and Town Halls.</b>
  </p>
</div>

<div class="card" style="background-color: #f9f7f5fc;" >
  <h1 style="display: block; width: 100%; max-width: 100%; box-sizing: border-box; text-align: left; margin-bottom: 10px;">
    Recommendation Summary
  </h1>
  <p style="margin-bottom: 15px; width: 100%; max-width: 100%; box-sizing: border-box;">
  <ul>
    <li>Identify cause of low voter turnout in low and middle income neighborhoods through GOTV and informed volunteer training.</li>
    <li>Redistribute time spent and increase candidate presence in low and middle income neighborhoods.</li>
    <li>Address concerns around policy stances across the map through more focused community meetings and town halls.</li>
    <li>Focus campaign outreach on spreading awareness about the candidate and his policies.</li>
  </ul>
  </p>
</div>

<!-- ```js
Plot.plot({
  width,
  marginLeft: 60,
  marks: [
    Plot.barX(latestYear, Plot.groupY(
      { x: "sum" },
      { y: "name", x: "count", fill: "ethnicity",  tip: true, 
        sort: { y: "x", reverse: true, limit: 20 } }
    ))
  ],
})
``` -->

<!-- 
```js
const allResults = results.map(d => ({
  ...d,
  total_votes: d.votes_candidate + d.votes_opponent,
  vote_margin: d.votes_candidate - d.votes_opponent,
  vote_share: 100 * (d.votes_candidate / (d.votes_candidate + d.votes_opponent))
}));
display(allResults)
```

```js
Plot.plot({
  marginLeft: 80,
  grid: true,
  color: { legend: true },
  y: {label: "Income Category"},
  color: {scheme: "pubugn"},
  marks: [
    Plot.barX(allResults, 
      Plot.groupY(
        { x: "count" },  // reducer
        { x: "income_category", y: "boro_cd", fill: "vote_share", tip: true } // options
      )),
    Plot.frame()
  ]
})
``` -->


<!-- ```js
const names = await FileAttachment("data/topNames.csv").csv({ typed: true });
display(names.slice(0, 10))
```

```js
const latestYear = names.filter(d => d.year === 2021);
display(latestYear);
const uniqueNames = Array.from(new Set(latestYear.map(name => name.name)));
display(uniqueNames);
const nameCounts = latestYear.reduce((acc, value) => ({
  ...acc, // keep existing work
  [value.name]: (acc[value.name] || 0) + value.rank
}), {});
const rankSum = Object.entries(nameCounts)
  .map((d) => ({ name: d[0], rankSum: d[1] }))
  .sort((a, b) => a.rankSum - b.rankSum);
display(rankSum)
```


```js
Plot.plot({
  width,
  marginLeft: 60,
  marks: [
    Plot.barX(latestYear, Plot.groupY(
      { x: "sum" },
      { y: "name", x: "count", fill: "ethnicity",  tip: true, 
        sort: { y: "x", reverse: true, limit: 20 } }
    ))
  ],
})
``` -->

<!-- ```js
// First Attempt - Simple rendering of candidate votes across districts
Plot.plot({
  // this projection is already zoomed into NYC
  projection: {
    domain: districts,
    type: "mercator",
  },
  color: {
    type: "quantize",
    scheme: "blues",
    label: "Candidate Votes",
    legend: true
  },
  marks: [
    Plot.geo(districts, {
      fill: (d, i) => {
        const rate = resultsCandidate.get(d.properties.BoroCD)
        return rate
      },
      // stroke: "black", 
      tip: true, 
      channels: { 
        "Borough District": d => d.properties.BoroCD,
        "Candidate Votes": d => resultsCandidate.get(d.properties.BoroCD),
        "Opponent Votes": d => resultsOpponent.get(d.properties.BoroCD)
        } 
    }),
  ]
})
``` -->

<!-- ```js
// Simple rendering of the NYC districts topoJSON
Plot.plot({
  // this projection is already zoomed into NYC
  projection: {
    domain: districts,
    type: "mercator",
  },
  marks: [
    Plot.geo(districts),
  ]
})
``` -->