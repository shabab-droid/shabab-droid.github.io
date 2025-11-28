# Sisyphus' Ruin

> "The struggle itself towards the heights is enough to fill a man's heart. One must imagine Sisyphus happy." — Albert Camus

**Sisyphus' Ruin** is a browser-based probabilistic simulation and incremental game that explores the mathematics of persistence, failure, and the absurd. It combines a clean, modern UI with a chaotic "Atmosphere" engine to visualize the struggle of pushing the boulder up the mountain.

## 🏔️ Overview

The goal is simple: Push the stone to the Summit (Height 20). However, the journey is governed by probability and chaos.

Each roll of the dice results in one of three outcomes:
1.  **Ascent:** The stone rises.
2.  **Steady:** The stone holds fast, and you build **Poise**.
3.  **Ruin:** Gravity wins. The stone returns to the bottom.

**Poise** is your defense. As you accumulate Poise (through "Steady" outcomes), your probability of Ruin decreases and your probability of Ascent increases.

## 🌪️ Features

### The Chaos Engine
The background features a live-rendered **Lorenz Attractor** (a strange attractor from chaos theory). This isn't just a visual effect; it acts as the "Atmosphere."
* The velocity of the chaotic system dictates **Time Dilation**.
* **Calm** weather allows for swift moves.
* **Storms** distort time, forcing long delays between attempts.
* *Note: The chaos affects the flow of time, but never the probability of the fall.*

### Statistical Analysis
The simulation includes a robust math engine (`SisyphusMath`) that calculates and displays:
* Real-time probability updates based on current Height and Poise.
* Monte Carlo simulations for expected win rates and cycle lengths.
* Heatmaps and histograms tracking time spent at specific Heights/Poise levels.

### Save & Load System
* **5 Local Save Slots:** Store different run states locally.
* **JSON Import/Export:** Export specific save slots or your current run to a file to share or back up.
* **Auto-Save:** The game preserves your state between sessions.

### Advanced Configuration
Open the **Advanced Parameters** accordion to tweak the laws of physics:
* Adjust the Max Height ($M$) and Max Poise ($L$).
* Modify the Beta ($\beta$) curve for difficulty scaling.
* Change the decay rate ($\alpha$) of Ruin probabilities.

## 🛠️ Technical Implementation

This project is built as a **Single Page Application (SPA)** with no external dependencies.

* **Core:** Vanilla JavaScript (ES6+ Classes).
* **Visuals:** HTML5 Canvas (for the Chaos Engine) and CSS3 Grid/Flexbox.
* **Styles:** Modern CSS variables for theming and animations.
* **Accessibility:** Includes ARIA announcers for screen readers and full keyboard navigation.

### Key Classes
* `GameController`: Manages the game loop, inputs, and save states.
* `SisyphusMath`: Handles the probability logic and expectation solving.
* `ChaosEngine`: Renders the Lorenz system and calculates time modifiers.
* `SisyphusUI`: Handles DOM updates, animations, and modal rendering.

## ⌨️ Controls

* **Space / Enter**: Roll (Push the stone).
* **A**: Toggle Auto-Play.
* **R**: Reset the simulation.
* **?**: Open Help/Shortcuts.

## 📄 License

[MIT License](LICENSE) - Feel free to fork, modify, and distribute.
