<script>
    export let width;
    export let yScale;
    export let margin;
    const ticks = [0, 20, 40, 60];
</script>

<g transform="translate({margin.left} {margin.top})">
    {#each ticks as tick}
        <text x="0" y={yScale(tick)} fill="#cecece"
            >{tick} {tick == 60 ? " hours studied" : ""}</text
        >
        <line
            x1="0"
            y1={yScale(tick)}
            x2={width - margin.right}
            y2={yScale(tick)}
            stroke={tick == 0 ? "black" : "#cecece"}
        ></line>
    {/each}
</g>
