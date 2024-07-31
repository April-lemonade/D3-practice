<script>
    import * as d3 from "d3";
    export let data;
    export let xValue = (d) => d.Protein;
    export let yValue = (d) => d.Level;
    let xScale;
    export let margin;
    let width = window.innerWidth;
    let height = window.innerHeight;
    let groupedData = [];
    let yAxis, yScale, xAxis;
    let uniqueProteins;
    const colors = [
        "#191d20ff",
        "#565a5dff",
        "#6b6f70ff",
        "#8e908fff",
        "#b5b5b3ff",
        "#cfd0cbff",
    ];
    let innerRadius = 341.331;
    let outerRadius = 373.159;
    let lineOffset = 10;
    colors.reverse();
    let x = 265.25;
    let y = 472.5;
    let begin = -1;
    let a = 1.421;
    let b = 1.822;
    let value = 1;
    let figures = [];
    for (let i = 1; i < 7; i++) {
        figures.push({
            id: i,
            x: 0,
            y: 0,
            scale: 0.1,
        });
    }
    let figureBeginX = 222.091;
    let figureBeginY = -54.866;
    let figureBeginScale = 0.338;
    let figureOffsetX = 11.4;
    let figureOffsetY = 6.13;
    let figureOffsetScale = 0.035;

    function addZerosToGroups(groupdata, begin) {
        return groupdata.map(([year, group]) => [
            year,
            [
                { Level: begin, Protein: "blank1", target_yrs: year },
                ...group,
                { Level: begin, Protein: "blank2", target_yrs: year },
            ],
        ]);
    }

    $: if (data.length > 0) {
        groupedData = Array.from(d3.group(data, (d) => d.target_yrs).entries());
        groupedData = addZerosToGroups(groupedData, begin);
        console.log(groupedData);
        uniqueProteins = new Set(data.map(xValue));
        uniqueProteins = Array.from(uniqueProteins);
        uniqueProteins.unshift("blank1");
        uniqueProteins.push("blank2");
        groupedData.reverse();
        console.log(groupedData);
        xScale = d3.scalePoint().domain(uniqueProteins).range([a, b]);
        console.log(xScale.domain());
        yScale = d3
            .scaleLinear()
            .domain([d3.min(data, yValue), d3.max(data, yValue)])
            .range([innerRadius, outerRadius]);
        console.log("x:", x, ",y:", y, ",begin:", begin, ",a:", a, ",b:", b);
    }

    function getLineCoordinates(pointIndex, lineOffset, groupedData) {
        if (groupedData.length > 0) {
            const firstYearData = groupedData[2][1];
            const lastYearData = groupedData[groupedData.length - 2][1];
            // console.log(groupedData[groupedData.length - 2][0]);
            if (
                firstYearData.length > pointIndex &&
                lastYearData.length > pointIndex
            ) {
                const firstPoint = firstYearData[pointIndex];
                const lastPoint = lastYearData[pointIndex];

                let firstPointAngle = createAdjustedXScale(2)(
                    xValue(firstPoint),
                );
                let firstPointRadius = adjustedRadius(
                    firstPoint,
                    2,
                    pointIndex,
                    firstYearData,
                    lineOffset,
                    groupedData,
                );

                let lastPointAngle = createAdjustedXScale(
                    groupedData.length - 2,
                )(xValue(lastPoint));
                let lastPointRadius = adjustedRadius(
                    lastPoint,
                    groupedData.length - 2,
                    pointIndex,
                    lastYearData,
                    lineOffset,
                    groupedData,
                );

                return {
                    start: {
                        x: Math.sin(firstPointAngle) * firstPointRadius,
                        y: -Math.cos(firstPointAngle) * firstPointRadius,
                    },
                    end: {
                        x: Math.sin(lastPointAngle) * lastPointRadius,
                        y: -Math.cos(lastPointAngle) * lastPointRadius,
                    },
                };
            }
        }
        return null;
    }

    function createAdjustedXScale(i, index) {
        // if (index === 0 || index === groupedData.length - 1) {
        //     return d3.scalePoint().domain(uniqueProteins).range([a, b]);
        // }
        if (i === 2) {
            return d3.scalePoint().domain(uniqueProteins).range([a, b]);
        }
        const adjustmentFactor = i / 2 - 1;
        const newRange = [
            a + (adjustmentFactor * (b - a)) / uniqueProteins.length / 2,
            b - (adjustmentFactor * (b - a)) / uniqueProteins.length / 2,
        ];
        return d3.scalePoint().domain(uniqueProteins).range(newRange);
    }

    function adjustedRadius(d, i, index, yearData, lineOffset, groupedData) {
        if (index === 0 || index === yearData.length - 1) {
            return yScale(yValue(d)); // 不偏移端点
        }
        return yScale(yValue(d)) + (groupedData.length - i) * lineOffset;
    }
    let proteinLines = [];
    $: proteinLines = [];
    $: if (data.length > 0 && groupedData.length > 0) {
        proteinLines = uniqueProteins
            .map((_, i) => getLineCoordinates(i, lineOffset, groupedData))
            .filter((d) => d !== null);
    }
