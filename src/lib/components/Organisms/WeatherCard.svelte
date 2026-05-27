<script>
    let { weather, saveCity } = $props();
</script>

<article class="weather-card">
    <h2>{weather.name}</h2>

    <p class="temp">
        {weather.main.temp}°C
    </p>

    <p>
        {weather.weather[0].description}
    </p>

    <p>
        {weather.wind.speed} m/s
    </p>
</article>

<button onclick={saveCity}> Save city </button>

<style>
 .weather-card {
  margin-top: 20px;
  padding: 18px;
  border-radius: 16px;
  background: white;
  text-align: center;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.06);
  border: 1px solid #f1f5f9;
  container-name: weather;
  container-type: inline-size;

  .temp {
    font-size: 2.5rem;
    font-weight: 700;
    color: #0f172a;
    margin: 10px 0;
  }

  button {
    padding: 12px;
    font-size: 15px;
    border: none;
    border-radius: 10px;
    background: linear-gradient(135deg, #2563eb, #1d4ed8);
    color: white;
    cursor: pointer;
    transition: 0.2s ease;
    font-weight: 500;
    margin-top: 20px;

    &:hover {
      transform: translateY(-1px);
      box-shadow: 0 6px 18px rgba(37, 99, 235, 0.25);
    }
  }

  @container weather (min-width: 400px) {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 12px;
    text-align: left;

    .temp {
      font-size: 2rem;
    }
  }

  @media (prefers-color-scheme: dark) {
    background: #111827;
    border: 1px solid #1f2937;
  }
}
</style>
