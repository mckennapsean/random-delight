<script>
  import { onMount } from 'svelte';

  /** The list of items to pick from */
  export let items = [];

  // Create a shuffled, reactive copy of the items for elimination
  let visibleItems = [];
  let winner = null;
  
  onMount(() => {
    // 1. Create a shuffled copy on mount
    visibleItems = [...items].sort(() => Math.random() - 0.5);
    
    // 2. Start the animation
    runElimination();
  });

  /** A simple promise-based delay */
  function delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }

  /** The core animation logic */
  async function runElimination() {
    // This is the simple "suspense" logic
    while (visibleItems.length > 1) {
      // Pick a random item to remove
      const removeIndex = Math.floor(Math.random() * visibleItems.length);
      
      // Get the item's DOM element to add a "fading" class
      const allElements = Array.from(document.querySelectorAll(`[data-id="${visibleItems[removeIndex]}"]`));
      const elementToRemove = allElements.find(el => !el.classList.contains('eliminating')); // Find the one in *this* component
      
      if (elementToRemove) {
          elementToRemove.classList.add('eliminating');
      }

      // Wait for the "fade out" CSS transition
      await delay(300);

      // Actually remove the item from the array to trigger reactivity
      visibleItems.splice(removeIndex, 1);
      visibleItems = visibleItems; // This assignment forces Svelte to update
      
      // Wait a moment before the next removal
      await delay(100);
    }
    
    // Set the winner
    winner = visibleItems[0];
  }
</script>

<div class="grid-container">
  {#if winner}
    <div class="winner-display">
      🏆 {winner} 🏆
    </div>
  {:else}
    <div class="grid">
      {#each visibleItems as item}
        <div class="item" data-id={item}>
          {item}
        </div>
      {/each}
    </div>
  {/if}
</div>

<style>
  .grid {
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem;
    margin-top: 1rem;
  }
  .item {
    background-color: #f4f4f4;
    padding: 0.5rem 1rem;
    border-radius: 4px;
    font-weight: 500;
    transition: all 0.3s ease-out;
  }
  .item.eliminating {
    /* The "fade out" animation */
    opacity: 0;
    transform: scale(0.8);
  }
  .winner-display {
    margin-top: 1rem;
    font-size: 2rem;
    font-weight: bold;
    text-align: center;
    color: #3772ff;
    animation: pop-in 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  }

  @keyframes pop-in {
    0% {
      transform: scale(0.5);
      opacity: 0;
    }
    100% {
      transform: scale(1);
      opacity: 1;
    }
  }
</style>

