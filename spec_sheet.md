# Software Requirements & Technical Specification (SRS)

## Project: APEX PIT WALL OS — Dual-Car Strategy Simulator (20-Driver Grid)

### 1. System Overview

**APEX PIT WALL OS** is a browser-based, zero-dependency, single-file interactive motorsports simulation application. The user acts as Head of Strategy for the team, managing two drivers: **Yuki Tsunoda (Car #22)** and **Daniel Ricciardo (Car #3)** against a full 20-driver Formula 1 grid during a 35-lap Grand Prix at Silverstone Circuit.

The system simulates dynamic tire degradation, weather shifts, variable safety car states, thermal friction modeling, procedural audio synthesis, and team pit-stop logistics (including double-stack queuing).

### 2. Architecture & Design Principles

| Constraint / Pillar | Specification |
| :--- | :--- |
| **Delivery Model** | Standalone Single-File Architecture (`index.html`). Zero build pipeline (`npm`, `vite`, `webpack`). |
| **Styling & Theme** | Tailwind CSS CDN + custom CSS glassmorphism, carbon-fiber textures, and FIA high-contrast pit telemetry palette. |
| **Typography** | Google Fonts: `Chakra Petch` (timing/racing headers) and `JetBrains Mono` (telemetry/monospaced numbers). |
| **Audio Engine** | Web Audio API procedural synthesis (pure mathematical wave synthesis, no external audio assets). |
| **Graphics** | HTML5 Canvas 2D for circuit geometry, track conditions, and car positional telemetry blips. |
| **Tick Model** | Decoupled $\Delta t$ delta-time loop driven by `requestAnimationFrame`, with time-acceleration multipliers ($1\times, 2\times, 5\times, 10\times$). |
| **Grid Size** | Full 20-car F1 grid with authentic numbers, liveries, and team allocations. |

### 3. Grid Roster & Driver Allocation

The simulation models 10 teams and 20 drivers. The user commands Car #22 and Car #3:

| Pos / No. | Driver | Team | Color | Role / Control |
| :---: | :--- | :--- | :---: | :--- |
| **#22** | **Yuki Tsunoda** | **VCARB / Apex Strategy** | `#00f0ff` (Cyan) | **Player Car 1** |
| **#3** | **Daniel Ricciardo** | **VCARB / Apex Strategy** | `#ffaa00` (Amber) | **Player Car 2** |
| **#1** | Max Verstappen | Red Bull Racing | `#1e40af` | AI Competitor |
| **#11** | Sergio Pérez | Red Bull Racing | `#1e3a8a` | AI Competitor |
| **#4** | Lando Norris | McLaren | `#ff8000` | AI Competitor |
| **#81** | Oscar Piastri | McLaren | `#f97316` | AI Competitor |
| **#16** | Charles Leclerc | Ferrari | `#dc2626` | AI Competitor |
| **#55** | Carlos Sainz | Ferrari | `#b91c1c` | AI Competitor |
| **#63** | George Russell | Mercedes-AMG | `#00a19c` | AI Competitor |
| **#44** | Lewis Hamilton | Mercedes-AMG | `#059669` | AI Competitor |
| **#14** | Fernando Alonso | Aston Martin | `#00594f` | AI Competitor |
| **#18** | Lance Stroll | Aston Martin | `#047857` | AI Competitor |
| **#10** | Pierre Gasly | Alpine | `#2563eb` | AI Competitor |
| **#31** | Esteban Ocon | Alpine | `#3b82f6` | AI Competitor |
| **#23** | Alexander Albon | Williams | `#0082fa` | AI Competitor |
| **#2** | Logan Sargeant | Williams | `#38bdf8` | AI Competitor |
| **#27** | Nico Hülkenberg | Haas | `#e11d48` | AI Competitor |
| **#20** | Kevin Magnussen | Haas | `#be123c` | AI Competitor |
| **#77** | Valtteri Bottas | Kick Sauber | `#52e252` | AI Competitor |
| **#24** | Zhou Guanyu | Kick Sauber | `#22c55e` | AI Competitor |

### 4. Functional Specifications

#### 4.1 Five-Light Gantry & Start Sequence Engine
* **Pre-Grid Phase:** All 20 cars populate their starting positions in an authentic staggered 2-by-2 grid layout at a standstill.
* **Sequence Trigger:** Strategic operator initiates the FIA start sequence.
* **Gantry Step Progression:**
  $$\text{Interval} = 1000\text{ ms per light } (L_1 \to L_2 \to L_3 \to L_4 \to L_5)$$
  Each red light step triggers a synchronized $520\text{ Hz}$ sine tone.
* **Randomized Abrupt Extinguishment:**
  $$\Delta t_{\text{hold}} \in [800\text{ ms}, 2400\text{ ms}]$$
  Following $L_5$, lights hold for a randomized interval before extinguishing simultaneously ("Lights Out!").
* **Launch Physics:** Launch reaction dispersion ($\pm 0.0020$ lap delta) is assigned across all 20 cars. Launch tone ($1180\text{ Hz}$) and engine rev acoustics trigger.

#### 4.2 Dual-Car Strategy Consoles (Tsunoda & Ricciardo)
The strategist manages both drivers via split pit-wall telemetry decks:
* **Car #22 (Yuki Tsunoda):** Cyan accent theme (`#00f0ff`).
* **Car #3 (Daniel Ricciardo):** Amber accent theme (`#ffaa00`).

Each console features:
1. **Pace Mode Selectors:**
   * `SAVE`: $-15\%$ fuel burn, $-35\%$ tire thermal stress, $+0.85\text{s}$ lap delta penalty.
   * `BAL`: Nominal baseline ($1.0\times$ burn, $1.0\times$ thermals, $0.0\text{s}$ delta).
   * `PUSH`: $+15\%$ fuel burn, $+40\%$ tire thermal stress, $-0.65\text{s}$ lap delta bonus.
   * `QUALI`: $+35\%$ fuel burn, $+85\%$ tire thermal stress, $-1.10\text{s}$ lap delta bonus.
2. **ERS Overtake Toggle:** Deploys stored battery energy ($-0.75\text{s}$ lap time) discharging at $4.8\%/\text{s}$, recharging at $1.6\%/\text{s}$ when inactive.
3. **Four-Corner Tire Thermals ($T_{\text{FL}}, T_{\text{FR}}, T_{\text{RL}}, T_{\text{RR}}$):** Modeled dynamically against ambient track surface temperatures, aggressive slip angles, and weather wetness.
4. **Pit Stop Calling:** Individual `BOX THIS LAP` toggles and compound selectors (`SOFT`, `MEDIUM`, `HARD`, `INTER`, `WET`).

#### 4.3 Pit Lane Operations & Double-Stack Queuing
* **Nominal Stationary Time:** $2.6\text{ seconds}$ baseline tire change.
* **Pit Speed Limiter:** Velocity restricted to $22.2\text{ m/s}$ ($80\text{ km/h}$) along the pit trajectory ($\sim 22.5\text{s}$ pit loss).
* **Double-Stack Condition:** If both Tsunoda (#22) and Ricciardo (#3) box on the same lap within $< 4.5\text{ seconds}$ of each other, the trailing car enters `DOUBLE STACK WAIT`, waiting stationary in the queue until the lead car is dropped and released.

### 5. Mathematical & Physics Models

#### 5.1 Lap Pace Calculation
For any car $i$ on tick $t$:
$$T_{\text{lap}} = T_{\text{base}} + \Delta T_{\text{compound}} + \Delta T_{\text{fuel}} + \Delta T_{\text{wear}} + \Delta T_{\text{weather}} + \Delta T_{\text{mode}} - (S_i - 1.0) \cdot 1.8$$

Where:
* $T_{\text{base}} = 87.50\text{ s}$ (Silverstone baseline).
* $\Delta T_{\text{fuel}} = m_{\text{fuel}} \cdot 0.030\text{ s/kg}$.
* $\Delta T_{\text{wear}} = \begin{cases} (100 - W) \cdot 0.022, & \text{if } A \le C_{\text{lap}} \\ 0.4 \cdot (A - C_{\text{lap}})^{1.4}, & \text{if } A > C_{\text{lap}} \text{ (Tire Cliff)} \end{cases}$
* $S_i$ is individual driver skill coefficient ($\approx 0.992 - 1.032$).

#### 5.2 Tire Compound Matrix
| Compound | Code | Base Delta ($\Delta T$) | Deg Rate / Lap | Cliff Lap ($C_{\text{lap}}$) | Optimal Wetness |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Soft** | `S` (Red) | $-0.75\text{s}$ | $2.70\%$ | 12 | $0\% - 15\%$ |
| **Medium** | `M` (Yellow) | $\pm 0.00\text{s}$ | $1.45\%$ | 22 | $0\% - 20\%$ |
| **Hard** | `H` (White) | $+0.65\text{s}$ | $0.82\%$ | 32 | $0\% - 22\%$ |
| **Intermediate** | `I` (Green) | $+3.50\text{s}$ | $1.25\%$ | 24 | $18\% - 65\%$ |
| **Full Wet** | `W` (Blue) | $+6.80\text{s}$ | $1.10\%$ | 28 | $55\% - 100\%$ |

### 6. Procedural Audio Synthesis
All audio is synthesized client-side via Web Audio API:
* **Gantry Lights:** $520\text{ Hz}$ Sine tone ($180\text{ ms}$).
* **Lights Out:** $1180\text{ Hz}$ Triangle wave launch chime ($450\text{ ms}$).
* **Throttle Burst:** $95\text{ Hz} \to 260\text{ Hz}$ Sawtooth sweep.
* **Team Radio:** Two-tone step chime ($920\text{ Hz} \to 1840\text{ Hz}$).
* **Warning Tone:** $460\text{ Hz} \to 320\text{ Hz}$ Sawtooth alarm.
* **Pit Limiter:** $210\text{ Hz}$ Square wave pulse.

### 7. Competitor AI Logic
The 18 AI competitors make real-time strategic calls based on:
* **Tire Wear & Cliff:** Triggers pit request if health $< 28\%$ or age $> C_{\text{lap}} + 2$.
* **Dynamic Rain Crossover:** Pits for `INTERMEDIATE` or `WET` when track surface moisture exceeds optimal slick thresholds.
* **Safety Car / VSC Opportunism:** $35\%$ pit loss discount under SC conditions triggers opportunistic pit stops for aging compounds ($> 10$ laps old).