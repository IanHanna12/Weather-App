<script>
    import { createEventDispatcher } from 'svelte';

    export let city = '';
    export let days = 1;

    const dispatch = createEventDispatcher();

    function handleSubmit(event) {
        event.preventDefault();
        dispatch('search', { city, days });
    }
</script>

<div class="search-form-container">
    <form on:submit={handleSubmit} class="search-form">
        <div class="form-group">
            <label for="city">City</label>
            <div class="input-with-icon">
                <svg class="input-icon" xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <path d="M20 10c0 6-8 12-8 12s-8-6-8-12a8 8 0 0 1 16 0Z"/>
                    <circle cx="12" cy="10" r="3"/>
                </svg>
                <input
                        type="text"
                        id="city"
                        bind:value={city}
                        placeholder="Enter city name..."
                        required
                />
            </div>
        </div>

        <div class="form-group">
            <label for="days">Forecast Days</label>
            <select id="days" bind:value={days}>
                <option value={1}>Today</option>
                <option value={2}>2 Days</option>
                <option value={3}>3 Days</option>
                <option value={4}>4 Days</option>
                <option value={5}>5 Days</option>
            </select>
        </div>

        <button type="submit" class="search-button">
            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <circle cx="11" cy="11" r="8"/>
                <path d="m21 21-4.3-4.3"/>
            </svg>
            Get Weather
        </button>
    </form>
</div>

<style>
    .search-form-container {
        background-color: white;
        border-radius: 12px;
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
        padding: 2rem;
        margin-bottom: 2rem;
    }

    .search-form {
        display: flex;
        flex-wrap: wrap;
        gap: 1rem;
        align-items: flex-end;
    }

    .form-group {
        flex: 1;
        min-width: 200px;
    }

    label {
        display: block;
        margin-bottom: 0.5rem;
        font-weight: 500;
        color: #555;
    }

    .input-with-icon {
        position: relative;
    }

    .input-icon {
        position: absolute;
        left: 12px;
        top: 50%;
        transform: translateY(-50%);
        color: #999;
    }

    input, select {
        width: 100%;
        padding: 0.75rem;
        padding-left: 2.5rem;
        border: 1px solid #e0e0e0;
        border-radius: 6px;
        font-size: 1rem;
        transition: border-color 0.2s, box-shadow 0.2s;
    }

    select {
        padding-left: 0.75rem;
        appearance: none;
        background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='24' height='24' viewBox='0 0 24 24' fill='none' stroke='%23999' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='m6 9 6 6 6-6'/%3E%3C/svg%3E");
        background-repeat: no-repeat;
        background-position: right 0.75rem center;
        background-size: 1rem;
    }

    input:focus, select:focus {
        outline: none;
        border-color: #1976d2;
        box-shadow: 0 0 0 3px rgba(25, 118, 210, 0.2);
    }

    .search-button {
        background-color: #1976d2;
        color: white;
        border: none;
        border-radius: 6px;
        padding: 0.75rem 1.5rem;
        font-size: 1rem;
        font-weight: 500;
        cursor: pointer;
        display: flex;
        align-items: center;
        gap: 0.5rem;
        transition: background-color 0.2s;
    }

    .search-button:hover {
        background-color: #1565c0;
    }

    .search-button:active {
        background-color: #0d47a1;
    }

    @media (max-width: 768px) {
        .search-button {
            width: 100%;
            justify-content: center;
            margin-top: 0.5rem;
        }
    }
</style>
