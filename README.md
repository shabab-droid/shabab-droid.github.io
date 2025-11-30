# Sisyphus' Ruin

> "The struggle itself towards the heights is enough to fill a man's heart. One must imagine Sisyphus happy."  
> — Albert Camus, *The Myth of Sisyphus*

A browser-based meditation on persistence, probability, and the absurd. Push the stone. Watch it fall. Begin again.

---

## The Burden

You are Sisyphus. Your task is eternal: push the stone to the Summit.

Each attempt to move the stone yields one of three fates:

**Ascent** — The stone rises. You climb closer to the Summit, but the effort shatters your composure. Your Poise resets, leaving you vulnerable.

**Steady** — The stone holds fast against the slope. Nothing gained, nothing lost—except that in stillness, you gather yourself. Your Poise grows.

**Ruin** — Gravity wins. The stone tumbles back to the base of the mountain. You follow it down. The climb begins anew.

### The Role of Poise

Poise is your defense against the mountain. As it accumulates through Steady outcomes, two things shift in your favor: the probability of Ascent increases, and the probability of Ruin decreases. But every successful Ascent costs you—Poise is consumed by the effort of climbing, resetting you to a precarious state.

### The Atmosphere

The background renders a Lorenz attractor—a chaotic system that never repeats, never settles. Its velocity determines the *Time Dilation* between your moves:

- **Calm** skies allow rapid attempts
- **Windy** conditions slow your pace  
- **Storms** force long pauses between rolls

The chaos affects only *time*, never *probability*. The odds of Ruin remain what they are. You simply must wait longer to face them.

### Controls

| Key | Action |
|-----|--------|
| `Space` / `Enter` | Roll (push the stone) |
| `A` | Toggle auto-play |
| `R` | Reset the simulation |
| `?` | Show keyboard shortcuts |

---

## Technical Documentation

### Architecture

Sisyphus' Ruin is a single-page application with zero external runtime dependencies. All logic, rendering, and state management occur client-side using vanilla JavaScript (ES6+ classes), HTML5 Canvas, and CSS3.

### Core Modules

#### `SisyphusMath` — Probability Engine

Handles all stochastic calculations and expectation solving.

**Probability Model:**

For a state at Height `k` (1 to M) and Poise `ℓ` (0 to L):

```
Base win rate:     w_base = w_min + (w_max - w_min) × ((k-1)/(M-1))^β
Poise multiplier:  ε(ℓ)   = (1 - ρ) + ρ × (ℓ/L)^γ
Final win prob:    P(win) = w_base × ε(ℓ)

Loss probability:  P(loss) = c_min + (c_max - c_min) × e^(-α × ℓ)
Neutral prob:      P(neu)  = 1 - P(win) - P(loss)
```

**Expectation Solver:**

Uses value iteration to solve a system of Bellman equations across the full (k, ℓ) state space. Computes:
- `TopHit[k][ℓ]` — Probability of eventually reaching Summit from state (k, ℓ)
- `CycleLen[k][ℓ]` — Expected steps until Ruin or Summit
- `WinCount[k][ℓ]` — Expected Ascents before cycle ends
- `TimeLen[k][ℓ]` — Expected real-time duration (incorporating weather delays)

**Caching:**

A `ProbabilityCache` (LRU, max 400 entries) memoizes probability calculations keyed by state and full parameter set, avoiding redundant computation during auto-play.

#### `SisyphusModel` — Game State

Maintains all mutable game state:

```javascript
state: { k, l, step, topHit, cycleSteps }
stats: { wins, neus, loss, maxK, longestCycle }
distributions: { k[], l[], kl[][], cycles[] }
cycleSummaries: [ { id, length, wins, neus, maxK, path, outcome } ]
```

State transitions occur in `step()`, which samples from the probability distribution and updates all tracking structures.

#### `ChaosEngine` — Lorenz Attractor Renderer

Simulates the Lorenz system with parameters σ=10, ρ=28, β=8/3:

