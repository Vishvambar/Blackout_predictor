# Blackout_predictor

Blackout_predictor is a small, research-focused repository of Jupyter notebooks and example data that demonstrates preprocessing and a simple instability detector for high-frequency electrical grid sensor streams. The notebooks show how to synthesize or ingest high-resolution sensor readings, align and resample them, compute rolling correlations on frequency channels, and collapse those correlations into a single grid "sync score" that can be used as a blackout / instability indicator.

This README replaces a corrupted placeholder and documents the notebooks, data files, how the pipeline fits together, and how to reproduce the results locally.

## Quick summary

- Primary artifacts: Jupyter notebooks (data.ipynb, processing_data.ipynb, find_moment.ipynb, corr.ipynb).
- Data files: raw_grid_data.csv (synthetic/raw readings), processed_grid_data.csv (aligned/resampled data), rolling_correlation.csv (heavy intermediate), Sync_scre.csv (sync score time-series).
- Goal: demonstrate an end-to-end flow from high-frequency sensor streams to a simple grid instability indicator (rolling-correlation → sync score).

## Repository layout

```
README.md                     # this file
data.ipynb                    # synthetic data generation (creates raw_grid_data.csv)
processing_data.ipynb         # ingest & resample (creates processed_grid_data.csv)
find_moment.ipynb             # analysis: rolling correlation, sync score, plots (exports rolling_correlation.csv, Sync_scre.csv)
corr.ipynb                    # small demo / correlation examples
raw_grid_data.csv             # (committed) raw sensor readings (synthetic)
processed_grid_data.csv       # (committed) resampled & pivoted data used by analysis
rolling_correlation.csv       # (committed) rolling correlation output (large)
Sync_scre.csv                 # (committed) collapsed sync score time-series
.gitignore
```

## What each notebook does (recommended order)

1. data.ipynb
   - Generates synthetic high-frequency sensor streams for multiple substations.
   - Simulates sampling jitter, desynchronization, and injected fault behavior that mimics cascading instability.
   - Outputs: `raw_grid_data.csv` (timestamped rows per sensor).

2. processing_data.ipynb
   - Loads `raw_grid_data.csv`, converts Timestamp to datetime, pivots so each metric+sensor is a column, and resamples to a uniform 100 ms cadence.
   - Interpolates and forward/back-fills to remove NaNs, then writes `processed_grid_data.csv`.

3. find_moment.ipynb
   - Loads `processed_grid_data.csv`, isolates `Frequency_*` columns, computes a rolling correlation matrix (window=50 samples), collapses it to a single per-timestep sync score.
   - Produces visualizations of the sync score and writes `rolling_correlation.csv` and `Sync_scre.csv`.

4. corr.ipynb
   - Small, illustrative examples of correlation calculations used during exploration and debugging.

## How to run the project locally

Prerequisites
- Python 3.8+ (3.9/3.10 recommended)
- ~2–4 GB free memory for the small example files; larger datasets will need more.

Suggested packages (add these to `requirements.txt`):
- numpy
- pandas
- matplotlib
- jupyterlab or notebook

Create a virtual environment and install dependencies:

```bash
python -m venv .venv
source .venv/bin/activate    # Windows: .venv\Scripts\activate
pip install --upgrade pip
pip install numpy pandas matplotlib jupyterlab
```

Run notebooks interactively

1. Start Jupyter Lab or Notebook:
   - jupyter lab
   - or jupyter notebook
2. Open and run cells in order: `data.ipynb` → `processing_data.ipynb` → `find_moment.ipynb`.

Run notebooks non-interactively (execute & save)

```bash
# Example: execute the processing notebook and save the executed notebook
jupyter nbconvert --to notebook --execute processing_data.ipynb --output outputs/processing_data_executed.ipynb
```

Notes on computational cost
- The rolling-correlation step in `find_moment.ipynb` computes a full pairwise correlation across frequency sensors inside a rolling window; this is O(n_sensors^2) per timestep and can become expensive as sensor count or window size grows. For larger problems consider approximations (subset pairing, PCA on frequency channels, or incremental correlation methods).

## Data format and expectations

- raw_grid_data.csv: long-form table with columns similar to Timestamp, Sensor_ID, Voltage, Current, Frequency. This file is committed as a small synthetic example.
- processed_grid_data.csv: pivoted wide-form table where columns are metric_sensor (e.g. Voltage_Substation_01, Frequency_Substation_01) and rows are uniformly sampled timestamps.
- rolling_correlation.csv: output of freq-channel rolling correlation; typically large and used for debugging/analysis.
- Sync_scre.csv: collapsed sync score (one value per timestep) that can be plotted or thresholded to flag instability.

If you replace the example files with real data, ensure the same column naming conventions or update the pivoting/column filtering cells in `processing_data.ipynb` and `find_moment.ipynb`.

## Reproducing the demonstration

1. From a fresh clone, run `data.ipynb` to regenerate `raw_grid_data.csv` (or skip if you already have `raw_grid_data.csv`).
2. Run `processing_data.ipynb` to build `processed_grid_data.csv`.
3. Run `find_moment.ipynb` to compute `rolling_correlation.csv` and `Sync_scre.csv` and to visualize the sync score.

## Next steps and suggestions

- Add a `requirements.txt` or `environment.yml` to lock dependencies.
- Add a short `Makefile` or scripts/automation to run the notebooks in order.
- Replace committed CSVs with small sample datasets and keep large raw datasets out of the repo; provide download instructions for large data.
- Consider implementing a lighter-weight sync detector (e.g., PCA-based) for scaling to many sensors.
- Add tests for preprocessing steps (e.g., shape checks, no-NaN assertions) and a small example unit to ensure notebooks run in CI.

## Contributing

Contributions are welcome. Please open an issue describing the change or a small pull request. Useful contributions:
- Add dependency files (requirements.txt / environment.yml)
- Provide a Dockerfile or Binder config for reproducible notebooks
- Add comments and documentation cells in notebooks that explain the math and parameter choices

  
## Contact
Maintainer: Vishvambar