</script>

<link
    rel="stylesheet"
    href="https://cdn.jsdelivr.net/npm/svelte-material-ui@7.0.0/bare.min.css"
/>

<div class="chart-container">
    <!-- <div class="slidecontainer">
        x位移：{x}
        <input type="range" min="0" max={width} bind:value={x} />
    </div>
    <div class="slidecontainer">
        y位移：{y}
        <input type="range" min="0" max={height} bind:value={y} />
    </div> -->

    <div class="slidecontainer">
        起始点数值：{begin}
        <input type="range" min="-1" max="1" step="0.01" bind:value={begin} />
        起始角度：{((a * 180) / Math.PI).toFixed(2)}
        <input type="range" min="0" max={Math.PI} step="0.001" bind:value={a} />
        结束角度： {((b * 180) / Math.PI).toFixed(2)}
        <input type="range" min="0" max={Math.PI} step="0.001" bind:value={b} />
        内半径： {innerRadius}
        <input
            type="range"
            min="0"
            max={height}
            step="0.001"
            bind:value={innerRadius}
        />
        外半径： {outerRadius}
        <input
            type="range"
            min="0"
            max={height}
            step="0.001"
            bind:value={outerRadius}
        />
        线的offset: {lineOffset}
        <input
            type="range"
            min="0"
            max={height}
            step="0.001"
            bind:value={lineOffset}
        />
        头的起始x: {figureBeginX}
        <input
            type="range"
            min="0"
            max={width}
            step="0.001"
            bind:value={figureBeginX}
        />
        头的起始y: {figureBeginY}
        <input
            type="range"
            min={-height}
            max={height}
            step="0.001"
            bind:value={figureBeginY}
        />
        头的起始scale: {figureBeginScale}
        <input
            type="range"
            min="0"
            max="1"
            step="0.001"
            bind:value={figureBeginScale}
        />
        头x偏移: {figureOffsetX}
        <input
            type="range"
            min={-width / 2}
            max={width / 2}
            step="0.001"
            bind:value={figureOffsetX}
        />
        头y偏移: {figureOffsetY}
        <input
            type="range"
            min={-height / 2}
            max={height / 2}
            step="0.001"
            bind:value={figureOffsetY}
        />
        头scale偏移: {figureOffsetScale}
        <input
            type="range"
            min="0"
            max={figureBeginScale}
            step="0.001"
            bind:value={figureOffsetScale}
        />
    </div>
    <svg {width} {height} viewBox="{width / 7} 0 {width / 3.5} {height / 40}">
        <g id="waves">
            {#each groupedData as [year, yearData], i}
                {#if i % 2 == 0 && i !== 0}
                    <g class={"year-" + year}>
                        <path
                            class="figure"
                            id={"figure" + i / 2}
                            stroke={colors[i / 2 - 1]}
                            stroke-width="2"
                            style={`transform:translate(${figureBeginX + (i / 2 - 1) * figureOffsetX}px, ${figureBeginY + (i / 2 - 1) * figureOffsetY}px) scale(${figureBeginScale - (i / 2 - 1) * figureOffsetScale})`}
                            fill={colors[i / 2 - 1]}
                            d="M224.25,15.07c-7.78,3.51-29.86,13.67-50.34,34.29-9.34,9.41-17.93,20.57-31.38,38.04-6.74,8.75-9.45,12.68-11.9,16.56-2.56,4.04-10.36,16.78-16.91,35.09-5.3,14.84-8.17,28.83-9.61,41.12-8.44,14.94-6.25,26.8,1.68,38.04,33.47,29.2-64.23,53.47-1.99,91.1,1.36-.08,.63,5.09,1.99,5,.55,0,1,.91,1,2.03,5.61,10.6,14.25,19.75,16,32.47-.96,5.94,4.14,5.93,1,10-3.06,10.74,1.8,25.04,4.5,34.86,3.76,10.89-1.57,37.36,12.54,43.64,14.16,11.33,31.78,.43,41.95,14.07,7.3,5.33,16.67,9.65,21,17.93,1.11,3.9,5.04,3.09,6,6.56,.59,2.15,5.7,3.58,1.97,6.24-5.07,8.45-3.23-.11-8.6,8.96-1.54,1.68-4.07,1.76-4.68,4.31-.23,2.52-4.48,1.24-4.69,3.36-1.43,2.04-4.51,4.66-6.97,5.57-4.62,4.3-8.43,12.31-15.03,14-.54,1.53-4.17,.61-4.52,3.07-.73,2.65-3.6,2.87-5.23,4.56-1.87,2.13-.76,3.81-3.75,5.78-3.43,2.97-3.23,6.73-5.5,9.6-1.34-.14-.66,7.14-2,7-.55,0-1,1.08-1,2.39,.08,3.25-3.14,2.74-3,6.08-.44,3.89-2.22,1.79-2,7.03,.03,6.89-3.79,4.92-4,11.5,.02,5.54-2.16,2.62-2,8.5,0,3.26-.4,4.5-1.45,4.5-2.26-.65-.48,8.71-2.33,8.76-1.58-.03-.84,9.19-2.22,9.24-.61,5.33-2.37,13.03-3,19.53,.44,10.43-3.38,3.75-3,14.02-.6,6.93-1.23,8.56-3,13.45-1.47,2.05-.76,19.36-3,21.57-.49,6.73-6.4,11.74-7,18.41,0,1.11-.45,2.01-1,2.01-1.35-.09-.65,4.09-2,4-1.33-.16-.67,7.15-1.98,7-9.26,23.89-18.99,48.3-32.04,71-3.56,9.69-7.99,22-12.98,31-.55,0-1,1.13-1,2.5,.25,3.11-2.02,1.75-2,4.5-2.57,2.98-1.04,8.94-3.5,11.36-1.99,1.74-2.56,15.41-4.5,18.14-.55,.34-1,3.97-1,8.06,.36,9.31-1.97,4.98-2,13.11-10.5,37.58-8.32,77.91-18,115.89-.36,5.1-1.26,4.51-3.16,7.66-1.55,.43-1.23,10.22-2.79,10.75-3.85,4.72,2.27,24.13-3.02,26.74-1.78,.46-1.96,1.39-1.94,10.06-1,13.44-2.67,14.72-2.01,26.49-1.09,7.79-3.74-.04-2.53,12.5,1.3,15.84-5.17,22.06-6.55,36.58,1.08,9.94-3.2,5.48-2.71,12.07-1.42,11.12-4.73,21.89-4.71,33.6-11.16,27.43,3.97,4.49,347.29,12.01,374.13-11.33,346.8,25.71,351.13-22.71,9.48-126.31-22.4-255.05-23.36-380.33-1.46-17.49-4.98-34.74-6.56-51.52-2.61-4.59-.03-12.68-4.83-16.21-14.39-93.63-66.56-180.97-139.15-241.92-11.69-8.5-18.64-13.92-19.17-28.72-6.49-47.64-67.17-45.1-54.55-75.08,11.23-19.4,15.87-36.19,18.02-47.45,1.83-9.54,2.28-17.51,3.17-33.44,.83-14.75,.82-27.7,.81-43.85,0-8.58-.09-8.06-.15-22.54-.09-21.59,.28-24.88,0-32.72-.45-12.57-1.38-29.3-3.36-49.4-1.16-10.55-4.3-26.9-13.87-44.1-15.99-28.72-40.13-42.19-60.56-53.59C359.59,5.76,325.67,1.95,318.06,1.19c-14.86-1.47-52.47-4.73-93.8,13.88Z"
                        />
                        <path
                            class="wave"
                            d={d3
                                .lineRadial()
                                .curve(d3.curveCardinal)
                                .angle((d, index) =>
                                    createAdjustedXScale(i, index)(xValue(d)),
                                )
                                .radius((d, index) =>
                                    adjustedRadius(
                                        d,
                                        i,
                                        index,
                                        yearData,
                                        lineOffset,
                                        groupedData,
                                    ),
                                )(yearData)}
                            fill={colors[i / 2 - 1]}
                        ></path>

                        {#each yearData as d, index}
                            {#if d.Protein != "blank1" && d.Protein != "blank2"}
                                <circle
                                    cx={Math.sin(
                                        createAdjustedXScale(
                                            i,
                                            index,
                                        )(xValue(d)),
                                    ) *
                                        adjustedRadius(
                                            d,
                                            i,
                                            index,
                                            yearData,
                                            lineOffset,
                                            groupedData,
                                        )}
                                    cy={-Math.cos(
                                        createAdjustedXScale(
                                            i,
                                            index,
                                        )(xValue(d)),
                                    ) *
                                        adjustedRadius(
                                            d,
                                            i,
                                            index,
                                            yearData,
                                            lineOffset,
                                            groupedData,
                                        )}
                                    r="1"
                                    fill="white"
                                ></circle>
                            {/if}
                        {/each}
                    </g>
                {/if}
            {/each}
        </g>
        <g id="protein-lines">
            {#each proteinLines as protein, i}
                {#if i != 0 && i != proteinLines.length - 1}
                    <!-- <line
                        x1={getLineCoordinates(i, lineOffset).start.x}
                        y1={getLineCoordinates(i, lineOffset).start.y}
                        x2={getLineCoordinates(i, lineOffset).end.x}
                        y2={getLineCoordinates(i, lineOffset).end.y}
                        stroke="white"
                        stroke-width="0.5"
                    /> -->
                {/if}
            {/each}
        </g>
    </svg>
</div>

<style>
    .chart-container {
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100vh;
        flex-direction: column;
    }

    /* .figure {
        width: 100%;
        transform: scale(0.1);
        fill: red;
    } */

    /* svg,
    path {
        stroke: blue;
        fill: none;
    } */

    /* #waves {
     transform: scale(1) translate(205px, 445px);   
    } */
    .slidecontainer {
        width: 80%;
    }

    input {
        width: 100%;
    }

    /* .wave {
        display: block;
        stroke: blue;
        fill: none;
    }

    .figure {
        stroke: blue;
        fill: none;
    } */
</style>
