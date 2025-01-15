<script>
    import * as d3 from "d3";
    import { tick } from "svelte";
    export let data;
    export let xValue, yValue;
    export let margin;
    export let HRs;
    import { onMount } from "svelte";
    import headtext from "../src/assets/headtxt.svg";

    let adGroup, acdGroup, vadGroup, sortedData;
    let width = 1080,
        height = 1920;
    // $: width = (height / 4) * 3;
    let groupedData = [],
        xGroups = [];
    let yAxis, yScale, xAxis, xScale;
    let globalMax, globalMin;
    let proteinNames, uniqueProteinNames;
    let flag = false;
    let flag1 = false;

    let svg;
    let rangeMin = height - 450;
    let rangeMax = 1100;
    let source; // 初始化 DOM 元素引用
    let scrollTop = 0; // 存储相对屏幕顶部的距离
    let updateScrollTop;
    let scrolldomain;
    let falling;
    let title;

    const introWidth = 300;
    const introHeight = 150;

    const linkGen = d3
        .linkVertical()
        .x((d) => d[0])
        .y((d) => d[1]);

    const lineGenerator = d3
        .line()
        .x((d) => d[0])
        .y((d) => d[1])
        .curve(d3.curveBasis);

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

    let yearValue, yearScale, levelScale;

    let areaData;

    let cumulativeHeight = 0;
    let spacing = 10;
    let circlePositions = [];
    let newPositions = [];

    let HRValue, HRScale;

    $: if (data.length > 0 && HRs.length > 0) {
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
        areaData = groupedData.map(([year, yearData]) => {
            let firstData = {
                ...yearData[0],
                Protein: "blank0",
            };
            let lastData = {
                ...yearData[yearData.length - 1],
                Protein: "blank1",
            };

            let newYearData = [firstData, ...yearData, lastData];

            return [year, newYearData];
        });
        areaData = areaData.filter((_, i) => i % 2 == 0 && i != 0);
        console.log("areaData", areaData);
        groupedData = groupedData.filter((_, i) => i % 2 == 0 && i != 0);
        groupedData.forEach((item, index) => {
            let radius = 4 * (groupedData.length - index);
            newPositions.push({
                cx: 100,
                cy: cumulativeHeight + radius,
                radius: radius,
                year: item[0],
            });
            cumulativeHeight += 2 * radius + spacing;
            circlePositions = newPositions;
        });

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
            .range([10, width - 10])
            .padding(0.3);

        if (!flag) {
            yScale = d3
                .scaleLinear()
                .domain([globalMin, globalMax])
                .range([height - 450, 1500]);
            flag = true;
        }

        xAxis = d3.axisBottom(xScale);
        // svg.selectAll(".x-axis")
        //     .data([null])
        //     .join("g")
        //     .attr("class", "x-axis")
        //     .attr("transform", `translate(0, ${height - 100})`)
        //     .call(xAxis);

        // yAxis = d3
        //     .axisLeft(yScale)
        //     .ticks(10)
        //     .tickSize(-width + margin * 2);

        yAxis = d3.axisLeft(yScale).ticks(5);

        let yAxisG = svg
            .selectAll(".y-axis")
            .data([null])
            .join("g")
            .attr("class", "y-axis")
            .attr("transform", `translate(40, 0)`)
            .call(yAxis);

        yAxisG
            .selectAll(".tick line")
            .style("stroke", "#afafaf") // 设置为浅灰色
            .style("stroke-width", "0.5px"); // 设置为较细的线
        yAxisG.select(".domain").remove();
        let yAxisTop = yScale(yScale.domain()[1]);
        yAxisG.selectAll(".y-axis-label")?.remove();
        yAxisG.selectAll(".tick line").attr("stroke", "black");
        yAxisG
            .selectAll(".tick text")
            .style("font-size", "8pt")
            .style("font-family", "'Montserrat', sans-serif")
            .style("fill", "#1f1f1f");

        yAxisG
            .append("text")
            .attr("class", "y-axis-label")
            .attr("font-family", "'Open Sans', sans-serif")
            .attr("x", width * 0.02)
            .attr("y", yAxisTop)
            .attr("text-anchor", "start") // 左对齐主文本
            .attr("fill", "#1f1f1f")
            .style("font-size", "10pt")
            .style("font-weight", "lighter")
            .text("Plasma Protein")
            .append("tspan") // 添加一个tspan元素用于新行
            .attr("x", width * 0.02) // 确保tspan在文本块中正确位置
            .attr("dy", "1.2em") // 垂直移动到新行
            .text("(ng/mL)");

        d3.select(".y-axis").selectAll(".tick line").style("stroke", "none"); // 隐藏刻度线

        xGroups = Array.from(
            d3
                .group(
                    groupedData.flatMap(([_, yearData]) => yearData),
                    (d) => xValue(d),
                )
                .entries(),
        );
        yearValue = (d) => -parseFloat(d[0]);
        yearScale = d3
            .scalePoint()
            .domain([3, 5, 7, 9, 11, 13].reverse())
            .range([0, 1.8 * Math.PI]);
        levelScale = d3
            .scaleLinear()
            .domain([globalMin, globalMax + 0.2])
            .range([60, 80]);
        // console.log(xGroups);

        HRValue = (d) => d.HR;
        HRScale = d3
            .scaleLinear()
            .domain([1.5, 3.0])
            .range([0, 1.8 * Math.PI]);
        console.log(HRs);
    }

    let oldMan, stick;

    onMount(() => {
        falling = document.getElementById("falling");
        title = document.getElementById("headtext");
        oldMan = document.getElementById("path1");
        stick = document.getElementById("path2");
        yScale = d3
            .scaleLinear()
            .domain([globalMin, globalMax])
            .range([height - 450, 1100]);
        let container = document.getElementById("container");
        updateScrollTop = () => {
            scrollTop = container.scrollTop; // 获取页面的滚动偏移量
            console.log(scrollTop);
            scrolldomain = d3
                .scaleLinear()
                .domain([0, 1200])
                .range([height - 450, 950]);
            yScale = d3
                .scaleLinear()
                .domain([globalMin, globalMax])
                .range([height - 450, scrolldomain(scrollTop)]);

            if (scrollTop !== 0) {
                falling.style.transform = `translate(34%, ${12 + (scrollTop / 10) * 1.5}%) scale(${0.5 - scrollTop / 4000}) rotate(-40deg)`;
                oldMan.style.transform = `rotate(0deg)`;
                stick.style.transform = `translate(9%, -5%) rotate(40deg)`;
            } else {
                // oldMan.style.transform = `translate(34%, ${12 + (scrollTop / 10) * 1.5}%) scale(${0.5 - scrollTop / 4000}) rotate(-20deg)`;
                // stick.style.transform = `translate(34%, ${12 + (scrollTop / 10) * 1.5}%) scale(${0.5 - scrollTop / 4000}) rotate(-20deg)`;
                falling.style.transform = `translate(34%, ${12 + (scrollTop / 10) * 1.5}%) scale(${0.5 - scrollTop / 4000}) rotate(-20deg)`;
                stick.style.transform = `translate(0%, 0%) rotate(0deg)`;
            }

            falling.style.opacity = `${Math.max(1 - scrollTop / 1000, 0.5)}`;
            // title.style.opacity = 0;
        };
        container.addEventListener("scroll", updateScrollTop);
        updateScrollTop();

        const scrollContainer = document.getElementById("container"); // 替换为实际容器的选择器
        // let scrollSpeed = 0.5; // 滚动速度
        // function smoothScroll() {
        //     scrollContainer.scrollBy(0, scrollSpeed); // 滚动容器内
        //     requestAnimationFrame(smoothScroll); // 下一帧调用
        // }
        // setTimeout(() => {
        //     requestAnimationFrame(smoothScroll); // 延迟后启动平滑滚动
        // }, 1000); // 延迟 1 秒
        
        // setTimeout(() => {
        //     setInterval(() => {
        //         container.scrollBy(0, 5); // 滚动容器内
        //     }, 100);
        // }, 1000); // 延迟 1 秒（1000 毫秒）
    });

    $: if (source) {
        console.log(source.getBoundingClientRect().top);
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

<link
    href="https://fonts.googleapis.com/css2?family=Tienne:wght@400;700;900&display=swap"
    rel="stylesheet"
/>
<div
    id="container"
    class="chart-container"
    bind:clientWidth={width}
    style="height: 119vh;overflow:scroll;margin-top:-9%"
>
    <svg
        bind:this={svg}
        {width}
        {height}
        viewBox="0 0 {width} {height}"
        style="background-color: #FAFAFA;position: relative;cursor: none;"
    >
        <g id="lines">
            <!-- yScale(d3.max(values, (d) => yValue(d))) -250 -->
            {#if yScale}
                {#each yScale.ticks() as tick, i}
                    <!-- {#if i % 2 === 0}
                        <line
                            x1={xScale(xScale.domain()[0])}
                            x2={xScale(
                                xScale.domain()[xScale.domain().length - 1],
                            )}
                            y1={yScale(tick)}
                            y2={yScale(tick)}
                            stroke="#afafaf"
                            stroke-width="0.5"
                        />
                    {/if} -->
                    {#each uniqueProteinNames as name, j}
                        {#if i % 2 === 0}
                            <line
                                x1={xScale(name) - 5}
                                x2={xScale(name) + 5}
                                y1={yScale(tick)}
                                y2={yScale(tick)}
                                class="line"
                            />{/if}
                    {/each}
                {/each}
            {/if}
            {#each xGroups as [x, values], i}
                <line
                    class="lines"
                    x1={xScale(x)}
                    x2={xScale(x)}
                    y1={yScale(d3.min(values, (d) => yValue(d)))}
                    y2={1100}
                ></line>
                <path
                    fill="transparent"
                    class="lines"
                    stroke="black"
                    stroke-linejoin="bevel"
                    d={linkGen
                        ? linkGen({
                              source: [xScale(x), 1100],
                              target: [
                                  xScale(xGroups[Math.floor(i / 3) * 3 + 1][0]),
                                  750,
                              ],
                          })
                        : ""}
                />
            {/each}
            {#each xGroups as [x, values], i}
                <rect
                    x={xScale(x) - 20}
                    y={yScale(d3.max(values, (d) => yValue(d))) / 2 + 400}
                    width="40"
                    height="30"
                    fill="#FAFAFA"
                    stroke="none"
                />

                <text
                    x={xScale(x)}
                    y={yScale(d3.max(values, (d) => yValue(d))) / 2 + 415}
                    class="labelText"
                    text-anchor="middle"
                    dominant-baseline="middle"
                >
                    {x.split("_")[1]}
                </text>
                <g transform="translate(0 0)">
                    {#if i == 0}
                        <path
                            d="M {xScale(x) - 10} {yScale(
                                d3.max(values, (d) => yValue(d)),
                            ) - 265} Q 0 400 {width * 0.09} 350"
                            class="introLine"
                        />
                    {/if}

                    {#if i == 4}
                        <path
                            d="M {xScale(x) - 10} {yScale(
                                d3.max(values, (d) => yValue(d)),
                            ) - 235} Q {width * 0.25} 800 {width * 0.38} 480"
                            class="introLine"
                        />
                    {/if}

                    {#if i == 8}
                        <path
                            d="M {xScale(x) + 10} {yScale(
                                d3.max(values, (d) => yValue(d)),
                            ) - 240} Q {width} 800 {width * 0.9} 420"
                            class="introLine"
                        />
                    {/if}
                </g>

                {#if i % 3 === 0}
                    <!-- <circle
                        cx={xScale(xGroups[Math.floor(i / 3) * 3 + 1][0])}
                        cy={600 - 60}
                        r="50"
                        fill="none"
                        stroke="#dfdfdf"
                    ></circle> -->
                    <!-- <circle
                        cx={xScale(xGroups[Math.floor(i / 3) * 3 + 1][0])}
                        cy={700 - 60}
                        r="50"
                        fill="#fafafa"
                        stroke="none"
                    ></circle> -->

                    <text
                        x={xScale(xGroups[Math.floor(i / 3) * 3 + 1][0])}
                        class="labelText"
                        y={720}
                        text-anchor="middle"
                        style="font-size: 10pt;"
                    >
                        {#if x.split("_")[0] === "AD"}
                            Alzheimer’s
                            <tspan
                                dy="1.4em"
                                x={xScale(
                                    xGroups[Math.floor(i / 3) * 3 + 1][0],
                                )}
                            >
                                Disease
                            </tspan>
                        {:else if x.split("_")[0] === "ACD"}
                            All-Cause
                            <tspan
                                dy="1.4em"
                                x={xScale(
                                    xGroups[Math.floor(i / 3) * 3 + 1][0],
                                )}
                            >
                                Dementia
                            </tspan>
                        {:else if x.split("_")[0] === "VAD"}
                            Vascular
                            <tspan
                                dy="1.4em"
                                x={xScale(
                                    xGroups[Math.floor(i / 3) * 3 + 1][0],
                                )}
                            >
                                Dementia
                            </tspan>
                        {/if}
                    </text>
                {/if}

                <g
                    transform="translate({xScale(
                        xGroups[Math.floor(i / 3) * 3 + 1][0],
                    )}, {750 - 120})"
                >
                    <!-- <defs>
                        <pattern
                            id="diagonalHatch"
                            patternUnits="userSpaceOnUse"
                            width="20"
                            height="20"
                        >
                            <path
                                d="M 0 20 L 20 0"
                                stroke="#afafaf"
                                stroke-width="1"
                                shape-rendering="crispEdges"
                            />
                        </pattern>
                    </defs>
                    <defs>
                        <pattern
                            id="dense-grid"
                            patternUnits="userSpaceOnUse"
                            width="20"
                            height="20"
                        >
                            <path
                                d="M 0 0 L 20 0 M 0 20 L 0 0"
                                fill="none"
                                stroke="#cfcfcf"
                                stroke-width="2"
                                shape-rendering="crispEdges"
                            />
                        </pattern>
                    </defs> -->
                    <!-- <path
                        d={d3
                            .areaRadial()
                            .angle((d) => yearScale(-d.target_yrs))
                            .innerRadius(50)
                            .outerRadius((d) => levelScale(parseFloat(d.Level)))
                            .curve(d3.curveCardinal)(values)}
                        fill="none"
                        stroke={i % 3 === 2 ? "#afafaf" : "#dfdfdf"}
                        stroke-width="0.5"
                    ></path> -->
                    <path
                        d={d3
                            .lineRadial()
                            .angle((d) => yearScale(-d.target_yrs))
                            .radius((d) => levelScale(parseFloat(d.Level)))
                            .curve(d3.curveCardinalClosed)(values)}
                        fill="none"
                        stroke="#1f1f1f"
                        stroke-width="0.5"
                    ></path>

                    {#if HRs}
                        <line
                            x1="0"
                            y1="0"
                            x2={i % 3 == 0
                                ? d3.pointRadial(HRScale(HRs[i].HR), 50)[0]
                                : i % 3 == 1
                                  ? d3.pointRadial(HRScale(HRs[i].HR), 40)[0]
                                  : d3.pointRadial(HRScale(HRs[i].HR), 25)[0]}
                            y2={i % 3 == 0
                                ? d3.pointRadial(HRScale(HRs[i].HR), 50)[1]
                                : i % 3 == 1
                                  ? d3.pointRadial(HRScale(HRs[i].HR), 40)[1]
                                  : d3.pointRadial(HRScale(HRs[i].HR), 25)[1]}
                            stroke={i % 3 == 0 ? "#1f1f1f" : "black"}
                            stroke-width={i % 3 == 0 ? 0.5 : 1}
                        />
                    {/if}
                    {#each HRScale.ticks() as tick, i}
                        <line
                            x1={d3.pointRadial(HRScale(tick), 55)[0]}
                            y1={d3.pointRadial(HRScale(tick), 55)[1]}
                            x2={d3.pointRadial(HRScale(tick), 60)[0]}
                            y2={d3.pointRadial(HRScale(tick), 60)[1]}
                        />
                    {/each}

                    <!-- <line
                        stroke-dasharray="4 2"
                        x1={d3.pointRadial(
                            yearScale(-values[0].target_yrs),
                            levelScale(parseFloat(values[0].Level)),
                        )[0]}
                        y1={d3.pointRadial(
                            yearScale(-values[0].target_yrs),
                            levelScale(parseFloat(values[0].Level)),
                        )[1]}
                        x2={d3.pointRadial(
                            yearScale(-values[values.length - 1].target_yrs),
                            levelScale(
                                parseFloat(values[values.length - 1].Level),
                            ),
                        )[0]}
                        y2={d3.pointRadial(
                            yearScale(-values[values.length - 1].target_yrs),
                            levelScale(
                                parseFloat(values[values.length - 1].Level),
                            ),
                        )[1]}
                    /> -->
                    {#each values as value}
                        <!-- <text
                            x={d3.pointRadial(
                                yearScale(-value.target_yrs),
                                levelScale(parseFloat(value.Level)),
                            )[0]}
                            y={d3.pointRadial(
                                yearScale(-value.target_yrs),
                                levelScale(parseFloat(value.Level)),
                            )[1]}
                            font-size="8pt"
                        >
                            {x}
                        </text> -->
                    {/each}
                </g>
            {/each}
            <!-- {#each xGroups as [x, values], i}
                {#each values as value}
                    <circle
                        transform="translate({xScale(
                            xGroups[Math.floor(i / 3) * 3 + 1][0],
                        )}, {600 - 60})"
                        cx={d3.pointRadial(
                            yearScale(-value.target_yrs),
                            levelScale(parseFloat(value.Level)),
                        )[0]}
                        cy={d3.pointRadial(
                            yearScale(-value.target_yrs),
                            levelScale(parseFloat(value.Level)),
                        )[1]}
                        r="1"
                        stroke="none"
                        fill="#1f1f1f"
                    />
                {/each}
            {/each} -->
        </g>

        <g id="falling">
            <g id="g1">
                <path
                    id="path1"
                    class="cls-1"
                    d="M462.37,435.14c-6.35,3.94-13.3,7.85-17.01,9.14-2.97,1.03-7.97,1.6-12.02,2.15-3.97,.54-8.83,.19-11.45,1.07-2.95,.99-2.88,1.93-4.83,2.98-2.58,1.4-3.25,2.62-10.58,4.97-4.89,1.57-10.23,2.83-12.68,2.33-1.82-.37-2.5-1.12-3.43-2.28-1.07-1.33-1.96-4.68-2.94-7.64-1.05-3.19-2.35-7.21-2.76-9.95-.37-2.43,.32-5.62-.21-7.54-.77-2.74-1.3-3.48-3.96-6.96-2.19-2.87-6.77-6.44-9.87-9.84-2.94-3.22-5.84-7.07-7.5-9.06-18.29-18.95-27.45-48.87-53.9-59.5-18.42,9.7-42.26,.28-61.42,8.01-46.53,28.34-44.21,14.65-89.4,.97-4.6-1.18-13.03-3.74-20.65-5.68-8.06-2.04-18.27-4.3-24.79-5.77-5.85-1.32-13.67-2.67-18.84-4.09-5.19-1.43-12.79-4.03-18.52-6.22-5.21-1.99-10.67-4.41-13.35-5.71-2.35-1.14-6.42-3.29-9.83-5.36-3.78-2.29-9.13-5.46-12.5-8.37-3.66-3.15-5.88-5.29-7.1-7.78-1.06-2.16-1.6-6.6-1.82-10.49-.23-4.01,.77-8.76,.67-11.77-.13-3.8-.78-5.14-1.2-9.6-.41-4.44-1.25-4.79-.46-7.49,.46-1.55,2.59-2.5,2.48-3.36-.12-.93-1.99-1.7-3.72-3.04-2-1.55-4.52-3.44-6.3-4.64-1.41-.95-2.5-1.68-3.3-2.04-.79-.35-2.29-.28-3.62-1.03-1.34-.75-2.21-2.04-2.77-2.57-.43-.41-.99-.97-1.56-1.49-.95-.87-3.38-2.36-5.13-4.49-2.58-3.13-3.86-5.09-5.02-8.66-.87-2.69-1.3-6.6-.96-9.31,.36-2.85,1.54-5.8,3.04-8.22,1.43-2.33,4.37-5.08,6.55-7.06,2.34-2.12,6.09-5.04,9.16-6.98,3.17-2,7.73-4.4,11.13-5.75,3.45-1.36,8.71-2.4,12.49-3.54,3.82-1.16,8.88-3.23,12.66-4.08,4.56-1.02,5.63-2.28,7.32-.94,1.13,.9,.71,3.08,.49,4.91-.23,1.93-2.02,4.37-2.24,6.41-.27,2.43-.04,2.73,.52,4.3,.48,1.35,1.15,2.45,2.3,3.66,1.67,1.78,2.63,2.22,6.56,4.68,2.69,1.69,5.92,2.21,6.35,3.48,.33,.94-1.25,2.31-1.65,3.84-.55,2.12-.98,3.37-.58,5.85,.28,1.74,1.07,3.23,1.53,3.43,.64,.29,2.94-.64,5.26-1.71,3.41-1.56,5.94-2.93,8.78-5.34,2.26-1.91,4.59-5.02,6.12-7.07,1.46-1.94,3.24-4.79,4.41-6.85,1.05-1.83,1.89-4.08,2.61-5.18,.58-.89,1.62-1.78,2.2-2.7,.63-.99,1.23-2.41,1.84-3.81,.66-1.53,1.5-3.56,1.98-5.19,.58-1.94,1.06-4.7,1.65-7.57,.61-2.96,1.23-6.89,1.87-9.71,.63-2.76,1.86-6.33,2.38-8.96,.49-2.47,.95-5.66,1.03-7.75,.07-2.06-.4-5.06-.45-7.51-.05-2.71-.08-6.02,.23-8.89,.31-2.89,1.34-6.96,1.88-10.02,.52-2.89,1.4-6.32,1.44-8.46,.05-2.25-.22-4.67-.95-7.52-.76-2.95-2.59-7.38-4.24-10.77-1.91-3.92-4.58-7.46-6.32-8.88-.93-.76,.45-2.93,1.62-3.23,1.52-.39,6.63,4.31,7.82,3.61,1.7-1-.46-3.75,1.52-6.89,1.61-2.56,3.27-6.17,4.58-8.69,.78-1.49,5.01-5.9,6.22-5.26,1.2,.64-1.02,7.34-1.75,9.06-1.57,3.69-1.86,5.3-1.5,5.46,.42,.19,1.6-5.08,3.12-8.69,1.95-4.62,5.84-8.87,6.72-8.43,.35,.17,.96,1.48,.62,3.11-.92,4.4-6.14,14.41-5.49,14.64,.97,.35,6.81-14.14,8.59-15.26,.97-.61,1.98-1.21,2.73,.75,.53,1.39-5.37,13.44-4.99,15.51,.23,1.28,6.73-10.91,7.96-10.29,3.05,1.51-.69,8.32-.79,9.93-.29,4.47-2.06,10.36-2.84,14.61-.73,3.97-1.9,8.9-2.12,12.15-.23,3.44,.04,4.19,.23,5.96,.21,2.01,.9,3.62,1.44,8.4,.46,4.06,.51,9.2,.71,12.52,.2,3.23,.71,7.98,.76,11.66,.06,4.13-.22,10.59-.56,15.7-.35,5.3-.87,12.43-1.69,17.3-.85,5.05-3.07,12.55-4.42,18.47-1.14,5.04-3.6,9.93-2.26,10.59,.94,.46,2.49-2.75,4.45-5.26,2.35-3,5.37-7.18,7.32-10.18,1.71-2.63,3.49-5.65,4.34-7.47,.61-1.31,.57-3.16,1.22-3.41,.56-.21,1.6,.26,2.37,1.31,1.16,1.56,.93,2.96,1.09,5.22,.11,1.6-.69,3.02-.23,3.39,.59,.47,2.61-.87,4.71-1.4,2.71-.68,6.89-1.75,10.31-2.13,3.14-.35,6.51-.63,8.04-.15,.43,.14,5.53,1.34,1.77,2.93-5.41,2.29-20.38,5.21-19.88,5.56,.81,.56,23.24-4.96,23.97-4.43,.79,.56,1.74,1.01,2.36,2.11,.25,.45-.51,2.11-3.85,3.35-4.81,1.78-20.29,6.35-20.25,6.92,.04,.62,14.58-3.8,19.5-4.19,3.02-.24,4.72,.84,4.59,1.99-.05,.48-3.54,1.89-8.33,3.59-5.61,1.99-11.93,3.89-12.3,4.7-.63,1.39,16.35-2.7,16.85-1.2,.8,2.37-7,3.81-8.17,4.32-3.76,1.66-8.49,4.68-12.49,6.4-3.79,1.63-8.56,3.44-11.31,4.99-2.96,1.66-4.95,2.57-6.82,4.73-1.2,1.38-2,2.85-1.74,3.45,.39,.92,3.69,1.08,6.64,2.02,4.13,1.31,11.16,3.59,17.07,5.69,6.2,2.2,14.24,5.15,19.46,7.35,4.76,2.01,10.6,5.34,14.75,6.88,4.68,1.74,7,3.08,10.48,2.94,2.15-.09,3.99-1.19,4.85-1.79,.66-.46,1.27-1.54,1.9-2.25,.8-.9,2.38-2.36,3.58-3.82,1.94-2.37,5.56-7.76,9.19-12.84,3.96-5.55,8.99-12.55,12.16-17.08,2.76-3.94,6.16-8.8,8.21-11.89,1.94-2.94,4.45-7.6,6.85-10.83,2.29-3.08,4.94-6.95,7.3-8.42,2.9-1.8,5.43-2.08,10.14-2.64,3.88-.46,9.26,.39,13.03,.27,3.31-.1,6.89-.46,8.9-.75,2.57-.37,8.13-1.84,13.75-2.8,6.03-1.03,13.31-1.79,17.62-2.45,4.23-.64,11.16-1.78,16.86-2.76,5.86-1.01,13.41-2.22,18.29-3.28,4.19-.91,9.35-1.43,11.81-3,2.9-1.86,3.2-2.96,4.99-6.15,1.17-2.09,1.52-4.72,2.11-5.95,.48-1.01,1.59-2.34,2.17-3.54,.52-1.09,.74-2.33,1.04-2.9,.24-.45,.67-.92,1.08-1.48s.82-1.04,1.08-1.48c.36-.61,.97-1.95,1.56-3.34,.77-1.82,2.1-4.76,2.82-7.37,.74-2.69,1.41-6.2,1.48-8.5,.07-2.39-.82-6.2-1.08-9.41-.26-3.15-.97-6.8-.43-8.94,.43-1.7,1.51-3.42,2.48-4.13,.82-.6,2.21-1.05,3.08-.75,1.49,.51,3.04,3.03,5.08,6.39,2.24,3.68,4.54,9.53,6.2,13.9,1.57,4.12,3.18,9.14,3.88,12.19,.65,2.83,1.29,7.2,1.53,10.65,.25,3.66-.58,8.92-.05,12.7,.7,4.98,1.22,4.75,2.04,6.99,.92,2.53,2.19,2.55,4.02,9.12,1.24,4.44,1.88,9.59,1.58,12.27-.21,1.9-.79,4.01-1.84,4.99-.99,.92-2.98,.94-4.41,1.26-1.59,.35-4,.69-5.93,.97-2.18,.32-5.52,1.12-8.24,1.05-2.8-.07-6.3-1.59-8.83-1.58-3.1,0-2.68-.61-3.98,.74-1.16,1.19-.46,4.73-1.15,7.41-.67,2.65-1.38,5.62-2.45,6.79-1.13,1.25-4.31,2.39-7.3,3.44-3.45,1.21-8.5,2.36-12.19,2.91-3.78,.56-9.32,.18-13.36,.82-6.21,1-8.28,1.72-16.03,4.22-5.79,1.87-12.74,5.18-18.44,7.07-5.21,1.73-12.05,3.3-16.37,4.64-3.56,1.11-7.75,1.82-9.34,3.15-2.02,1.69-2.8,4.43-5.43,9.86-2.28,4.71-4.43,11.13-6.38,15.15-1.82,3.75-4.86,8.52-6.61,12-1.41,2.8-3.34,5.34-2.84,6.23,.55,.98,4.17,.63,7.73,.94,4.77,.41,12.46,.74,18.69,.97,6.87,.25,16.93,.15,24.46,.54,6.93,.36,14.82,.27,18.94,1.53,3.84,1.18,6.73,3.11,9.65,5.45,2.37,1.9,4.68,5.1,6.34,7.02,1.86,2.15,4.99,6.07,7.8,9.38,2.85,3.36,6.3,7.32,8.35,9.65,1.79,2.03,4.33,4.85,6.15,6.86,2.02,2.23,5.32,5.79,8,8.69,2.6,2.82,5.6,6.08,7.32,7.93,1.84,1.97,5.29,5.7,8.47,9,3.44,3.58,8,8.21,11.07,11.28,2.83,2.83,6.61,6.47,9.11,8.93,2.1,2.06,5.31,3.82,5.48,5.39,.28,2.64-2.31,3.62-7.47,11.28-4.9,7.28-8.24,8.68-7.54,11.03,.3,1.02,2.3,1.21,3.49,1.45,1.25,.25,3.06-.19,4.33,0,1.34,.21,3.11,.89,4.68,1.5,1.69,.65,3.85,1.83,5.54,2.52,1.48,.6,3.07,1.1,3.88,1.38,1.07,.38,3.58,1.44,6.31,2.19,3.08,.84,7.09,1.78,9.77,1.99,2.36,.18,5.22-.02,7.01-.34,2.14-.38,6.02-2.17,9.76-3.08,3.67-.89,7.48-1.99,9.38-1.61,1.41,.28,3.69,1.2,4.22,2.67,.84,2.3,.55,2.65-1.95,6.07-2.74,3.75-9.03,7.09-16.26,11.59Z"
                />
                <path
                    id="path2"
                    class="cls-2"
                    d="M267.8,179.59c-2.84-.04-7.09-7.28-12.64-13.81-7.85-9.24-20.35-25.96-31.41-39.86-12.11-15.22-29.23-36.22-41.29-51.38-11.12-13.97-25.29-30.25-32.38-41.08-6.91-10.56-11.46-15.36-11.07-20.73,.26-3.61,3.32-7.32,7.26-9.81,4.7-2.98,7.55-3.5,11.91-2.36,3.48,.91,7.55,4.85,9.87,8.41,2.18,3.34,3.58,6.07,2.15,7.23-1.37,1.12-4.21-1.89-6.97-3.75-2.93-1.97-5.37-5.55-7.51-6.23-1.83-.58-4.78-.8-6.78,.54-2.5,1.67-3.74,3.78-3.55,6.97,.31,5.29,9.49,15.23,17.42,26.11,10.08,13.82,27.1,34.09,40.03,50.34,13.52,17,32.5,40.16,45.35,56.99,10.44,13.68,22.28,25.07,21.95,29.97-.08,1.16-1.23,2.47-2.37,2.46Z"
                />
            </g>
        </g>

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
                                opacity={index == 1
                                    ? Math.min(
                                          Math.pow(
                                              scrollTop / (height * 0.45),
                                              2,
                                          ),
                                          1,
                                      )
                                    : 1}
                                class="head"
                                d="M195.5,3.86c-32.4,18.14-89.9,30.33-103.7,56.12-11.67,5.7,7.15,8.24-13.32,10.5-9.81-3.83-8.01,18.83-10.94,6.86,13.92,56.83-1.1,7.75-41.24,145.35-10.3,18.23-7.63,32.7,2.05,46.41,40.84,35.63-78.37,65.24-2.43,111.15,1.66-.1,.77,6.21,2.43,6.1,.67,0,1.22,1.11,1.22,2.48,6.84,12.93,17.39,24.1,19.52,39.62-1.17,7.25,5.05,7.24,1.22,12.2-3.73,13.1,2.2,30.55,5.49,42.53,4.59,13.29-1.92,45.58,15.3,53.24,17.28,13.82,38.77,.52,51.18,17.17,14.57,12.57,52.68,30.65,24.85,48.43-20.06,20.21-48.43,34.85-61.46,61.31-9.44,25.49-18.68,52.91-24.4,79.31-7.56,37.43-9.59,80.05-26.84,113.46l677.45,.3c-17.56-114.24-81.21-220.8-169.77-295.16-14.26-10.37-22.74-16.98-23.39-35.04-7.92-58.12-81.95-55.03-66.56-91.6,30.99-30.14,25.47-83,25.41-122.25-.06-46.38-5.52-80.42-5.54-119.1-.02-31.85-.38-65.49-14.25-92.36-6.27-10.48-4.82-14.07-23.06-31.72-28.93-28-68.82-44.79-107.67-57.22-31.07-5.99-76.33-6.62-103.16-11.91-7.46,1.28-24.29,8.06-28.4,3.86Z"
                            />
                        </mask>
                    </defs>
                    <g mask="url(#mask{index})">
                        {#if areaData}
                            {#each areaData as [year, yearData], i}
                                <g>
                                    <path
                                        class="area"
                                        id="area{i}"
                                        fill={index == 0
                                            ? bgColors[i]
                                            : colors[i]}
                                        d={d3
                                            .area()
                                            .curve(d3.curveMonotoneX)
                                            .x((d, i1) => {
                                                if (i1 == 0) return 0;
                                                if (i1 == yearData.length - 1)
                                                    return width;
                                                return xScale(xValue(d));
                                            })
                                            .y0(height)
                                            .y1((d) => yScale(yValue(d)))(
                                            yearData,
                                        )}
                                        opacity={i === groupedData.length - 1
                                            ? Math.min(
                                                  Math.pow(
                                                      scrollTop /
                                                          (height * 0.2),
                                                      2,
                                                  ),
                                                  0.5,
                                              )
                                            : Math.min(
                                                  Math.pow(
                                                      scrollTop /
                                                          ((height / (i + 1)) *
                                                              0.5),
                                                      2,
                                                  ),
                                                  0.9 - i * 0.1,
                                              )}
                                    />
                                </g>
                            {/each}
                        {/if}

                        {#if index === 1}
                            <!-- <img src={headtext} /> -->

                            <foreignObject
                                {width}
                                {height}
                                id="headText"
                                style="opacity: {Math.min(
                                    Math.pow(scrollTop / (height * 0.1), 2),
                                    1,
                                )};cursor: none;"
                            >
                                <embed
                                    style="cursor: none;"
                                    {width}
                                    {height}
                                    src="../src/assets/final5.svg"
                                    type="image/svg+xml"
                                />
                            </foreignObject>
                        {/if}
                    </g>
                </g>
            {/each}
        </g>

        <g id="circlesAndtext">
            {#each groupedData as [year, yearData], i}
                {#each yearData as d, i1}
                    {#if xValue(d).includes("GFAP")}
                        {#if i === groupedData.length - 1 || i === 0}
                            <!-- <circle
                                cx={xScale(xValue(d))}
                                cy={yScale(yValue(d))}
                                r="18"
                            />
                            <text
                                style="fill: white;font-size:9pt"
                                class="number"
                                text-anchor="middle"
                                dominant-baseline="middle"
                                x={xScale(xValue(d))}
                                y={yScale(yValue(d))}
                            >
                                {yValue(d).toFixed(2)}
                            </text> -->
                        {:else}
                            <!-- <line
                                stroke="black"
                                x1={xScale(xValue(d)) - 5}
                                x2={xScale(xValue(d)) - 10}
                                y1={yScale(yValue(d))}
                                y2={yScale(yValue(d))}
                            /> -->
                            <!-- <line
                                stroke="black"
                                x1={xScale(xValue(d)) + 5}
                                x2={xScale(xValue(d)) + 10}
                                y1={yScale(yValue(d))}
                                y2={yScale(yValue(d))}
                            /> -->
                            <!-- <circle
                                cx={xScale(xValue(d))}
                                cy={yScale(yValue(d))}
                                r="1.5"
                            /> -->
                        {/if}
                        <!-- {:else if xValue(d).includes("GDF15")} -->
                        <!-- <circle
                            cx={xScale(xValue(d))}
                            cy={yScale(yValue(d))}
                            r="1.5"
                        /> -->
                        <!-- <line
                            x1={xScale(xValue(d)) - 3}
                            x2={xScale(xValue(d)) + 3}
                            y1={yScale(yValue(d))}
                            y2={yScale(yValue(d))}
                            style="mix-blend-mode: difference;stroke:white"
                        ></line> -->
                    {:else}
                        <!-- <line
                            x1={xScale(xValue(d)) - 3}
                            x2={xScale(xValue(d)) + 3}
                            y1={yScale(yValue(d)) - 3}
                            y2={yScale(yValue(d)) + 3}
                            style="mix-blend-mode: difference;stroke:white"
                        ></line>
                        <line
                            x1={xScale(xValue(d)) - 3}
                            x2={xScale(xValue(d)) + 3}
                            y1={yScale(yValue(d)) + 3}
                            y2={yScale(yValue(d)) - 3}
                            style="mix-blend-mode: difference;stroke:white"
                        ></line> -->
                        <!-- {#if i1 != 0} -->
                        <!-- <line
                            stroke="black"
                            x1={xScale(xValue(d)) - 5}
                            x2={xScale(xValue(d)) - 10}
                            y1={yScale(yValue(d))}
                            y2={yScale(yValue(d))}
                        /> -->
                        <!-- {/if}} -->
                        <!-- {#if i1 != yearData.length - 1} -->
                        <!-- <line
                            stroke="black"
                            x1={xScale(xValue(d)) + 5}
                            x2={xScale(xValue(d)) + 10}
                            y1={yScale(yValue(d))}
                            y2={yScale(yValue(d))}
                        /> -->
                        <!-- {/if} -->

                        <!-- <circle
                            cx={xScale(xValue(d))}
                            cy={yScale(yValue(d))}
                            r="1.5"
                        /> -->
                    {/if}

                    <!-- <g style="mix-blend-mode: multiply;fill:#afafaf">
                        {#if i === groupedData.length - 1}
                            <polygon
                                points="
                        {xScale(xValue(d)) +
                                    10 * Math.cos((0 * Math.PI) / 4)}, {yScale(
                                    yValue(d),
                                ) +
                                    10 * Math.sin((0 * Math.PI) / 4)}
                        {xScale(xValue(d)) +
                                    10 * Math.cos((1 * Math.PI) / 4)}, {yScale(
                                    yValue(d),
                                ) +
                                    10 * Math.sin((1 * Math.PI) / 4)}
                        {xScale(xValue(d)) +
                                    10 * Math.cos((2 * Math.PI) / 4)}, {yScale(
                                    yValue(d),
                                ) +
                                    10 * Math.sin((2 * Math.PI) / 4)}
                        {xScale(xValue(d)) +
                                    10 * Math.cos((3 * Math.PI) / 4)}, {yScale(
                                    yValue(d),
                                ) +
                                    10 * Math.sin((3 * Math.PI) / 4)}
                        {xScale(xValue(d)) +
                                    10 * Math.cos((4 * Math.PI) / 4)}, {yScale(
                                    yValue(d),
                                ) +
                                    10 * Math.sin((4 * Math.PI) / 4)}
                        {xScale(xValue(d)) +
                                    10 * Math.cos((5 * Math.PI) / 4)}, {yScale(
                                    yValue(d),
                                ) +
                                    10 * Math.sin((5 * Math.PI) / 4)}
                        {xScale(xValue(d)) +
                                    10 * Math.cos((6 * Math.PI) / 4)}, {yScale(
                                    yValue(d),
                                ) +
                                    10 * Math.sin((6 * Math.PI) / 4)}
                        {xScale(xValue(d)) +
                                    10 * Math.cos((7 * Math.PI) / 4)}, {yScale(
                                    yValue(d),
                                ) +
                                    10 * Math.sin((7 * Math.PI) / 4)}
                    "
                            />
                        {:else if i === groupedData.length - 2}
                            <polygon
                                points="
        {xScale(xValue(d)) + 10 * Math.cos((0 * 2 * Math.PI) / 7)}, {yScale(
                                    yValue(d),
                                ) +
                                    10 * Math.sin((0 * 2 * Math.PI) / 7)}
        {xScale(xValue(d)) + 10 * Math.cos((1 * 2 * Math.PI) / 7)}, {yScale(
                                    yValue(d),
                                ) +
                                    10 * Math.sin((1 * 2 * Math.PI) / 7)}
        {xScale(xValue(d)) + 10 * Math.cos((2 * 2 * Math.PI) / 7)}, {yScale(
                                    yValue(d),
                                ) +
                                    10 * Math.sin((2 * 2 * Math.PI) / 7)}
        {xScale(xValue(d)) + 10 * Math.cos((3 * 2 * Math.PI) / 7)}, {yScale(
                                    yValue(d),
                                ) +
                                    10 * Math.sin((3 * 2 * Math.PI) / 7)}
        {xScale(xValue(d)) + 10 * Math.cos((4 * 2 * Math.PI) / 7)}, {yScale(
                                    yValue(d),
                                ) +
                                    10 * Math.sin((4 * 2 * Math.PI) / 7)}
        {xScale(xValue(d)) + 10 * Math.cos((5 * 2 * Math.PI) / 7)}, {yScale(
                                    yValue(d),
                                ) +
                                    10 * Math.sin((5 * 2 * Math.PI) / 7)}
        {xScale(xValue(d)) + 10 * Math.cos((6 * 2 * Math.PI) / 7)}, {yScale(
                                    yValue(d),
                                ) +
                                    10 * Math.sin((6 * 2 * Math.PI) / 7)}
    "
                            />
                        {:else if i === 3}
                            <polygon
                                points="
                                {xScale(xValue(d)) + 10 * Math.cos(0)}, {yScale(
                                    yValue(d),
                                ) +
                                    10 * Math.sin(0)}
                                {xScale(xValue(d)) +
                                    10 * Math.cos(Math.PI / 3)},
                                 {yScale(yValue(d)) +
                                    10 * Math.sin(Math.PI / 3)}
                                {xScale(xValue(d)) +
                                    10 * Math.cos((2 * Math.PI) / 3)},
                                 {yScale(yValue(d)) +
                                    10 * Math.sin((2 * Math.PI) / 3)}
                                {xScale(xValue(d)) +
                                    10 * Math.cos(Math.PI)}, {yScale(
                                    yValue(d),
                                ) +
                                    10 * Math.sin(Math.PI)}
                                {xScale(xValue(d)) +
                                    10 * Math.cos((4 * Math.PI) / 3)}, {yScale(
                                    yValue(d),
                                ) +
                                    10 * Math.sin((4 * Math.PI) / 3)}
                                {xScale(xValue(d)) +
                                    10 * Math.cos((5 * Math.PI) / 3)}, {yScale(
                                    yValue(d),
                                ) +
                                    10 * Math.sin((5 * Math.PI) / 3)}
                            "
                            />
                        {:else if i === 2}
                            <polygon
                                points="
                        {xScale(xValue(d)) +
                                    10 * Math.cos(-Math.PI / 10)}, {yScale(
                                    yValue(d),
                                ) +
                                    10 * Math.sin(-Math.PI / 10)}
                        {xScale(xValue(d)) +
                                    10 *
                                        Math.cos(
                                            (2 * Math.PI) / 5 - Math.PI / 10,
                                        )}, {yScale(yValue(d)) +
                                    10 *
                                        Math.sin(
                                            (2 * Math.PI) / 5 - Math.PI / 10,
                                        )}
                        {xScale(xValue(d)) +
                                    10 *
                                        Math.cos(
                                            (4 * Math.PI) / 5 - Math.PI / 10,
                                        )}, {yScale(yValue(d)) +
                                    10 *
                                        Math.sin(
                                            (4 * Math.PI) / 5 - Math.PI / 10,
                                        )}
                        {xScale(xValue(d)) +
                                    10 *
                                        Math.cos(
                                            (6 * Math.PI) / 5 - Math.PI / 10,
                                        )}, {yScale(yValue(d)) +
                                    10 *
                                        Math.sin(
                                            (6 * Math.PI) / 5 - Math.PI / 10,
                                        )}
                        {xScale(xValue(d)) +
                                    10 *
                                        Math.cos(
                                            (8 * Math.PI) / 5 - Math.PI / 10,
                                        )}, {yScale(yValue(d)) +
                                    10 *
                                        Math.sin(
                                            (8 * Math.PI) / 5 - Math.PI / 10,
                                        )}
                    "
                            />
                        {:else if i === 1}
                            <polygon
                                points="
        {xScale(xValue(d)) - 5}, {yScale(yValue(d)) - 5}
        {xScale(xValue(d)) + 5}, {yScale(yValue(d)) - 5}
        {xScale(xValue(d)) + 5}, {yScale(yValue(d)) + 5}
        {xScale(xValue(d)) - 5}, {yScale(yValue(d)) + 5}
    "
                            />
                        {:else if i === 0}
                            <polygon
                                points="
        {xScale(xValue(d))}, {yScale(yValue(d)) - 8.66}
        {xScale(xValue(d)) - 5}, {yScale(yValue(d)) + 4.33}
        {xScale(xValue(d)) + 5}, {yScale(yValue(d)) + 4.33}
    "
                            />
                        {/if}
                    </g> -->

                    <!-- <circle
                        fill="none"
                        stroke="#3f3f3f"
                        cx={xScale(xValue(d))}
                        cy={yScale(yValue(d))}
                        r={10 + 4 * (groupedData.length - i)}
                        style="box-shadow: inset 0 0 0 10px black;"
                    /> -->
                    <circle
                        cx={xScale(xValue(d))}
                        cy={yScale(yValue(d))}
                        r={16 - (groupedData.length - i)}
                        fill="none"
                        stroke="#ededed"
                        stroke-width={2 * (groupedData.length - i)}
                        opacity="0.25"
                    />
                    <circle
                        cx={xScale(xValue(d))}
                        cy={yScale(yValue(d))}
                        r={12 - (groupedData.length - i)}
                        fill="none"
                        stroke="#4f4f4f"
                        stroke-width={2 * (groupedData.length - i)}
                    />
                {/each}
            {/each}
        </g>

        <g id="legend">
            <!-- {#if circlePositions}
                <text x="40" y="-15" class="legendText">
                    Time to diagnosis (years)
                </text>
                {#each circlePositions as circle, i}
                    <circle
                        fill="none"
                        stroke="black"
                        cx={circle.cx}
                        cy={circle.cy}
                        r={circle.radius}
                    />
                    <text
                        x={circle.cx +
                            d3.max(circlePositions, (d) => d.radius) +
                            5}
                        y={circle.cy}
                        dominant-baseline="middle"
                    >
                        {-circle.year}
                    </text>
                {/each}
            {/if} -->
            <g>
                <text x="100" y="-10" class="legendTitle">
                    Time to diagnosis
                </text>
                <text x="100" y="5" class="legendTitle"> (yrs) </text>
                <!-- {#each groupedData as [year, yearData], i}
                    <circle
                        cx={100}
                        cy={100 - (10 + 4 * (groupedData.length - i))}
                        r={10 + 4 * (groupedData.length - i)}
                        fill="none"
                        stroke-width="0.5"
                        stroke="#3f3f3f"
                    />
                    <line
                        x1="100"
                        y1={100 -
                            (10 + 4 * (groupedData.length - i)) -
                            (10 + 4 * (groupedData.length - i))}
                        x2={i % 2 === 0 ? 100 + 40 : 100 - 40}
                        y2={100 -
                            (10 + 4 * (groupedData.length - i)) -
                            (10 + 4 * (groupedData.length - i))}
                        stroke="#3f3f3f"
                        stroke-width="0.3"
                    />
                    <text
                        class="legendText"
                        style="font-weight: lighter;font-size:8pt"
                        x={i % 2 === 0 ? 100 + 45 : 100 - 45}
                        y={100 -
                            (10 + 4 * (groupedData.length - i)) -
                            (10 + 4 * (groupedData.length - i))}
                        dominant-baseline="middle"
                        text-anchor={i % 2 === 0 ? "start" : "end"}
                    >
                        {-year}
                    </text>
                {/each} -->
                {#each groupedData as [year, yearData], i}
                    <circle
                        cx={80}
                        cy={35 + 33 * i}
                        r={16 - (groupedData.length - i)}
                        fill="none"
                        stroke="#ededed"
                        stroke-width={2 * (groupedData.length - i)}
                    />
                    <circle
                        cx={80}
                        cy={35 + 33 * i}
                        r={12 - (groupedData.length - i)}
                        fill="none"
                        stroke="#4f4f4f"
                        stroke-width={2 * (groupedData.length - i)}
                    />
                    <text
                        x={120}
                        y={35 + 33 * i}
                        dominant-baseline="middle"
                        text-anchor="end"
                        class="legendText"
                    >
                        {-year}
                    </text>
                {/each}
            </g>
            <g transform="translate({width * 0.87} -1400)">
                <!-- <text x="0" y="0" class="legendTitle">
                    Temporal trajectories of plasma proteins
                </text> -->
                <text
                    x="10"
                    y="20"
                    text-anchor="middle"
                    class="legendTitle"
                    style="font-size: 10pt;"
                >
                    <!-- Dementia -->
                    Protein HR for Disease
                </text>
                <g transform="translate(10 90)">
                    <g style="transform:  scale(0.6);">
                        <!-- <circle x="0" y="0" r="140" fill="none" stroke="#dfdfdf"
                        ></circle> -->
                        <!-- {#if levelScale}
                            {#each yearScale.domain() as yearTick, i}
                                <line
                                    x1={d3.pointRadial(
                                        yearScale(yearTick),
                                        levelScale(levelScale.domain()[0]),
                                    )[0]}
                                    y1={d3.pointRadial(
                                        yearScale(yearTick),
                                        levelScale(levelScale.domain()[0]),
                                    )[1]}
                                    x2={d3.pointRadial(
                                        yearScale(yearTick),
                                        levelScale(
                                            levelScale.domain()[1] + 0.2,
                                        ),
                                    )[0]}
                                    y2={d3.pointRadial(
                                        yearScale(yearTick),
                                        levelScale(
                                            levelScale.domain()[1] + 0.2,
                                        ),
                                    )[1]}
                                    stroke="#dfdfdf"
                                    stroke-width="2"
                                />
                                <text
                                    dominant-baseline="middle"
                                    text-anchor="middle"
                                    x={d3.pointRadial(
                                        yearScale(yearTick),
                                        levelScale(
                                            levelScale.domain()[1] + 0.35,
                                        ),
                                    )[0]}
                                    y={d3.pointRadial(
                                        yearScale(yearTick),
                                        levelScale(
                                            levelScale.domain()[1] + 0.35,
                                        ),
                                    )[1]}
                                >
                                    {yearTick}
                                </text>

                                {#each levelScale.ticks() as levelTick, j}
                                    {#if j % 3 == 2}
                                        <circle
                                            cx="0"
                                            cy="0"
                                            r={levelScale(levelTick)}
                                            fill="none"
                                            stroke="#dfdfdf"
                                        ></circle>
                                    {/if}
                                {/each}
                            {/each}
                        {/if} -->

                        {#each HRs.slice(0, 3) as HR, i}
                            <!-- <path
                                d={d3
                                    .areaRadial()
                                    .angle((d) => yearScale(-d.target_yrs))
                                    .innerRadius(50)
                                    .outerRadius((d, index) =>
                                        levelScale(
                                            parseFloat(d.Level) +
                                                (index % 2 === 0 ? 0.2 : -0.1),
                                        ),
                                    )
                                    .curve(d3.curveCardinal)(values)}
                                fill={i % 3 === 0
                                    ? "#dfdfdf"
                                    : i % 3 === 2
                                      ? "url(#diagonalHatch)"
                                      : "url(#dense-grid)"}
                                stroke={i % 3 === 0 ? "#dfdfdf" : "#afafaf"}
                                stroke-width="0.5"
                            ></path> -->
                            <!-- <line
                                stroke="black"
                                class="line"
                                stroke-dasharray="4 2"
                                x1={d3.pointRadial(
                                    yearScale(-values[0].target_yrs),
                                    levelScale(
                                        parseFloat(values[0].Level) + 0.2,
                                    ),
                                )[0]}
                                y1={d3.pointRadial(
                                    yearScale(-values[0].target_yrs),
                                    levelScale(
                                        parseFloat(values[0].Level) + 0.2,
                                    ),
                                )[1]}
                                x2={d3.pointRadial(
                                    yearScale(
                                        -values[values.length - 1].target_yrs,
                                    ),
                                    levelScale(
                                        parseFloat(
                                            values[values.length - 1].Level,
                                        ) - 0.1,
                                    ),
                                )[0]}
                                y2={d3.pointRadial(
                                    yearScale(
                                        -values[values.length - 1].target_yrs,
                                    ),
                                    levelScale(
                                        parseFloat(
                                            values[values.length - 1].Level,
                                        ) - 0.1,
                                    ),
                                )[1]}
                            /> -->
                            <line
                                x1="0"
                                y1="0"
                                x2={i % 3 == 0
                                    ? d3.pointRadial(
                                          HRScale(HRs[i].HR) - 0.3,
                                          50,
                                      )[0]
                                    : i % 3 == 1
                                      ? d3.pointRadial(
                                            HRScale(HRs[i].HR) - 0.8,
                                            40,
                                        )[0]
                                      : d3.pointRadial(
                                            HRScale(HRs[i].HR) - 0.5,
                                            25,
                                        )[0]}
                                y2={i % 3 == 0
                                    ? d3.pointRadial(
                                          HRScale(HRs[i].HR) - 0.3,
                                          50,
                                      )[1]
                                    : i % 3 == 1
                                      ? d3.pointRadial(
                                            HRScale(HRs[i].HR) - 0.8,
                                            40,
                                        )[1]
                                      : d3.pointRadial(
                                            HRScale(HRs[i].HR) - 0.5,
                                            25,
                                        )[1]}
                                stroke={i % 3 == 0 ? "#1f1f1f" : "black"}
                                stroke-width={i % 3 == 0 ? 0.5 : 1}
                            />
                            <line
                                x1={i % 3 == 0
                                    ? d3.pointRadial(
                                          HRScale(HRs[i].HR) - 0.3,
                                          50,
                                      )[0]
                                    : i % 3 == 1
                                      ? d3.pointRadial(
                                            HRScale(HRs[i].HR) - 0.8,
                                            40,
                                        )[0]
                                      : d3.pointRadial(
                                            HRScale(HRs[i].HR) - 0.5,
                                            25,
                                        )[0]}
                                y1={i % 3 == 0
                                    ? d3.pointRadial(
                                          HRScale(HRs[i].HR) - 0.3,
                                          50,
                                      )[1]
                                    : i % 3 == 1
                                      ? d3.pointRadial(
                                            HRScale(HRs[i].HR) - 0.8,
                                            40,
                                        )[1]
                                      : d3.pointRadial(
                                            HRScale(HRs[i].HR) - 0.5,
                                            25,
                                        )[1]}
                                x2={i % 3 == 0
                                    ? d3.pointRadial(
                                          HRScale(HRs[i].HR) - 0.1,
                                          220,
                                      )[0]
                                    : i % 3 == 1
                                      ? d3.pointRadial(
                                            HRScale(HRs[i].HR) - 1,
                                            160,
                                        )[0]
                                      : d3.pointRadial(
                                            HRScale(HRs[i].HR) - 0.4,
                                            110,
                                        )[0]}
                                y2={i % 3 == 0
                                    ? d3.pointRadial(
                                          HRScale(HRs[i].HR) - 0.3,
                                          50,
                                      )[1]
                                    : i % 3 == 1
                                      ? d3.pointRadial(
                                            HRScale(HRs[i].HR),
                                            40,
                                        )[1]
                                      : d3.pointRadial(
                                            HRScale(HRs[i].HR) - 0.5,
                                            25,
                                        )[1]}
                                stroke-dasharray="4 2"
                                stroke="#1f1f1f"
                                stroke-width="0.5"
                            />
                            <text
                                x={i % 3 == 0
                                    ? d3.pointRadial(
                                          HRScale(HRs[i].HR) + 0.1,
                                          200,
                                      )[0]
                                    : i % 3 == 1
                                      ? d3.pointRadial(
                                            HRScale(HRs[i].HR) - 1,
                                            190,
                                        )[0]
                                      : d3.pointRadial(
                                            HRScale(HRs[i].HR) - 0.15,
                                            140,
                                        )[0]}
                                y={i % 3 == 0
                                    ? d3.pointRadial(
                                          HRScale(HRs[i].HR) - 0.3,
                                          50,
                                      )[1]
                                    : i % 3 == 1
                                      ? d3.pointRadial(
                                            HRScale(HRs[i].HR) - 0.8,
                                            40,
                                        )[1]
                                      : d3.pointRadial(
                                            HRScale(HRs[i].HR) - 0.5,
                                            25,
                                        )[1]}
                                dominant-baseline="middle"
                                text-anchor="middle"
                                class="legendText"
                            >
                                {HR.Protein.split("_")[1]}
                            </text>
                        {/each}
                        {#each xGroups.slice(0, 3) as [x, values], i}
                            <!-- {#each values as value, index}
                                <circle
                                    cx={d3.pointRadial(
                                        yearScale(-value.target_yrs),
                                        levelScale(
                                            parseFloat(value.Level) +
                                                (index % 2 === 0 ? 0.2 : -0.1),
                                        ),
                                    )[0]}
                                    cy={d3.pointRadial(
                                        yearScale(-value.target_yrs),
                                        levelScale(
                                            parseFloat(value.Level) +
                                                (index % 2 === 0 ? 0.2 : -0.1),
                                        ),
                                    )[1]}
                                    r="1"
                                    stroke="none"
                                    fill="#1f1f1f"
                                />
                            {/each} -->
                            <path
                                d={d3
                                    .lineRadial()
                                    .angle((d) => yearScale(-d.target_yrs))
                                    .radius((d) =>
                                        levelScale(parseFloat(d.Level)),
                                    )
                                    .curve(d3.curveCardinalClosed)(values)}
                                fill="none"
                                stroke="#1f1f1f"
                                stroke-width="0.5"
                            ></path>
                        {/each}
                        {#if HRScale}
                            {#each HRScale.ticks() as tick, i}
                                <line
                                    x1={d3.pointRadial(HRScale(tick), 55)[0]}
                                    y1={d3.pointRadial(HRScale(tick), 55)[1]}
                                    x2={d3.pointRadial(HRScale(tick), 60)[0]}
                                    y2={d3.pointRadial(HRScale(tick), 60)[1]}
                                    class="lines"
                                    stroke="black"
                                />
                                <text
                                    x={d3.pointRadial(HRScale(tick), 85)[0]}
                                    y={d3.pointRadial(HRScale(tick), 85)[1]}
                                    dominant-baseline="middle"
                                    text-anchor="middle"
                                    class="legendText"
                                >
                                    {tick.toFixed(1)}
                                </text>
                            {/each}
                        {/if}
                    </g>
                    <foreignObject
                        x="-300"
                        y="-50"
                        width="205"
                        height="200"
                        class="legendText"
                    >
                        <div style="text-align: right;font-size:9pt">
                            * HR (Hazard Ratio) measures the relative risk of
                            developing a dementia based on the presence or level
                            of specific proteins. An HR above 1 suggests a
                            higher risk, while an HR below 1 indicates a
                            protective effect.
                        </div>
                    </foreignObject>
                    <!-- {#if levelScale}
                        {#each yearScale.domain() as yearTick, i}
                            {#if i === 0}
                                <line
                                    x1={(d3.pointRadial(
                                        yearScale(yearTick),
                                        levelScale(
                                            levelScale.domain()[1] + 0.35,
                                        ),
                                    )[0] -
                                        10) *
                                        0.6}
                                    y1={d3.pointRadial(
                                        yearScale(yearTick),
                                        levelScale(
                                            levelScale.domain()[1] + 0.35,
                                        ),
                                    )[1] * 0.6}
                                    x2={(d3.pointRadial(
                                        yearScale(yearTick),
                                        levelScale(
                                            levelScale.domain()[1] + 0.35,
                                        ),
                                    )[0] -
                                        35) *
                                        0.6}
                                    y2={d3.pointRadial(
                                        yearScale(yearTick),
                                        levelScale(
                                            levelScale.domain()[1] + 0.35,
                                        ),
                                    )[1] * 0.6}
                                    stroke="#1f1f1f"
                                    stroke-width="1"
                                    stroke-dasharray="3 3"
                                />
                                <text
                                    class="legendText"
                                    font-size="8pt"
                                    text-anchor="end"
                                    dominant-baseline="middle"
                                    x={(d3.pointRadial(
                                        yearScale(yearTick),
                                        levelScale(
                                            levelScale.domain()[1] + 0.35,
                                        ),
                                    )[0] -
                                        40) *
                                        0.6}
                                    y={d3.pointRadial(
                                        yearScale(yearTick),
                                        levelScale(
                                            levelScale.domain()[1] + 0.35,
                                        ),
                                    )[1] * 0.6}>Time to diagnosis (years)</text
                                >
                            {/if}
                        {/each}
                        {#each levelScale.ticks() as levelTick, j}
                            {#if j % 3 == 2}
                                <text
                                    x="0"
                                    y={(levelScale(levelTick) + 10) * 0.6}
                                    text-anchor="middle"
                                    dominant-baseline="midlle"
                                    style="font-family:'Open Sans', sans-serif"
                                    font-weight="lighter"
                                    font-size="7pt"
                                >
                                    {levelTick}
                                </text>
                            {/if}
                            {#if j === levelScale.ticks().length - 1}
                                <line
                                    x1="0"
                                    x2="0"
                                    y1={(levelScale(levelTick) + 10) * 0.6 + 5}
                                    y2={(levelScale(levelTick) + 10) * 0.6 + 20}
                                    stroke-dasharray="3 3"
                                    stroke="#1f1f1f"
                                />
                                <text
                                    x="40"
                                    y={(levelScale(levelTick) + 10) * 0.6 + 30}
                                    class="legendText"
                                    dominant-baseline="middle"
                                    font-size="8pt"
                                    text-anchor="middle"
                                >
                                    Plasma Protein (ng/mL)
                                </text>
                            {/if}
                        {/each}
                    {/if} -->

                    <g>
                        <!-- {#each ["GDF15", "NEFL", "GFAP"] as p, i}
                            <circle
                                transform="scale(0.6)"
                                cx="0"
                                cy={230 + i * 50}
                                r="20"
                                stroke="#1f1f1f"
                                fill={i === 0
                                    ? "#dfdfdf"
                                    : i === 1
                                      ? "url(#diagonalHatch)"
                                      : "url(#dense-grid)"}
                            />
                            <text
                                transform="scale(0.6)"
                                style="font-size: 14.4pt;"
                                x="30"
                                y={230 + i * 50}
                                class="legendText"
                                dominant-baseline="middle"
                            >
                                {p}
                            </text>
                        {/each} -->
                    </g>
                </g>
            </g>
        </g>

        <g id="proteinIntro" transform="translate(0 130)">
            <g id="NEFLIntro">
                <foreignObject
                    x={width * 0.7}
                    y="250"
                    width={introWidth}
                    height={introHeight}
                >
                    <div class="intoTitle">NEFL</div>
                    <div class="introSubTitle">(Neurofilament Light Chain)</div>
                    <div class="introContent">
                        Levels of NEFL, which indicate axonal damage and are
                        linked to various neurological diseases, begin to change
                        significantly ahead of a dementia diagnosis,
                        highlighting their predictive potential.
                    </div>
                </foreignObject>
            </g>
            <g id="GFAPIntro">
                <foreignObject
                    x={width * 0.4}
                    y="300"
                    width={introWidth}
                    height={introHeight}
                >
                    <div class="intoTitle">GFAP</div>
                    <div class="introSubTitle">
                        (Glial Fibrillary Acidic Protein)
                    </div>
                    <div class="introContent">
                        Levels of GFAP begin to change at least 10 years before
                        a dementia diagnosis, showing high specificity for
                        dementia prediction, indicating its potential use in
                        screening high-risk populations.
                    </div>
                </foreignObject>
            </g>
            <g id="GDFIntro">
                <foreignObject
                    x={width * 0.1}
                    y="170"
                    width={introWidth}
                    height={introHeight}
                >
                    <div class="intoTitle">GDF15</div>
                    <div class="introSubTitle">
                        (Growth Differentiation Factor 15)
                    </div>
                    <div class="introContent">
                        Levels of GDF15, a stress-response protein associated
                        with inflammation and tissue damage, show marked changes
                        before the onset of dementia, underscoring its potential
                        as an early biomarker for the disease.
                    </div>
                </foreignObject>
            </g>
        </g>

        <g id="source">
            <foreignObject x="1170" y="35" width="255" height="58">
                <div>
                    <p>SOURCE:</p>
                    <p>
                        GUO Y, YOU J, ZHANG Y, ect. Plasma proteomic profiles
                        predict future dementia in healthy adults[J/OL]. Nature
                        Aging, 2024, 4(2): 247-260.
                    </p>
                </div>
            </foreignObject>
        </g>
    </svg>
</div>

<style>
    #falling {
        transform: translate(34%, 12%) scale(0.5) rotate(-20deg);
        fill: #1f1f1f;
    }
    .head {
        transform: translate(28%, 56%) scale(1);
        transform-origin: center bottom;
    }

    * {
        margin: 0;
        padding: 0;
        overflow: hidden;
        cursor: none;
    }

    .labelText {
        font-family: "Open Sans", sans-serif;
        padding: 0% 5%;
        color: #1f1f1f;
        font-size: 10pt;
        font-weight: normal;
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
        stroke: #1f1f1f;
        stroke-width: 0.5;
    }

    #number {
        font-family: "Montserrat", sans-serif;
        padding: 0% 5%;
        color: #1f1f1f;
        font-size: 4pt;
    }

    #legend {
        transform: translate(0%, 82%);
        transform-origin: center bottom;
    }

    .legendText {
        font-family: "Open Sans", sans-serif;
        font-size: 10pt;
        font-weight: lighter;
    }

    .legendTitle {
        font-size: 10pt;
        dominant-baseline: middle;
        text-anchor: middle;
        font-family: "Open Sans", sans-serif;
        font-weight: lighter;
    }

    #source {
        transform: translate(2.5%, 94%);
        font-family: "Open Sans", sans-serif;
        font-size: 8pt;
        color: #afafaf;
    }

    .intoTitle {
        font-family: "Tienne", serif;
        font-weight: bold;
        font-size: 12pt;
    }

    .introSubTitle {
        font-family: "Open Sans", sans-serif;
        font-weight: 600;
        font-size: 10pt;
    }

    .introContent {
        font-family: "Open Sans", sans-serif;
        font-weight: lighter;
        font-size: 10pt;
    }

    .introLine {
        stroke: #dfdfdf;
        stroke-width: 2;
        fill: none;
    }
</style>
