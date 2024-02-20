<script>
    import Graph from '../components/Graphs.svelte';
    export let name = "World";
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
    <h1>Hello {name}!</h1>
    <p>Visit <a href="https://svelte.dev/tutorial"> Svelte tutorial</a> to learn how to build Svelte apps.</p>
    <input placeholder="What do you need to do?" value="" />

    <section class="graph">
        <h2 style="margin-top: 15px">todo pie</h2>
        <div>
            <Graph {player_data} {team_data}/>
          </div>
    </section>
</main>

<style>
    main {
        text-align: center;
    }
</style>
