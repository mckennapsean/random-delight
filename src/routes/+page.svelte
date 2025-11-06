<script>
  import EliminationGrid from '$lib/EliminationGrid.svelte';
  
  let rawText = '';
  let results = []; // This will be our simple, in-memory "results log"

  /**
   * This is a "dumb" proof-of-concept parser.
   * It only checks for 'coin' or a comma-separated list.
   */
  function parsePoc(text) {
    const trimmed = text.trim();

    if (trimmed.toLowerCase() === 'coin') {
      return {
        id: Date.now(), // Use a unique ID for the #each block key
        items: ['Heads', 'Tails'],
        raw: 'coin'
      };
    }

    if (trimmed.includes(',')) {
      return {
        id: Date.now(),
        items: trimmed.split(',').map(s => s.trim()).filter(Boolean),
        raw: trimmed
      };
    }

    // Not a valid PoC command
    return null;
  }

  function handleSubmit() {
    const command = parsePoc(rawText);

    if (command && command.items.length > 1) {
      // Add the new command to the top of the results log
      results = [command, ...results];
    }
    
    // Clear the input
    rawText = '';
  }
</script>

<svelte:head>
  <title>Randomizer PoC</title>
</svelte:head>

<main>
  <h1>🎲 Randomizer PoC</h1>
  
  <form on:submit|preventDefault={handleSubmit}>
    <input
      type="text"
      bind:value={rawText}
      placeholder="Type 'coin' or 'a,b,c'"
    />
    <button type="submit">Go</button>
  </form>

  <div class="results-log">
    {#each results as result (result.id)}
      <div class="result-entry">
        <small>You ran: "{result.raw}"</small>
        <EliminationGrid items={result.items} />
      </div>
    {/each}
  </div>
</main>

<style>
  main {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
    max-width: 600px;
    margin: 2rem auto;
  }
  form {
    display: flex;
    gap: 0.5rem;
    margin-bottom: 2rem;
  }
  input {
    flex: 1;
    font-size: 1.2rem;
    padding: 0.5rem;
  }
  button {
    font-size: 1.2rem;
    padding: 0.5rem 1rem;
    background-color: #3772ff;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }
  .results-log {
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
  }
  .result-entry {
    border: 1px solid #eee;
    padding: 1rem;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.05);
  }
  small {
    color: #666;
  }
</style>

