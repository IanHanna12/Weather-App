<script>
    import Navbar from "./navbar/Navbar.svelte";
    import WeatherForecast from "./WeatherForecast.svelte";
    import Footer from "./footer/Footer.svelte";
    import { onMount } from 'svelte';

    let city = 'Heilbronn';
    let days = 5;
    let lang = 'de';
    let weatherData = [];
    let loading = false;
    let error = null;
    let isRecording = false;
    let isConnected = true;
    let triggerWord = "Wetter";
    let isListening = false;
    let recognition;

    const API_BASE_URL = "http://localhost:8080";

    function setupSpeechRecognition() {
        if (typeof window === 'undefined') return;

        try {
            // @ts-ignore
            const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
            if (!SpeechRecognition) {
                console.log("Speech Recognition not supported");
                return;
            }

            recognition = new SpeechRecognition();
            recognition.lang = lang === 'de' ? 'de-DE' : 'en-US';
            recognition.continuous = false;

            recognition.onstart = () => isRecording = true;
            recognition.onend = () => {
                isRecording = false;
                if (isListening) {
                    setTimeout(() => recognition.start(), 300);
                }
            };

            recognition.onresult = (event) => {
                const transcript = event.results[0][0].transcript.toLowerCase();
                console.log("Heard:", transcript);

                if (!transcript.includes(triggerWord.toLowerCase())) return;

                // Extract command after trigger word
                const parts = transcript.split(triggerWord.toLowerCase());
                if (parts.length < 2) return;

                const command = parts[1].trim();
                if (!command) return;

                // Parse command
                parseVoiceCommand(command);
            };

            isListening = true;
            recognition.start();
        } catch (e) {
            console.error("Speech recognition error:", e);
        }
    }

    function parseVoiceCommand(command) {
        // Extract city and days from command
        const words = command.trim().split(/\s+/);
        let newDays = 5;
        let cityWords = [];

        for (let i = 0; i < words.length; i++) {
            const num = parseInt(words[i]);
            if (!isNaN(num)) {
                newDays = Math.min(Math.max(num, 1), 5); // Limit between 1-5
            } else {
                cityWords.push(words[i]);
            }
        }

        const newCity = cityWords.join(" ").trim();
        if (newCity) {
            city = newCity;
            days = newDays;
            fetchWeather();
        }
    }

    function checkConnection() {
        isConnected = navigator.onLine;
        window.addEventListener('online', () => isConnected = true);
        window.addEventListener('offline', () => isConnected = false);
    }

    onMount(() => {
        setupSpeechRecognition();
        checkConnection();

        return () => {
            if (recognition) {
                isListening = false;
                try { recognition.stop(); } catch(e) {}
            }
        };
    });

    async function fetchWeather() {
        if (!city) return;

        loading = true;
        error = null;

        try {
            const url = `${API_BASE_URL}/api/weather/${encodeURIComponent(city)}?days=${days}&lang=${lang}`;
            const response = await fetch(url);

            if (!response.ok) {
                throw new Error(`HTTP error: ${response.status}`);
            }

            const data = await response.json();
            weatherData = processWeatherData(data);
        } catch (err) {
            console.error('Error fetching weather:', err);
            error = err.message;
            weatherData = [];
        } finally {
            loading = false;
        }
    }

    function processWeatherData(data) {
        if (!Array.isArray(data)) {
            // Handle single data point
            return [{
                city: data.city || city,
                forecastData: data.forecastDate,
                temperature: data.temperature || 0,
                minTemparature: data.minTemperature || 0,
                maxTemparature: data.maxTemperature || 0,
                humidity: data.humidity || 0,
                description: data.description || '',
                iconCode: data.iconCode || '01d',
                rain: data.rain || 0
            }];
        }

        // Group by date
        const byDate = {};
        data.forEach(item => {
            const date = item.forecastDate.slice(0, 10);
            if (!byDate[date])
                byDate[date] = [];
            byDate[date].push(item);
        });

        // Process each day
        const processed = Object.keys(byDate).map(date => {
            const items = byDate[date];
            const noon = items.find(i => i.forecastDate.includes('T12:00')) || items[0];

            // Calculate min/max temperatures
            let minTemp = Infinity;
            let maxTemp = -Infinity;

            items.forEach(item => {
                const temp = item.temperature || 0;
                if (temp < minTemp) minTemp = temp;
                if (temp > maxTemp) maxTemp = temp;
            });

            return {
                city: noon.city || city,
                forecastData: noon.forecastDate,
                temperature: noon.temperature || 0,
                minTemparature: minTemp === Infinity ? 0 : minTemp,
                maxTemparature: maxTemp === -Infinity ? 0 : maxTemp,
                humidity: noon.humidity || 0,
                description: noon.description || '',
                iconCode: noon.iconCode || '01d',
                rain: noon.rain || 0
            };
        });

        return processed.slice(0, days);
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

<Navbar bind:lang />

<main>
    <div class="header-section">
        <h1>Wetter Informations-News</h1>
        <div class="status-indicators">
            <div class="connection-status {isConnected ? 'connected' : 'disconnected'}">
                {isConnected ? '🟢 Verbunden' : '🔴 Nicht verbunden'}
            </div>
            <div class="recording-status {isRecording ? 'recording' : ''}">
                {isRecording ? '🎤 Aufnahme läuft...' : '🎤 Bereit für Sprachbefehle'}
            </div>
        </div>
    </div>

    <div class="search-container">
        <div class="city-display">
            <h2>Aktuell: {city} für {days} {days === 1 ? 'Tag' : 'Tage'}</h2>
        </div>

        <div class="voice-command-example">
            <p>Trigger-Wort: <strong>{triggerWord}</strong></p>
            <p>Beispiele:</p>
            <ul>
                <li>"<em>{triggerWord} Berlin</em>" - Wetter für Berlin (5 Tage)</li>
                <li>"<em>{triggerWord} München 3 Tage</em>" - Wetter für München für 3 Tage</li>
            </ul>
            <p class="listening-status">
                {isListening ? 'Spracherkennung ist aktiv' : 'Spracherkennung wird initialisiert...'}
            </p>
        </div>
    </div>

    {#if loading}
        <div class="loading">
            <p>Wetterdaten werden geladen...</p>
        </div>
    {:else if error}
        <div class="error">
            <p>Fehler beim Laden der Wetterdaten: {error}</p>
        </div>
    {:else if weatherData.length > 0}
        <WeatherForecast weatherData={weatherData} city={city} useSymbols={true} />
    {:else}
        <div class="initial-state">
            <p>Sprechen Sie "{triggerWord} [Stadt]" um Wetterdaten zu laden.</p>
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
        margin: 0 0 1rem 0;
    }

    h2 {
        font-size: 1.5rem;
        color: var(--text-color);
        margin: 0 0 1rem 0;
    }

    .status-indicators {
        display: flex;
        justify-content: center;
        gap: 1.5rem;
        margin-top: 0.5rem;
    }

    .connection-status, .recording-status {
        padding: 0.5rem 1rem;
        border-radius: 20px;
        font-size: 0.9rem;
        display: inline-flex;
        align-items: center;
        gap: 0.5rem;
    }

    .connected {
        background-color: rgba(72, 187, 120, 0.2);
        color: #2f855a;
    }

    .disconnected {
        background-color: rgba(245, 101, 101, 0.2);
        color: #c53030;
    }

    .recording {
        background-color: rgba(237, 100, 166, 0.2);
        color: #b83280;
        animation: pulse 1.5s infinite;
    }

    @keyframes pulse {
        0% { opacity: 1; }
        50% { opacity: 0.6; }
        100% { opacity: 1; }
    }

    .search-container {
        background-color: var(--card-background);
        border-radius: 12px;
        padding: 1.5rem;
        margin-bottom: 2rem;
        box-shadow: 0 4px 12px var(--shadow-color);
    }

    .city-display {
        margin-bottom: 1rem;
        padding-bottom: 1rem;
        border-bottom: 1px solid var(--border-color);
    }

    .voice-command-example {
        padding: 1rem;
        background-color: rgba(66, 153, 225, 0.1);
        border-radius: 8px;
        font-size: 0.9rem;
    }

    .voice-command-example p {
        margin: 0.3rem 0;
    }

    .voice-command-example strong {
        color: #4299e1;
    }

    .voice-command-example ul {
        margin: 0.5rem 0;
        padding-left: 1.5rem;
    }

    .voice-command-example li {
        margin-bottom: 0.3rem;
    }

    .listening-status {
        margin-top: 1rem;
        font-weight: 500;
        color: #4299e1;
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

    @media (max-width: 768px) {
        .status-indicators {
            flex-direction: column;
            gap: 0.5rem;
        }

        h1 { font-size: 2rem; }
        h2 { font-size: 1.3rem; }
    }
</style>
