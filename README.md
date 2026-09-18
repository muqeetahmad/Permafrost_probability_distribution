# Permafrost Probability Distribution

A logistic regression workflow (Google Colab notebook) for mapping the **probability of permafrost occurrence** from rock glacier inventory data and two topoclimatic predictors: **Mean Annual Air Temperature (MAAT)** and **Potential Incoming Solar Radiation (PISR)**.

## Overview

Permafrost extent is difficult to observe directly over large areas, but rock glaciers act as useful indicators of its presence or absence:

- **Intact rock glaciers (RGID = 1)** — associated with the presence of permafrost
- **Relict rock glaciers (RGID = 0)** — associated with the absence of permafrost

Points mapped along the initiation lines of intact and relict rock glaciers, together with MAAT and PISR values extracted at those points, are used as training data for a binary logistic regression classifier. Once trained, the model is applied to MAAT/PISR raster grids covering the full study area to produce a spatially continuous **permafrost probability map**.

## Method Summary

1. **Training data** — CSV of point observations (`RGID`, `final_maat`, `Solar_radi`)
2. **Train/test split** — 75% training, 25% testing (`random_state = 0`)
3. **Standardization** — features scaled with `StandardScaler`
4. **Model** — `LogisticRegression` (scikit-learn), fit on standardized MAAT/PISR
5. **Evaluation** — confusion matrix on the held-out test set
6. **Gridded prediction** — MAAT and PISR raster grids (`.tif`) are flattened, stacked, standardized, and passed to the trained model via `predict` and `predict_proba`
7. **Output rasters** — probability-of-absence and probability-of-presence grids are reshaped back to the original raster dimensions and saved as `.tif`
8. **Georeferencing** — spatial reference (geotransform + projection) from the source MAAT raster is applied to the output using GDAL, producing a georeferenced probability map ready for GIS software

## Repository Contents

| File | Description |
|---|---|
| `Permafrost_probability_distribution.ipynb` | End-to-end Colab notebook: data loading, model training, evaluation, gridded prediction, and export of georeferenced probability rasters |

## Requirements

The notebook is designed to run in **Google Colab** with the following libraries:

- `numpy`, `pandas`
- `matplotlib`, `seaborn`, `plotnine`
- `scikit-learn`
- `Pillow` (PIL)
- `gdal` (osgeo)

To run locally instead of Colab, install the equivalent packages, e.g.:

```bash
pip install numpy pandas matplotlib seaborn plotnine scikit-learn pillow gdal
```

## Input Data

The notebook expects a Google Drive folder named **`Permafrost_probability_KU`** containing:

| File | Purpose |
|---|---|
| `trainingdata.csv` | Point observations: `RGID` (0/1), `final_maat` (MAAT), `Solar_radi` (PISR) |
| `MAAT.tif` | Mean Annual Air Temperature raster, covering the full study area |
| `PISR.tif` | Potential Incoming Solar Radiation raster, covering the same area/extent as `MAAT.tif` |

> The raster reshape step in the notebook assumes a grid size of **5000 × 11208**. If you use a different study area, update this shape to match your raster dimensions.

## Usage

1. Open `Permafrost_probability_distribution.ipynb` in Google Colab.
2. Mount your Google Drive and upload the `Permafrost_probability_KU` folder (containing `trainingdata.csv`, `MAAT.tif`, and `PISR.tif`).
3. Run the notebook cells in order:
   - Load and inspect the training data
   - Train and evaluate the logistic regression model
   - Load the MAAT/PISR grids and generate the probability map
   - Export the georeferenced permafrost probability raster
4. Retrieve the output rasters from your Google Drive:
   - `Probability_permafrost_presence.tif`
   - `Probability_permafrost_absence.tif`
   - `Probability_permafrost_presence_georeferenced.tif`

## Outputs

- **Confusion matrix** — visual summary of classifier performance on the test set
- **Probability rasters** — pixel-wise probability (0–1) of permafrost presence/absence across the study area
- **Georeferenced GeoTIFF** — final output ready for further analysis or visualization in GIS software (e.g., ArcMap, QGIS)

## Notes & Limitations

- The raw output rasters from `predict_proba` are **not georeferenced** by default; the final notebook step applies the projection and geotransform from the MAAT raster to fix this.
- Only two predictors (MAAT and PISR) are used; model accuracy depends on the quality and spatial coverage of the rock glacier inventory used for training.
- The workflow assumes MAAT and PISR rasters share identical dimensions and spatial extent.

## License

No license file is currently included in this repository. Add one (e.g., MIT, Apache-2.0) if you intend to formalize reuse terms.
