<script>
    let { favorites, deleteCity } = $props();
</script>

<h2>Favorite Cities</h2>

{#if favorites.length === 0}
    <p class="empty">You don’t have any saved cities yet</p>
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

<style>
.favorite {
  display: grid;
  gap: 12px;
  margin-top: 15px;

  .favorite-city {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 14px;
    border-radius: 14px;
    background: white;
    box-shadow: 0 8px 20px rgba(0, 0, 0, 0.05);
    transition: 0.2s;

    &:hover {
      transform: translateY(-2px);
    }

    .delete-btn {
      background: transparent;
      border: 1px solid #ef4444;
      color: #ef4444;
      padding: 6px 12px;
      border-radius: 10px;
      cursor: pointer;
      font-size: 13px;
      transition: 0.2s ease;

      &:hover {
        background: #ef4444;
        color: white;
      }
    }
  }

  .empty {
    text-align: center;
    color: #94a3b8;
    margin-top: 15px;
    font-size: 14px;
  }

  @media (prefers-color-scheme: dark) {
    .favorite-city {
      background: #111827;
      border: 1px solid #1f2937;
    }
  }
}
</style>
