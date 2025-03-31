<script>
    export let weather;
    export let index = 0;
    export let useSymbols = true;

    function formatDate(dateString) {
        const date = new Date(dateString);
        return {
            day: new Intl.DateTimeFormat('de-DE', { weekday: 'long' }).format(date),
            date: new Intl.DateTimeFormat('de-DE', { day: '2-digit', month: '2-digit' }).format(date)
        };
    }

    $: formattedDate = formatDate(weather.forecastData);
    $: rainProbability = weather.rain || 0;
</script>

<div class="weather-card">
    <div class="card-header">
        <h3>{formattedDate.day}</h3>
        <p class="date">{formattedDate.date}</p>
    </div>

    <div class="card-body">
        <img
                src={`https://openweathermap.org/img/wn/${weather.iconCode}@2x.png`}
                alt={weather.description}
                class="weather-icon"
        />

        <p class="weather-description">{weather.description}</p>

        <div class="weather-details">
            <div class="detail-row">
                <div class="detail">
                    {#if useSymbols}
                        <i class="fas fa-temperature-low icon-min-temp"></i>
                    {:else}
                        <span>Min</span>
                    {/if}
                    <span>{weather.minTemparature.toFixed(1)}°C</span>
                </div>

                <div class="detail">
                    {#if useSymbols}
                        <i class="fas fa-temperature-high icon-max-temp"></i>
                    {:else}
                        <span>Max</span>
                    {/if}
                    <span>{weather.maxTemparature.toFixed(1)}°C</span>
                </div>
            </div>

            <div class="detail-row">
                <div class="detail">
                    {#if useSymbols}
                        <i class="fas fa-tint icon-humidity"></i>
                    {:else}
                        <span>Feucht.</span>
                    {/if}
                    <span>{weather.humidity}%</span>
                </div>

                <div class="detail">
                    {#if useSymbols}
                        <i class="fas fa-cloud-rain icon-rain"></i>
                    {:else}
                        <span>Regen</span>
                    {/if}
                    <span>{rainProbability}%</span>
                </div>
            </div>
        </div>
    </div>
</div>

<svelte:head>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.4/css/all.min.css">
</svelte:head>

<style>
    .weather-card {
        background-color: white;
        border-radius: 10px;
        overflow: hidden;
        box-shadow: 0 3px 10px rgba(0, 0, 0, 0.1);
        height: 100%;
        display: flex;
        flex-direction: column;
    }

    .card-header {
        background: linear-gradient(135deg, #4299e1, #63b3ed);
        color: white;
        padding: 0.75rem;
        text-align: center;
    }

    .card-header h3 {
        margin: 0;
        font-size: 1.1rem;
        font-weight: 600;
        text-transform: capitalize;
    }

    .date {
        margin: 0.1rem 0 0;
        font-size: 0.8rem;
        opacity: 0.9;
    }

    .card-body {
        padding: 1rem;
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    .weather-icon {
        width: 80px;
        height: 80px;
    }

    .weather-description {
        margin: 0 0 1rem;
        font-size: 1rem;
        color: #333;
        text-transform: capitalize;
    }

    .weather-details {
        width: 100%;
        display: flex;
        flex-direction: column;
        gap: 0.75rem;
    }

    .detail-row {
        display: flex;
        justify-content: space-between;
        width: 100%;
        padding-top: 0.75rem;
        border-top: 1px solid #eee;
    }

    .detail {
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 0.25rem;
        font-size: 0.9rem;
        min-width: 40%;
    }

    .detail i {
        font-size: 1.1rem;
    }

    .icon-humidity { color: #FFD700; }
    .icon-rain { color: #4299e1; }
    .icon-min-temp { color: #1E90FF; }
    .icon-max-temp { color: #FF4500; }
</style>
