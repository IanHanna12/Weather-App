<script>
    export let weatherData;
    export let useSymbols = false;

    function formatDate(dateString) {
        try {
            if (!dateString) return "Invalid date";

            const date = new Date(dateString);

            if (isNaN(date.getTime())) return "Invalid date";

            return date.toLocaleDateString(undefined, {
                weekday: 'long',
                year: 'numeric',
                month: 'long',
                day: 'numeric'
            });
        } catch (e) {
            console.error("Error formatting date:", e, "Date string was:", dateString);
            return "Invalid date";
        }
    }

    // Helper function to get weather icon URL
    function getWeatherIconUrl(iconCode) {
        return `https://openweathermap.org/img/wn/${iconCode}@2x.png`;
    }
</script>

<div class="weather-card">
    <div class="date">{formatDate(weatherData.forecastData)}</div>

    <div class="weather-info">
        <div class="temperature-container">
            <div class="current-temp">{Math.round(weatherData.temperature)}°C</div>
            <div class="min-max">
                <span class="min">{Math.round(weatherData.minTemparature)}°</span> /
                <span class="max">{Math.round(weatherData.maxTemparature)}°</span>
            </div>
        </div>

        <div class="weather-icon">
            <img src={getWeatherIconUrl(weatherData.iconCode)} alt={weatherData.description} />
            <div class="description">{weatherData.description}</div>
        </div>
    </div>

    <div class="details">
        <div class="detail-item">
            <span class="label">Luftfeuchtigkeit:</span>
            <span class="value">{weatherData.humidity}%</span>
        </div>

        <div class="detail-item">
            <span class="label">Regenwahrscheinlichkeit:</span>
            <span class="value">{weatherData.rain}%</span>
        </div>
    </div>
</div>

<style>
    .weather-card {
        background-color: var(--card-background);
        border-radius: 12px;
        padding: 1.5rem;
        box-shadow: 0 4px 12px var(--shadow-color);
        transition: transform 0.2s;
    }

    .weather-card:hover {
        transform: translateY(-5px);
    }

    .date {
        font-size: 1.2rem;
        font-weight: 600;
        margin-bottom: 1rem;
        color: var(--text-color);
    }

    .weather-info {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 1.5rem;
    }

    .temperature-container {
        display: flex;
        flex-direction: column;
    }

    .current-temp {
        font-size: 2.5rem;
        font-weight: 700;
        color: var(--text-color);
    }

    .min-max {
        font-size: 1rem;
        color: #666;
    }

    .min {
        color: #4299e1;
    }

    .max {
        color: #f56565;
    }

    .weather-icon {
        text-align: center;
    }

    .weather-icon img {
        width: 80px;
        height: 80px;
    }

    .description {
        font-size: 1rem;
        color: #666;
        text-transform: capitalize;
    }

    .details {
        border-top: 1px solid var(--border-color);
        padding-top: 1rem;
    }

    .detail-item {
        display: flex;
        justify-content: space-between;
        margin-bottom: 0.5rem;
    }

    .label {
        color: #666;
    }

    .value {
        font-weight: 600;
        color: var(--text-color);
    }
</style>
