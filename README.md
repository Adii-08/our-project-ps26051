# ThermaSetu: Area-Specific Thermal Shelter Optimizer

**ThermaSetu** is a physics-based tactical shelter screening and microclimate digital twin engine developed for **DRDO Problem Statement SIH26051** (*Software Based Model Development for Design of Area Specific Shelter for Thermal Comfort Maintenance*).

The software integrates classical chemical engineering transport phenomena (transient conduction, wind-coupled external convection, and sol-air radiation), high-altitude tropospheric barometric lapse adjustments, apparent heat capacity Phase Change Material (PCM) latent buffering, and psychrometric interstitial frost diagnostics into a decoupled web architecture.

---

## Key Capabilities

* **Barometric Air Density Correction:** Dynamically calculates localized air density $\rho_{air}(z)$ via the barometric lapse formula (e.g., $0.70\text{ kg/m}^3$ at Siachen vs. $1.225\text{ kg/m}^3$ at sea level) to avoid overestimating ventilation and infiltration heat losses by 35–45%.
* **Latent Thermal Mass Storage (PCM):** Uses the Apparent Heat Capacity method to capture daytime solar absorption and nocturnal latent discharge around a $21^\circ\text{C}$ phase transition band.
* **Wind-Coupled Forced Convection:** Dynamically computes exterior boundary layer film coefficients as a function of local gale wind velocity ($h_{out} = 5.7 + 3.8 v_{wind}$).
* **Psychrometric Interstitial Frost Diagnostics:** Applies the Magnus-Tetens relationship to evaluate occupant respiration dew point ($T_{dp}$) against interior wall boundary temperatures ($T_{w,in}$), flagging freeze-thaw degradation risks.
* **Multi-Objective Pareto Optimization:** Iteratively analyzes insulation thickness, core substrate selection (Aerogel, PUF, VIP), and PCM mass to balance comfort compliance against structural payload limits.
* **Empirical Validation & Numerical Auditing:** Includes a measured-data CSV evaluation workflow ($R^2$, MAE, RMSE) and a step-halving numerical timestep convergence test ($\Delta t \rightarrow \Delta t/2 \rightarrow \Delta t/4$).

---

## Core Governing Equations

### 1. Dynamic Energy Conservation

The room air node temperature ($T_{in}$) is integrated over discrete time intervals $\Delta t$:

$$(C_{air} + C_{envelope} + m_{pcm}C_{app})\frac{dT_{in}}{dt} = \dot{Q}_{solar}(t) + \dot{Q}_{occ} + \dot{Q}_{aux}(t) - \dot{Q}_{cond}(t) - \dot{Q}_{vent}(t) - \dot{Q}_{rad}(t)$$

### 2. Apparent Heat Capacity Formulation (PCM Latent Spike)

To avoid tracking a discontinuous moving phase boundary, latent heat of fusion ($L_f$) is modeled as a temperature-dependent Gaussian enthalpy spike:

$$C_{app}(T) = C_{p,sensible} + \left[ \frac{L_f}{\sqrt{2\pi}\sigma} \right] \exp\left( -\frac{(T - T_m)^2}{2\sigma^2} \right)$$

* $T_m$: Melting plateau ($21.0^\circ\text{C}$)
* $L_f$: Latent heat of fusion ($190{,}000\text{ J/kg}$ for paraffin wax RT21HC)
* $\sigma$: Transition half-spread ($0.75^\circ\text{C} - 0.80^\circ\text{C}$)

### 3. Tropospheric Barometric Lapse Air Density

At high altitudes, air density drops significantly with elevation $z$ (meters):

$$\rho_{air}(z) = \frac{P_0}{R_{spec}(T_{amb} + 273.15)} \left(1 - \frac{0.0065 z}{288.15}\right)^{5.255}$$

Infiltration and ventilation load:

$$\dot{Q}_{vent}(t) = \left(\frac{\text{ACH} \cdot V_{room}}{3600}\right) \rho_{air}(z) C_{p,air} (T_{in}(t) - T_{amb}(t))$$

