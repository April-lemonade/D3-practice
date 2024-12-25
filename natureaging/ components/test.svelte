<script>
    import * as d3 from "d3";
    export let data;
    export let xValue, yValue;
    export let margin;

    let adGroup, acdGroup, vadGroup, sortedData;
    let width = window.innerWidth,
        height = window.innerHeight;
    // $: width = (height / 4) * 3;
    let groupedData = [],
        xGroups = [];
    let yAxis, yScale, xAxis, xScale;
    let globalMax, globalMin;
    let proteinNames, uniqueProteinNames;

    const linkGen = d3
        .linkVertical()
        .x((d) => d[0])
        .y((d) => d[1]);

    const bgColors = [
        "#FAFAFA",
        "#EBEBEB",
        "#D9D9D9",
        "#CCCCCC",
        "#BFBFBF",
        "#B3B3B3",
    ];
    bgColors.reverse();
    const colors = [
        "#1F1F1F",
        "#363636",
        "#4A4A4A",
        "#5C5C5C",
        "#666666",
        "#7A7A7A",
    ];
    colors.reverse();

    const diseases = [
        "AD (Alzheimer’s Disease): Alzheimer’s Disease is a progressive neurodegenerative disorder that primarily affects older adults. It is the most common cause of dementia, characterized by memory loss, cognitive decline, and behavioral changes. The exact cause of AD is still not fully understood, but it involves the accumulation of amyloid-beta plaques and tau tangles in the brain.",
        "ACD (All-Cause Dementia): All-Cause Dementia refers to the broader category of dementia, encompassing all types of dementia including Alzheimer’s Disease, Vascular Dementia, Lewy body dementia, frontotemporal dementia, and others. It is used as a general term to describe cognitive decline that interferes with daily life due to various underlying causes.",
        "VAD (Vascular Dementia): Vascular Dementia is the second most common form of dementia, caused by conditions that block or reduce blood flow to the brain, depriving brain cells of oxygen and nutrients. It can result from stroke, small vessel disease, or other vascular issues, leading to cognitive impairment that can vary in severity.",
    ];

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

        const svg = d3.select("svg");

        groupedData = Array.from(
            d3.group(sortedData, (d) => d.target_yrs).entries(),
        );
        groupedData.reverse();
        groupedData = groupedData.filter((_, i) => i % 2 == 0 && i != 0);
        console.log(groupedData);
        globalMax = Math.max(
            ...groupedData.flatMap(([_, yearData]) => yearData.map(yValue)),
        );
        globalMin = Math.min(
            ...groupedData.flatMap(([_, yearData]) => yearData.map(yValue)),
        );
        xScale = d3
            .scalePoint()
            .domain(uniqueProteinNames)
            .range([margin, width - margin])
            .padding(0.2);

        yScale = d3
            .scaleLinear()
            .domain([globalMin, globalMax])
            .range([height - 450, 1100]);

        xAxis = d3.axisBottom(xScale);
        // svg.selectAll(".x-axis")
        //     .data([null])
        //     .join("g")
        //     .attr("class", "x-axis")
        //     .attr("transform", `translate(0, ${height - 100})`)
        //     .call(xAxis);

        // yAxis = d3
        //     .axisLeft(yScale)
        //     .ticks(5)
        //     .tickSize(-width + margin * 2);
        yAxis = d3.axisLeft(yScale).ticks(6);

        let yAxisG = svg
            .selectAll(".y-axis")
            .data([null])
            .join("g")
            .attr("class", "y-axis")
            .attr("transform", `translate(25, 0)`)
            .call(yAxis);

       

       

        xGroups = Array.from(
            d3
                .group(
                    groupedData.flatMap(([_, yearData]) => yearData),
                    (d) => xValue(d),
                )
                .entries(),
        );
        console.log(xGroups);
    }
</script>

<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" />
<link
    href="https://fonts.googleapis.com/css2?family=Life+Savers&display=swap"
    rel="stylesheet"
/>
<link
    href="https://fonts.googleapis.com/css2?family=Life+Savers:wght@400;700;800&family=Open+Sans:ital,wght@0,300..800;1,300..800&display=swap"
    rel="stylesheet"
/>
<link
    href="https://fonts.googleapis.com/css2?family=Life+Savers:wght@400;700;800&family=Open+Sans:ital,wght@0,300..800;1,300..800&family=PT+Serif:ital,wght@0,400;0,700;1,400;1,700&display=swap"
    rel="stylesheet"
