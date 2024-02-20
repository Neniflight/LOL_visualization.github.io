<script>
	import * as d3 from 'd3';
    import { onMount } from 'svelte';
    import { scaleLinear, scaleBand } from 'd3-scale';

    export let player_data = []; // Define player_data as a prop
    export let team_data = []; // Define team_data as a prop


    let selectedPatch = 13.05; // Initial patch
    let selectedRegion = 'LCK'; // Initial region

    let svg;
    let xScale;
    let yScale;
    let sortedChampions;
    let graphInitialized = false;
    let availablePatches = []; // Array to store available patches based on the selected league
    let groupedData;

    let svg2;
    let xScale2;
    let yScale2;
    let sortedWorstChampions;

    let svgBanned;
    let xScaleBanned;
    let yScaleBanned;
    let sortedBannedChampions;

    onMount(async () => {

        // Check if team_data is populated
       // Introduce a short delay (e.g., 100 milliseconds)
        await new Promise(resolve => setTimeout(resolve, 200));
        console.log('what')
        // Call your function to group data by region, patch, and champion
        console.log(player_data)
        groupedData = groupDataByRegionPatchAndChampion(player_data);
        console.log(groupedData)

        // Set available patches for the initial region
        updateAvailablePatches(groupedData);        // Update the patch slider

        console.log(availablePatches)

        console.log("slider updated, creating graph")
        // Create initial graph
        createGraph(groupedData.get(selectedRegion).get(selectedPatch));
        createWorstGraph(groupedData.get(selectedRegion).get(selectedPatch));
        createBannedChampionsGraph(team_data)
    });

    function groupDataByRegionPatchAndChampion(data) {
        // Group data using d3
        return d3.group(data, d => d.league, d => d.patch, d => d.champion);
    }

    function createGraph(data) {
        console.log(data)
        let flattenedData = Array.from(data, ([champion, results]) => [champion, results.length >= 5 ? d3.mean(results, d => d.result) : 0]);

        // Sort champions by win rate
        sortedChampions = flattenedData.sort((a, b) => b[1] - a[1]).slice(0, 5);
                // Create scales
                xScale = d3.scaleLinear()
            .domain([0, d3.max(sortedChampions, d => d[1])])
            .range([0, 500]); // Adjust width as needed

        // Create scales
        xScale = d3.scaleLinear()
            .domain([0, 1]) // Adjusted to represent percentage
            .range([0, 300]); // Adjust width as needed

        yScale = d3.scaleBand()
            .domain(sortedChampions.map(d => d[0]))
            .range([0, 120]) // Adjust height as needed
            .padding(0.1);

        // Create SVG
        svg = d3.select('#graph')
            .append('svg')
            .attr('width', 500)
            .attr('height', 200); // Adjust dimensions as needed

        // Create y-axis
        svg.append("g")
            .attr("transform", "translate(100,0)") // Adjust x-position for the y-axis
            .call(d3.axisLeft(yScale));

        // Create x-axis
        svg.append("g")
            .attr("transform", "translate(100,120)") // Adjust y-position for the x-axis
            .call(d3.axisBottom(xScale).ticks(5, "%")); // Display ticks in percentage form

        // Add champion images
        svg.selectAll('.champion-image')
            .data(sortedChampions)
            .enter()
            .append('image')
            .attr("class", "champion-image")
            .attr("xlink:href", d => `${d[0]}Square.png`)
            .attr("x", 35) // Adjust x-position to display images on the left side of the graph
            .attr("y", d => yScale(d[0])) // Adjust y-position to center images with the corresponding y-axis label
            .attr("width", 20) // Adjust image size as needed
            .attr("height", 20); // Adjust image size as needed
        
        // Create bars
        svg.selectAll('rect')
            .data(sortedChampions)
            .enter()
            .append('rect')
            .attr('x', 100) // Adjust x-position to start bars from the left
            .attr('y', d => yScale(d[0]))
            .attr('width', d => xScale(d[1]))
            .attr('height', yScale.bandwidth())
            .attr('fill', 'steelblue'); // Adjust color as needed

        // Add win rate labels
        svg.selectAll('.win-rate')
            .data(sortedChampions)
            .enter()
            .append('text')
            .attr('class', 'win-rate')
            .text(d => `${(d[1] * 100).toFixed(2)}%`) // Display win rate in percentage
            .attr('x', d => {
                let width = xScale(d[1]);
                if (width < 60) return 100 + width + 5; // Adjust position for narrow bars
                else return 100 + width - 5; // Align text to the end of the bars
            })            .attr('y', d => yScale(d[0]) + yScale.bandwidth() / 2)
            .attr('dy', '0.35em')
            .attr('dx', 5)
            .attr('fill', 'black'); // Adjust color as needed


            
       
        console.log('graph created')
        graphInitialized = true
    }

        function updateGraph() {
            if (!graphInitialized) return; // Check if the graph is initialized
            console.log('updating')
            console.log(selectedPatch)

            // Get the new patch selected from the slider
            let selectedPatchData = groupDataByRegionPatchAndChampion(player_data).get(selectedRegion).get(selectedPatch);

            // Update sorted champions based on the selected patch
            let flattenedData = Array.from(selectedPatchData, ([champion, results]) => [champion, results.length >= 5 ? d3.mean(results, d => d.result) : 0]);

            // Filter champions with fewer than 5 games
            flattenedData = flattenedData.filter(([champion, meanResult]) => meanResult !== 0);

            // Sort champions by win rate
            sortedChampions = flattenedData.sort((a, b) => b[1] - a[1]).slice(0, 5);

            // Sort champions by win rate (ascending for worst)
            sortedWorstChampions = flattenedData.sort((a, b) => a[1] - b[1]).slice(0, 5);

            console.log(sortedChampions)
            
            // Update y-axis scale with new domain
            yScale.domain(sortedChampions.map(d => d[0]));

            // Remove existing images
            svg.selectAll(".champion-image").remove();

            // Add new images for each champion
            sortedChampions.slice(0, 5).forEach((champion, index) => {
                svg.append("image")
                    .attr("class", "champion-image")
                    .attr("xlink:href", `${champion[0]}Square.png`)
                    .attr("x", 30) // Adjust x-position to display images on the left side of the graph
                    .attr("y", yScale(champion[0])) // Adjust y-position to center images with the corresponding y-axis label
                    .attr("width", 20) // Adjust image size as needed
                    .attr("height", 20); // Adjust image size as needed
            });

            // Update y-axis
            svg.select("g")
                .call(d3.axisLeft(yScale));

            // Redraw graph when patch or region changes
            svg.selectAll('rect')
                .data(sortedChampions)
                .transition()
                .duration(500)
                .attr('width', d => xScale(d[1]));

            // Redraw win rate labels
            svg.selectAll('.win-rate')
                .data(sortedChampions)
                .text(d => `${(d[1] * 100).toFixed(2)}%`) // Display win rate in percentage
                .attr('x', d => {
                    let width = xScale(d[1]);
                    if (width < 60) return 100 + width + 5; // Adjust position for narrow bars
                    else return 100 + width - 5; // Align text to the end of the bars
                });
            

            // Update y-axis scale with new domain
            yScale2.domain(sortedWorstChampions.map(d => d[0]));

            // Update y-axis
            svg2.select("g")
                .call(d3.axisLeft(yScale2));

            
            // Remove existing images
            svg2.selectAll(".champion-image").remove();

            // Add new images for each champion
            sortedWorstChampions.slice(0, 5).forEach((champion, index) => {
                svg2.append("image")
                    .attr("class", "champion-image")
                    .attr("xlink:href", `${champion[0]}Square.png`)
                    .attr("x", 30) // Adjust x-position to display images on the left side of the graph
                    .attr("y", yScale2(champion[0])) // Adjust y-position to center images with the corresponding y-axis label
                    .attr("width", 20) // Adjust image size as needed
                    .attr("height", 20); // Adjust image size as needed
            });


            // Create bars
            svg2.selectAll('rect')
                .data(sortedWorstChampions)
                .transition()
                .duration(500)
                .attr('width', d => xScale(d[1]));

            // Add win rate labels
            svg2.selectAll('.win-rate')
                .data(sortedWorstChampions)
                .text(d => `${(d[1] * 100).toFixed(2)}%`) // Display win rate in percentage
                .attr('x', d => {
                    let width = xScale(d[1]);
                    if (width < 60) return 100 + width + 5; // Adjust position for narrow bars
                    else return 100 + width; // Align text to the end of the bars
                });


                
                // Update the banned champions graph
                // Recalculate and update the most banned champions
            let regionData = team_data.filter(d => d.league === selectedRegion);
            let patchData = regionData.filter(d => d.patch === selectedPatch);
            let bannedChampionsCount = {};
            patchData.forEach(d => {
                for (let i = 1; i <= 5; i++) {
                    let ban = d[`ban${i}`];
                    if (bannedChampionsCount[ban]) {
                        bannedChampionsCount[ban]++;
                    } else {
                        bannedChampionsCount[ban] = 1;
                    }
                }
            });
            let bannedChampionsArray = Object.entries(bannedChampionsCount);
            bannedChampionsArray.sort((a, b) => b[1] - a[1]);
            sortedBannedChampions = bannedChampionsArray.slice(0, 5);

            // Update the most banned champions graph

            // Update y-axis scale with new domain
            yScaleBanned.domain(sortedBannedChampions.map(d => d[0]));

            // Update y-axis
            svgBanned.select("g")
                .call(d3.axisLeft(yScaleBanned));
            // Remove existing images
            svgBanned.selectAll(".champion-image").remove();
            // Remove existing bars and labels
            //svgBanned.selectAll('rect').remove();
            //svgBanned.selectAll('.banned-champion-count').remove();


            // Add new images for each champion
            sortedBannedChampions.slice(0, 5).forEach((champion, index) => {
                svgBanned.append("image")
                    .attr("class", "champion-image")
                    .attr("xlink:href", `${champion[0]}Square.png`)
                    .attr("x", 30) // Adjust x-position to display images on the left side of the graph
                    .attr("y", yScaleBanned(champion[0])) // Adjust y-position to center images with the corresponding y-axis label
                    .attr("width", 20) // Adjust image size as needed
                    .attr("height", 20); // Adjust image size as needed
            });



            console.log(sortedBannedChampions)
            
            // Create bars
            svgBanned.selectAll('rect')
                .data(sortedBannedChampions)
                .transition()
                .duration(500)
                .attr('width', d => xScaleBanned(d[1]));

            // Add labels for banned champions
            // Redraw win rate labels
            svgBanned.selectAll('.banned-champion-count')
                .data(sortedBannedChampions)
                .text(d => d[1]) // Display win rate in percentage
                .attr('x', d => {
                    let width = xScaleBanned(d[1]);
                    if (width < 60) return 100 + width + 10; // Adjust position for narrow bars
                    else return 100 + width; // Align text to the end of the bars
                });
        }

    function updateAvailablePatches() {
        // Get available patches for the selected league
        let leagueData = groupedData.get(selectedRegion);
        availablePatches = [...leagueData.keys()];
        if (!availablePatches.includes(selectedPatch)) {
            selectedPatch = 13.01;
        }
        console.log(availablePatches)
        updateGraph();
    }

    function createWorstGraph(data) {
        let flattenedData = Array.from(data, ([champion, results]) => [champion, results.length >= 5 ? d3.mean(results, d => d.result) : 1]);

        // Filter champions with fewer than 5 games
        flattenedData = flattenedData.filter(([champion, meanResult]) => meanResult !== 0);

        // Sort champions by win rate (ascending for worst)
        sortedWorstChampions = flattenedData.sort((a, b) => a[1] - b[1]).slice(0, 5);

        // Create scales
        xScale2 = d3.scaleLinear()
            .domain([0, 1]) // Adjusted to represent percentage
            .range([0, 300]); // Adjust width as needed

        yScale2 = d3.scaleBand()
            .domain(sortedWorstChampions.map(d => d[0]))
            .range([0, 120]) // Adjust height as needed
            .padding(0.1);


        // Create SVG
        svg2 = d3.select('#worst-graph')
            .append('svg')
            .attr('width', 500)
            .attr('height', 200); // Adjust dimensions as needed

        // Create y-axis
        svg2.append("g")
            .attr("transform", "translate(100,0)") // Adjust x-position for the y-axis
            .call(d3.axisLeft(yScale2));

        // Create x-axis
        svg2.append("g")
            .attr("transform", "translate(100,120)") // Adjust y-position for the x-axis
            .call(d3.axisBottom(xScale2).ticks(5, "%")); // Display ticks in percentage format


        // Add champion images
        svg2.selectAll('.champion-image')
            .data(sortedWorstChampions)
            .enter()
            .append('image')
            .attr("class", "champion-image")
            .attr("xlink:href", d => `${d[0]}Square.png`)
            .attr("x", 35) // Adjust x-position to display images on the left side of the graph
            .attr("y", d => yScale2(d[0])) // Adjust y-position to center images with the corresponding y-axis label
            .attr("width", 20) // Adjust image size as needed
            .attr("height", 20); // Adjust image size as needed

        // Create bars
        svg2.selectAll('rect')
            .data(sortedWorstChampions)
            .enter()
            .append('rect')
            .attr('x', 100) // Adjust x-position to start bars from the left
            .attr('y', d => yScale2(d[0]))
            .attr('width', d => xScale2(d[1]))
            .attr('height', yScale2.bandwidth())
            .attr('fill', 'steelblue'); // Adjust color as needed



        // Add win rate labels
        svg2.selectAll('.win-rate')
            .data(sortedWorstChampions)
            .enter()
            .append('text')
            .attr('class', 'win-rate')
            .text(d => `${(d[1] * 100).toFixed(2)}%`) // Display win rate in percentage
            .attr('x', d => {
                let width = xScale2(d[1]);
                if (width < 60) return 100 + width + 5; // Adjust position for narrow bars
                else return 100 + width + 3; // Align text to the end of the bars
            })
            .attr('y', d => yScale2(d[0]) + yScale2.bandwidth() / 2)
            .attr('dy', '0.35em')
            .attr('fill', 'black'); // Adjust color as needed
    }
    function createBannedChampionsGraph(data) {
        // Filter data for the selected region
        let regionData = data.filter(d => d.league === selectedRegion);

        // Filter data for the selected patch
        let patchData = regionData.filter(d => d.patch === selectedPatch);

        // Create an object to count bans for each champion
        let bannedChampionsCount = {};
        patchData.forEach(d => {
            for (let i = 1; i <= 5; i++) {
                let ban = d[`ban${i}`];
                if (bannedChampionsCount[ban]) {
                    bannedChampionsCount[ban]++;
                } else {
                    bannedChampionsCount[ban] = 1;
                }
            }
        });

        // Convert the object into an array of [champion, count] pairs
        let bannedChampionsArray = Object.entries(bannedChampionsCount);

        // Sort the array by count in descending order
        bannedChampionsArray.sort((a, b) => b[1] - a[1]);

        // Take the top 5 most banned champions
        sortedBannedChampions = bannedChampionsArray.slice(0, 5);
            xScaleBanned = d3.scaleLinear()
                .domain([0, d3.max(bannedChampionsArray, d => d[1])])
                .range([0, 300]);

        yScaleBanned = d3.scaleBand()
            .domain(sortedBannedChampions.map(d => d[0]))
            .range([0, 120])
            .padding(0.1);

        svgBanned = d3.select('#banned-champions-graph')
            .append('svg')
            .attr('width', 500)
            .attr('height', 200);

        svgBanned.append("g")
            .attr("transform", "translate(100,0)")
            .call(d3.axisLeft(yScaleBanned));

        svgBanned.append("g")
            .attr("transform", "translate(100,120)")
            .call(d3.axisBottom(xScaleBanned).ticks(5));

        svgBanned.selectAll('rect')
            .data(sortedBannedChampions)
            .enter()
            .append('rect')
            .attr('x', 100)
            .attr('y', d => yScaleBanned(d[0]))
            .attr('width', d => xScaleBanned(d[1]))
            .attr('height', yScaleBanned.bandwidth())
            .attr('fill', 'steelblue');

        svgBanned.selectAll('.banned-champion-count')
            .data(sortedBannedChampions)
            .enter()
            .append('text')
            .attr('class', 'banned-champion-count')
            .text(d => d[1])
            .attr('x', d => 100 + xScaleBanned(d[1]) + 5)
            .attr('y', d => yScaleBanned(d[0]) + yScaleBanned.bandwidth() / 2)
            .attr('dy', '0.35em')
            .attr('fill', 'black');

        console.log(sortedBannedChampions)
        svgBanned.selectAll('.champion-image')
            .data(sortedBannedChampions)
            .enter()
            .append('image')
            .attr('class', 'champion-image')
            .attr('xlink:href', d => `${d[0]}Square.png`)
            .attr('x', 35)
            .attr('y', d => yScaleBanned(d[0]))
            .attr('width', 20)
            .attr('height', 20);
    }


</script>
<h1>Best Win Rate Champions Per Region Per Patch</h1>
<div id="graph"></div>
<h1>Worst Win Rate Champions Per Region Per Patch</h1>
<div id="worst-graph"></div>
<h1>Most Banned Champions Per Region Per Patch</h1>
<div id="banned-champions-graph"></div>

<!-- Selector to choose patch -->
<select bind:value={selectedPatch} on:change={updateGraph}>
    {#each availablePatches as patch}
        <option value={patch}>{patch.toFixed(2)}</option>
    {/each}
</select>

<!-- Control to select region -->
<select bind:value={selectedRegion} on:change={updateAvailablePatches}>
  <option value="LCK">LCK</option>
  <option value="LEC">LEC</option>
  <option value="LPL">LPL</option>
  <option value="LCS">LCS</option>
</select>