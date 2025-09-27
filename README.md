# Urban Transit Simulations in Simulink — Bus Line 73 (Duomo–Linate) & Metro M4

This repository contains two Simulink models that simulate **vehicle dynamics, operations, and energy** for:
- **Bus Line 73 (Duomo → Linate)** — articulated hybrid-electric bus with series architecture (IVECO Urbanway 18).  
- **Metro Line 4 (M4, Milan)** — driverless metro train, including energy/adhesion and lateral dynamics on curved track.

The work includes **kinematic & dynamic blocks**, **resistances**, **braking logic**, **pathway information lookup**, and (for the bus) a **series HEV thermostat control strategy (TCS)** with battery model. For the metro, the model adds **track cant & speed limits**, **adhesion checks**, and **lateral dynamics** (non-compensated acceleration, roll angle, lateral displacement).  
Source: our course project report and slides.

---

## Key features

### Bus Line 73 (Duomo–Linate)
- **Route**: ~7.24 km, **19 stops**, **17 curves** (curvature via clothoids), slope ~0 (Δalt < 10 m).
- **Series hybrid**: diesel ICE → generator (≈200–210 kW) + **160 kW** traction motor, **11 kWh** Li‑ion battery; TCS keeps ICE at constant operating point between SOC bounds. 
- **Dynamic model**: selects tractive effort as min of **motor limit / comfort / adhesion**; computes **Pel, Eel**, and operation modes (traction, free-running, braking, stop). 
- **Kinematic block**: integrates **a → v → s**, dwell times at stops (e.g., 10–15 s).
- **Resistances**: aerodynamic (`ρ, Cd, A`) + rolling friction (**fv ≈ 0.014**).
- **Battery & SOC**: current from power balance (gen + motor + auxiliaries), SOC via coulomb counting; **regen ~30%** of mech. braking.
- *(Optional)* **Vibrations & comfort (ISO 2631)**: quarter-car model, Fourier road profile, weighted perception metrics.

### Metro M4
- **Route**: ~14.198 km, **21 stations**, **21 curves**, slope in ±5‰ band; cant up to **160 mm** with linear ramp (3 mm/m) for speed limits.
- **Dynamic model**: traction curve and operation modes; **energy** with traction/braking efficiencies and auxiliaries.
- **Adhesion**: speed‑dependent adhesion force with adherent mass ratio (motorized vs total wheelsets).
- **Resistances**: grade (sinθ), curve (von Rockl), drag (empirical rail formula for low speeds).
- **Lateral dynamics**: non-compensated acceleration (cant + curvature), roll angle θ, lateral offset y.
---

## Repository layout

Transit-Simulations-Simulink/
├── bus-line-73/        # Simulink model for Line 73 (series HEV, dynamics, kinematics, resistances, brake)
└── metro-m4/           # Simulink model for M4 (dynamics, energy/adhesion, resistances, lateral dynamics)

