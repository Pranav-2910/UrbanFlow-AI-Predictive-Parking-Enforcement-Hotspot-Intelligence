# 🚓 Bengaluru Parking Enforcement Intelligence (B.T.P. Operational Prototype)

An AI-driven, high-performance geospatial decision support system built to assist the **Bengaluru Traffic Police (B.T.P.)** in optimizing enforcement resource allocation. The prototype detects illegal parking hotspots, ranks them dynamically using multi-weight scoring algorithms, and simulates autonomous patrol dispatch routing in real time.

This project was developed for the **Flipkart Gridlock 2.0 Hackathon (Round 2)** to address urban congestion caused by unregulated street parking in Bengaluru.

---

## 📖 Key Vocabulary & Concepts

To understand the analytical models running under the hood of the dashboard, familiarize yourself with the following terms:

| Term | Definition | Operational Impact |
| :--- | :--- | :--- |
| **Violation Volume** | The raw density and frequency of traffic violations recorded in a given jurisdiction. | Helps identify high-frequency enforcement zones where fine collection and visible policing can be maximized. |
| **Congestion Proxy** | An index measuring the obstruction potential of a parking violation on traffic flow, based on road characteristics and transit density. | Prioritizes clearing violations that cause the highest downstream traffic delays rather than just high raw volumes. |
| **Chronicity** | The persistent, recurring nature of violations outside standard peak weekend cycles. | Highlights areas with chronic weekday violations (e.g., commercial zones with daily delivery blockages). |
| **Peak Overlap** | The percentage of violations occurring during core commute rush hours (08:00–11:00 AM and 05:00–08:00 PM). | Identifies spots that actively paralyze the road network during peak traffic hours. |
| **Road Narrowness** | A capacity multiplier calculated based on the road class (e.g., single-lane local streets vs. multi-lane arterials). | Upweights narrow roads because a single parked vehicle can block a larger percentage of the lane capacity. |
| **POI Proximity** | Proximity (within a 300m radius) to major public Points of Interest (Metro stations, transit hubs, commercial malls). | Identifies high-risk zones where illegal parking triggers cascading traffic bottlenecks. |
| **Clearance Delta** | The reduction in active violations achieved by dispatching a patrol to a hotspot. | Simulates the dynamic restoration of lane capacity post-enforcement. |

---

## 🧮 Mathematical Formulations & Scoring Modes

The dashboard operates in two primary modes, each utilizing distinct mathematical models to calculate priority ranks:

### 1. Violation Volume Mode
This mode ranks hotspots based on raw violation frequency, public reporting verification, and persistent temporal habits.

$$\text{Priority Score} = W_{\text{vol}} \cdot S_{\text{vol}} + W_{\text{app}} \cdot S_{\text{app}} + W_{\text{chr}} \cdot S_{\text{chr}}$$

Where:
* **$W_{\text{vol}}, W_{\text{app}}, W_{\text{chr}}$** are user-defined weights that auto-balance to sum to exactly $100\%$.
* **$S_{\text{vol}}$ (Volume Score)**: The normalized count of active violations in the area, capped at 100:
  $$S_{\text{vol}} = \min\left(100.0, \frac{\text{FilteredCount}}{100.0} \cdot 100.0\right)$$
* **$S_{\text{app}}$ (Approval Score)**: The verification rate of citizen-reported violations:
  $$S_{\text{app}} = \frac{\text{Approved Reports}}{\text{Approved Reports} + \text{Rejected Reports}} \cdot 100.0$$
* **$S_{\text{chr}}$ (Chronicity Score)**: The share of violations occurring on weekdays:
  $$S_{\text{chr}} = \frac{\text{Weekday Count}}{\text{Total Count}} \cdot 100.0$$

---

### 2. Congestion Proxy Mode
This mode evaluates the physical impact of illegal parking on urban mobility. It prioritizes clearing street bottlenecks that choke traffic flow.

$$\text{Congestion Score} = \min\left(100.0, \frac{\text{RawScore}}{0.820} \cdot 100.0\right)$$

Where the **$\text{RawScore}$** is calculated as:

