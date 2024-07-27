<script>
    export let yScale;
    export let groupByContinent;
    const ticks = yScale.domain();

    import { fade } from "svelte/transition";
</script>

<!-- We can not use ticks here because there are names, so this is a scale for NOMINAL / ORDINAL VALUES-->

<g class="axis y">
    {#each ticks as tick, index}
    {#if groupByContinent}
    <g class="tick" in:fade={{delay: index * 100}} out:fade> 
        <text y={yScale(tick)} x={0}>
            {tick}
        </text>
    </g>
    {/if}
    {/each}
    </g>