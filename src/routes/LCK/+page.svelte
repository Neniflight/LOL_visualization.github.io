<script>
    import Graph from '../../components/Graphs.svelte';
    import * as d3 from 'd3';
    import { onMount } from 'svelte';
    
    let player_data = [];
    let team_data = [];

    onMount(async () => {

        const res1 = await fetch('player_data.csv'); 
        const csv1 = await res1.text();
        player_data = d3.csvParse(csv1, d3.autoType)
        console.log(player_data);

        
        const res2 = await fetch('team_data.csv'); 
        const csv2 = await res2.text();
        team_data = d3.csvParse(csv2, d3.autoType)
        console.log(team_data);
    });
</script>

<main>
    <section class="graph">
        <div>
            <Graph {player_data} {team_data} selectedRegion="LCK"/>
          </div>
    </section>
</main>

<style>
    main {
        text-align: center;
    }
</style>
