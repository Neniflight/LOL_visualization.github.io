<script>
  import { json, select } from "d3";
  import { geoPath, geoNaturalEarth1 } from "d3";
  import { onMount } from 'svelte';
  import App from '../components/App.svelte';

  let dataset = null;
  let path = null;
  let projection = null;
  //pins to mark where the regions are
  let pins = [
    { name: "USA", region:"North America", initials:"LCS", coordinates: [-100, 43] },
    { name: "France", region:"Europe", initials:"LEC", coordinates: [5, 53] },
    { name: "China", region:"China", initials:"LPL", coordinates: [107, 30] },
    { name: "South Korea", region:"South Korea", initials:"LCK", coordinates: [129, 40] },
    // Add more pins as needed
  ];

  onMount(() => {
    json(
      "https://raw.githubusercontent.com/holtzy/D3-graph-gallery/master/DATA/world.geojson"
    ).then((data) => {
      dataset = data;
      projection = geoNaturalEarth1();
      path = geoPath(projection);
    });
  });

  function handleMouseOver(event, feature) {
    const countryName = feature.properties.name;
    const isPinned = pins.some(pin => pin.name === countryName);
    if (isPinned) {
      select(event.target).attr("fill", "#C89B3C"); // Change fill color to highlight
      select(event.target).attr("stroke-width", "1");
      console.log("Mouse Over");
    }
  }

  function handleMouseOut(event, feature) {
    select(event.target).attr("fill", "white"); // Reset fill color
    select(event.target).attr("stroke-width", "0.3");

    console.log("Mouse Off");

  }

  function handleFocus(event, feature) {
    handleMouseOver(event, feature); // Trigger mouseover effect
  }

  function handleBlur(event, feature) {
    handleMouseOut(event, feature); // Trigger mouseout effect
  }

  
</script>

<style>
  @import '../routes/style.css';
</style>

<div class="container">
  <h1>League of Legends Visualizations by Region</h1>
  {#if dataset}
    <!-- World map drawn by paths -->
    <svg class="worldmap" viewBox="0 0 1000 550"> 
      {#each dataset.features as feature}
        <path
          d={path(feature)}
          fill="white"
          stroke="#010A13"
          stroke-width="0.3"
          on:mouseover={(e) => handleMouseOver(e, feature)}
          on:mouseout={(e) => handleMouseOut(e, feature)}
          on:focus={(e) => handleFocus(e, feature)}
          on:blur={(e) => handleBlur(e, feature)}
          role="region"
        />
      {/each}
      {#each pins as pin}
      <!-- location of pins -->
        <a href={"/" + pin.initials} class="router">
          <line 
          x1={projection(pin.coordinates)[0]} 
          y1={projection(pin.coordinates)[1]+16} 
          x2={projection(pin.coordinates)[0]} 
          y2={projection(pin.coordinates)[1]+5} 
          stroke="gray" 
          stroke-width="5"
        />
        <circle
          cx={projection(pin.coordinates)[0]}
          cy={projection(pin.coordinates)[1]}
          r="8"
          fill="red"
        />
        <rect
          width=80
          height=20
          x={projection(pin.coordinates)[0] + 12}
          y={projection(pin.coordinates)[1] - 8}
          rx=5
          ry=5
          fill="#C8AA6E"
        ></rect>
        <text
          x={projection(pin.coordinates)[0] + 14}
          y={projection(pin.coordinates)[1] + 4}
          font-size="12px"
          font-family="Merriweather Sans"
          font-weight="bold"
          fill="#1E282D"
        >
          {pin.region}
        </text>
        </a>
      {/each}
    </svg>
  {:else}
    <p>Loading...</p>
  {/if}
</div>