$$\text{RawScore} = \text{Density}_{\text{norm}} \cdot \text{Narrowness} \cdot (1.0 + \text{PeakOverlap}) \cdot (1.0 + \text{POI}_{\text{score}})$$

* **$\text{Density}_{\text{norm}}$ (Normalized Density)**: Active violation count divided by the maximum historical city density ($2512$ violations):
  $$\text{Density}_{\text{norm}} = \frac{\text{Active Count}}{2512.0}$$
* **$\text{Narrowness}$**: Multiplier assigned based on road classification ($1.5$ for local/narrow roads, $1.2$ for secondary roads, $1.0$ for primary/arterial roads).
* **$\text{PeakOverlap}$**: The proportion of violations coinciding with commute rush hours:
  $$\text{PeakOverlap} = \frac{\text{Count}_{\text{rush}}}{\text{Count}_{\text{total}}}$$
* **$\text{POI}_{\text{score}}$**: Proximity score calculated based on the count of transit links and public hubs in a 300-meter radius (e.g., $+0.2$ per nearby Metro station).

---

## 💎 Core Interactive Features & Upgrades

The prototype features a rich, glassmorphic dark-theme user interface designed to maximize operational visibility:

1. **☀️ Light/Dark Mode Switcher**
   * Remaps background, text, card borders, and list styling on the fly.
   * Synchronizes Leaflet maps by swapping tile layers (CartoDB Dark Matter $\leftrightarrow$ CartoDB Voyager light tiles) and dynamically updates Chart.js axis/label text colors.
   * Contrast-stabilized toggle button and telemetry HUD prevent visual decay in either mode.

2. **⏳ Temporal Time-Lapse Slider**
   * Features a playback deck (Play, Pause, Speed: Slow/Normal/Fast) that filters and aggregates over $162,000$ historical violation records in under $15\text{ms}$ per frame.
   * Animates the visual "ebb and flow" of violations across the city heatmap hour-by-hour.

3. **🎚️ Interactive Formula Weight Tuner**
   * Normalizes weights automatically: dragging one weight slider scales the other two proportionally so their sum is always $100\%$.
   * Recalculates composite scores and re-orders the junction priority lists in real time.

4. **🚓 Live Patrol Dispatch Simulator**
   * Clicking `⚡ Dispatch Nearest Patrol` on any Leaflet map popup initiates an active response.
   * Computes the nearest Police Station from coordinate centers (`psCenters`) and draws a dashed routing path.
   * Animates a patrol vehicle along the route at 60fps using coordinate interpolation (`requestAnimationFrame`).
   * Displays a translucent telemetry HUD showing distance (km) and live ETA countdown (seconds).
   * Triggers a clearance pulse upon arrival, reducing the hotspot count gradually and updating dashboard statistics dynamically.

5. **🔮 Predictive Incident & Patrol Forecasting**
   * Computes the Top 10 Police Jurisdictions predicted to have the highest violation density for any selected hour/day.
   * Click-to-load triggers update the **Patrol Pre-Position Alert** panel immediately with target jurisdiction recommendations.

---

## ⚡ System Performance & Honest Disclosures

* **Model Performance**: The predictive forecasting models operate with a validation Mean Absolute Error (**MAE**) of **2.19 violations/hour** (representing a **4.15% error reduction** over baseline averages).
* **Nighttime Sync Anomaly**: We document a synchronization lag between **02:00 AM and 05:00 AM** where live reports experience delayed ingestion. The system mitigates this by applying historical smoothing averages during these hours.

---

## 🚀 Getting Started

The dashboard is structured as a standalone front-end web application that runs entirely in the browser using CDN-loaded dependencies.

### Prerequisites
* A modern web browser (Google Chrome, Microsoft Edge, or Mozilla Firefox).

### Running Locally
1. Clone the repository:
   ```bash
   git clone https://github.com/your-username/Bengaluru-Parking-Intelligence.git
   cd Bengaluru-Parking-Intelligence
   ```
2. Open `index.html` directly in your browser, or serve it using a local HTTP server:
   ```bash
   # Using python
   python -m http.server 8000
   ```
3. Navigate to `http://localhost:8000` in your web browser.

---

*Developed for the Bengaluru Traffic Police (B.T.P.) Resource Optimization Proposal.*
