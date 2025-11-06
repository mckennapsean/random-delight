### **Project Plan: "Delightful Randomizer" App**

**1. Project Goal:**
Create a web-based randomizer app using SvelteKit. The app will feature a single "Smart Command Center" interface that accepts text input (e.g., `1d20`, `coin`, or pasted lists) and uses a unified, suspense-building "elimination" animation to display the results.

**2. Tech Stack:**

  * **Framework:** SvelteKit (for Svelte, routing, and a modern build process).
  * **Hosting:** Static adapter for GitHub Pages.
  * **Persistence:** Browser `localStorage` (no backend).

**3. Core Architecture: "Smart Command Center"**
The app will be a single-page application. The UI is centered around two main components:

1.  **A "Smart Input" bar:** A text input where the user types commands or pastes lists.
2.  **"Quick-Pick" buttons:** A set of icons below the input that pre-fill the text bar with common commands (e.g., `coin`, `1d20`) to aid discoverability.
3.  **A "Results Log":** A scrollable area above the input that displays the history of all randomization results and their animations.

-----

### **4. Project File Structure (SvelteKit)**

```
/src
├── lib/
│   ├── components/
│   │   ├── Header.svelte         # App title, settings icon, etc.
│   │   ├── SmartInput.svelte     # The main text input bar.
│   │   ├── QuickPicks.svelte     # The row of icon buttons (coin, dice, etc.).
│   │   ├── ResultsLog.svelte     # The scrollable area for history.
│   │   ├── renderers/
│   │   │   └── EliminationGrid.svelte  # V1: The *only* renderer.
│   │   └── Icon.svelte           # A helper component for SVG icons.
│   │
│   ├── stores.js         # Svelte stores for state management.
│   ├── parser.js         # The core "smart parser" logic.
│   ├── random.js         # The core randomness utilities.
│   └── types.js          # TypeScript/JSDoc types for our data.
│
└── routes/
    └── +page.svelte      # The main (and only) page.
```

-----

### **5. Data & State Management (`src/lib/stores.js`)**

We will use Svelte's `writable` stores to manage the application state.

```javascript
import { writable } from 'svelte/store';
import { browser } from '$app/environment';

// Type definition for a single result
/**
 * @typedef {Object} Result
 * @property {string} id - A unique ID (e.g., new Date().getTime())
 * @property {string} rawInput - The original text (e.g., "2d6+2")
 * @property {import('./types').ParsedCommand} command - The parsed command object
 */

// The main store for the results log.
// It will be synced with localStorage.
const initialHistory = browser ? JSON.parse(localStorage.getItem('randomHistory') || '[]') : [];
export const history = writable(initialHistory);

// Subscribe to history changes and save to localStorage
if (browser) {
  history.subscribe((value) => {
    localStorage.setItem('randomHistory', JSON.stringify(value));
  });
}
```

-----

### **6. Component & Logic Breakdown**

#### `src/lib/random.js` (Core Utilities)

This module centralizes all randomness functions.

  * `getRandomInt(min, max)`: Returns a random integer. Uses `Math.random()`.
  * `shuffleArray(array)`: Returns a new, shuffled copy of an array (Fisher-Yates algorithm).

#### `src/lib/types.js` (Data Structures)

This file defines the data structures the parser will output.

```javascript
/**
 * @typedef {Object} ParsedCommand
 * @property {'list' | 'dice'} type - The type of randomization.
 * @property {Array<Object>} tasks - A list of randomization tasks to run.
 * @property {number} [modifier] - A number to add/subtract (for dice).
 *
 * @typedef {Object} RandomizationTask
 * @property {Array<string | number>} items - The list of items to pick from.
 * @property {number} pickCount - How many items to pick from the list (usually 1).
 */
```

#### `src/lib/parser.js` (The Smart Parser)

This is the "brain" of the app. It takes raw text and turns it into a `ParsedCommand`.

