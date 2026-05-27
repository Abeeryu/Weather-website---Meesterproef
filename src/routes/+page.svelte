<script>
  import { supabase } from "$lib/supabaseClient";
  import { onMount } from "svelte";

  let city = $state("");
  let weather = $state(null);
  let error = $state("");
  let favorites = $state([]);
  let loading = $state(false);

  async function getWeather(event) {
    event.preventDefault();

    error = "";
    loading = true;

    if (!city.trim()) {
      error = "Please enter a city";
      loading = false;
      return;
    }

    try {
      const res = await fetch(
        `https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=712c664b4430d30580f2d2e27501801d&units=metric`,
      );

      const data = await res.json();

      if (data.cod != 200) {
        error = data.message;
        weather = null;
        return;
      }

      weather = data;
    } catch {
      error = "Something went wrong";
    } finally {
      loading = false;
    }
  }

  async function loadFavorites() {
    const { data, error } = await supabase
      .from("favorite_cities")
      .select("*")
      .order("created_at", { ascending: false });

    if (!error) {
      favorites = data;
    }
  }

  onMount(() => {
    loadFavorites();
  });

  async function saveCity() {
    if (!weather) return;

    // check if already exists
    const exists = favorites.some(
      (f) => f.city.toLowerCase() === weather.name.toLowerCase(),
    );

    if (exists) return;

    const { error } = await supabase
      .from("favorite_cities")
      .insert([{ city: weather.name }]);

    if (!error) {
      loadFavorites();
    }
  }

  async function deleteCity(id) {
    const { error } = await supabase
      .from("favorite_cities")
      .delete()
      .eq("id", id);

    if (error) {
      console.error(error);
      return;
    }

    loadFavorites();
  }
</script>

<main class="weather-app">
  <div class="container">
    <header>
      <h1>Weather App</h1>
    </header>

    <form class="search" onsubmit={getWeather}>
      <label for="city"></label>

      <input
        id="city"
        value={city}
        oninput={(e) => (city = e.target.value)}
        placeholder="Enter city"
      />

      <button type="submit">Search</button>
    </form>

    {#if error}
      <p class="error">{error}</p>
    {/if}

    {#if loading}
      <p class="loading">Loading weather...</p>
    {/if}

    {#if weather}
      <article class="weather-card">
        <h2>{weather.name}</h2>
        <p class="temp">{weather.main.temp}°C</p>
        <p>{weather.weather[0].description}</p>
        <p>{weather.wind.speed} m/s</p>
      </article>
      <button onclick={saveCity}> Save city </button>
    {/if}

    <h2>Favorite Cities</h2>

    {#if favorites.length === 0}
      <p class="empty">
        You don’t have any saved cities yet. Search and save one
      </p>
    {:else}
      <section class="favorite">
        {#each favorites as fav}
          <article class="favorite-city">
            <span>{fav.city}</span>

            <button class="delete-btn" onclick={() => deleteCity(fav.id)}>
              Remove
            </button>
          </article>
        {/each}
      </section>
    {/if}
  </div>
</main>

<!-- (MOBILE FIRST) -->
<style>
  .weather-app {
    min-height: 100vh;
    width: 100%;
    margin: 0;

    font-family: sans-serif;
    padding: 16px;

    background: linear-gradient(135deg, #dbeafe, #f8fafc, #e0f2fe);
  }

  .container {
    max-width: 500px;
    margin: 0 auto;
  }

  /* HEADER */
  h1 {
    font-size: 1.8rem;
    text-align: center;
    margin-bottom: 10px;
    font-weight: 700;
    color: #0f172a;
  }

  /* SEARCH CARD */
  .search {
    display: flex;
    flex-direction: column;
    gap: 10px;
    margin-top: 20px;
    padding: 16px;
    background: white;
    border-radius: 14px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.05);
  }

  .sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  overflow: hidden;
}

  label {
    font-size: 14px;
    color: #64748b;
  }

  input {
    padding: 12px;
    font-size: 16px;
    border: 1px solid #e2e8f0;
    border-radius: 10px;
    outline: none;
    transition: 0.2s;
  }

  input:focus {
    border-color: #2563eb;
    box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.1);
  }

  /* BUTTONS */
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
  }

  button:hover {
    transform: translateY(-1px);
    box-shadow: 0 6px 18px rgba(37, 99, 235, 0.25);
  }

  /* ERROR */
  .error {
    color: #ef4444;
    margin-top: 12px;
    text-align: center;
    font-size: 14px;
  }

  /* LOADING */
  .loading {
    text-align: center;
    margin-top: 15px;
    font-weight: 500;
    color: #64748b;
  }

  /* WEATHER CARD */
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
  }

  .temp {
    font-size: 2.5rem;
    font-weight: 700;
    color: #0f172a;
    margin: 10px 0;
  }

  /* FAVORITES */
  .favorite {
    display: grid;
    gap: 12px;
    margin-top: 15px;
  }

  .favorite-city {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 14px;
    border-radius: 14px;
    background: white;
    box-shadow: 0 8px 20px rgba(0, 0, 0, 0.05);
    transition: 0.2s;
  }

  .favorite-city:hover {
    transform: translateY(-2px);
  }

  /* DELETE BUTTON */
  .delete-btn {
    background: transparent;
    border: 1px solid #ef4444;
    color: #ef4444;
    padding: 6px 12px;
    border-radius: 10px;
    cursor: pointer;
    font-size: 13px;
    transition: 0.2s ease;
    align-items: center;
  }

  .delete-btn:hover {
    background: #ef4444;
    color: white;
  }

  /* EMPTY STATE */
  .empty {
    text-align: center;
    color: #94a3b8;
    margin-top: 15px;
    font-size: 14px;
  }

  /* RESPONSIVE */
  @container weather (min-width: 400px) {
  .weather-card {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 12px;
    text-align: left;
  }

  .temp {
    font-size: 2rem;
  }
}

  @media (min-width: 600px) {
    .search {
      flex-direction: row;
      align-items: end;
    }

    input {
      flex: 1;
    }

    button {
      width: 120px;
    }
  }

  @media (prefers-color-scheme: dark) {
  .weather-app {
    background: #0b1220;
    color: #e2e8f0;
  }

  .search,
  .weather-card,
  .favorite-city {
    background: #111827;
    border: 1px solid #1f2937;
  }

  input {
    background: #0f172a;
    color: white;
    border: 1px solid #334155;
  }
}
</style>