/>
<link
    href="https://fonts.googleapis.com/css2?family=Montserrat:ital,wght@0,100..900;1,100..900&display=swap"
    rel="stylesheet"
/>
<div class="chart-container" style="height: 100vh;" bind:clientWidth={width}>
    <svg {width} {height} style="background-color: #FAFAFA;">
        <g id="areas">
            {#each [0, 1] as i, index}
                <g id="area{index}">
                    <defs>
                        <mask id="mask{index}">
                            <rect
                                {width}
                                {height}
                                x="0"
                                y="0"
                                id="rect{index}"
                                fill={index == 0 ? "white" : "black"}
                            />
                            <path
                                id="head{index}"
                                fill={index == 0 ? "black" : "white"}
                                class="head"
                                d="M195.5,3.86c-32.4,18.14-89.9,30.33-103.7,56.12-11.67,5.7,7.15,8.24-13.32,10.5-9.81-3.83-8.01,18.83-10.94,6.86,13.92,56.83-1.1,7.75-41.24,145.35-10.3,18.23-7.63,32.7,2.05,46.41,40.84,35.63-78.37,65.24-2.43,111.15,1.66-.1,.77,6.21,2.43,6.1,.67,0,1.22,1.11,1.22,2.48,6.84,12.93,17.39,24.1,19.52,39.62-1.17,7.25,5.05,7.24,1.22,12.2-3.73,13.1,2.2,30.55,5.49,42.53,4.59,13.29-1.92,45.58,15.3,53.24,17.28,13.82,38.77,.52,51.18,17.17,14.57,12.57,52.68,30.65,24.85,48.43-20.06,20.21-48.43,34.85-61.46,61.31-9.44,25.49-18.68,52.91-24.4,79.31-7.56,37.43-9.59,80.05-26.84,113.46l677.45,.3c-17.56-114.24-81.21-220.8-169.77-295.16-14.26-10.37-22.74-16.98-23.39-35.04-7.92-58.12-81.95-55.03-66.56-91.6,30.99-30.14,25.47-83,25.41-122.25-.06-46.38-5.52-80.42-5.54-119.1-.02-31.85-.38-65.49-14.25-92.36-6.27-10.48-4.82-14.07-23.06-31.72-28.93-28-68.82-44.79-107.67-57.22-31.07-5.99-76.33-6.62-103.16-11.91-7.46,1.28-24.29,8.06-28.4,3.86Z"
                            />
                        </mask>
                    </defs>
                    <g mask="url(#mask{index})">
                        {#each groupedData as [year, yearData], i}
                            <g>
                                <path
                                    class="area"
                                    id="area{i}"
                                    fill={index == 0 ? bgColors[i] : colors[i]}
                                    d={d3
                                        .area()
                                        .curve(d3.curveCardinal)
                                        .x((d) => xScale(xValue(d)))
                                        .y0(height)
                                        .y1((d) => yScale(yValue(d)))(yearData)}
                                    opacity={i === groupedData.length - 1
                                        ? 0.5
                                        : 1 - i * 0.1}
                                />
                            </g>
                        {/each}
                    </g>
                </g>
            {/each}
        </g>
    </svg>
</div>

<style>
    #fallingMan {
        transform: translate(34%, 10%) scale(0.7) rotate(-45deg);
    }
    .head {
        transform: translate(20%, 56%) scale(1);
        transform-origin: center bottom;
    }

    * {
        margin: 0;
        padding: 0;
        overflow: hidden;
    }

    .labelText {
        font-family: "Open Sans", sans-serif;
        padding: 0% 5%;
        color: #1f1f1f;
        font-size: 8pt;
    }

    .yearLabel {
        font-family: "Open Sans", sans-serif;
        font-size: 8pt;
        fill: white;
        dominant-baseline: hanging;
        mix-blend-mode: exclusion;
    }

    #bgText {
        font-family: "PT Serif", serif;
        padding: 2% 5%;
        width: 90%;
        color: #ebebeb;
    }

    #lines {
        stroke: black;
        stroke-width: 0.5;
    }

    #number {
        font-family: "Montserrat", sans-serif;
        padding: 0% 5%;
        color: #1f1f1f;
        font-size: 4pt;
    }
</style>
