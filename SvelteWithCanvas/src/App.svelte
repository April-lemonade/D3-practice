<script>
  import Canvas from "../components/Canvas.svelte";
  import Circle from "../components/Circle.svelte";
  import * as d3 from "d3";
  let data = [];
  setInterval(() => {
    data = Array.from({ length: 10000 }).map(() => {
      return {
        a: Math.random(),
        b: Math.random(),
        r: Math.random(),
        fill: `rgb(${Math.random() * 255}, ${Math.random() * 255}, ${
          Math.random() * 255
        })`,
      };
    });
  }, 2000);
  let width = 1000;
  let height = 500;

  $: xScale = d3.scaleLinear().domain([0, 1]).range([0, width]);

  $: yScale = d3.scaleLinear().domain([0, 1]).range([height, 0]);

  $: rScale = d3
    .scaleLinear()
    .domain([0, 1])
    .range([5, width / 100]);
</script>

<main bind:clientHeight={height} bind:clientWidth={width}>
  <Canvas {width} {height}>
    {#each data as { a, b, r, fill }}
      <Circle x={xScale(a)} y={yScale(b)} r={rScale(r)} {fill} />
    {/each}
  </Canvas>
</main>

<style>
  * {
    margin: 0;
    padding: 0;
  }
</style>
