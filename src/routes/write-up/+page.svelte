<body>
    <div class="button-container">
        <a href="/">
            <button>Back to Region</button>
        </a>
    </div>
    <div class="container">
        <div class="text-container">
            <span class="header"><h1>Write Up</h1></span>
            <h2>Overview</h2>
            <p> We created this visualization to answer the question of which champions and players are doing well based on the region, patch, and role. This allows us to see the current meta of League of Legends and will allow Riot to help improve champions that are currently struggling. 
                We chose a world map as your homepage for the initial visualization because it allows the reader to get a geographical sense of where these regions are. Additionally, selecting options from a map is more intuitive for people who are familiar with maps and for those who are not, we have pins to indicate what options are available. We ended up with a bar mark positioned on a common scale. We display the champions with the highest win rate, the lowest win rates, and the highest ban rate. Additionally, these stats can be filtered by the appropriate region and patch. This allows a user to see which champions are meta based on region and patch and which region needs buffs/reworks. Additionally, we decided to have a scatter mark with rectangle marks when you hover over specific dots (scatterplot). This scatter plot shows important stats related to specific roles. For instance, a top player would value stats like cspm (creep score per minute) and kill participation, while a jungler values monster kills. Dots/players toward the top right are better compared to players in the bottom left. We did initially consider other stats, but due to missing data from leagues like China, we decided not to proceed. Additionally, we decided to show all four charts at once because it lets you analyze multiple stats and players at once from one patch. Separating these charts would not allow users to see immediately which champions are meta and bad. 
                Alternatively, we considered a heat map using the Valorant API to see what areas that players control throughout the map at different points of time. However, this was impossible because the Valorant API is locked behind a paywall and there are no datasets available to us.</p>
            <h2>Development Process</h2>
            <p>We had 2 people developing this process. Together, we roughly spent 15 hours together working on this project where each of us roughly spent the same amount of time. Calvin worked on routing, the world map with home page, styling components, and creating the clean dataset. Kevin worked on the visualizations for the bar and scatter plots and the logic behind the code. What took the most time was coding out the visualizations because we needed to figure out how Svelte and D3 worked and how to create these visualizations. We found similarities between Svelte and React, but the logic of the coding is definitely different. </p>
        </div>
    </div>
</body>

<style>
    .text-container {
        display: flex;
        flex-direction: column;
        padding: 3rem;
        font-family: "Merriweather Sans", sans-serif;
        max-width: 1200px;
        width: 100%;
        color: #C8AA6E;
    }
    
    .header {
        text-align: center;
    }

    h2 {
        color: #F0E6D2;
    }

    button {
        border-radius: 3px;
        color: #F0E6D2;
        background-color: #1E282D;
        border-color: #C89B3C;
        font-size: 1.5rem;
        margin-top: 3rem;
    }

    body {
        height: 100vh;
        background-color: #091428;
    }

</style>