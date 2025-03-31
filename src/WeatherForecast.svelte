<script>
    import WeatherCard from "./weathercard/WeatherCard.svelte";

    export let weatherData = [];
    export let city = '';
    export let useSymbols = true; // Standardmäßig Symbole verwenden
</script>

{#if weatherData.length > 0}
    <div class="city-header">
        <h2>{city}</h2>
    </div>

    <div class="weather-grid">
        {#each weatherData as weather, index}
            <WeatherCard {weather} {index} {useSymbols} />
        {/each}
    </div>
{:else}
    <div class="nos-data">
        <p>Keine Wetterdaten verfügbar. Bitte suchen Sie nach einer Stadt.</p>
    </div>
{/if}

<style>
    .city-header {
        text-align: center;
        margin-bottom: 2rem;
        background-color: var(--card-background);
        border-radius: 12px;
        padding: 1rem;
        box-shadow: 0 4px 12px var(--shadow-color);
    }

    .city-header h2 {
        font-size: 2rem;
        color: var(--text-color);
        margin: 0;
    }

    .weather-grid {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
        gap: 1.5rem;
        margin-bottom: 2rem;
    }

    @media (max-width: 768px) {
        .weather-grid {
            grid-template-columns: 1fr;
        }
    }
</style>
