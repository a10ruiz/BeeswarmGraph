<script>
    export let xScale;
    export let height;
    export let width;

    $: ticks = xScale.ticks(4);	
</script>

<g class="axis x">
    {#each ticks as tick}
        <g class="tick" transform="translate({xScale(tick)}, 0)">
            <text x="0" y={height} text-anchor="middle">{tick}</text>
            <line x1="0" y1="0" x2="0" y2={height-16} stroke="#c4c4c4"></line>
        </g>

    {/each}
    <text class="axis-title" x={width} y={height}
    text-anchor="end"
    dominant-baseline="hanging"
    dy="6">
        Happines, out of 10 &rarr;
    </text>
</g>