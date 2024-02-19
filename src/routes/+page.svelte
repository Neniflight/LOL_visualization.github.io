
<script>
  import { json } from "d3";
  import { geoPath, geoNaturalEarth1 } from "d3";
  import { onMount } from 'svelte';

  let dataset = [];
  json(
    "https://raw.githubusercontent.com/holtzy/D3-graph-gallery/master/DATA/world.geojson"
  ).then((data) => {
    dataset = data.features;
  });
  const width = 1200;
  const height = 600;

  const projection = geoNaturalEarth1();
  const path = geoPath(projection);
</script>

<main>
  <svg {width} {height}> 
    {#each dataset as data}
    <path
      d={path(data)}
    />
    {/each}
  </svg>
</main>