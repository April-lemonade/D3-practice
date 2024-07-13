<script>
  import * as d3 from "d3";
  import Scatter from "../components/Scatter.svelte";
  import ReuseableScatter from "../components/ReuseableScatter.svelte";
  import Select, { Option } from "@smui/select";
  let width = window.innerWidth;
  let height = window.innerHeight;
  const csvUrl = "./data/iris.csv";
  const { csv } = d3;
  let data = [];
  let xValue, yValue, symbolValue;
  let columns = [
    "petal_width",
    "sepal_width",
    "petal_length",
    "sepal_length",
    "species",
  ];
  let value = "petal_length";
  let value1 = "sepal_length";
  let xType = false;
  let yType = false;
  const parseRow = (d) => {
    d.sepal_length = +d.sepal_length;
    d.sepal_width = +d.sepal_width;
    d.petal_length = +d.petal_length;
    d.petal_width = +d.petal_width;
    return d;
  };
  const main = async () => {
    data = await csv(csvUrl, parseRow);
    // yValue = (d) => d.sepal_length;
    symbolValue = (d) => d.species;
  };
  $: xValue = (d) => d[value];
  $: yValue = (d) => d[value1];
  $: if (value == "species") {
    xType = true;
  } else {
    xType = false;
  }
  $: if (value1 == "species") {
    yType = true;
  } else {
    yType = false;
  }
  main();
</script>

<main>
  <!-- SMUI Styles -->
  <link
    rel="stylesheet"
    href="https://cdn.jsdelivr.net/npm/svelte-material-ui@7.0.0/bare.min.css"
  />

  <div class="chart-container">
    <Scatter {width} height={height / 2}></Scatter>
  </div>
  <div class="columns margins" style="justify-content: flex-start;">
    <Select bind:value label="X Axis">
      {#each columns as col}
        <Option value={col}>{col}</Option>
      {/each}
    </Select>
    <Select bind:value={value1} label="Y Axis">
      {#each columns as col}
        <Option value={col}>{col}</Option>
      {/each}
    </Select>
  </div>
  <div class="chart-container">
    <ReuseableScatter
      {width}
      {height}
      {data}
      {xValue}
      {yValue}
      {symbolValue}
      {xType}
      {yType}
    ></ReuseableScatter>
  </div>
</main>

<style>
  * {
    margin: 0;
    padding: 0;
  }
</style>
