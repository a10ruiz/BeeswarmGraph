<script>
    export let data;
    export let colorScale;
    export let width;
    import {fly, fade } from "svelte/transition"

    let tooltipWidth;
    $: xPosition = data.x + tooltipWidth > width ? 
    data.x - tooltipWidth : data.x;
</script>


<div class="tootlip"
    in:fly={{ y: 10, duration: 200, delay: 100 }}
    out:fade
    style="position:absolute;
    top: {data.y}px;
    left: {xPosition}px;"
bind:clientWidth={tooltipWidth}>

<h1>{data.country}
    <span class="continent"
    style="background-color: {colorScale(data.continent)};">
    {data.continent}
</span></h1>

<div class="info">
<span class='score'>
    {data.happiness}/10
</span>
</div>

<!-- Creamos una barra dentro del tooltip-->
<span class="bar background" />
<span class="bar foreground" 
style ="background-color: {colorScale(data.continent)};
width:{data.happiness *10}%;"/>
</div>

<style>
    .tootlip {
        background: white;
        box-shadow: 2px 3px 8px rgba(0, 0, 0, 0.15);
        padding: 8px 6px;
        border-radius: 5px;
        pointer-events: none;
        display: flex;
        justify-content: space-between;
        flex-direction: column;
        transition: top 300ms ease, left 300ms ease; /* para que el tooltip pase de un circulo a otro sin que desaparezca y vuelva a aparecer */
    }
    
    .score {
        font-weight: bold;
        font-size: 0.7em;
        }

    .continent {
        font-size: 0.6em;
        text-transform: uppercase;
        padding: 2px;
        border-radius: 6px;
        }

    .info {
        display: flex;
        justify-content: space-between;
        column-gap: 8px;
        padding-top: 7%;
        }

    h1 {
        font-size: 15px;
        font-weight: 500;
        margin-bottom: 4px;
    }

    .bar {
        position: absolute;
        bottom: 0;
        left: 0;
        height: 4px;
        width: 100%;
    }

    .bar.background {
        background-color: #c4c4c4;
    }
</style>