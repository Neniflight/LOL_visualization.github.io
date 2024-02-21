<script>
	import * as d3 from 'd3';
    import { afterUpdate,  onMount } from 'svelte';
    import { scaleLinear, scaleBand } from 'd3-scale';
    import { writable, get } from 'svelte/store';

    // Store for selected patch
    const selectedPatchStore = writable("");

    export let player_data = []; // Define player_data as a prop
    export let selectedRegion = 'LCK'; // Default selected region
    export let selectedPatch = 13.05; // Default selected patch



    const positions = ['top', 'jng', 'mid', 'bot', 'sup']; // Positions
    
    let groupedData = [];
    let svgScatter;
    let tooltip;

    onMount(async () => {
        await new Promise(resolve => setTimeout(resolve, 500));
        console.log('wdff')
        console.log(player_data)
        console.log(selectedPatch)
        console.log(selectedRegion)
        groupedData = d3.group(player_data, d => d.league, d => d.patch, d => d.position, d => d.playername);
        console.log(groupedData)
        svgScatter = d3.select('#scatter-plot')
            .append('svg')
            .attr('width', 600)
            .attr('height', 400);

        tooltip = d3.select('body')
            .append('div')
            .style('opacity', 0)
            .attr('class', 'tooltip');
            
        renderScatterPlot('top')
    });

      // Subscribe to changes in selectedPatch
      $: {
    selectedPatchStore.set(selectedPatch);
  }

  selectedPatchStore.subscribe(newPatch => {
    updateScatter('top');
  });
    function renderScatterPlot(position) {
    // Clear existing scatter plot elements
    svgScatter.selectAll('*').remove();

    let data = groupedData.get(selectedRegion).get(selectedPatch).get(position);
    let test_data = Array.from(data, ([playername, cspm]) => [playername, cspm.length >= 5 ? d3.mean(cspm, d => d.result) : 0]);

    function calculateKillParticipation(data) {
        return Array.from(data, ([playername, stats]) => {
            let mean_kills = d3.mean(stats, d => d.kills);
            let mean_assists = d3.mean(stats, d => d.assists);
            let mean_teamkills = d3.mean(stats, d => d.teamkills);
            let killParticipation = (mean_kills + mean_assists) / mean_teamkills;
            return [playername, killParticipation];
        });
    }

    function calculateKDA(data) {
        return Array.from(data, ([playername, stats]) => {
            let mean_kills = d3.mean(stats, d => d.kills);
            let mean_assists = d3.mean(stats, d => d.assists);
            let mean_deaths = d3.mean(stats, d => d.deaths);
            let kda = (mean_kills + mean_assists) / mean_deaths;
            return [playername, kda];
        });
    }
    console.log(data)
    // Define accessors for x and y values
    const xAccessorFunc = d => {
        if (position === 'top') return Array.from(data, ([playername, cspm]) => [playername, cspm.length >= 2 ? d3.mean(cspm, d => d.cspm) : 0]);
        else if (position === 'jng') return Array.from(data, ([playername, monsterkills]) => [playername, monsterkills.length >= 2 ? d3.mean(monsterkills, d => d.monsterkills) : 0]);
        else if (position === 'mid') return Array.from(data, ([playername, earnedgpm]) => [playername, earnedgpm.length >= 2 ? d3.mean(earnedgpm, d => d.earnedgpm) : 0]);
        else if (position === 'bot') return calculateKDA(data);
        else if (position === 'sup') return Array.from(data, ([playername, vspm]) => [playername, vspm.length >= 2 ? d3.mean(vspm, d => d.vspm) : 0]);
    };
    const xAccessor = xAccessorFunc(position);
    const yAccessor = calculateKillParticipation(data);
    console.log(xAccessor)

    const margin = { top: 20, right: 20, bottom: 30, left: 40 };
    const width = 600 - margin.left - margin.right;
    const height = 400 - margin.top - margin.bottom;

    const svg = svgScatter.append('g')
        .attr('transform', `translate(${margin.left},${margin.top})`);

    const xScale = d3.scaleLinear()
        .domain([0, d3.max(xAccessor, d => d[1])]).nice()
        .range([0, width]);

    const yScale = d3.scaleLinear()
        .domain([0, d3.max(yAccessor, d => d[1])]).nice()
        .range([height, 0]);

    const xAxis = d3.axisBottom(xScale);
    const yAxis = d3.axisLeft(yScale);

    svg.append('g')
        .attr('transform', `translate(0,${height})`)
        .call(xAxis);

    svg.append('g')
        .call(yAxis);

    let txt = '';

    if (position == 'top') txt = 'CS Per Minute';
    else if (position == 'jng') txt = 'Monster Kills';
    else if (position == 'mid') txt = 'Gold Per Min';
    else if (position == 'bot') txt = 'KDA';
    else if (position == 'sup') txt = 'Vision Score per Min';

    // Add labels
    svg.append('text')
        .attr('transform', `translate(${width / 2},${height + margin.top + 10})`)
        .style('text-anchor', 'middle')
        .text(txt)
        .selectAll("text")
            .attr("fill", "#F0E6D2");

    svg.append('text')
        .attr('transform', 'rotate(-90)')
        .attr('y', 0 - margin.left)
        .attr('x', 0 - (height / 2))
        .attr('dy', '1em')
        .style('text-anchor', 'middle')
        .text('Kill Participation');
    svg.selectAll("text")
            .attr("fill", "#F0E6D2");

    svg.selectAll('circle')
        .data(xAccessor)
        .enter().append('circle')
        .attr('cx', (d, i) => xScale(xAccessor[i][1]))
        .attr('cy', (d, i) => yScale(yAccessor[i][1]))
        .attr('r', 5)
        .attr('fill', 'steelblue')
        .on('mouseover', handleMouseOver)
        .on('mouseout', handleMouseOut);
    svg.selectAll('.domain')
        .attr("stroke", "#F0E6D2"); // Change color of axis lines
    // Add styling to axis lines
    svg.selectAll(".tick line")
        .attr("stroke", "#F0E6D2"); // Change color of axis lines
        

    function handleMouseOver(d, i) {
        const [xPlayername, xValue] = i;
        const yIndex = yAccessor.findIndex(([playername]) => playername === xPlayername);
        const [yPlayername, yValue] = yAccessor[yIndex];

        const tooltipWidth = 180; // Adjust as needed
        const tooltipHeight = 70; // Adjust as needed
        const padding = 10; // Adjust as needed
        const roundedXValue = parseFloat(xValue).toFixed(2); // Round to 2 decimal points
        const roundedYValue = parseFloat(yValue).toFixed(2); // Round to 2 decimal points

        const tooltip = svg.append('g')
            .attr('id', 'tooltip');

        tooltip.append('rect')
            .attr('width', tooltipWidth)
            .attr('height', tooltipHeight)
            .attr('fill', 'white')
            .attr('stroke', 'black')
            .attr('stroke-width', 1);

        tooltip.append('text')
            .attr('x', padding)
            .attr('y', padding + 15)
            .style('font-weight', 'bold')
            .text(xPlayername);

        tooltip.append('text')
            .attr('x', padding)
            .attr('y', padding + 35)
            .text(`${txt}: ${roundedXValue}`);

        tooltip.append('text')
            .attr('x', padding)
            .attr('y', padding + 55)
            .text(`Kill Participation: ${roundedYValue}`);

        // Position tooltip relative to the mouse cursor
        const mouseX = d.screenX;
        const mouseY = d.screenY;
        tooltip.attr('transform', `translate(${mouseX/2 - tooltipWidth-100},${mouseY/2 - tooltipHeight - padding-100})`);
    }

    function handleMouseOut() {
        svg.select('#tooltip').remove();
    }}


    
    function updateScatter(position){
        if (!svgScatter) return; // If SVG element does not exist, return

        renderScatterPlot(position)
    }

</script>

<div id="scatter-plot"></div>

<select on:change="{e => updateScatter(e.target.value)}">
    {#each positions as position}
        <option value="{position}">{position}</option>
    {/each}
</select>


<style>
    .tooltip {
        position: absolute;
        background-color: white;
        border: 1px solid #ddd;
        padding: 5px;
    }
</style>