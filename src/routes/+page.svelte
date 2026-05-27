 <script>
  import { supabase } from "$lib/supabaseClient";
  import { onMount } from "svelte";

  // Components
  import WeatherSearch from "$lib/components/Molecules/WeatherSearch.svelte";
  import WeatherCard from "$lib/components/Organisms/WeatherCard.svelte";
  import FavoritesList from "$lib/components/Organisms/FavoritesList.svelte";

  // State variables
  let city = $state("");
  let weather = $state(null);
  let error = $state("");
  let favorites = $state([]);
  let loading = $state(false);

  // Fetch weather data from OpenWeather API
  async function getWeather(event) {
    console.log("CITY:", city);
    event.preventDefault();

    error = "";
    loading = true;

    // Validate input
    if (!city.trim()) {
      error = "Please enter a city";
      loading = false;
      return;
    }

    try {
      // Call weather API
      const res = await fetch(
        `https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=712c664b4430d30580f2d2e27501801d&units=metric`
      );

      const data = await res.json();

      // Handle API errors
      if (data.cod != 200) {
        error = data.message;
        weather = null;
        return;
      }

      // Store weather result
      weather = data;
    } catch {
      error = "Something went wrong";
    } finally {
      loading = false;
    }
  }

  // Load favorites from Supabase
  async function loadFavorites() {
    const { data, error } = await supabase
      .from("favorite_cities")
      .select("*")
      .order("created_at", { ascending: false });

    if (!error) {
      favorites = data;
    }
  }

  // Run once when page loads
  onMount(() => {
    loadFavorites();
  });

  // Save city to favorites
  async function saveCity() {
    if (!weather) return;

    // Prevent duplicates
    const exists = favorites.some(
      (f) => f.city.toLowerCase() === weather.name.toLowerCase()
    );

    if (exists) return;

    // Insert into Supabase
    const { error } = await supabase
      .from("favorite_cities")
      .insert([{ city: weather.name }]);

    if (!error) {
      loadFavorites();
    }
  }

  // Delete city from favorites
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

     <!-- Search component -->
    <WeatherSearch bind:city {getWeather} />

    <!-- Error display -->
    {#if error}
      <p class="error">{error}</p>
    {/if}

    <!-- Loading state -->
    {#if loading}
      <p class="loading">Loading weather...</p>
    {/if}

    <!-- Weather result -->
    {#if weather}
      <WeatherCard {weather} {saveCity} />
    {/if}

    <!-- Favorites list -->
    <FavoritesList {favorites} {deleteCity} />
  </div>
</main>

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

    /* Dark mode support */
    @media (prefers-color-scheme: dark) {
      background: #0b1220;
      color: #e2e8f0;
    }
  }
</style>