```javascript
/**
 * Parses raw text input into a structured command object.
 * @param {string} text
 * @returns {import('./types').ParsedCommand | null}
 */
export function parseCommand(text) {
  const trimmedText = text.trim();

  // 1. Dice Notation (e.g., "2d6+2", "d20")
  const diceRegex = /^(\d+)?d(\d+)([+\-]\d+)?$/i;
  const diceMatch = trimmedText.match(diceRegex);
  if (diceMatch) {
    const count = parseInt(diceMatch[1] || '1', 10);
    const sides = parseInt(diceMatch[2], 10);
    const modifier = parseInt(diceMatch[3] || '0', 10);
    
    const diceList = Array.from({ length: sides }, (_, i) => i + 1);
    
    return {
      type: 'dice',
      tasks: Array.from({ length: count }, () => ({
        items: diceList,
        pickCount: 1,
      })),
      modifier: modifier,
    };
  }

  // 2. Keyword: "coin" or "flip"
  if (['coin', 'flip'].includes(trimmedText.toLowerCase())) {
    return {
      type: 'list',
      tasks: [{ items: ['Heads', 'Tails'], pickCount: 1 }],
      modifier: 0,
    };
  }

  // 3. Pasted List (comma, newline, or space separated)
  const listItems = trimmedText.split(/[\s,n]+/g).filter(Boolean);
  if (listItems.length > 1) {
    return {
      type: 'list',
      tasks: [{ items: listItems, pickCount: 1 }],
      modifier: 0,
    };
  }

  // No match
  return null;
}
```

#### `src/routes/+page.svelte` (The Main UI)

This assembles the app.

  * Imports `SmartInput`, `QuickPicks`, and `ResultsLog`.
  * Imports the `history` store.
  * Defines a `handleCommand(event)` function that:
    1.  Gets the raw text from the `SmartInput` component's event.
    2.  Calls `parseCommand(rawText)`.
    3.  If a valid command is returned, it creates a new `Result` object (with a unique ID) and adds it to the `history` store (`$history = [...$history, newResult]`).

#### `src/lib/components/ResultsLog.svelte` (The Renderer)

This is the "Smart Renderer" (V1).

  * Subscribes to the `history` store: `{#each $history as result (result.id)} ... {/each}`.
  * Inside the loop, it renders the raw input: `<div>You ran: {result.rawInput}</div>`.
  * It then loops over `result.command.tasks`: `{#each result.command.tasks as task}`.
  * For **every task**, it renders the V1 component: `<EliminationGrid items={task.items} />`.
  * If `result.command.modifier` exists, it renders the modifier after the grid(s).

#### `src/lib/components/renderers/EliminationGrid.svelte` (The Animation)

This is the core animation component.

  * **Props:** `export let items = [];` (e.g., `[1, 2, ..., 20]` or `["Alice", "Bob"]`).
  * **State:**
      * `let remainingItems = [];`
      * `let winner = null;`
      * `let isRunning = false;`
  * **OnMount:**
      * It should immediately set `remainingItems` to a *shuffled* copy of the `items` prop (using `shuffleArray`).
      * It should *not* start the animation automatically. (We might add a "Re-run" button later).
      * It *should* start the animation.
  * **Animation Logic:**
    1.  Create an `async function runElimination()` that sets `isRunning = true`.
    2.  Use a `while` loop: `while (remainingItems.length > 1)`.
    3.  Inside the loop, calculate how many items to remove based on the current length. This is your "adaptive animation" logic:
          * If `length > 50`, remove 10 items at a time.
          * If `length > 15`, remove 3 items at a time.
          * If `length <= 15`, remove 1 item at a time (for suspense).
    4.  Create a `delay(ms)` helper function (`new Promise(resolve => setTimeout(resolve, ms))`).
    5.  `await delay(ms)`: The delay should get *longer* as the list gets smaller (e.g., 50ms for batch removal, 300ms for final one-by-one removal).
    6.  After the delay, update the state: `remainingItems = remainingItems.slice(0, remainingItems.length - itemsToRemove);`. Svelte's reactivity will handle the UI update.
    7.  When the loop finishes, set `winner = remainingItems[0]` and `isRunning = false`.
  * **Markup:**
      * Use `{#each remainingItems as item}` to render the grid.
      * Use CSS Grid (`display: grid`) for the layout.
      * When `winner` is set, apply a "winner" class to highlight it and make it larger. The other items should be rendered with an "eliminated" class (e.g., `opacity: 0.2`).
      * The suspense is built by Svelte re-rendering the list as the `remainingItems` array shrinks.

-----

### **7. Future Work (V2) - Not for this Plan**

  * **Additional Renderers:** Add new components (e.g., `DiceSkitter.svelte`, `CoinFlip.svelte`) and update `ResultsLog.svelte` to pick the right one based on `result.command.type`.
  * **Enhanced Parser:** Support "pick 3 from list" syntax.
  * **History Re-run:** Add a "re-run" button to each log entry.
  * **Secure Crypto:** Add a setting to toggle the `random.js` functions between `Math.random()` and `window.crypto.getRandomValues()`.

