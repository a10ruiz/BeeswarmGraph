<script>
    export let colorScale;
    export let hoveredLegend;

</script>

<!-- Legend -->
 <!-- we use div instead of g because its outside the svg chart -->
<div class="legend" on:mouseleave={() => { hoveredLegend = null; }}>
    {#each colorScale.domain() as continent}

    <!-- svelte-ignore a11y-mouse-events-have-key-events -->
    <p
    on:mouseover={() => { hoveredLegend = continent; }}
    class:unhovered={hoveredLegend &&
    hoveredLegend !== continent}
    >
        <span style="background-color: {colorScale(continent)}"/>
        
        {continent}</p>
    {/each}
</div>

<style>
    .legend {
        display: flex;
        flex-direction: row;
        justify-content: center;
        flex-wrap: wrap; /* if there is not enough space, it will wrap */
        column-gap: 12px;
        row-gap: 6px;
        margin-bottom: 10px;
    }

    span{
        width: 10px;
        height: 10px;
        display: inline-block;
        border-radius: 50%;
        border: 1px solid rgba(0, 0, 0, 0.5);
        text-anchor: middle;
    }

    p{
        font-size: 0.75rem;
        text-transform: uppercase;
        display: flex;
        align-items: center;
        column-gap: 3px;
        transition: opacity 300ms ease;
        cursor: pointer;
    }

    .unhovered{
        opacity: 0.5;
    }
</style>