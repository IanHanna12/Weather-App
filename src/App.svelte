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
    let lastFetchTime = null;
    let isRecording = false;
    let isConnected = true;
    let triggerWord = "Wetter";
    let isListening = false;

    const API_BASE_URL = "http://localhost:8080";


    // Speech recognition setup
    let recognition;
    let recognitionSupported = false;

    function initSpeechRecognition() {
        // Check if we're in a browser environment
        if (typeof window === 'undefined') {
            return;
        }

        // Safely check for SpeechRecognition support
        try {
            // @ts-ignore - Ignore TypeScript errors for browser API detection
            const isSpeechRecognitionSupported = 'SpeechRecognition' in window || 'webkitSpeechRecognition' in window;

            if (!isSpeechRecognitionSupported) {
                console.log("Speech Recognition API is not supported in this browser");
                return;
            }

            recognitionSupported = true;

            // @ts-ignore - Create the appropriate SpeechRecognition instance
            const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
            recognition = new SpeechRecognition();

            // Configure the recognition
            recognition.lang = lang === 'de' ? 'de-DE' : 'en-US';
            recognition.continuous = false;
            recognition.interimResults = false;

            // Set up event handlers
            recognition.onstart = () => {
                isRecording = true;
            };

            recognition.onend = () => {
                isRecording = false;

                // Restart listening if continuous listening is enabled
                if (isListening && recognitionSupported) {
                    setTimeout(() => {
                        try {
                            recognition.start();
                        } catch (e) {
                            console.error("Fehler beim Neustarten der Spracherkennung:", e);
                        }
                    }, 300);
                }
            };

            recognition.onresult = (event) => {
                const transcript = event.results[0][0].transcript.toLowerCase();
                console.log("Erkannt:", transcript);

                if (transcript.includes(triggerWord.toLowerCase())) {
                    // Extract city name after trigger word
                    const parts = transcript.split(triggerWord.toLowerCase());
                    if (parts.length > 1 && parts[1].trim()) {
                        const command = parts[1].trim();

                        // Try to extract city and days
                        let newCity = command;
                        let newDays = 5; // Default

                        // Check for number of days in the command
                        const dayPatterns = [
                            { regex: /(\d+)\s*tag(e)?/i, group: 1 },
                            { regex: /für\s*(\d+)\s*tag(e)?/i, group: 1 },
                            { regex: /nächsten\s*(\d+)\s*tag(e)?/i, group: 1 }
                        ];

                        for (const pattern of dayPatterns) {
                            const match = command.match(pattern.regex);
                            if (match && match[pattern.group]) {
                                newDays = parseInt(match[pattern.group]);
                                // Remove the days part from the city string
                                newCity = command.replace(pattern.regex, '').trim();
                                break;
                            }
                        }

                        // Limit days to 1-5
                        newDays = Math.min(Math.max(newDays, 1), 5);

                        // Update state and fetch weather
                        city = newCity;
                        days = newDays;
                        fetchWeather();
                    }
                }
            };

            recognition.onerror = (event) => {
                console.error("Spracherkennung Fehler:", event.error);
                isRecording = false;
            };

            // Start listening automatically
            startContinuousListening();

        } catch (e) {
            console.error("Fehler bei der Initialisierung der Spracherkennung:", e);
            recognitionSupported = false;
        }
    }

    function startContinuousListening() {
        if (recognition && recognitionSupported) {
            try {
                isListening = true;
                recognition.start();
            } catch (e) {
                console.error("Fehler beim Starten der Spracherkennung:", e);
            }
        }
    }

    function stopContinuousListening() {
        if (recognition && recognitionSupported) {
            try {
                isListening = false;
                recognition.stop();
            } catch (e) {
                console.error("Fehler beim Stoppen der Spracherkennung:", e);
            }
        }
    }

    function checkConnection() {
        isConnected = navigator.onLine;
        window.addEventListener('online', () => isConnected = true);
        window.addEventListener('offline', () => isConnected = false);
    }

    // Initialize speech recognition and connection check when component mounts
    onMount(() => {
        initSpeechRecognition();
        checkConnection();

        // Clean up on component unmount
        return () => {
            stopContinuousListening();
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
                const errorData = await response.json().catch(() => ({}));
                throw new Error(errorData.error || `HTTP-Fehler: ${response.status}`);
            }

            const data = await response.json();
            console.log("Rohe API-Antwort:", data);

            // Process and group the data by day
            if (Array.isArray(data)) {
                // Group forecasts by day
                const forecastsByDay = {};
                const currentDateTime = new Date();

                data.forEach(item => {
                    // Extract just the date part (YYYY-MM-DD)
                    const dateKey = item.forecastDate.split('T')[0];

                    if (!forecastsByDay[dateKey]) {
                        forecastsByDay[dateKey] = [];
                    }

                    forecastsByDay[dateKey].push({
                        city: item.city || city,
                        forecastData: item.forecastDate,
                        forecastTime: new Date(item.forecastDate),
                        temperature: item.temperature || 0,
                        minTemparature: item.minTemperature || 0,
                        maxTemparature: item.maxTemperature || 0,
                        humidity: item.humidity || 0,
                        description: item.description || '',
                        iconCode: item.iconCode || '01d',
                        rain: item.rain || 0
                    });
                });

                // Create one entry per day with min/max values
                weatherData = Object.keys(forecastsByDay).map(dateKey => {
                    const forecastsForDay = forecastsByDay[dateKey];

                    // Sort forecasts by time
                    forecastsForDay.sort((a, b) => a.forecastTime.getTime() - b.forecastTime.getTime());

                    // Choose the representative forecast for this day
                    let dailyRepresentativeForecast;

                    // For today, find forecast closest to current time
                    if (dateKey === currentDateTime.toISOString().split('T')[0]) {
                        const currentTime = currentDateTime.getTime();
                        dailyRepresentativeForecast = forecastsForDay.reduce((closestForecast, currentForecast) => {
                            const currentTimeDiff = Math.abs(currentForecast.forecastTime.getTime() - currentTime);
                            const closestTimeDiff = Math.abs(closestForecast.forecastTime.getTime() - currentTime);
                            return currentTimeDiff < closestTimeDiff ? currentForecast : closestForecast;
                        }, forecastsForDay[0]);
                    } else {
                        // For future days, prefer noon forecast or use middle of day
                        dailyRepresentativeForecast = forecastsForDay.find(forecast =>
                            forecast.forecastData.includes('T12:00')
                        ) || forecastsForDay[Math.floor(forecastsForDay.length / 2)];
                    }

                    // Get min/max temperatures for the day
                    const dailyTemperatures = forecastsForDay.map(forecast => forecast.temperature);

                    return {
                        city: dailyRepresentativeForecast.city,
                        forecastData: dailyRepresentativeForecast.forecastData,
                        temperature: dailyRepresentativeForecast.temperature,
                        minTemparature: Math.min(...dailyTemperatures),
                        maxTemparature: Math.max(...dailyTemperatures),
                        humidity: dailyRepresentativeForecast.humidity,
                        description: dailyRepresentativeForecast.description,
                        iconCode: dailyRepresentativeForecast.iconCode,
                        rain: dailyRepresentativeForecast.rain
                    };
                }).slice(0, days);
            } else {
                // Handle single forecast data
                weatherData = [{
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

            lastFetchTime = new Date();
            console.log("Verarbeitete Wetterdaten:", weatherData);
        } catch (err) {
            console.error('Fehler beim Laden der Wetterdaten:', err);
            error = err.message;
            weatherData = [];
        } finally {
            loading = false;
        }
    }

    function handleSearch(event) {
        const { city: newCity, days: newDays } = event.detail;
        city = newCity;
        days = newDays;
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

<Navbar bind:city bind:days bind:lang on:search={handleSearch} />

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
                <li>"<em>{triggerWord} Hamburg für 2 Tage</em>" - Wetter für Hamburg für 2 Tage</li>
            </ul>
            <p class="listening-status">
                {isListening ? 'Spracherkennung ist aktiv - Sprechen Sie jetzt' : 'Spracherkennung wird initialisiert...'}
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
        margin: 0;
        margin-bottom: 1rem;
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
        0% {
            opacity: 1;
        }
        50% {
            opacity: 0.6;
        }
        100% {
            opacity: 1;
        }
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
        .status-indicators {
            flex-direction: column;
            gap: 0.5rem;
        }

        h1 {
            font-size: 2rem;
        }

        h2 {
            font-size: 1.3rem;
        }
    }
</style>
