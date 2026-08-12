# GPFA Latent Dynamics Analysis

This repo fits **Gaussian Process Factor Analysis (GPFA)** to single-unit spiking
data from extracellular recordings, in order to extract low-dimensional latent
trajectories that summarize population-level neural dynamics on a trial-by-trial
basis, and relate those trajectories to behavior.

## What's in this notebook

`GPFA_data.ipynb` walks through the full pipeline:

1. **Load session data** — spikes and trial event times for a recording session
2. **Unit selection** — keep well-isolated ("good") units with enough spikes to
   support GPFA fitting
3. **Build GPFA-ready trials** — convert raw spike times into per-trial,
   per-neuron spike trains aligned to trial onset
4. **Cross-validate latent dimensionality** — fit GPFA across a range of candidate
   dimensionalities and score each by leave-neuron-out prediction error, with a
   held-out test set for a final unbiased evaluation
5. **Fit the final model** at the selected dimensionality and save results
6. **Visualize latent trajectories** — single-trial and trial-averaged dynamics,
   3D trajectories, latent traces alongside spike rasters, and the GPFA loading
   matrix
7. **Relate latents to behavior** — correlate each latent dimension with measured
   behavioral variables (e.g. licking, whisking, pupil size, respiration)

## Requirements

- Python 3.11
- [`elephant`](https://elephant.readthedocs.io/) (spike train analysis + GPFA)
- [`neo`](https://neo.readthedocs.io/) and [`quantities`](https://python-quantities.readthedocs.io/)
- `numpy`, `scipy`, `scikit-learn`, `matplotlib`

```bash
pip install elephant neo quantities numpy scipy scikit-learn matplotlib
```

This notebook also depends on two local modules from this repo, imported via a
path relative to the notebook's location:

- `data_paths.py` — a `DATA_PATH` dict mapping session keys to `.mat` file paths
- `Analysis_library/` — a local package providing `file_loading.SessionData` (for
  loading a session's spikes/trial data) and `analysis.get_good_units` (for
  filtering to well-isolated units)

Update `data_paths.py` to point at your own data before running.

## Usage

1. Set `SESSION_KEY` in the "Load Session Data" section to the session you want
   to analyze.
2. Adjust analysis parameters as needed:
   - `BIN_SIZE_MS` — GPFA time-bin width (default 30 ms; consider 10 ms for finer
     temporal resolution at the cost of more compute)
   - `PRE_TRIAL_S` / `POST_TRIAL_S` — trial window around each trial event
   - `MIN_SPIKES` — minimum total spikes for a unit to be included
   - `X_DIMS` — candidate latent dimensionalities to cross-validate over
   - `N_FOLDS` — number of cross-validation folds
3. Run the notebook top to bottom. The cross-validation section is the slowest
   step (it fits `len(X_DIMS) * N_FOLDS + 2` GPFA models).
4. Fitted models and results are saved to `ISS/gpfa_results.pkl`.

For the behavior-correlation section at the end, you'll need a `binned_behaviors`
dict (behavior name → array of shape `[trials, time bins]`, aligned to the same
trial windows and bin size used above) — this notebook doesn't build it, so load
or construct it from your session's behavioral traces first.

## Known limitations / TODOs

- The final GPFA model isn't currently saved in a form that supports inference on
  new data without refitting.
- Raster plots aren't yet shown alongside every latent trajectory plot.
- The behavior-correlation analysis assumes `binned_behaviors` has already been
  built elsewhere and doesn't validate its alignment with `binned_trials`.

## Output

- `ISS/gpfa_results.pkl` — pickled dict containing the fitted GPFA model,
  estimated parameters, selected `x_dim`, cross-validation errors, unit
  selections, and the analysis settings used to produce them.
