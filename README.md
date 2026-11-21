# CE 766: Watershed Management — Final Course Project

**Course:** CE 766 – Watershed Management (Fall 2026)  
**Instructor:** Prof. Riddhi Singh  
**Student:** Prabhakar Yadav  
**GitHub Repository:** [https://github.com/Prabhakar-Yadav/CE766_Project-](https://github.com/Prabhakar-Yadav/CE766_Project-)  
**Study Area:** Malaprabha Sub-Watershed, Upper Krishna Basin, Karnataka, India  
**Analysis Period:** 2010–2020 (11-Year Daily Hydrometeorological Forcing)

---

## 1. Executive Summary & Scope Refinement

This project evaluates the hydrological efficacy of four discrete watershed management intervention strategies against baseline conditions in the semi-arid Malaprabha Sub-Watershed (Karnataka, India) across the 2010–2020 decadal climate cycle. A continuous daily-step lumped conceptual water-balance model was developed and verified with exact mass-balance closure ($< 10^{-12}\text{ mm}$ error). The model couples USDA Soil Conservation Service Curve Number (SCS-CN) surface runoff generation, a root-zone soil-moisture bucket accounting model, and daily storage-routing reservoir dynamics.

### Proposal Feedback & Scope Alignment:
Following feedback on the initial project proposal from Prof. Riddhi Singh, the scope was formally refined:
- **Omitted:** Multi-objective genetic algorithm optimization (NSGA-II / `pymoo`) and Latin Hypercube Sampling (LHS) / Monte Carlo uncertainty propagation.
- **In-Depth Focus:** Physical rigor, continuous soil-moisture accounting, exact mass-balance verification, and an in-depth comparative evaluation of four discrete watershed intervention scenarios (Check Dams, Farm Ponds, Afforestation, and a Combined Portfolio) subjected to observed historical monsoon variability.

---

## 2. Study Area & Hydrometeorological Setting

- **Catchment:** Malaprabha Sub-Watershed, a major tributary sub-basin of the Upper Krishna River system in northern Karnataka.
- **Spatial Bounding Box:** Latitude 15.50°N to 16.15°N, Longitude 74.50°E to 75.85°E (catchment area $A \approx 2,500\text{ km}^2$; $1\text{ mm}$ catchment depth $\equiv 2.5\text{ MCM}$).
- **Soil & Land Cover:** Vertisols and Alfisols (ISRIC SoilGrids Hydrologic Soil Group C/D); predominantly rainfed agriculture, fallow croplands, and open scrubland (NRSC Bhuvan).
- **Climate Regime:** Semi-arid tropical monsoon regime characterized by intense southwest monsoon storms (July–September) followed by prolonged dry periods, severe inter-annual rainfall variability (Mean Annual Rainfall: $822.68\text{ mm/yr}$, CV: $23.3\%$), high evaporative demand (Mean Annual PET: $1731.67\text{ mm/yr}$), and intensive groundwater abstraction.

---

## 3. XLRM Decision-Analytical Framework

The study is structured using the XLRM framework for robust decision analysis:

| XLRM Element | Description in This Study | Quantitative Values / Specifications |
| :--- | :--- | :--- |
| **X (Exogenous Factors)** | Observed historical climate variability (2010–2020) from IMD gridded records | • Mean Annual Rainfall: $822.68\text{ mm/yr}$<br>• Driest Year (2016): $545.39\text{ mm/yr}$<br>• Wettest Year (2019): $1156.29\text{ mm/yr}$<br>• Inter-annual Std Dev: $191.73\text{ mm/yr}$ (CV: $23.3\%$) |
| **L (Policy Levers)** | 4 Discrete Watershed Interventions vs. Baseline | • **S0 (Baseline):** $CN = 78.0$, no storage<br>• **S1 (Check Dams):** $V_{max} = 15\text{ mm}$, $\alpha = 0.60$, $k_{seep} = 0.06\text{ d}^{-1}$<br>• **S2 (Farm Ponds):** $V_{max} = 10\text{ mm}$, $\alpha = 0.35$, $k_{seep} = 0.04\text{ d}^{-1}$<br>• **S3 (Afforestation):** $CN = 72.0$, no structures<br>• **S4 (Combined Portfolio):** $CN = 72.0 \rightarrow$ Ponds $\rightarrow$ Dams |
| **R (Relationships / Model)** | Continuous daily conceptual water-balance model | • SCS-CN surface runoff: $Q_t = (P_t - I_a)^2 / (P_t - I_a + S)$<br>• Soil-moisture balance: $SM_{max} = 140\text{ mm}$, $SM_{fc} = 105\text{ mm}$<br>• Drainage rate: $k_{perc} = 0.20\text{ d}^{-1}$<br>• Hargreaves-Samani (1985) PET equation<br>• Reservoir mass-balance routing with open-water evaporation |
| **M (Performance Measures)** | Hydrological efficacy and trade-off metrics | • Mean Annual Direct Surface Runoff Outflow ($mm/yr$, % reduction)<br>• Mean Annual Net Groundwater Recharge ($mm/yr$, % gain)<br>• Open-Water Evaporative Structural Loss ($mm/yr$)<br>• Drought-Year (2016) vs. Wet-Year (2019) Stress Performance |

---

## 4. Mathematical Model Formulation

### 4.1 Daily Meteorological Forcing & Potential Evapotranspiration
Daily potential evapotranspiration ($PET_t$, $mm/day$) is computed via the **Hargreaves-Samani (1985)** method:
```text
PET_t = 0.0023 * Ra * (T_mean + 17.8) * sqrt(T_max - T_min)
```
where extraterrestrial solar radiation $R_a$ (converted to equivalent $mm/day$ water depth) is computed analytically from the sub-watershed centroid latitude ($\phi = 15.825^\circ\text{N}$) and solar declination across day of year $J \in [1, 365]$ (Allen et al., 1998; FAO-56).

### 4.2 Surface Runoff & Root-Zone Soil Moisture Accounting
Direct surface runoff is computed using the SCS-CN formulation (USDA-SCS, 1972):
```text
S = (25400 / CN) - 254
Ia = 0.2 * S
Q_t = ((P_t - Ia)^2) / (P_t - Ia + S)   if P_t > Ia, else 0.0
F_t = P_t - Q_t
```
Root-zone soil moisture ($SM_t$) is tracked with field capacity $SM_{fc} = 0.75 \cdot SM_{max} = 105\text{ mm}$ and maximum capacity $SM_{max} = 140\text{ mm}$:
```text
SM_temp = SM_{t-1} + F_t
AET_t = min(SM_temp, PET_t * min(1.0, SM_temp / SM_fc))
SM_post = SM_temp - AET_t
R_sat = max(0.0, SM_post - SM_max)
R_drain = k_perc * max(0.0, min(SM_post, SM_max) - SM_fc)
R_nat_t = R_sat + R_drain
SM_t = SM_post - R_nat_t
```

### 4.3 Storage Routing Mass Balance (Check Dams & Farm Ponds)
For water-harvesting structures:
```text
Inflow_t = alpha * Q_in,t
V_temp = V_{t-1} + Inflow_t
Spill_t = max(0.0, V_temp - V_max)
V_avail = min(V_max, V_temp)
Evap_t = min(V_avail, f_evap * PET_t)
V_after_evap = V_avail - Evap_t
Seep_t = min(V_after_evap, k_seep * V_after_evap)
V_t = V_after_evap - Seep_t
Q_out,t = (1.0 - alpha) * Q_in,t + Spill_t
R_total,t = R_nat_t + Seep_t
```

---

## 5. Summary of Scenario Results

All metrics represent 11-year mean annual values (2010–2020) over the Malaprabha Sub-Watershed:

| Scenario | Composite CN | Capacity ($mm$) | Capture $\alpha$ | Runoff Outflow ($mm/yr$) | Runoff Reduction (%) | Net Recharge ($mm/yr$) | Recharge Increase (%) | Structure Evap ($mm/yr$) | 2016 Drought Runoff ($mm$) | 2016 Drought Recharge ($mm$) | 2019 Wet Runoff ($mm$) | 2019 Wet Recharge ($mm$) | Mass Balance Error ($mm$) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Baseline (S0)** | 78.0 | 0.0 | 0.00 | 19.59 | 0.0% | 127.82 | 0.0% | 0.00 | 2.28 | 15.24 | 54.71 | 386.02 | $0.00 \times 10^0$ |
| **Check Dams (S1)** | 78.0 | 15.0 | 0.60 | 8.02 | 59.1% | 133.16 | +4.2% | 6.22 | 0.91 | 15.31 | 23.91 | 404.27 | $-1.82 \times 10^{-12}$ |
| **Farm Ponds (S2)** | 78.0 | 10.0 | 0.35 | 12.73 | 35.0% | 129.02 | +0.9% | 5.65 | 1.48 | 15.24 | 35.56 | 391.58 | $0.00 \times 10^0$ |
| **Afforestation (S3)** | 72.0 | 0.0 | 0.00 | 8.46 | 56.8% | 135.84 | +6.3% | 0.00 | 0.09 | 15.70 | 25.89 | 413.39 | $-1.82 \times 10^{-12}$ |
| **Combined Portfolio (S4)** | 72.0 | 25.0 | 0.95 | 2.20 | 88.8% | 137.28 | +7.4% | 4.82 | 0.02 | 15.70 | 6.73 | 420.12 | $-1.82 \times 10^{-12}$ |

---

## 6. Key Hydrological Findings & Discussion

1. **Non-Additive Portfolio Interaction & Diminishing Returns:**  
   Individual implementation indicates recharge gains of $+5.35\text{ mm/yr}$ (Check Dams), $+1.21\text{ mm/yr}$ (Farm Ponds), and $+8.02\text{ mm/yr}$ (Afforestation), totaling $+14.58\text{ mm/yr}$ in isolation. However, when combined in Scenario 4, the net recharge gain is only $+9.46\text{ mm/yr}$ ($+7.4\%$). This occurs because upstream afforestation lowers runoff at the source by $56.8\%$, leaving less inflow for downstream farm ponds and check dams. Consequently, downstream structures fill less frequently and spill less, demonstrating diminishing marginal returns for structural interventions when coupled with vegetative restoration.

2. **Evaporative Penalty in Semi-Arid Climates:**  
   Open-water storage structures suffer significant evaporative losses ($6.22\text{ mm/yr}$ for Check Dams, $5.65\text{ mm/yr}$ for Farm Ponds). In the severe drought year of 2016 ($P = 545.39\text{ mm}$), farm ponds generated $0.00\text{ mm}$ of net recharge gain because the small captured runoff volume was fully evaporated before it could percolate.

3. **Buffer Dynamics in Wet Monsoon Years:**  
   During the extreme monsoon of 2019 ($P = 1156.29\text{ mm}$), baseline runoff reached $54.71\text{ mm}$. The Combined Portfolio reduced outflow to $6.73\text{ mm}$ (an $87.7\%$ peak reduction) while boosting recharge to $420.12\text{ mm/yr}$, demonstrating substantial flood mitigation and groundwater recharge potential during high-rainfall periods.

4. **Field Implementation Readiness & Data Requirements:**  
   These lumped modeling results provide catchment-scale planning insights but cannot be directly implemented in the field without:
   - **Streamflow Calibration:** Validating simulated runoff against daily discharge observations from Central Water Commission (CWC) or CAMELS-IND gauging stations.
   - **Groundwater Validation:** Calibrating recharge rates against Central Ground Water Board (CGWB) observation well water-table hydrographs.
   - **Spatial Siting:** High-resolution DEM topography (Cartosat/SRTM) and stream-order analysis to position structures at optimal geomorphological sites.
   - **Siltation & Economics:** Incorporating reservoir siltation dynamics, desiltation maintenance, and economic capital cost-benefit evaluations.

---

## 7. Deliverables & Repository Structure

```text
CE766_Project-/
├── CE766_project.ipynb                   # Fully executed master notebook (14 cells, zero errors)
├── Figure_1_Intervention_Comparison.png  # 2-panel visual: Mean annual runoff & recharge (300 DPI)
├── Figure_2_Interannual_Performance.png  # 2-panel visual: 2010-2020 inter-annual dynamics (300 DPI)
├── malaprabha_2010_2020_cleaned.csv      # Cleaned monthly hydrometeorological time series (132 months)
├── scenario_summary_results.csv          # Comprehensive metrics for all 5 scenarios
├── .gitignore                            # Excludes raw IMD binary grids (imd_data/) and cache files
└── README.md                             # Complete project documentation and submission report
```

---

## 8. Setup & Reproducibility Instructions

### Prerequisites:
- Python 3.9+ (tested on Python 3.13)
- Required packages: `imdlib`, `xarray`, `netCDF4`, `pandas`, `numpy`, `matplotlib`, `scipy`

### Installation & Execution:
```bash
# 1. Clone the repository
git clone https://github.com/Prabhakar-Yadav/CE766_Project-.git
cd CE766_Project-

# 2. Install dependencies
pip install imdlib xarray netCDF4 pandas numpy matplotlib scipy

# 3. Open and run the notebook
jupyter notebook CE766_project.ipynb
```
*Note: The notebook automatically downloads the official IMD gridded daily rainfall and temperature files on first run if they are not already cached in `imd_data/`.*

---

## 9. References & Data Portals

1. **Pai, D. S., Sridhar, L., Rajeevan, M., Sreejith, O. P., Satbhai, N. S., & Mukhopadhyay, B. (2014).** Development of a new high spatial resolution ($0.25^\circ \times 0.25^\circ$) long period (1901–2010) daily gridded rainfall data set over India. *MAUSAM*, 65(1), 1–18. [https://mausamjournal.imd.gov.in/index.php/MAUSAM/article/view/851](https://mausamjournal.imd.gov.in/index.php/MAUSAM/article/view/851)
2. **USDA-SCS (Soil Conservation Service). (1972).** *National Engineering Handbook, Section 4: Hydrology*. U.S. Department of Agriculture, Washington, D.C.
3. **Mangukiya, N. K., Kumar, K. B., Dey, P., Sharma, S., Bejagam, V., Mujumdar, P. P., & Sharma, A. (2025).** CAMELS-IND: hydrometeorological time series and catchment attributes for 228 catchments in Peninsular India. *Earth System Science Data*, 17(2), 461–491. [https://doi.org/10.5194/essd-17-461-2025](https://doi.org/10.5194/essd-17-461-2025)
4. **Hargreaves, G. H., & Samani, Z. A. (1985).** Reference crop evapotranspiration from temperature. *Applied Engineering in Agriculture*, 1(2), 96–99.
5. **Nandi, S., Patel, P., & Swain, S. (2024).** IMDLIB: An open-source library for retrieval, processing and spatiotemporal exploratory assessments of gridded meteorological observation datasets over India. *Environmental Modelling & Software*, 171, 105869.
6. **Central Ground Water Board (CGWB). (2020).** *Ground Water Year Book, Karnataka Region*. Ministry of Jal Shakti, Government of India. [https://cgwb.gov.in/](https://cgwb.gov.in/)
