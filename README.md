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

See `Technical Report.pdf` for the full technical write-up and diagrams.
---

## Requirements

- **MATLAB R2020a+** with **Simulink**.  
- No Simscape required; standard Simulink + MATLAB Function blocks are used.  
- OS: Windows / macOS / Linux.

*(Optional)* For the comfort/vibrations demo, basic Control/Signal Processing functions are used within MATLAB; no special toolbox dependency is assumed.

---

## How to run

### GUI
1. Open MATLAB → Simulink.
2. Open  `URBANWAY.mat` (for bus dynamic sim) or `METRO.mat` (for metro dynamic sym) to load data on the workspace
4. Click **Run** in Simulink (Setting sim duration 1600 s for bus and 1200 s for metro).

Outputs: scope plots or logged signals for Space/Speed/Acceleration, Tractive & Braking Effort, Resistances (grade/curve/drag), Energy & SOC (bus), Adhesion check, Lateral dynamics (metro); sample figures are in the report.

## Modeling notes

- Bus dynamic selection: min{motor limit, comfort (a≈0.8 m/s²), adhesion} → operation mode via state chart (traction/free/brake/stop).
- HEV control (bus): TCS toggles generator based on SOC bounds; ICE off < 20 km/h; regen ~30% during braking.
- Metro speed limits from cant with linear ramp 3 mm/m; adhesion speed dependence; resistances per rail empirical formulas.
- Lateral dynamics (metro) computes non‑compensated acceleration, roll angle, and lateral offset given curvature, cant, and speed.


## Validation (examples from report)

- Bus: simulated speed/space/acceleration profiles compared to real on-board sensor data on the same route.
- Metro: simulated speed, energy, adhesion margin, and lateral response along M4 reference path.

## License
Released under the MIT License. See LICENSE.
