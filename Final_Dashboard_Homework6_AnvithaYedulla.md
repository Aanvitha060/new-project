
---
title: Homework6_AnvithaYedulla
---

<p style="font-size: 20px; font-weight: normal; margin-bottom: 10px;">
  Anvitha Yedulla - 801427412
</p>

<h1 style="white-space: nowrap; margin-bottom: 20px;">Exploring Anxiety and Depression Trends</h1>

<h3 style="white-space: nowrap;">Dataset Overview</h3>

This dataset contains survey responses measuring anxiety and depression levels
across individuals of different demographics, capturing factors like age group,
gender, and mental health scores.

---

<style>
  .filter-box select {
    font-size: 18px;
    padding: 10px 14px;
    width: 240px;
    border-radius: 6px;
    border: 1px solid #0d0c0c;
    margin-top: 5px;
    background: rgb(249, 246, 246);
    color: black;
  }
  .filter-box label {
    font-size: 17px;
    margin-bottom: 4px;
    color: #0d0c0c;
  }
  .filter-container {
    display: flex;
    gap: 30px;
    flex-wrap: wrap;
    margin-bottom: 30px;
  }
  .viz-card {
    background: rgb(247, 244, 244);
    border: 1px solid #444;
    border-radius: 12px;
    padding: 24px;
    margin-bottom: 30px;
    box-shadow: 0 4px 8px rgba(0,0,0,0.3);
  }
  .viz-card h2 {
    font-size: 24px;
    margin-bottom: 16px;
    color: rgb(14, 13, 13);
  }
  .viz-card ul {
    font-size: 18px;
    line-height: 1.9;
    color: #171616;
    padding-left: 24px;
  }
  .viz-card li {
    margin-bottom: 12px;
  }
</style>

<div class="filter-container">
  <div class="filter-box">
    <label for="genderDropdown">Select Gender:</label>
    <select id="genderDropdown"></select>
  </div>
  <div class="filter-box">
    <label for="ageDropdown">Select Age Group:</label>
    <select id="ageDropdown"></select>
  </div>
</div>

<div style="display: flex; flex-wrap: wrap; gap: 20px;">
  <div class="viz-card" style="flex: 1;">
    <h2>Average Anxiety Score by Age (Bar Chart)</h2>
    <div id="barChart"></div>
  </div>
  <div class="viz-card" style="flex: 1;">
    <h2>Distribution of Depression Scores (Histogram)</h2>
    <div id="histogram"></div>
  </div>
</div>

```js
import * as d3 from "https://cdn.jsdelivr.net/npm/d3@7/+esm";

let MyData;

FileAttachment("./data/anxiety_depression_data.csv").csv({ typed: true }).then(data => {
  MyData = data;
  populateDropdowns(MyData);
  updateCharts(MyData);

  document.getElementById("genderDropdown").addEventListener("change", () => {
    updateCharts(MyData);
  });
  document.getElementById("ageDropdown").addEventListener("change", () => {
    updateCharts(MyData);
  });
});

function populateDropdowns(data) {
  const genderSet = [...new Set(data.map(d => d.Gender))];
  const ageSet = [...new Set(data.map(d => d.Age))];

  const genderDropdown = document.getElementById("genderDropdown");
  const ageDropdown = document.getElementById("ageDropdown");

  genderDropdown.innerHTML = "";
  genderSet.forEach(g => {
    const option = document.createElement("option");
    option.value = g;
    option.textContent = g;
    genderDropdown.appendChild(option);
  });

  ageDropdown.innerHTML = "";
  ageSet.forEach(a => {
    const option = document.createElement("option");
    option.value = a;
    option.textContent = a;
    ageDropdown.appendChild(option);
  });

  genderDropdown.value = genderSet[0];
  ageDropdown.value = ageSet[0];
}

function updateCharts(data) {
  const selectedGender = document.getElementById("genderDropdown").value;
  const selectedAge = document.getElementById("ageDropdown").value;

  const filtered = data.filter(d => d.Gender === selectedGender || d.Age === selectedAge);

  createBarChart(filtered);
  createHistogram(filtered);
}

function createBarChart(data) {
  const margin = { top: 20, right: 30, bottom: 40, left: 70 };
  const width = 400 - margin.left - margin.right;
  const height = 400 - margin.top - margin.bottom;
  d3.select("#barChart").html("");

  const svg = d3.select("#barChart")
    .append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
    .append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`);

  const avgData = d3.rollup(data, v => d3.mean(v, d => d.Anxiety_Score), d => d.Age);

  const x = d3.scaleBand()
    .domain(Array.from(avgData.keys()))
    .range([0, width])
    .padding(0.2);

  const y = d3.scaleLinear()
    .domain([0, d3.max(avgData.values())])
    .range([height, 0]);

  svg.append("g").attr("transform", `translate(0,${height})`).call(d3.axisBottom(x).tickFormat(d => d).tickSizeOuter(0));
  svg.append("g").call(d3.axisLeft(y));

  svg.selectAll("rect")
    .data(Array.from(avgData))
    .enter()
    .append("rect")
    .attr("x", d => x(d[0]))
    .attr("y", d => y(d[1]))
    .attr("width", x.bandwidth())
    .attr("height", d => height - y(d[1]))
    .style("fill", "orange");
}

function createHistogram(data) {
  const margin = { top: 20, right: 30, bottom: 40, left: 70 };
  const width = 400 - margin.left - margin.right;
  const height = 400 - margin.top - margin.bottom;
  d3.select("#histogram").html("");

  const svg = d3.select("#histogram")
    .append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
    .append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`);

  const x = d3.scaleLinear()
    .domain(d3.extent(data, d => d.Depression_Score))
    .range([0, width]);

  const histogram = d3.histogram()
    .value(d => d.Depression_Score)
    .domain(x.domain())
    .thresholds(x.ticks(20));

  const bins = histogram(data);

  const y = d3.scaleLinear()
    .domain([0, d3.max(bins, d => d.length)])
    .range([height, 0]);

  svg.append("g").attr("transform", `translate(0,${height})`).call(d3.axisBottom(x));
  svg.append("g").call(d3.axisLeft(y));

  svg.selectAll("rect")
    .data(bins)
    .enter()
    .append("rect")
    .attr("x", 1)
    .attr("transform", d => `translate(${x(d.x0)},${y(d.length)})`)
    .attr("width", d => x(d.x1) - x(d.x0) - 1)
    .attr("height", d => height - y(d.length))
    .style("fill", "skyblue");
}
```

<div class="viz-card">
  <h2>Design Decisions & Insights</h2>
  <ul>
    <li>The bar chart shows how average anxiety scores vary across different age groups.</li>
    <li>The histogram reveals how depression scores are distributed in the sample.</li>
    <li>Dropdown filters allow quick exploration based on gender and age group.</li>
    <li>Simple color themes and clear labeling enhance usability.</li>
  </ul>
</div>
