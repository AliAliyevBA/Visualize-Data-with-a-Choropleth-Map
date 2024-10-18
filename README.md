# Visualize-Data-with-a-Choropleth-Map
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Choropleth Map</title>
    <script src="https://d3js.org/d3.v7.min.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
        }

        svg {
            display: block;
            margin: 0 auto;
        }

        .county {
            stroke: #fff;
        }

        #tooltip {
            position: absolute;
            background-color: lightgray;
            padding: 5px;
            border-radius: 5px;
            display: none;
            font-size: 14px;
        }

        #legend {
            font-size: 12px;
        }
    </style>
</head>
<body>
    <h1 id="title">United States Educational Attainment</h1>
    <p id="description">Percentage of adults age 25 and older with a bachelor's degree or higher (2010-2014)</p>
    
    <svg id="chart" width="960" height="600"></svg>
    <div id="tooltip"></div>

    <!-- Load the FCC Test Script -->
    <script src="https://cdn.freecodecamp.org/testable-projects-fcc/v1/bundle.js"></script>

    <script>
        const countyDataUrl = "https://cdn.freecodecamp.org/testable-projects-fcc/data/choropleth_map/counties.json";
        const educationDataUrl = "https://cdn.freecodecamp.org/testable-projects-fcc/data/choropleth_map/for_user_education.json";

        const svg = d3.select("#chart");
        const path = d3.geoPath();
        const tooltip = d3.select("#tooltip");

        const width = 960;
        const height = 600;

        const colors = d3.scaleThreshold()
            .domain([3, 12, 21, 30, 39])
            .range(d3.schemeBlues[6]);

        // Load both datasets
        Promise.all([d3.json(countyDataUrl), d3.json(educationDataUrl)])
            .then(([us, educationData]) => {
                // Convert education data to an object by FIPS for faster lookup
                const education = {};
                educationData.forEach(d => {
                    education[d.fips] = d;
                });

                // Draw counties
                svg.append("g")
                    .selectAll("path")
                    .data(topojson.feature(us, us.objects.counties).features)
                    .enter().append("path")
                    .attr("class", "county")
                    .attr("d", path)
                    .attr("fill", d => {
                        const educationRate = education[d.id]?.bachelorsOrHigher;
                        return educationRate ? colors(educationRate) : "#ccc";
                    })
                    .attr("data-fips", d => d.id)
                    .attr("data-education", d => education[d.id]?.bachelorsOrHigher || 0)
                    .on("mouseover", (event, d) => {
                        const edu = education[d.id];
                        tooltip.style("display", "block")
                            .style("left", event.pageX + "px")
                            .style("top", event.pageY - 28 + "px")
                            .html(`County: ${edu.area_name}, ${edu.state}<br>
                                   Education: ${edu.bachelorsOrHigher}%`)
                            .attr("data-education", edu.bachelorsOrHigher);
                    })
                    .on("mouseout", () => {
                        tooltip.style("display", "none");
                    });

                // Draw legend
                const legend = svg.append("g")
                    .attr("id", "legend")
                    .attr("transform", `translate(${width - 400}, ${height - 40})`);

                const legendScale = d3.scaleLinear()
                    .domain([0, 40])
                    .range([0, 300]);

                const legendAxis = d3.axisBottom(legendScale)
                    .tickSize(13)
                    .tickValues(colors.domain())
                    .tickFormat(d => d + "%");

                legend.selectAll("rect")
                    .data(colors.range().map(color => {
                        const d = colors.invertExtent(color);
                        if (!d[0]) d[0] = legendScale.domain()[0];
                        if (!d[1]) d[1] = legendScale.domain()[1];
                        return d;
                    }))
                    .enter().append("rect")
                    .attr("height", 13)
                    .attr("x", d => legendScale(d[0]))
                    .attr("width", d => legendScale(d[1]) - legendScale(d[0]))
                    .attr("fill", d => colors(d[0]));

                legend.append("g")
                    .attr("transform", "translate(0,13)")
                    .call(legendAxis);
            });
    </script>
</body>
</html>
