# Dopamine_photometry

Analysis code for the GRAB-DA fiber photometry data in:

> **Cerebellar induced VTA plasticity underlies chronic stress-induced depression-like behaviors**
> *Cell Reports* (under revision)

| Notebook | Figure panels | Content |
|---|---|---|
| `Figure 3&5.ipynb` | Fig. 3I–K, Fig. 5D–F | Feeding-evoked (phasic) and tonic DA release |
| `Figure S8.ipynb` | Fig. S8A, S8B | Tail pinch-evoked DA response |

---

## Requirements

```bash
pip install -r requirements.txt
```

Raw data were recorded with a TDT RZ system (Synapse). Data blocks are read with the [`tdt`](https://pypi.org/project/tdt/) Python package.

## Data organization

Each recording is a TDT block folder stored inside a per-mouse folder:

```
TDT_data/                      <- set as save_dir in each notebook
├── <MouseID>/
│   ├── <TDT block folder>/    <- 1st recording (before)
│   └── <TDT block folder>/    <- 2nd recording (after)
├── <MouseID>/
│   └── ...
```

Before running a notebook:
1. Set `save_dir` to your data folder.
2. Fill in the path lists, entering 1st and 2nd recordings in the same mouse order:
   - `Figure 3&5.ipynb`: `*_before_paths` / `*_after_paths`
   - `Figure S8.ipynb`: `pinch_1st_paths` / `pinch_2nd_paths`

Output files (CSV, SVG, PNG) are written to `save_dir`.

Streams and event markers used:
- `_465A`: GRAB-DA2h signal (465 nm excitation)
- `_405A`: isosbestic control (405 nm excitation)
- `Note` epoc: event markers
  - Code `2003` = pellet biting (Fig. 3/5)
  - 2000-series codes = tail pinch onset (Fig. S8)

---

## Signal processing (common to both notebooks)

1. Downsampling: non-overlapping 10-sample block mean.
2. Motion/bleaching correction: least-squares linear fit of the 405 nm signal to the 465 nm signal (`np.polyfit`, degree 1).
3. ΔF/F = (F465 − fitted F405) / fitted F405.
4. z-score = (ΔF/F − mean of baseline) / SD of baseline (SD with ddof = 1).

---

## `Figure 3&5.ipynb`: feeding-evoked DA (Fig. 3I–K, 5D–F)

| Parameter | Value (t = 0 at pellet biting, Note code 2003) |
|---|---|
| z-score baseline | −330 to −30 s |
| Mean trace window | −60 to +120 s |
| Phasic window (peak) | 0 to 30 s |
| Tonic window | −300 to −120 s (3 min) |

Cohorts:
- Fig. 3: Ctrl, Stress, Gi + CNO
- Fig. 5: vGluT2 O/E + RS, vGluT2 O/E Ctrl, GFP + RS, GFP Ctrl

**Outputs**

| File | Used for |
|---|---|
| `Zscore_<cohort>_MeanOnly.svg` | Fig. 3I, 5D, 5E mean ± SEM traces (1st vs 2nd) |
| `zscore_0_30s_ALL_COHORTS.csv` | Per-mouse z-score, 0–30 s, all cohorts (phasic) |
| `tonic_Mean465_Mean405_win_-300_-120.csv` | Per-recording raw Mean465, Mean405 and Mean465/Mean405 (tonic) |

**Downstream analysis (performed in OriginPro using the CSV values above)**

- Phasic:
  - Peak = maximum z-score within 0–30 s for each recording.
  - R<sub>DA-phasic</sub> = peak(2nd) / peak(1st).
- Tonic:
  - R<sub>DA-tonic</sub> = (Mean465/Mean405)(2nd) / (Mean465/Mean405)(1st).
- Statistics:
  - Within-group 1st vs 2nd: paired t-test on 1st-normalized values (1 vs 2nd/1st ratio).
  - Between-group R<sub>DA</sub>: one-way ANOVA followed by Fisher's LSD post hoc test.

**Plot notes**
- SEM in the mean traces is computed with ddof = 0.
- All traces are saved with a fixed y-axis (−4.13 to 12.43).
- In the published figure, the axis range of each panel was rescaled in the vector graphics editor while preserving the same scale ratio.

---

## `Figure S8.ipynb`: tail pinch-evoked DA (Fig. S8A, S8B)

| Parameter | Value (t = 0 at tail pinch onset) |
|---|---|
| Event | Latest Note code in the 2000–2999 range |
| z-score baseline | −60 to −10 s |
| Group classification (1st recording) | ΔZ = mean z(0 to 3 s) − mean z(−60 to −10 s); ΔZ > 0 → Increase, otherwise Decrease |
| Peak windows (inclusive) | Increase: max, 0–30 s; Decrease: min, 0–3 s and max, 3–30 s |
| Mean trace | −60 to +120 s, interpolated to a 50 ms grid |
| Display smoothing | 0.5 s centered moving average per mouse (traces only; not used for classification or peaks) |

Peaks are taken from the downsampled samples, not from the interpolated traces.

**Outputs**

| File | Used for |
|---|---|
| `FigS8A_deltaZ.csv` | Fig. S8A, per-mouse ΔZ and group |
| `FigS8B_peaks.csv` | Fig. S8B, per-mouse peaks (1st, 2nd, 2nd − 1st) |
| `FigS8B_paired_ttest.csv` | Fig. S8B, paired t-test (2nd vs 1st) and Cohen's d<sub>z</sub> |
| `FigS8B_mean_traces.csv` / `.svg` / `.png` | Fig. S8B, mean ± SEM traces (smoothed and raw) |

---
