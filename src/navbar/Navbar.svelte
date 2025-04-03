<script>
    export let city = 'Heilbronn';
    export let days = 5;
    export let lang = 'de';
</script>

<nav class="navbar">
    <div class="container">
        <div class="logo">
            <a href="/">Weather App</a>
        </div>
    </div>
</nav>

<style>
    .navbar {
        background-color: #4299e1;
        color: white;
        padding: 1rem 0;
        box-shadow: 0 2px 5px rgba(0, 0, 0, 0.05);
    }

    .container {
        max-width: 1200px;
        margin: 0 auto;
        padding: 0 1rem;
        display: flex;
        justify-content: space-between;
        align-items: center;
    }

    .logo a {
        color: white;
        font-size: 1.5rem;
        font-weight: bold;
        text-decoration: none;
    }
</style>