### 4. Overall Composite Transmittance ($U$-Value)

For multi-layer envelopes (exterior cladding, insulation core, and PCM liner):

$$R_{total} = \frac{1}{h_{out} A} + \sum_{j=1}^{n}\frac{L_j}{k_j A} + \frac{1}{h_{in} A}$$

$$U = \frac{1}{R_{total} A} \quad [\text{W/m}^2\cdot\text{K}]$$

### 5. Magnus-Tetens Psychrometric Dew Point & Frost Criterion

Occupants release water vapor at $\approx 50\text{ g/h/soldier}$, raising indoor relative humidity ($RH$). The dew point is calculated via:

$$\gamma(T_{in}, RH) = \frac{17.625 \cdot T_{in}}{243.04 + T_{in}} + \ln\left(\frac{RH}{100}\right), \quad T_{dp} = \frac{243.04 \cdot \gamma}{17.625 - \gamma}$$

The inner wall surface boundary temperature:

$$T_{w,in} = T_{in} - \frac{U_{wall}(T_{in} - T_{amb})}{h_{in}}$$

$$\text{Frost Risk Flag} =  \begin{cases}  \text{CRITICAL HAZARD}, & \text{if } T_{w,in} \le T_{dp} \text{ and } T_{w,in} < 0^\circ\text{C} \\  \text{SAFE (DRY)}, & \text{if } T_{w,in} > T_{dp}  \end{cases}$$

### 6. Tactical Fuel & Carbon Offset

Savings relative to a baseline military canvas tent ($U = 3.2\text{ W/m}^2\cdot\text{K}$, $1.2\text{ ACH}$):

$$\text{Diesel Saved (L/day)} = \frac{\int_{0}^{24\text{ h}} \max\left(0, \dot{Q}_{deficit,tent}(t) - \dot{Q}_{deficit,shelter}(t)\right) dt}{\text{LHV}_{diesel} \cdot \rho_{diesel} \cdot \eta_{heater}}$$

$$\text{CO}_2 \text{ Offset (kg/day)} = \text{Diesel Saved (L)} \times 2.68\text{ kg CO}_2/\text{L}$$

---

## Project Structure

```text
thermasetu/
├── backend/
│   ├── main.py              # FastAPI application & NumPy ODE thermal solver
│   ├── requirements.txt     # Python dependencies
│   └── test_solver.py       # Numerical verification unit tests
├── frontend/
│   └── index.html           # Standalone dashboard UI (HTML5, Canvas, CSS variables)
├── data/
│   ├── sample_climate.csv   # Field sample microclimate data (hour, temp, solar, wind, rh)
│   └── sample_measured.csv  # Field sensor validation dataset (hour, measuredIndoorTemp)
├── docs/
│   └── DRDO_SIH26051_Spec.pdf
└── README.md

```

---

## Getting Started

### Prerequisites

* Python 3.10 or higher
* Modern web browser (Chrome, Edge, Firefox, Safari)

### 1. Backend Setup (FastAPI Microservice)

Clone the repository and enter the backend directory:

```bash
git clone https://github.com/your-username/thermasetu.git
cd thermasetu/backend

```

Create and activate a virtual environment:

```bash
# Linux / macOS
python3 -m venv venv
source venv/bin/activate

# Windows
python -m venv venv
venv\Scripts\activate

```

Install dependencies:

```bash
pip install -r requirements.txt

```

Start the simulation API server:

```bash
uvicorn main:app --reload --host 0.0.0.0 --port 8000

```

The API interactive documentation will be accessible at `http://localhost:8000/docs`.

### 2. Frontend Setup

The dashboard is self-contained in `frontend/index.html`. You can open it directly in a web browser, or serve it via Python:

```bash
cd ../frontend
python3 -m http.server 3000

```

Open `http://localhost:3000` in your browser.

