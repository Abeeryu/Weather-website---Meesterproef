<script>
    let { city = $bindable(), getWeather } = $props();
</script>

<form class="search" onsubmit={getWeather}>
    <label for="city" class="sr-only"> Enter city </label>

    <input id="city" bind:value={city} placeholder="Enter city" />

    <button type="submit">Search</button>
</form>

<style>
.search {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-top: 20px;
  padding: 16px;
  background: white;
  border-radius: 14px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.05);

  label {
    font-size: 14px;
    color: #64748b;
  }

  .sr-only {
    position: absolute;
    width: 1px;
    height: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    white-space: nowrap;
    border: 0;
  }

  input {
    padding: 12px;
    font-size: 16px;
    border: 1px solid #e2e8f0;
    border-radius: 10px;
    outline: none;
    transition: 0.2s;

    &:focus {
      border-color: #2563eb;
      box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.1);
    }
  }

  @media (min-width: 600px) {
    flex-direction: row;
    align-items: end;

    input {
      flex: 1;
    }

    button {
      width: 120px;
    }
  }

  @media (prefers-color-scheme: dark) {
    background: #111827;
    border: 1px solid #1f2937;

    input {
      background: #0f172a;
      color: white;
      border: 1px solid #334155;
    }
  }
}
</style>
