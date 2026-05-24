<script>
  let city = $state('');
  let weather = $state(null);
  let error = $state('');

  async function getWeather(event) {
    event.preventDefault();

    error = '';

    if (!city.trim()) {
      error = 'Please enter a city';
      return;
    }

    const res = await fetch(
      `https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=712c664b4430d30580f2d2e27501801d&units=metric`
    );

    const data = await res.json();

    if (data.cod != 200) {
      error = data.message;
      weather = null;
      return;
    }

    weather = data;
  }
</script>

<div class="app">
  <h1>Weather App</h1>

  <form class="search" onsubmit={getWeather}>
    <input
      value={city}
      oninput={(e) => city = e.target.value}
      placeholder="Enter city"
    />

    <button type="submit">Search</button>
  </form>

  <p class="error">{error}</p>

  {#if weather}
    <div class="card">
      <h2>{weather.name}</h2>
      <p class="temp">{weather.main.temp}°C</p>
      <p>{weather.weather[0].description}</p>
      <p>{weather.wind.speed} m/s</p>
    </div>
  {/if}
</div>

<!-- (MOBILE FIRST) -->
<style>
  .app {
    font-family: sans-serif;
    padding: 16px;
    max-width: 500px;
    margin: 0 auto;
  }

  h1 {
    font-size: 1.5rem;
    text-align: center;
  }

  .search {
    display: flex;
    flex-direction: column;
    gap: 10px;
    margin-top: 16px;
  }

  input {
    padding: 12px;
    font-size: 16px;
    border: 1px solid #ccc;
    border-radius: 8px;
  }

  button {
    padding: 12px;
    font-size: 16px;
    border: none;
    border-radius: 8px;
    background: #2563eb;
    color: white;
    cursor: pointer;
  }

  button:hover {
    background: #1d4ed8;
  }

  .error {
    color: red;
    margin-top: 10px;
    text-align: center;
  }

  .card {
    margin-top: 20px;
    padding: 16px;
    border-radius: 12px;
    background: #f3f4f6;
    text-align: center;
  }

  .temp {
    font-size: 2rem;
    font-weight: bold;
  }

  /* 📱 TABLET + DESKTOP */
  @media (min-width: 600px) {
    .search {
      flex-direction: row;
    }

    input {
      flex: 1;
    }

    button {
      width: 120px;
    }
  }
</style>