> **Offline Architecture:** The frontend automatically checks for the FastAPI backend at `http://localhost:8000/api/simulate`. If the backend is unreachable, it seamlessly switches to its built-in client-side JavaScript numerical engine.

---

## API Reference

### `POST /api/simulate`

Computes the 24-hour dynamic heat balance, comfort compliance, and logistics metrics.

#### Request Body (JSON)

```json
{
  "altitude": 3500.0,
  "latitude": 34.15,
  "doy": 15,
  "tmean": -12.0,
  "tamp": 8.5,
  "solar": 850.0,
  "wind": 8.0,
  "rh": 35.0,
  "length": 6.0,
  "width": 4.0,
  "height": 2.6,
  "win_area": 2.8,
  "door_area": 1.8,
  "win_u": 2.4,
  "azimuth": 180.0,
  "tilt": 15.0,
  "occupants": 4,
  "ach": 0.5,
  "start_temp": 10.0,
  "target_temp": 21.0,
  "tolerance": 3.0,
  "mode": "heat",
  "setpoint": 18.5,
  "equip_kw": 4.0,
  "efficiency": 0.75,
  "dt_minutes": 5.0,
  "wall_layers": [
    { "name": "Wood", "mm": 15.0, "k": 0.13, "rho": 550.0, "cp": 1600.0, "isPCM": false },
    { "name": "PUF", "mm": 60.0, "k": 0.022, "rho": 42.0, "cp": 1400.0, "isPCM": false },
    { "name": "PCM_RT21", "mm": 15.0, "k": 0.20, "rho": 880.0, "cp": 2000.0, "isPCM": true, "Lf": 190000.0, "Tm": 21.0, "sigma": 0.8 }
  ],
  "roof_layers": [
    { "name": "Aerogel", "mm": 40.0, "k": 0.015, "rho": 110.0, "cp": 1000.0, "isPCM": false }
  ],
  "floor_layers": [
    { "name": "PUF", "mm": 60.0, "k": 0.022, "rho": 42.0, "cp": 1400.0, "isPCM": false }
  ]
}

```

#### Response (JSON Summary)

```json
{
  "u_avg": 0.312,
  "rho_air": 0.824,
  "comfort_compliance": 94.2,
  "avg_temp": 20.4,
  "min_temp": 18.6,
  "max_temp": 22.8,
  "diesel_saved_liters": 19.4,
  "co2_offset_kg": 52.1,
  "t_dew_point": 5.8,
  "t_wall_inner": 16.4,
  "is_frost_risk": false,
  "total_mass": 2840.0,
  "totals_kwh": {
    "solar": 42.5,
    "conduction": 21.2,
    "ventilation": 8.4,
    "radiation": 9.1,
    "heat": 0.8
  },
  "records": [ ... ]
}

```

---

## Verification & Validation

| Verification Scheme | Method | Standard Applied |
| --- | --- | --- |
| **Numerical Convergence** | Timestep step-halving ($\Delta t = 20, 10, 5, 2.5, 1.25\text{ min}$) | Cauchy Criterion ($\Vert{}T_{\Delta t} - T_{\Delta t/2}\Vert{} < 0.1^\circ\text{C}$) |
| **Field Sensor Agreement** | Pearson correlation and statistical error vs. uploaded CSV | $R^2 \ge 0.85$, $\text{MAE} \le 1.8^\circ\text{C}$, $\text{RMSE} \le 2.2^\circ\text{C}$ |
| **Psychrometry & Condensation** | Magnus-Tetens boundary tracking | CEN EN ISO 13788 Glaser method equivalent |
| **Material Physics** | Knudsen diffusion & apparent heat capacity | ASHRAE Fundamentals / EnergyPlus Algorithms |

---

## Contributing

1. Fork the repository.
2. Create a feature branch (`git checkout -b feature/rk4-solver`).
3. Commit changes (`git commit -m 'Add 4th-order Runge-Kutta numerical solver'`).
4. Push to the branch (`git push origin feature/rk4-solver`).
5. Open a Pull Request.

---
