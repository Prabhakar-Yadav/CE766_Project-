# CE 766: Hydrological Assessment of Watershed Interventions in the Malaprabha Sub-Watershed

**Author:** Prabhakar Yadav  
**Course:** CE 766 Watershed Management | **Instructor:** Prof. Riddhi Singh

## 1. Repository Contents
* `baseline_model.py`: Python script for spatial aggregation of IMD gridded rainfall data, baseline SCS Curve Number ($CN = 78$) water balance calculation, and hydrograph plotting.
* `malaprabha_2010_2020_cleaned.csv`: Aggregated monthly precipitation and baseline simulated surface runoff (mm) for the Malaprabha Sub-Watershed (2010–2020).
* `baseline_runoff.png`: Output hydrograph of baseline monthly precipitation vs. direct runoff.

## 2. Original Data Sources
1. **IMD Gridded Rainfall & Temperature:** India Meteorological Department (IMD), $0.25^\circ \times 0.25^\circ$ daily gridded dataset (`https://www.imdpune.gov.in/`) accessed via `imdlib`.
2. **CAMELS-IND Dataset:** Catchment Attributes and Meteorology for Large-sample Studies – India, Zenodo (`https://doi.org/10.5281/zenodo.14005378`).
3. **Land Cover (LULC):** National Remote Sensing Centre (NRSC) Bhuvan portal (`https://bhuvan.nrsc.gov.in/`).
4. **Soil Attributes:** ISRIC World Soil Information – SoilGrids (`https://soilgrids.org/`).

## 3. Current Project Status
* **Completed:** Acquisition of 2010–2020 meteorological forcing data, derivation of composite baseline Curve Number ($CN = 78$), and execution of Baseline Scenario (Scenario 0) water balance model.
* **Underway:** Implementation of storage-routing modules for Scenario 1 (Check Dams), Scenario 2 (Farm Ponds), and Scenario 3 (Afforestation) to evaluate net groundwater recharge improvements.
