# Design Spec: Live MUF & Grayline Propagation Map Widget

This document outlines the technical design for a live-updating Day/Night (Grayline) world map overlay with Maximum Usable Frequency (MUF) thermal propagation grids, designed for amateur radio callsign **AC3GX**.

## 🚀 Objectives
- Provide a high-fidelity visual propagation tool displaying the real-time location of the solar terminator (Grayline).
- Display a live-updating thermal heatmap of estimated Maximum Usable Frequencies (MUF) scaling with current NOAA SFI (Solar Flux Index) and K-index telemetry.
- Maintain **100% self-contained offline capability** with zero external image asset dependencies.

## 🛠️ Architecture & Components

### 1. HTML Canvas Component
An HTML5 canvas element (`#prop-map-canvas`) placed inside a resizable card panel within the Right Column of the console.
- **Aspect Ratio:** 2:1 (standard equirectangular projection mapping -180 to +180 longitude and +90 to -90 latitude).
- **Default dimensions:** 100% width of parent column, with custom heights.

### 2. Zero-Asset Vector Continent Mapping
Continents are defined as polygon nodes in longitude/latitude coordinate degrees:
- **Eurasia:** `[[-5,36],[30,30],[35,12],[45,15],[60,25],[77,8],[98,10],[108,18],[120,38],[140,35],[170,66],[100,75],[60,70],[30,60],[20,40]]`
- **Africa:** `[[-17,32],[32,31],[34,27],[51,11],[46,-10],[40,-25],[33,-34],[18,-34],[10,-5],[-14,14]]`
- **North America:** `[[-168,65],[-120,69],[-80,70],[-60,60],[-55,48],[-80,25],[-99,15],[-105,20],[-115,30],[-125,48],[-160,58]]`
- **South America:** `[[-80,10],[-50,-5],[-35,-7],[-40,-20],[-65,-45],[-75,-55],[-70,-35],[-75,-20],[-81,-5]]`
- **Australia:** `[[113,-22],[135,-12],[142,-10],[153,-28],[148,-38],[115,-34],[113,-26]]`
- **Greenland:** `[[-50,60],[-40,60],[-30,70],[-40,80],[-60,80],[-55,70]]`

### 3. Solar Astronomy Math
To trace the day/night boundary, the dashboard calculates the solar declination ($\delta$) and the subsolar longitude ($\lambda_0$) based on the current UTC date and time:
- **Solar Declination ($\delta$):**
  $$\theta = \frac{2\pi}{365} \cdot (\text{Day of Year} - 1)$$
  $$\delta \approx 0.409 \cdot \sin\left(\frac{2\pi}{365} \cdot (\text{Day of Year} - 80)\right)$$
- **Subsolar Longitude ($\lambda_0$):**
  $$\lambda_0 = -15 \cdot \left(\text{UTC Hour} + \frac{\text{UTC Minute}}{60} + \frac{\text{UTC Second}}{3600}\right)$$
- **Solar Zenith Angle ($\chi$)** for any latitude ($\phi$) and longitude ($\lambda$):
  $$\cos(\chi) = \sin(\phi)\sin(\delta) + \cos(\phi)\cos(\delta)\cos(\lambda - \lambda_0)$$
  - Night zone: $\cos(\dots) < 0$.
  - Grayline boundary: $\cos(\dots) \approx 0$.

### 4. MUF Heatmap Formulation
The Maximum Usable Frequency (MUF) represents the highest frequency supporting skywave transmission:
- **Dynamic Peak Scaling:** $\text{MUF}_{\text{day}} = 10 + (\text{NOAA\_SFI} \cdot 0.18)$ (MHz).
- **Nighttime Floor:** $\text{MUF}_{\text{night}} = 6 + (\text{NOAA\_SFI} \cdot 0.03)$ (MHz).
- **Geomagnetic Penalty:** If K-index $\ge 4$, scale the final MUF by $(1 - 0.08 \cdot (K - 3))$ to simulate storm degradation.
- **Pixel Grid Interpolation:** Render MUF values in a 5° latitude/longitude grid, blending colors using a screen blending composite operation.

### 5. Design Mockup (Thermal Colors)
- **Purple/Indigo:** $< 10 \text{ MHz}$ (Only lower HF bands like 80m/40m open).
- **Green:** $10 \text{ to } 18 \text{ MHz}$ (20m band wide open).
- **Amber/Orange:** $18 \text{ to } 28 \text{ MHz}$ (15m band open).
- **Red/Magenta:** $> 28 \text{ MHz}$ (10m band open).

## 🧪 Verification Plan
- Verify math equations produce reasonable day/night shadows matching actual commercial grayline maps.
- Ensure NOAA telemetry changes update the canvas real-time scale dynamically.
- Check canvas bounds responsiveness across desktop and mobile screens.
