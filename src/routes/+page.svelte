<script>
  import { supabase } from "$lib/supabaseClient";
  import { onMount } from "svelte";
  import WeatherSearch from "$lib/components/Molecules/WeatherSearch.svelte";
  import WeatherCard from "$lib/components/Organisms/WeatherCard.svelte";
  import FavoritesList from "$lib/components/Organisms/FavoritesList.svelte";

  let city = $state("");
  let weather = $state(null);
  let error = $state("");
  let favorites = $state([]);
  let loading = $state(false);

  async function getWeather(event) {
    console.log("CITY:", city);
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

    <WeatherSearch bind:city {getWeather} />

    {#if error}
      <p class="error">{error}</p>
    {/if}

    {#if loading}
      <p class="loading">Loading weather...</p>
    {/if}

    {#if weather}
      <WeatherCard {weather} {saveCity} />
    {/if}

    <FavoritesList {favorites} {deleteCity} />
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

  .container {
    max-width: 500px;
    margin: 0 auto;
  }

  h1 {
    font-size: 1.8rem;
    text-align: center;
    margin-bottom: 10px;
    font-weight: 700;
    color: #0f172a;
  }

  .error {
    color: #ef4444;
    margin-top: 12px;
    text-align: center;
    font-size: 14px;
  }

  .loading {
    text-align: center;
    margin-top: 15px;
    font-weight: 500;
    color: #64748b;
  }

  @media (prefers-color-scheme: dark) {
    background: #0b1220;
    color: #e2e8f0;
  }
}
</style>
