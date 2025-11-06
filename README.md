# 🎲 Delightful Randomizer

A modern, fast, and "delightful" randomizer web app built with SvelteKit. This project focuses on a unique, unified user experience through a "Smart Command Center" interface, prioritizing fun, suspense-building animations for every randomization.

[Image or GIF of the app in action once it's built]

## ✨ Core Concept: The "Smart Command Center"

Unlike traditional apps that split tools into different pages, this app uses a single, powerful interface:

* **Unified Input:** A single text bar is your entry point for everything.
* **Smart Parser:** It intelligently understands what you want to do.
    * Type `2d6+5` or `d20` to roll dice.
    * Type `coin` or `flip` for a coin toss.
    * Paste a list of names (`Alice, Bob, Carol`) to pick a winner.
* **"Quick-Picks":** For discoverability, icon buttons (🪙, 🎲, 🎡) instantly add common commands to the input bar.
* **Results Log:** A persistent "chat-style" log shows your complete history of randomizations and their results.

## 🚀 Key Features

* **Suspenseful Animations:** The core of the "delightful" experience. Instead of just showing a result, the app uses a "suspenseful elimination" animation—showing all possible outcomes and rapidly, then slowly, eliminating them until only the winner remains.
* **Complex Dice Support:** Handles multi-die rolls (like `3d6`) by running the elimination animation for *each die* simultaneously.
* **Pasted List Support:** Instantly parses comma, space, or newline-separated lists for random picking.
* **Zero-Backend:** Runs entirely in your browser.
* **Persistent History:** Your results log is automatically saved to your browser's `localStorage` so it's there when you come back.

## 🛠️ Tech Stack

* **Framework:** [SvelteKit](https://kit.svelte.dev/)
* **Language:** JavaScript (with JSDoc for types)
* **State Management:** Svelte Stores
* **Styling:** Plain CSS / CSS Modules (or your preference)
* **Hosting:** Built as a static site, perfect for [GitHub Pages](https://pages.github.com/).

## 🏁 Getting Started (Development)

This project is built with SvelteKit. To get it running locally:

1.  **Clone the repository:**
    ```bash
    git clone [your-repo-url]
    cd [your-repo-name]
    ```

2.  **Install dependencies:**
    ```bash
    npm install
    ```

3.  **Run the development server:**
    ```bash
    npm run dev
    ```

    Open [http://localhost:5173](http://localhost:5173) to see the app in action.

## 📦 Building for Production (GitHub Pages)

This project uses the `adapter-static` for SvelteKit to build a pure static site.

1.  **Run the build command:**
    ```bash
    npm run build
    ```

2.  This will generate a `build/` directory. You can then configure GitHub Pages to deploy from this directory (or use a GitHub Action to automate this).
