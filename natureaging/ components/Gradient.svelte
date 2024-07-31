<script>
    import * as d3 from "d3";
    export let data;
    export let xValue, yValue;
    export let margin;
    export let text;
    let adGroup, acdGroup, vadGroup, sortedData;
    let xScale;
    let width = window.innerWidth;
    let height = window.innerHeight;

    let groupedData = [];
    let yAxis, yScale, xAxis;

    // const colors = [
    //     "#1C3659",
    //     "#27446E",
    //     "#325283",
    //     "#5A86A9",
    //     "#79A1C2",
    //     "#98BBDB",
    //     "#FCFCFC",
    // ];

    const colors = [
        "#191d20",
        "#565a5d",
        "#6b6f70",
        "#8e908f",
        "#b5b5b3",
        "#cfd0cb",
    ];

    colors.reverse();

    let proteinNames;
    let uniqueProteinNames;

    $: if (data.length > 0) {
        adGroup = data.filter((d) => d.Protein.startsWith("AD"));
        acdGroup = data.filter((d) => d.Protein.startsWith("ACD"));
        vadGroup = data.filter((d) => d.Protein.startsWith("VAD"));
        sortedData = [...adGroup, ...acdGroup, ...vadGroup];
        proteinNames = sortedData.map((d) => d.Protein);
        uniqueProteinNames = Array.from(new Set(proteinNames)).sort((a, b) => {
            if (a.startsWith("AD") && !b.startsWith("AD")) return -1;
            if (b.startsWith("AD") && !a.startsWith("AD")) return 1;
            if (a.startsWith("ACD") && !b.startsWith("ACD")) return -1;
            if (b.startsWith("ACD") && !a.startsWith("ACD")) return 1;
            return 0;
        });

        xScale = d3
            .scalePoint()
            .domain(uniqueProteinNames)
            .range([margin, width - margin]);

        yScale = d3
            .scaleLinear()
            .domain([d3.min(data, yValue), d3.max(data, yValue)])
            .range([height - 200, 0]);

        const svg = d3.select("svg");

        xAxis = d3.axisBottom(xScale);
        svg.selectAll(".x-axis")
            .data([null])
            .join("g")
            .attr("class", "x-axis")
            .attr("transform", `translate(0, ${height - 100})`)
            .call(xAxis);

        yAxis = d3.axisLeft(yScale);
        svg.selectAll(".y-axis")
            .data([null])
            .join("g")
            .attr("class", "y-axis")
            .attr("transform", `translate(30, 0)`)
            .call(yAxis);

        groupedData = Array.from(
            d3.group(sortedData, (d) => d.target_yrs).entries(),
        );
        groupedData.reverse();
    }
</script>

<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" />
<link
    href="https://fonts.googleapis.com/css2?family=Julee&display=swap"
    rel="stylesheet"
/>
<div
    class="chart-container"
    role="contentinfo"
    style="height: 100vh;"
    bind:clientWidth={width}
>
    <svg {height} {width} style="background: #D9D9D9" id="wave">
        {#each groupedData as [year, yearData], i}
            {#if i % 2 == 0 && i != 0}
                <g>
                    <path
                        id={"curve" + i}
                        fill="none"
                        d={d3
                            .line()
                            .curve(d3.curveCardinal)
                            .x((d) => xScale(xValue(d)))
                            .y((d) => yScale(yValue(d)) + (i / 2 - 1) * 30)(
                            yearData,
                        )}
                    />
                    <path
                        class="area"
                        fill={colors[i / 2 - 1]}
                        d={d3
                            .area()
                            .curve(d3.curveCardinal)
                            .x((d) => xScale(xValue(d)))
                            .y0(height)
                            .y1((d) => yScale(yValue(d)) + (i / 2 - 1) * 30)(
                            yearData,
                        )}
                    />
                </g>
            {/if}
        {/each}
        {#each groupedData as [year, yearData], i}
            {#if i % 2 == 0 && i != 0}
                <text>
                    <textPath
                        xlink:href={"#curve" + i}
                        style="font-size: 1.4rem;fill:white;dominant-baseline:hanging"
                    >
                        {text[i]}
                    </textPath>
                </text>
                <line
                    style="stroke: #D9D9D9;stroke-width: 0.5;"
                    x1="0"
                    y1={height - 200 + (i / 2 - 1) * 30}
                    x2={width}
                    y2={height - 200 + (i / 2 - 1) * 30}
                />
            {/if}
        {/each}
    </svg>
</div>
