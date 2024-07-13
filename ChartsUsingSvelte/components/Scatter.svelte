<script>
    import * as d3 from "d3";
    export let width;
    export let height;
    const csvUrl = "./data/iris.csv";
    // let width = window.innerWidth;
    // let height = window.innerHeight;
    const { csv } = d3;
    let data = [];
    let xValue, yValue;
    let xScale, yScale;
    let xAxis, yAxis;
    const margin = 40;
    let hoveredData = null;
    // $: console.log(hoveredData);
    const parseRow = (d) => {
        d.sepal_length = +d.sepal_length;
        d.sepal_width = +d.sepal_width;
        d.petal_length = +d.petal_length;
        d.petal_width = +d.petal_width;
        return d;
    };

    const main = async () => {
        data = await csv(csvUrl, parseRow);
    };

    $: if (data.length != 0) {
        xValue = (d) => d.petal_length;
        yValue = (d) => d.sepal_length;
        xScale = d3
            .scaleLinear()
            // .domain([d3.min(data, xValue), d3.max(data, xValue)])
            // .domain(d3.extent(data, xValue))
            .domain([0.5, d3.max(data, xValue)])
            .range([margin, width - margin]);
        yScale = d3
            .scaleLinear()
            // .domain(d3.extent(data, yValue))
            .domain([4, d3.max(data, yValue)])
            .range([height - margin, margin]);
        xAxis = d3.axisBottom(xScale);
        yAxis = d3.axisLeft(yScale).tickSize(-width + margin * 2); // 默认是向着axis后面的方向延伸，此处是left，要向反方向（此处是right），就要加负号
        d3.select("svg").selectAll(".x-axis, .y-axis").remove(); // 清除旧的轴，不然会有残影
        d3.select("svg")
            .append("g")
            .attr("class", "x-axis0")
            .attr("transform", `translate(0, ${height - margin})`)
            .call(xAxis);
        d3.select("svg")
            .append("g")
            .attr("class", "y-axis0")
            .attr("transform", `translate(${margin}, 0)`)
            .call(yAxis)
            .select(".domain")
            .remove();
        d3.selectAll(".y-axis .tick line")
            .style("stroke", "#cecece") // 设置刻度线颜色
            .style("stroke-width", "1px"); // 设置刻度线宽度
    }
    main();
</script>

<div
    role="contentinfo"
    class="chart-container"
    bind:clientWidth={width}
    on:mouseleave={() => {
        hoveredData = null;
    }}
>
    <h1>Iris-1</h1>
    <svg {width} {height}>
        {#each data.sort((a, b) => a.petal_width + a.sepal_width - b.pedal_width - b.sepal_width) as d}
            <circle
                tabindex="0"
                role="button"
                cx={xScale(d.petal_length)}
                cy={yScale(d.sepal_length)}
                opacity={d == hoveredData ? "1" : "0.3"}
                r={d == hoveredData
                    ? (d.petal_width + d.sepal_width) * 2
                    : (d.petal_width + d.sepal_width) }
                fill={d.species === "virginica"
                    ? "red"
                    : d.species === "versicolor"
                      ? "green"
                      : "blue"}
                on:mouseover={() => {
                    hoveredData = d;
                }}
                on:focus={() => {
                    hoveredData = d;
                }}
            ></circle>
        {/each}
    </svg>
    {#if hoveredData}
        <div
            class="tooltip"
            style="position: absolute;left:{xScale(
                hoveredData.petal_length,
            )}px;top:{yScale(hoveredData.sepal_length)}px;border:solid 1px"
        >
            <p>species:</p>
            <p>{hoveredData.species}</p>
        </div>
    {/if}
</div>

<style>
    * {
        margin: 0;
        padding: 0;
    }

    circle {
        transition: opacity 300ms ease;
        cursor: pointer;
    }

    circle:focus {
        outline: none;
    }

    .tooltip {
        padding: 5px;
        transition:
            top 300ms ease,
            left 300ms ease;
        pointer-events: none;
    }
</style>
