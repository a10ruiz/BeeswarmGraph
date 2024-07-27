<script>
  // Datos
  import data from "$data/data.js";

  // Creamos medidas básicas de la tabla

  let width = 400;
  let height = 400;

  const margin = {
    top: 0,
    right: 15,
    bottom: 30,
    left: 15,
  };

  $: innerWidth = width - margin.left - margin.right; // $ porque lo queremos reactivo
  let innerHeight = height - margin.top - margin.bottom; // por eso aquí no hace falta el $

  // EJES

  import {
    scaleLinear, // linear para escalas de datos numéricos
    scaleBand, // para escalas de datos categóricos
    scaleOrdinal, // para escala de colores
    scaleSqrt, // para un área proporcional a los datos
  } from "d3-scale";

  import { extent } from "d3-array"; // para el máximo y el mínimo de los datos en el domain
  // se puede hacer manual, pero esto lo voy a hacer así para tener la referencia por si hace falta en algún momento
  console.log(extent(data, (d) => d.happiness));

  $: xScale = scaleLinear()
    .domain(extent(data, (d) => d.happiness)) // es 1 y 9 por si no funciona después Jejejeje
    .range([50, innerWidth]);

  //old let yScale = scaleBand()
  // .domain(data.map(d => d.continent))
  // .range([innerHeight, 0]);

  import { mean, rollups } from "d3-array";

  // Generate the average for each continent, so that we can sort according to that
  const continents = rollups(
    data,
    (v) => mean(v, (d) => d.happiness),
    (d) => d.continent,
  ) // Group data by continent and return the group-wide mean
    .sort((a, b) => a[1] - b[1]) // Sort according to value
    .map((d) => d[0]); // Grab the continent name

  //Add a color to each circle according to its continent
  const colorRange = [
    "#c51b7d",
    "#e9a3c9",
    "#fde0ef",
    "#e6f5d0",
    "#a1d76a",
    "#4d9221",
  ];

  const colorScale = scaleOrdinal()
    .domain(continents) // INPUT
    .range(colorRange); // OUTPUT

  //new and ordered Yscale

  let yScale = scaleBand()
    .domain(continents)
    .range([innerHeight, 0])
    .paddingOuter(0.5);

  // Size of the circles (it changes deppending on the width - adaptative to mobile phones-)
  $: radiusScale = scaleSqrt()
    .domain([1, 9]) // happines data range just as in the Xaxis
    .range(width < 568 ? [2, 7] : [3, 9]);

  //FUERZAS DE D3
  import {
    forceSimulation, // this is for simulating forces
    forceY, // this is to arrange nodes vertically
    forceX, // this is to arrange nodes horizontally
    forceCollide, // this is to prevent them from overlapping with each other
  } from "d3-force";

  const RADIUS = 5;
  const simulation = forceSimulation(data); // es un objeto en el que pasan MUCHAS cosas: dentro están los nodos

  //Separamos la simulación en dos bloques
  // es una especie de If, si se actualiza X vuelve a ejecutar la simulation
  $: {
    simulation
      .force(
        "x", // llamamos a la fuerza después (es extraño, pero funciona así)
        forceX()
          .x((d) => xScale(d.happiness)) // igual que con los atributos: una función que llamamos d (pero podría ser X o cualquier cosa) y que equivaldrá a los datos de happines de cada objeto / nodo
          .strength(0.8),
      )
      .force(
        "y",
        forceY()
          .y((d) => (groupByContinent ? yScale(d.continent) : innerHeight / 2))
          .strength(0.2),
      )
      .force(
        "collide",
        forceCollide().radius((d) => radiusScale(d.happiness)),
      )
      .alpha(0.3) // [0, 1] The rate at which the simulation finishes. You should increase this if you want a faster simulation, or decrease it if you want more "movement" in the simulation.
      .alphaDecay(0.0005) // [0, 1] The rate at which the simulation alpha approaches 0. you should decrease this if your bubbles are not completing their transitions between simulation states.
      .restart();
  }

  $: nodes = simulation.nodes();

  let nodes = [];
  simulation.on("tick", () => {
    nodes = simulation.nodes();
  });


  // AXIS --------------------------------------------------------

  import AxisX from "$components/AxisX.svelte";
  import AxisY from "$components/AxisY.svelte";
  import Legend from "$components/Legend.svelte";
  
 // TOOLTIP ---------------------------------------------------------
  let hovered;
  import Tooltip from "$components/Tooltip.svelte";

  import { fade } from "svelte/transition";

// MAKING THE LEGEND INTERACTIVA -------------------------------------

  let hoveredLegend;
  let groupByContinent = false;
</script>

<body>

    <h1>Happiness by Country</h1>
    <Legend {colorScale} bind:hoveredLegend={hoveredLegend}/>
    
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <div class="chart-container" bind:clientWidth={width}
    on:click={(e) => {
      groupByContinent = !groupByContinent; //si clicamos hace que false de groupByContinent pase a ser TRUE
      hovered = null; // evita que se activen las interacciones por deslizar sobre cosas mientras se lleva a cabo esta interacción
    }}
    >

    <svg {width} {height}>
      <g class="chart" transform="translate({margin.left}, {margin.top})">
        <!-- We put the axis here because SVG renders in the order that its Written-->
        <AxisX xScale={xScale} height={innerHeight} width={innerWidth}/>
        <AxisY yScale={yScale} {groupByContinent}/>
        {#if hovered}
        
        <line 
          x1={hovered.x}
          x2={hovered.x}
          y1={innerHeight}
          y2={hovered.y}
          stroke={colorScale(hovered.continent)}
          stroke-width="2"
        />
        {/if}
        {#each nodes as node, index}
          <!-- svelte-ignore a11y-no-noninteractive-tabindex -->
          <circle 
            in:fade={{
              delay: 500 + index*10 // para la transición de inicio que tarde un 500 milisegundos, y luego cada 10 milisegundos un círculo nuevo así van en abanico
              }} 
            cx={node.x}
            cy={node.y}
            r={radiusScale(node.happiness)}
            fill={colorScale(node.continent)}
            stroke={hovered || hoveredLegend // if the circle OR (||) the contient in the Legend 
            ?  hovered === node || hoveredLegend === node.continent // if the nodeis hovered OR the continent is hovered ... 
            ? "black" : "transparent"
            : "transparent"}
            opacity={hovered || hoveredLegend 
             ? hovered === node || hoveredLegend === node.continent 
             ? 1 : 0.4
             : 1}
            on:mouseover={() => {
              hovered = node;
              }}
            on:focus={() => {
              hovered = node;
              }}
          tabindex="0"
            on:mouseleave={() => {
              hovered = null}}
          on:click={(event)=>{
            event.stopPropagation(); // if you click on the circle, it will not trigger anythingelse, it stops but if you click anywhere else everything will work normally
          }}

          />
        {/each}
      </g>
    </svg>

    {#if hovered}
    <Tooltip data={hovered} colorScale={colorScale} width={innerWidth}/>
    {/if}

  
  </div>
</body>

<style>
  :global(.tick text, .axis-title)
  {
    font-size: 12px;
    font-weight: 400;
    fill: hsla(212,10%, 53%, 1);
    user-select: none; /* prevents selection of the text */
  }

  h1
  {
    font-size: 1.25rem;
    margin-bottom: 10px;
    font-weight: 600;
    text-align: center;
  }

  circle {
    transition: stroke 300ms ease, opacity 300ms ease;
    cursor: pointer;
  }

  line {
    transition: stroke 300ms ease;
}
</style>
