<script>
    export let height;
    export let xScale;
    export let margin;
    const ticks = [0, 25, 50, 75, 100];
</script>

<g>
    {#each ticks as tick}
        <text
            x={xScale(tick) + margin.left}
            y={height - margin.bottom}
            fill="#cecece"
            dominant-baseline="hanging"
            text-anchor="middle"
            dy="6">{tick}%</text
        >
    {/each}
</g>
