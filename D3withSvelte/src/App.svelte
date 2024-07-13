<script>
  import data from "./data/data.js";
  import AxisX from "./components/AxisX.svelte";
  import AxisY from "./components/AxisY.svelte";
  import Tooltip from "./components/Tooltip.svelte";

  let width = window.innerWidth;
  let height = window.innerHeight;
  const margin = { top: 20, right: 40, left: 20, bottom: 20 };

  import { scaleLinear } from "d3-scale";
  import { max, sort } from "d3-array";
  $: xScale = scaleLinear()
    .domain([0, max(data, (d) => d.grade)])
    .range([0, width - margin.left - margin.right]);
  $: yScale = scaleLinear()
    .domain([0, max(data, (d) => d.hours)])
    .range([height - margin.top - margin.bottom, 0]);
  let hoveredData;
  $: console.log(hoveredData);
</script>

<h1>Study longer, score better!</h1>
<div
  class="chart-container"
  bind:clientWidth={width}
  on:mouseleave={() => (hoveredData = null)}
  role="main"
>
  <svg {width} {height}>
    <AxisX {height} {xScale} {margin} />
    <AxisY {width} {yScale} {margin} />
    <g class="circles" transform="translate({margin.left} {margin.top})">
      {#each data.sort((a, b) => a.grade - b.grade) as student}
        <circle
          role="button"
          cx={xScale(student.grade)}
          cy={yScale(student.hours)}
          r={hoveredData && hoveredData == student ? "20" : "10"}
          opacity={hoveredData && hoveredData == student ? "1" : "0.3"}
          fill="purple"
          stroke="black"
          on:mouseover={() => {
            hoveredData = student;
          }}
          on:focus={() => {
            hoveredData = student;
          }}
          tabindex="0"
        />
      {/each}
    </g>
  </svg>
  {#if hoveredData}
    <Tooltip data={hoveredData} {xScale} {yScale} />
  {/if}
</div>

<style>
  * {
    margin: 0;
    padding: 0;
  }

  circle {
    transition:
      r 300ms ease,
      opacity 300ms ease;
    cursor: pointer;
  }

  circle:focus {
    outline: none;
  }

  h1 {
    margin: 20px;
  }
</style>