```
dx/dt = σ(y - x)
dy/dt = x(ρ - z) - y  
dz/dt = xy - βz
```

The instantaneous velocity magnitude maps to a time dilation multiplier (0.5x to ~3x). The attractor is rendered via 3D→2D projection with continuous rotation, using a ring buffer of 10,000 points for the trail.

The `setFrozen(bool)` method pauses weather text updates during roll animations while continuing the visual simulation.

#### `SisyphusUI` — Rendering Layer

Manages all DOM updates with batched writes via `requestAnimationFrame` to minimize layout thrashing. Key optimizations:
- Pending updates accumulated in a `Map` and flushed once per frame
- Template-based cycle entry rendering using `<template>` elements
- CSS-driven animations for progress bars (barberpole effect via `background-position` animation)

The `visualizeRoll()` method animates the RNG marker across the probability bar using a dramatic cubic-bezier easing that lingers at extremes.

#### `GameController` — Orchestration

Coordinates all modules and handles:
- Manual vs. auto-play modes
- Time-dilated delays with animated progress bars
- 5-slot save system with JSON import/export
- Theme management and persistence
- Keyboard shortcuts

Uses a `BackgroundTimer` (Web Worker-based) to maintain accurate timing even when the browser tab is inactive.

### Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `M` | 20 | Summit height (max k) |
| `L` | 20 | Max Poise |
| `w_min` / `w_max` | 0.10 / 0.90 | Win probability bounds |
| `β` (beta) | 1.5 | Height progression curve (>1 = easier late game) |
| `ρ` (rho) | 0.6 | Poise influence weight |
| `γ` (gamma) | 1.5 | Poise scaling exponent |
| `c_min` / `c_max` | 0.03 / 0.10 | Loss probability bounds |
| `α` (alpha) | 0.3 | Poise decay rate for loss reduction |
| `g_n` | 1 | Poise gained on Steady |
| `s_w` | 20 | Poise spent on Ascent |
| `max_delay` | 45.0 | Base cooldown (seconds) at k=1, ℓ=0 |
| `chaos_dt` | 0.0025 | Lorenz simulation timestep |
| `chaos_intensity` | 1.0 | Weather effect magnitude |

### Theming System

The `ThemeManager` supports:
- 5 preset themes (Cosmic, Magma, Terminal, Academic, Abyss)
- OKLCH-based procedural palette generation with harmony modes (tetradic, triadic, complementary, analogous, monochromatic)
- Custom palette saving/loading to localStorage
- JSON theme import/export

Color generation uses perceptually uniform OKLCH color space, converting through OKLab to linear sRGB with proper gamma correction.

### Persistence

**Auto-save:** Current run saves to `localStorage` on every step under key `sisyphus_v9_delay`.

**Save Slots:** 5 named slots (`sisyphus_slot_1` through `sisyphus_slot_5`) store complete serialized state.

**Serialized State Includes:**
- All parameters
- Current state (k, l, step, topHit)
- Lifetime statistics
- Distribution histograms
- Cycle history and summaries
- Run start timestamp

### Accessibility

- ARIA live region for screen reader announcements of outcomes
- Full keyboard navigation
- `prefers-reduced-motion` could be respected (not yet implemented)
- Semantic HTML structure with proper labeling

### Performance Considerations

- Probability cache prevents redundant calculations during rapid auto-play
- Canvas rendering uses a ring buffer to cap memory usage
- UI updates are batched and throttled
- Web Worker timer ensures background tab accuracy
- Distribution histograms use fixed-size arrays

---

## Philosophy

The game is unwinnable in any permanent sense. You may reach the Summit, but the stone will fall again. The statistics accumulate; the expected values converge; the heatmap fills in. And still you roll.

Camus argued that we must imagine Sisyphus happy—not despite the futility, but because of it. The absurd hero finds meaning in the struggle itself, in the moment of conscious descent, in the decision to push again.

This is a game about that decision.

---

## License

MIT
