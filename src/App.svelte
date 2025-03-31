<script>
  import Navbar from "./navbar/Navbar.svelte";
  import WeatherForecast from "./WeatherForecast.svelte";
  import Footer from "./footer/Footer.svelte";

  let city = 'Heilbronn';
  let days = 5;
  let weatherData = [];
  let loading = false;
  let error = null;
  let lastFetchTime = null;

  const API_BASE_URL = "http://localhost:8080";

  // Testdaten für Fallback
  const mockWeatherData = [
    {
      city: "Heilbronn",
      forecastData: new Date().toISOString(),
      temperature: 22.5,
      minTemparature: 18.2,
      maxTemparature: 25.7,
      humidity: 65,
      description: "leicht bewölkt",
      iconCode: "02d",
      rain: 10
    },
    {
      city: "Heilbronn",
      forecastData: new Date(Date.now() + 86400000).toISOString(),
      temperature: 28.3,
      minTemparature: 22.5,
      maxTemparature: 31.1,
      humidity: 40,
      description: "sonnig",
      iconCode: "01d",
      rain: 0
    },
    {
      city: "Heilbronn",
      forecastData: new Date(Date.now() + 172800000).toISOString(),
      temperature: 21.8,
      minTemparature: 17.9,
      maxTemparature: 24.5,
      humidity: 85,
      description: "regnerisch",
      iconCode: "10d",
      rain: 75
    },
    {
      city: "Heilbronn",
      forecastData: new Date(Date.now() + 259200000).toISOString(),
      temperature: 18.2,
      minTemparature: 14.5,
      maxTemparature: 20.8,
      humidity: 90,
      description: "gewitter",
      iconCode: "11d",
      rain: 90
    },
    {
      city: "Heilbronn",
      forecastData: new Date(Date.now() + 345600000).toISOString(),
      temperature: 15.1,
      minTemparature: 12.7,
      maxTemparature: 17.2,
      humidity: 70,
      description: "neblig",
      iconCode: "50d",
      rain: 30
    }
  ];

  async function fetchWeather() {
    if (!city) return;
    loading = true;
    error = null;

    try {
      const url = `${API_BASE_URL}/api/weather/forecast?city=${encodeURIComponent(city)}&days=${days}`;
      const response = await fetch(url);

      if (!response.ok) {
        const errorData = await response.json();
        throw new Error(errorData.error || `HTTP-Fehler: ${response.status}`);
      }

      const data = await response.json();
      console.log("Rohe API-Antwort:", data);

      // Daten verarbeiten
      if (Array.isArray(data)) {
        weatherData = data;
      } else {
        weatherData = [data];
      }

      weatherData = weatherData.map(item => ({
        city: item.city || city,
        forecastData: item.forecastData || new Date().toISOString(),
        temperature: item.temperature || 0,
        minTemparature: item.minTemparature || 0,
        maxTemparature: item.maxTemparature || 0,
        humidity: item.humidity || 0,
        description: item.description || '',
        iconCode: item.iconCode || '01d',
        rain: item.rain || 0
      }));

      lastFetchTime = new Date();
      console.log("Verarbeitete Wetterdaten:", weatherData);
    } catch (err) {
      console.error('Fehler beim Laden der Wetterdaten:', err);
      error = err.message;
      weatherData = mockWeatherData.slice(0, days);
    } finally {
      loading = false;
    }
  }

  function handleSubmit() {
    fetchWeather();
  }
</script>

<svelte:head>
  <style>
    :root {
      --card-background: #ffffff;
      --text-color: #333333;
      --border-color: #e0e0e0;
      --main-background: #f5f7fa;
      --shadow-color: rgba(0, 0, 0, 0.05);
    }
  </style>
</svelte:head>

<Navbar />

<main>
  <div class="header-section">
    <h1>Wetter Informations-News</h1>
  </div>

  <div class="search-container">
    <form on:submit|preventDefault={handleSubmit} class="search-form">
      <div class="city-input">
        <input
                type="text"
                bind:value={city}
                placeholder="Stadt eingeben..."
                required
        />
      </div>

      <div class="days-slider">
        <label for="days">Tage: {days}</label>
        <input
                type="range"
                id="days"
                bind:value={days}
                min="1"
                max="5"
                step="1"
        />
      </div>

      <button type="submit" disabled={loading}>
        {loading ? 'Wird geladen...' : 'Aktualisieren'}
      </button>
    </form>
  </div>

  {#if loading}
    <div class="loading">
      <p>Wetterdaten werden geladen...</p>
    </div>
  {:else if error}
    <div class="error">
      <p>Fehler beim Laden der Wetterdaten: {error}</p>
      {#if weatherData.length > 0}
        <p class="using-cached">Zeige Fallback-Daten an.</p>
      {/if}
    </div>
  {:else if weatherData.length > 0}
    <WeatherForecast weatherData={weatherData.slice(0, days)} city={city} useSymbols={true} />

    {#if lastFetchTime}
      <div class="cache-info">
        <p>Daten aktualisiert: {lastFetchTime.toLocaleString()}</p>
      </div>
    {/if}
  {:else}
    <div class="initial-state">
      <p>Bitte klicken Sie auf "Aktualisieren", um Wetterdaten zu laden.</p>
    </div>
  {/if}
</main>

<Footer />

<style>
  :global(body) {
    background-color: var(--main-background);
    margin: 0;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  }

  main {
    max-width: 1200px;
    margin: 0 auto;
    padding: 2rem 1rem;
  }

  .header-section {
    text-align: center;
    margin-bottom: 2rem;
  }

  h1 {
    font-size: 2.5rem;
    color: var(--text-color);
    margin: 0;
  }

  .search-container {
    background-color: var(--card-background);
    border-radius: 12px;
    padding: 1.5rem;
    margin-bottom: 2rem;
    box-shadow: 0 4px 12px var(--shadow-color);
  }

  .search-form {
    display: flex;
    align-items: center;
    flex-wrap: wrap;
    gap: 1.5rem;
  }

  .city-input {
    flex: 2;
    min-width: 200px;
  }

  .days-slider {
    flex: 1;
    min-width: 150px;
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }

  input[type="text"] {
    width: 100%;
    padding: 0.75rem;
    border: 1px solid var(--border-color);
    border-radius: 6px;
    background-color: var(--card-background);
    color: var(--text-color);
    font-size: 1rem;
  }

  input[type="range"] {
    width: 100%;
    accent-color: #4299e1;
  }

  button {
    padding: 0.75rem 1.5rem;
    background: linear-gradient(135deg, #4299e1, #63b3ed);
    color: white;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    transition: opacity 0.2s;
    font-size: 1rem;
  }

  button:hover {
    opacity: 0.9;
  }

  button:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }

  .loading, .error, .initial-state {
    text-align: center;
    padding: 2rem;
    background-color: var(--card-background);
    border-radius: 12px;
    box-shadow: 0 4px 12px var(--shadow-color);
  }

  .error {
    color: #e53e3e;
  }

  .initial-state {
    color: #4a5568;
  }

  .cache-info {
    text-align: right;
    font-size: 0.8rem;
    color: #666;
    margin-top: 1rem;
  }

  .using-cached {
    font-style: italic;
    color: #666;
    margin-top: 0.5rem;
  }

  @media (max-width: 768px) {
    .search-form {
      flex-direction: column;
      align-items: stretch;
    }

    .city-input, .days-slider {
      width: 100%;
    }
  }
</style>
