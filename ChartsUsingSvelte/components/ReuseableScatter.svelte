<script>
    import * as d3 from "d3";
    export let width, height;
    export let data;
    export let xValue, yValue, symbolValue;
    export let xType, yType;
    let xScale, yScale;
    let xAxis, yAxis;
    let symbolScale;
    const margin = 60;
    let hoveredData = null;
    $: if (data.length > 0) {
        hoveredData = null;
        const transitionTime = d3.transition().duration(1000);
        /*在 D3 中，d3.transition() 生成的是一个过渡实例，该实例具有唯一的 ID。
        如果把它放在最外层，那么在 Svelte 的响应式更新机制下，过渡实例可能会被重复使用或丢失其上下文，导致 transition not found 错误。
        因此，最好在每次需要过渡时生成一个新的过渡实例，这样可以确保它在正确的上下文中被使用。*/
        xScale = (
            xType
                ? d3.scalePoint().domain(data.map(xValue)).padding(0.2)
                : d3
                      .scaleLinear()
                      .domain([d3.min(data, xValue), d3.max(data, xValue)])
        ).range([margin, width - margin]);
        yScale = (
            yType
                ? d3.scalePoint().domain(data.map(yValue)).padding(0.2)
                : d3
                      .scaleLinear()
                      .domain([d3.min(data, yValue), d3.max(data, yValue)])
        ).range([height - margin, margin]);
        symbolScale = d3
            .scaleOrdinal()
            .domain(data.map(symbolValue))
            .range(d3.symbols);
        xAxis = d3.axisBottom(xScale);
        yAxis = d3.axisLeft(yScale).tickSize(-width + margin * 2); // 默认是向着axis后面的方向延伸，此处是left，要向反方向（此处是right），就要加负号
        let svg = d3.selectAll("#s1");
        console.log(svg);

        svg.selectAll(".x-axis")
            .data([null])
            .join("g")
            .attr("class", "x-axis")
            .attr("transform", `translate(0, ${height - margin})`)
            .transition(transitionTime)
            .call(xAxis);

        svg.selectAll(".y-axis")
            .data([null])
            .join("g")
            .attr("class", "y-axis")
            .attr("transform", `translate(${margin}, 0)`)
            .transition(transitionTime)
            .call(yAxis);

        svg.selectAll(".y-axis .domain").remove();

        svg.selectAll(".y-axis .tick line")
            .style("stroke", "#cecece") // 设置刻度线颜色
            .style("stroke-width", "1px"); // 设置刻度线宽度
    }
</script>

<div
    role="contentinfo"
    class="chart-container1"
    bind:clientWidth={width}
    on:mouseleave={() => {
        hoveredData = null;
    }}
>
    <h1>Iris-2</h1>
    <svg {width} {height} id="s1">
        {#each data.sort((a, b) => a.petal_width + a.sepal_width - b.pedal_width - b.sepal_width) as d, i}
            <path
                tabindex="0"
                role="button"
                transform={`translate(${xScale(xValue(d))},${yScale(yValue(d))})`}
                style={`transition-delay: ${i * 10}ms;`}
                d={d3.symbol().type(symbolScale(symbolValue(d)))()}
                opacity={d == hoveredData ? "1" : "0.3"}
                fill={symbolValue(d) === "virginica"
                    ? "red"
                    : symbolValue(d) === "versicolor"
                      ? "green"
                      : "blue"}
                on:mouseover={() => {
                    hoveredData = d;
                }}
                on:focus={() => {
                    hoveredData = d;
                }}
            >
            </path>
        {/each}
    </svg>
    {#if hoveredData}
        <div
            class="tooltip"
            style="position: absolute;left:{xScale(
                xValue(hoveredData),
            )}px;top:{yScale(yValue(hoveredData))}px;border:solid 1px"
        >
            <p>species:</p>
            <p>{symbolValue(hoveredData)}</p>
        </div>
    {/if}
</div>

<style>
    * {
        margin: 0;
        padding: 0;
    }

    path {
        transition: transform 1000ms ease;
        cursor: pointer;
    }

    path:focus {
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
