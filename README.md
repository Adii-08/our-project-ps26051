# ThermaSetu (थर्मसेतु)
### High-Altitude Thermal Shelter Modeler & Optimizer
**Built for DRDO Problem Statement · Smart India Hackathon (SIH26051)**

ThermaSetu is a lightweight, browser-based simulation tool we put together to design and test thermal shelters for extreme high-altitude border posts.

When troops are stationed at spots like the Siachen Base Camp or Eastern Ladakh, ambient temps regularly drop past -25°C. Right now, a lot of forward camps still rely on basic canvas tents and unvented Bukhari kerosene heaters. They end up burning 20+ liters of fuel a day per tent just to keep the inside barely livable. 

If you want to digitally test a better design, your standard option is running a full 3D CFD mesh in something like ANSYS or OpenFOAM. But waiting hours (or days) for a mesh to converge just to test a different insulation thickness isn't practical for rapid field engineering. ThermaSetu bridges this gap. It runs an implicit 1D finite-difference heat transfer model completely in the browser, giving you accurate temperature profiles and fuel estimates in a fraction of a second.

---

## What the tool actually does

* **5-Node Transient Heat Solver:** Instead of relying on crude steady-state R-values, we discretize the wall assembly into 5 nodes (exterior face, structural shell, core insulation, PCM buffer, and interior wall). The engine solves these simultaneously using an implicit Crank-Nicolson formulation and the Thomas Algorithm (TDMA). This means it’s unconditionally stable—it won't blow up with `NaN` errors even if you crank the time step up or down.
* **Phase Change Material (PCM) Buffer Modeling:** In the real world, PCMs don't just magically melt at a single sharp temperature. We modeled their apparent heat capacity ($C_{\text{app}}$) as a smooth Gaussian bell curve over their phase change range. This lets us see if a PCM layer will *actually* cycle and release latent heat, or if it's just going to sit there as dead weight during a sub-zero winter.
* **Live Weather via Open-Meteo:** No need to guess solar radiation or wind speeds. Just hit "Fetch Live Weather" and the tool pings the Open-Meteo API for the exact coordinates of Ladakh, Siachen, Tawang, etc., pulling hourly ambient temps, solar irradiance, and wind speeds.
* **Altitude-Adjusted Air Density:** Air density drops by nearly 40% at 5,000 meters. If you calculate ventilation heat loss using sea-level air density, your numbers will be garbage. The tool automatically corrects air properties using the standard barometric formula based on your chosen altitude.
* **Fresh Air & Bukhari Safety Checks:** Sealing a shelter tight saves heat, but if soldiers are burning kerosene indoors, CO and CO₂ levels can become lethal fast. We wrote an air exchange check that calculates the exact minimum ACH (Air Changes per Hour) needed to keep the air safe based on occupant respiration and stove draft.
* **ISO 7730 Comfort Index (PMV/PPD):** Just looking at air temperature doesn't tell you if a soldier is actually freezing. The model calculates Fanger's Predicted Mean Vote (PMV) by factoring in radiant exchange with cold interior walls, military winter clothing levels (2.5 clo), and metabolic rates.
* **Pareto Optimizer:** Basically a screening tool that loops through combinations of Aerogel, PUF, VIP, and PCM thicknesses to rank designs. It finds the setups that give the highest comfort for the lowest material weight and fuel cost.
* **Field Sensor Validation:** Got measured data from a physical prototype? You can drop a simple CSV (`hour,measuredIndoorTemp`) into the app to plot your real-world readings directly against our simulated curves and automatically calculate MAE and RMSE values.
* **Built-in Assistant:** A floating AI copilot widget you can ask questions about your current setup. For instance, "Why isn't my PCM freezing?" or "Is this ACH safe for 4 troops?"

---

## System Architecture (Yes, it's one file)

We intentionally built the entire application as a single, self-contained HTML file (`index.html`) using vanilla JavaScript and HTML5 Canvas. 

- **Zero Node.js / NPM dependencies**
- **No build steps (no Webpack, Vite, etc.)**
- **No bloated external UI frameworks**

Why? Because forward-deployed field engineers, military officers, or hackathon evaluators shouldn't have to `npm install` just to run a thermal model. You can literally save the `.html` file to a flash drive, open it on an offline field laptop in the middle of nowhere, and it works flawlessly.

```text
thermasetu/
├── index.html            # The whole app (UI, numerical solver, charts, assistant)
├── ThermaSetu logo.png   # Shelter emblem displayed in the header
└── README.md             # You are here
