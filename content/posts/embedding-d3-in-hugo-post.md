+++
title = "Embedding D3 in Hugo post"
author = ["Pierre Mercatoris"]
date = 2019-04-22
lastmod = 2023-01-08T00:11:00+01:00
tags = ["d3", "hugo"]
draft = false
weight = 1001
summary = "How to show a D3 graph into a Hugo site post"
comments = false
profile = false
share = false
reading_time = false
[menu]
  [menu.posts]
    weight = 1001
    identifier = "embedding-d3-in-hugo-post"
+++

## Generating some data {#generating-some-data}

{{< highlight python >}}
import numpy as np
import pandas as pd

size = 100

df = pd.DataFrame({
    'x': np.arange(size),
    'y': np.random.normal(size=size),
    'type': np.random.choice(['yes','no'], size)
})
df.to_csv("../static/data/random_normal.csv", index=False)
df.head()
{{< /highlight >}}

\# Out[177]:

|   | x | y         | type |
|---|---|-----------|------|
| 0 | 0 | -0.792839 | no   |
| 1 | 1 | -0.66592  | no   |
| 2 | 2 | -0.284811 | no   |
| 3 | 3 | -1.84316  | no   |
| 4 | 4 | 0.249928  | yes  |


## Storing the D3 code {#storing-the-d3-code}

{{< highlight html >}}
<script src="https://d3js.org/d3.v5.min.js"></script>
<style>
    .bar {
        fill: steelblue;
    }

    .bar:hover {
        fill: orange;
    }
</style>

<div id="graph">

    <svg id="chart" width="650" height="420"></svg>


    Choose year:
    <select id="year"></select>

    <input type="checkbox" id="sort">
    Toggle sort

    <script>
        d3.csv("/data/data.csv").then(d => chart(d));

        function chart(csv) {

            csv.forEach(function(d) {
                var dates = d.date.split("-");
                d.year = dates[0];
                d.month = dates[1];
                d.value = +d.value;
                return d;
            })

            var months = [...new Set(csv.map(d => d.month))],
                years = [...new Set(csv.map(d => d.year))];

            var options = d3.select("#year").selectAll("option")
                .data(years)
                .enter()
                .append("option")
                .text(d => d)

            var svg = d3.select("#chart"),
                margin = {
                    top: 25,
                    bottom: 0,
                    left: 30,
                    right: 25
                },
                width = +svg.attr("width") - margin.left - margin.right,
                height = +svg.attr("height") - margin.top - margin.bottom;

            var x = d3.scaleBand()
                .range([margin.left, width - margin.right])
                .padding(0.1)
                .paddingOuter(0.2)

            var y = d3.scaleLinear()
                .range([height - margin.bottom, margin.top])

            var xAxis = g => g
                .attr("transform", "translate(0," + (height - margin.bottom) + ")")
                .call(d3.axisBottom(x).tickSizeOuter(0))

            var yAxis = g => g
                .attr("transform", "translate(" + margin.left + ",0)")
                .call(d3.axisLeft(y))

            svg.append("g")
                .attr("class", "x-axis")

            svg.append("g")
                .attr("class", "y-axis")

            update(d3.select("#year").property("value"), 0)

            function update(year, speed) {

                var data = csv.filter(f => f.year == year)

                y.domain([0, d3.max(data, d => d.value)]).nice()

                svg.selectAll(".y-axis").transition().duration(speed)
                    .call(yAxis);

                data.sort(d3.select("#sort").property("checked") ?
                    (a, b) => b.value - a.value :
                    (a, b) => months.indexOf(a.month) - months.indexOf(b.month))

                x.domain(data.map(d => d.month))

                svg.selectAll(".x-axis").transition().duration(speed)
                    .call(xAxis)

                var bar = svg.selectAll(".bar")
                    .data(data, d => d.month)

                bar.exit().remove();

                bar.enter().append("rect")
                    .attr("class", "bar")
                    .attr("fill", "steelblue")
                    .attr("width", x.bandwidth())
                    .merge(bar)
                    .transition().duration(speed)
                    .attr("x", d => x(d.month))
                    .attr("y", d => y(d.value))
                    .attr("height", d => y(0) - y(d.value))
            }

            chart.update = update;
        }

        var select = d3.select("#year")
            .style("border-radius", "5px")
            .on("change", function() {
                chart.update(this.value, 750)
            })

        var checkbox = d3.select("#sort")
            .style("margin-left", "45%")
            .on("click", function() {
                chart.update(select.property("value"), 750)
            })
    </script>
</div>
{{< /highlight >}}


## Plot {#plot}

{{< highlight md >}}
{{</* simpled3 */>}}
{{< /highlight >}}

{{< simpled3 >}}