See `/docs/RelazioneDinamica.pdf` for the full technical write-up and diagrams.  [1](https://gruppofsitaliane-my.sharepoint.com/personal/961941_rfi_it/Documents/File%20chat%20di%20Microsoft%20Copilot/RelazioneDinamica.pdf)

---

## 🔧 Requirements

- **MATLAB R2020a+** with **Simulink**.  
- No Simscape required; standard Simulink + MATLAB Function blocks are used.  
- OS: Windows / macOS / Linux.

*(Optional)* For the comfort/vibrations demo, basic Control/Signal Processing functions are used within MATLAB; no special toolbox dependency is assumed.  [1](https://gruppofsitaliane-my.sharepoint.com/personal/961941_rfi_it/Documents/File%20chat%20di%20Microsoft%20Copilot/RelazioneDinamica.pdf)

---

## 📂 Data files (CSV)

Place these under each model’s `data/` folder:

### `bus-line-73/data/`
- `stops_bus73.csv`: two columns `[stop_id, s_m]` with cumulative distance of each stop (m).  [1](https://gruppofsitaliane-my.sharepoint.com/personal/961941_rfi_it/Documents/File%20chat%20di%20Microsoft%20Copilot/RelazioneDinamica.pdf)  
- `curves_bus73.csv`: columns `[start_m, end_m, radius_m, clothoid_m]` (e.g., clothoid = 10 m).  [1](https://gruppofsitaliane-my.sharepoint.com/personal/961941_rfi_it/Documents/File%20chat%20di%20Microsoft%20Copilot/RelazioneDinamica.pdf)

### `metro-m4/data/`
- `stops_m4.csv`: `[station_id, s_m]` (0 … 14198 m).  [1](https://gruppofsitaliane-my.sharepoint.com/personal/961941_rfi_it/Documents/File%20chat%20di%20Microsoft%20Copilot/RelazioneDinamica.pdf)  
- `curves_m4.csv`: `[start_m, end_m, radius_m, clothoid_m]` (e.g., 50 m).  [1](https://gruppofsitaliane-my.sharepoint.com/personal/961941_rfi_it/Documents/File%20chat%20di%20Microsoft%20Copilot/RelazioneDinamica.pdf)  
- `slope_profile.csv`: piecewise slope between stations (±5‰ if not available).  [1](https://gruppofsitaliane-my.sharepoint.com/personal/961941_rfi_it/Documents/File%20chat%20di%20Microsoft%20Copilot/RelazioneDinamica.pdf)  
- `cant_profile.csv`: cant (mm) along track; speed limits computed from `H[mm] = 11.798 * v[km/h]^2 / R[m]` ⇒ `v = 4.68 * sqrt(R)` (capped at 80 km/h).  [1](https://gruppofsitaliane-my.sharepoint.com/personal/961941_rfi_it/Documents/File%20chat%20di%20Microsoft%20Copilot/RelazioneDinamica.pdf)

> You can digitize positions/radii from OpenStreetMap/Overpass Turbo and measure with Google Earth Pro, as done in the report.  [1](https://gruppofsitaliane-my.sharepoint.com/personal/961941_rfi_it/Documents/File%20chat%20di%20Microsoft%20Copilot/RelazioneDinamica.pdf)

---

## How to run

### GUI
1. Open MATLAB → Simulink.
2. Open `bus-line-73/Bus_Line_73.slx` (or `metro-m4/Metro_M4.slx`).
3. Ensure `data/` CSVs are loaded (via **From Spreadsheet** blocks / init script).
4. Click **Run**.

### Command line (example)
```matlab
% Bus Line 73
addpath('bus-line-73','bus-line-73/matlab','bus-line-73/data');
load_bus73_data;              % (optional) script to load stops/curves into workspace
simOut = sim('bus-line-73/Bus_Line_73.slx','StopTime','1600');

% Metro M4
addpath('metro-m4','metro-m4/matlab','metro-m4/data');
load_m4_data;
simOut = sim('metro-m4/Metro_M4.slx','StopTime','1200');
```

Outputs: scope plots or logged signals for Space/Speed/Acceleration, Tractive & Braking Effort, Resistances (grade/curve/drag), Energy & SOC (bus), Adhesion check, Lateral dynamics (metro); sample figures are in the report.

## Modeling notes

Bus dynamic selection: min{motor limit, comfort (a≈0.8 m/s²), adhesion} → operation mode via state chart (traction/free/brake/stop).
HEV control (bus): TCS toggles generator based on SOC bounds; ICE off < 20 km/h; regen ~30% during braking.
Metro speed limits from cant with linear ramp 3 mm/m; adhesion speed dependence; resistances per rail empirical formulas.
Lateral dynamics (metro) computes non‑compensated acceleration, roll angle, and lateral offset given curvature, cant, and speed.


## Validation (examples from report)

Bus: simulated speed/space/acceleration profiles compared to real on-board sensor data on the same route.
Metro: simulated speed, energy, adhesion margin, and lateral response along M4 reference path.

## License
Released under the MIT License. See LICENSE.
