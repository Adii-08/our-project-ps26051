# ThermaSetu (थर्मसेतु)
### High-Altitude Thermal Shelter Modeling & Optimization Tool
**Built for DRDO Problem Statement · Smart India Hackathon (SIH26051)**

When soldiers are deployed at forward posts like Siachen Base Camp, Kargil, or Eastern Ladakh, winter temperatures consistently stay between -20°C and -40°C. A surprising number of these outposts still rely on standard canvas tents paired with traditional kerosene Bukhari stoves.

The practical outcome of that setup is tough on troops and logistics:
- Each tent burns through 20 to 25 liters of kerosene every single day just to keep interior temperatures slightly above freezing.
- The indoor temperature gradient is severe: a soldier's head might be at +18°C while their feet near the ground sit at -2°C.
- Breath and stove moisture hit the freezing inner fabric at night, turn to rime frost, and melt into cold drips onto sleeping bags the moment the morning heater kicks on.
- Sealing the shelter to trap warmth routinely leads to dangerous carbon monoxide and carbon dioxide build-up in thin, oxygen-poor air.

To design something better, engineers usually have to fire up heavy 3D CFD suites like ANSYS Fluent or OpenFOAM. But setting up meshes and waiting 6 to 12 hours for a single case to converge is unhelpful when a field team needs to test twenty different insulation layups before dinner.

We built **ThermaSetu** to fix this tradeoff. It runs a transient 1D numerical heat transfer engine right inside your web browser. You change an insulation layer or alter the roof pitch, and you get immediate, data-backed operational figures: interior thermal curves, fuel burn, air safety boundaries, and condensation risks—all calculated in milliseconds.

---

## What the Software Actually Does

### 1. 5-Node 1D Transient Heat Solver (Implicit TDMA)
Most quick calculators just take an assembly's static U-value and multiply it by a temperature difference. That completely ignores how thermal mass delays and dampens cold snaps overnight.

Instead, we slice the exterior wall into five distinct physical nodes:
1. x0: The exterior boundary exposed to wind and ambient air
2. x1: The outer cladding layer
3. x2: The primary insulation core (Aerogel, PUF, or VIP)
4. x3: The phase-change material (PCM) buffer layer
5. x4: The interior wall surface facing the soldiers

Rather than using an explicit forward-Euler step (which quickly blows up into NaN values unless your time step is tiny), we implemented a fully implicit Crank-Nicolson formulation solved with the **Thomas Algorithm (Tridiagonal Matrix Algorithm - TDMA)**. The solver is unconditionally stable, meaning you can jump between 30-second steps and 30-minute steps without the math diverging.

### 2. Realistic Phase Change Material (PCM) Buffer Modeling
Most theoretical setups assume a PCM melts and freezes at an exact single-degree temperature point. In real life, especially with composite salts and paraffin waxes, phase transitions happen gradually across a temperature window.

We model the apparent heat capacity (C_app) of the PCM layer using a Gaussian distribution centered around its melting point (Tm):
C_app(T) = Cp + [ Lf / (sigma * sqrt(2*pi)) ] * exp( -(T - Tm)^2 / (2 * sigma^2) )

This lets us test whether a PCM like RT21 will actually complete its daily freeze-thaw cycle in a specific climate, or if it will simply freeze solid on day one and act as dead structural weight.

### 3. Live Geospatial Weather via Open-Meteo
You don't have to guess solar radiation, ambient temperatures, or wind patterns. Clicking **"Fetch Live Weather"** queries the Open-Meteo API using the coordinates of the selected deployment sector (Ladakh, Siachen, Kargil, Tawang, or Thar). The application downloads real 24-hour diurnal curves, solar irradiance, relative humidity, and wind speeds, feeding them directly into the solver.

### 4. Altitude Corrections for Thin Air
Air density drops from ~1.225 kg/m^3 at sea level down to roughly 0.70 kg/m^3 at an elevation of 5,000 meters. If you calculate convective draft losses or ventilation heat loss using standard sea-level numbers, your calculations overestimate heat loss by roughly 40%. ThermaSetu recalculates barometric pressure and localized air density using the tropospheric barometric lapse formula:
rho_air(z) = [ P0 / (R_spec * T_K) ] * (1 - 0.0065 * z / 288.15)^5.255

### 5. Life-Support Air Exchange & Flue Safety Checks
Sealing a military shelter tight stops heat loss, but if you have four soldiers breathing and an active Bukhari heater burning fuel in an enclosed space, oxygen levels fall while CO and CO2 accumulate rapidly.

The software calculates the minimum required Air Changes per Hour (ACH_min) based on soldier respiration rates and heater combustion requirements at high altitude. If your configured ventilation is set lower than what is safe, the dashboard flags a warning and lets you auto-correct the ventilation rate with a single click.

### 6. ISO 7730 Fanger PMV / PPD Ergonomics
Air temperature alone doesn't tell you if someone is comfortable. A room with 20°C air will still feel freezing if the surrounding walls are at -5°C because your body radiates heat straight into them.

The software computes:
- **Mean Radiant Temperature (T_mrt)** based on all surface temperatures.
- **Operative Temperature (T_op)** combining radiative and convective factors.
- **Fanger PMV (Predicted Mean Vote)** and **PPD (Percentage of People Dissatisfied)** configured for soldiers wearing 2.5 clo heavy winter combat gear under light metabolic activity (1.2 met).
- **Head-to-Ankle Vertical Temperature Gradient**, ensuring floor drafts don't violate ISO standards.

### 7. Multi-Objective Pareto Screening
The optimizer tab systematically evaluates combinations of Aerogel, PUF, and Vacuum Insulation Panels (VIP) along with varying thicknesses of phase-change material. It scores and ranks every viable setup against your chosen priorities: thermal comfort, daily kerosene saved, and total assembly weight.

### 8. Field Sensor Validation
If you have an experimental shelter or a test cubicle with temperature loggers installed, you can upload your CSV data directly (`hour,measuredIndoorTemp`). The engine plots your recorded sensor readings alongside the simulated curve and calculates the Mean Absolute Error (MAE), Root Mean Square Error (RMSE), and correlation.

### 9. Integrated Tactical Copilot
A floating assistant widget reads the current state of your simulation parameters directly from memory. You can ask specific questions like *"Why isn't my PCM charging in Siachen?"* or *"Is an infiltration rate of 0.3 ACH safe for 4 troops?"* and receive practical explanations tied directly to your inputs.

---
