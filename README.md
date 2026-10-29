# Seabed marine litter around Malta: Lagrangian transport modelling and machine learning

This repository contains the code used for my MSc thesis in Environmental Monitoring and Assessment at the University of Malta (Institute of Earth Systems).

The thesis combines two approaches to investigate marine litter around the Maltese Islands and the wider central Mediterranean:

1. **Particle-transport modelling with OpenDrift.** Plastic particles are released from coastal clean-up locations (2017–2025) to simulate their transport and potential delivery to the seabed. Fishing-related litter is also tracked backwards to identify possible source regions.
2. **Machine learning.** MEDITS seabed-litter records are combined with environmental, oceanographic and human-activity predictors to predict seabed-litter accumulation suitability at two extents (the wider central Mediterranean and Malta's 25 nm Fisheries Management Zone).

The simulated seabed locations are then compared with the predicted suitability maps.

## Repository contents

| File | Description |
|---|---|
| `01_OpenDrift_workflow_template.ipynb` | Full particle-transport workflow for one simulation |
| `02_ML_workflow_template.ipynb` | Full machine-learning workflow for one model set |
| `requirements.txt` | Python packages |

The notebooks are provided as templates, with each showing the full workflow for one example. The other simulations and models used in the thesis follow the same workflow, with changes to the settings in the initial code cells.

### 01: OpenDrift workflow

Example: forward simulation with currents + SSH + waves + wind (CSWaWi) and the Terminal Velocity Chain (TVC).

1. Forcing preparation: nearest-neighbour gap filling around Malta, land re-masking with a 4 km inward coastline buffer, bilinear resampling to a 1/120° (~1 km) grid
2. Cleaning the clean-up records and assigning compass regions
3. Moving release points into the sea (coastline snapping, random point within 150 m, check against OpenDrift's landmask)
4. Readers and model set-up, including an `OceanDrift` subclass where terminal velocity depends on each particle's age
5. Simulation run one calendar year at a time, with particle positions and ages carried between years
6. Particle summary and reclassification of deactivations shallower than 1 m as coastal grounding
7. Trajectory metrics, residence in the 25 nm zone and the paired comparison between configurations (Wilcoxon signed-rank, rank-biserial effect size, Holm correction)

### 02: Machine-learning workflow

Example: full extent, all predictors (V1).

1. Loading the aligned predictor rasters and MEDITS haul midpoints
2. Predictor screening (coverage, distinct values, duplicate layers, correlation, iterative VIF)
3. Pseudo-absences (5 km exclusion buffer and a surface range envelope at the 2.5-97.5 percentiles)
4. Spatial split into micro-clusters and six groups (one independent holdout, five cross-validation folds)
5. Random Forest, XGBoost, SVM, MaxEnt and MLP, with grid search (default vs tuned)
6. AUC-weighted ensemble, Youden thresholds, holdout evaluation with BCa bootstrap confidence intervals and DeLong tests
7. Permutation importance and SHAP
8. Prediction map
9. Compact model by cost-aware backward elimination

## Main settings

| Setting | Value |
|---|---|
| OpenDrift version | 1.14.10 (`OceanDrift`) |
| Time step / output interval | 1 h / 6 h |
| Vertical mixing | Constant diffusivity 0.001 m² s⁻¹, 300 s step |
| Coastline / seabed action | Return to previous position / deactivate |
| Terminal Velocity Chain | +0.021 (0–60 d), 0 (60–120 d), −0.001, −0.005, −0.010, −0.020 m s⁻¹ (every 60 days) |
| Windage | 0.02 (CSWaWi only) |
| ML grid | 1/120°, EPSG:4326 (distances in EPSG:3035) |
| Pseudo-absences | 1,000 per extent, seed 20260916 |
| Bootstrap | 2,000 resamples, BCa |

## Data

The data are **not** included in this repository.

| Data | Source |
|---|---|
| Currents, sea-surface height, temperature, salinity | Copernicus Marine Service, Mediterranean Sea Physics Reanalysis |
| Waves and Stokes drift | Copernicus Marine Service, Mediterranean Sea Waves Reanalysis |
| Chlorophyll-a | Copernicus Marine Service, Mediterranean Sea Biogeochemistry Reanalysis |
| Wind | ERA5 and CERRA (Copernicus Climate Change Service) |
| Bathymetry | EMODnet Bathymetry |
| Seabed substrate | EMODnet Seabed Habitats (EUSeaMap) |
| Vessel density, route density, ports | EMODnet Human Activities |
| Seabed-litter presences | MEDITS bottom-trawl survey, via EMODnet Chemistry |
| Coastal clean-up records | Żibel and Nature Trust Malta (shared for this study, not publicly available) |

## Installation

Python 3.11 is recommended.

```bash
git clone <repository-url>
cd <repository-name>
pip install -r requirements.txt
```

OpenDrift can also be installed with conda if pip has problems with its dependencies (see the [OpenDrift documentation](https://opendrift.github.io)).

## Usage

1. Download the data listed above and arrange it as described in the first code cell of each notebook.
2. Change the paths and settings in the first code cells.
3. Run the notebooks from top to bottom.

The year-by-year OpenDrift run and the full grid search both take many hours. To test the code quickly, reduce the grids or the number of release years.

## Notes

- The simulated seabed locations represent possible seafloor delivery, not long-term accumulation, as burial and resuspension are not modelled.
- The suitability maps are uncalibrated and show relative suitability, not absolute probability of litter.
- The OpenDrift configuration was evaluated against one surface drifter. The forward and backward runs use uncorrected currents (current drift factor 1.0).

## Citation

If you use this code, please cite the thesis:

> Pirotta, A. J. (2026). *Modelling marine litter surface transport and seabed accumulation suitability* (Master's dissertation)

## Acknowledgements

Supervised by Dr Liam Butler, with Dr Adam Gauci as co-supervisor. Thanks to Żibel and Nature Trust Malta for the clean-up records, the Department of Geosciences, University of Malta for the drifter data, and the Copernicus, EMODnet and MEDITS programmes for the open data.

## Licence


The datasets used in this study are not included in this repository. Data products created by EMODnet are generally licensed under Creative Commons Attribution 4.0 International (CC BY 4.0), unless otherwise stated. Individual datasets distributed through EMODnet may have additional licensing and attribution requirements. Copernicus Marine Service and Copernicus Climate Change Service data are subject to their respective licensing and attribution requirements. Users should consult the terms of use and associated metadata of the original data providers.
