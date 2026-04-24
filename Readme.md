# Visual EEG Simulation — Alpha and Gamma Rhythm Analysis

**Repository:** [[https://github.com/LI-ZHAODONG/eeg_visual_simulation_lac](https://github.com/LI-ZHAODONG/eeg_visual_simulation_lac)](https://github.com/sananakther6642/eeg-visual-task-analysis.git)

Replication and validation of findings from:

> **Dissociable Spatial and Feature Tuning of Gamma and Alpha Rhythms in Human Visual Cortex**
> Ghaffari, Yavari, Bonyadian, Ghofrani & Butler (2025). *bioRxiv* doi:10.1101/2025.08.09.669461

Using high-density EEG (64 channels, OpenNeuro ds006547, n=30–31 subjects across pipelines), we investigate how gamma (40–80 Hz) and alpha (8–13 Hz) rhythms differ in their spatial selectivity, orientation tuning, and summation properties during visual stimulation.

---

## Key Findings

1. **Alpha suppresses broadly** — Visual stimulation suppresses alpha power across posterior electrodes with little spatial selectivity (CV = −2.02 / −0.79)
2. **Gamma enhances focally** — Gamma power increases are retinotopically specific and localized over occipital cortex
3. **Divisive normalization governs alpha summation** — Alpha spatial summation is strongly subadditive (DivNorm outperforms linear by 2–10×); gamma is closer to linear than alpha (DivNorm advantage 1.2–3.8×), consistent with the paper's direction
4. **Gamma is sharply orientation-tuned; alpha is not** — Gamma responses are selective for grating orientation while alpha shows minimal tuning

---

## Approach

We validate these findings using two independent preprocessing pipelines on the same raw data:

- **Custom Pipeline** — Hand-coded Python scripts with automated ICLabel-based ICA, harmonic notch filtering, and per-trial artifact rejection (n=31 subjects)
- **BIDS Pipeline** — Standardized MNE-BIDS-Pipeline with config-driven preprocessing, variance-adaptive ICA, and automated EOG/ECG rejection (n=30 subjects)

### Methodology Comparison

| Stage | Original Paper | Custom Pipeline | BIDS Pipeline |
|-------|---------------|-----------------|---------------|
| **ICA** | Picard, 60 components, manual selection | Picard, 60 components, ICLabel (≥0.60) | Picard, `n_components=0.99`, EOG/ECG thresholds |
| **Notch filter** | Single 60 Hz | Harmonic series (60–480 Hz, 8 harmonics) | Single 60 Hz (width = 2 Hz) |
| **Bandpass** | 1–100 Hz (FIR) | 1–100 Hz (FIR) | 1–100 Hz (FIR) |
| **Epochs** | −0.5 to +3.5 s | −0.5 to +3.5 s | −0.5 to +3.0 s |
| **Gamma band** | 40–80 Hz | 40–80 Hz (full band) | 40–80 Hz (full band) |
| **Artifact rejection** | Broadband gamma outlier (z > 4) | Per-trial PTP rejection | 1 subject excluded (sub-30) |

---

## Results

| Finding | Original Paper | Custom Pipeline | BIDS Pipeline |
|---------|---------------|-----------------|---------------|
| **Alpha suppression** | Broad, posterior | Clear (CV=−2.02) | Clear, 1.6× stronger (CV=−0.79) |
| **Gamma enhancement** | Focal, retinotopic | Present, noisy (CV=+0.929) | Present, noisy (CV=+1.734) |
| **DivNorm > Linear (alpha)** | Yes — subadditive | Yes — 2–9× advantage | Yes — 2–10× advantage |
| **DivNorm > Linear (gamma)** | No — linear wins | DivNorm wins, small margin (1.2–3.8×) | DivNorm wins, small margin (1.3–2.5×) |
| **Orientation tuning** | Strong gamma selectivity | TI ratio = 0.33 | Gamma TI > alpha TI |
| **Cross-subject consistency** | Assumed (n=30) | Alpha reliable; gamma variable | Same pattern |

Both pipelines show DivNorm winning for alpha (strongly, matching the paper) and for gamma (weakly). Gamma is substantially closer to linear summation than alpha across both pipelines, consistent with the paper's direction — gamma at the scalp reflects a mixture of signals from multiple brain sources, pushing the combined response toward linear summation.

---

## Repository Structure

```text
.
├── Custom_pipeline_Dataset/
│   ├── Scripts/              # Custom pipeline scripts + Phase_1/2.sh + Phase_1/2.bat
│   │   └── condition_mapping.json
│   └── outputs/              # Per-subject and group-level results
├── Bids_pipeline_Dataset/
│   ├── Scripts/              # BIDS pipeline scripts + Phase_1/2.sh + Phase_1/2.bat
│   └── outputs/
│       ├── group_level/
│       ├── derivatives/      # MNE-BIDS-Pipeline outputs + bids_analysis/
│       └── sub-01/ ... sub-29/ sub-31/   # sub-30 excluded
├── Team_LAC_Li_Akther_Cai_Report.ipynb   # Final report notebook
├── requirements.txt
└── Readme.md
```

---

## Pipelines

### Custom Pipeline

Located in `Custom_pipeline_Dataset/Scripts/`. Two-phase execution:

**Phase 1** — Per-subject: preprocessing, ICA, band power extraction, retinotopy, orientation tuning, ERSP

**Phase 2** — Group-level: grand averages, topomaps, statistics, model fitting, ERSP cluster tests

| Platform | Phase 1 | Phase 2 |
|----------|---------|---------|
| Mac / Linux | `Phase_1.sh` | `Phase_2.sh` |
| Windows | `Phase_1.bat` | `Phase_2.bat` |

### BIDS Pipeline

Located in `Bids_pipeline_Dataset/Scripts/`. Phase 1 runs MNE-BIDS-Pipeline preprocessing; Phase 2 runs custom analysis scripts (band power, retinotopy, orientation tuning, model fitting, group statistics).

| Platform | Phase 1 | Phase 2 |
|----------|---------|---------|
| Mac / Linux | `Phase_1.sh` | `Phase_2.sh` |
| Windows | `Phase_1.bat` | `Phase_2.bat` |

---

## Installation

**Mac / Linux:**

```bash
python3 -m venv eeg-env
source eeg-env/bin/activate
pip install -r requirements.txt
```

**Windows (Command Prompt):**

```bat
python -m venv eeg-env
eeg-env\Scripts\activate.bat
pip install -r requirements.txt
```

> **Windows note:** Python must be installed from [python.org](https://python.org) with **"Add Python to PATH"** checked. Do not use the Microsoft Store version. If you see `Python was not found`, go to `Settings → Apps → Advanced app settings → App execution aliases` and disable the Python aliases.
>
> **Git Bash users:** Use the `.sh` scripts instead of `.bat` — Git Bash runs shell scripts natively:
> ```bash
> bash Custom_pipeline_Dataset/Scripts/Phase_1.sh
> ```

**Prerequisites for running the pipelines** (not needed for the notebook):
- Raw BIDS dataset from [OpenNeuro ds006547](https://openneuro.org/datasets/ds006547)
- Place `ds006547/` as a sibling of this repository (same parent folder), or set the environment variable:

```
# Default layout (ds006547 next to this repo)
parent/
├── ds006547/                    # BIDS dataset
└── eeg_visual_simulation_lac/   # this repo
```

**Mac / Linux — override path:**
```bash
export BIDS_ROOT=/path/to/ds006547
```

**Windows — override path:**
```bat
set BIDS_ROOT=C:\path\to\ds006547
```

**Downloading the dataset (git-annex):**

OpenNeuro datasets use DataLad/git-annex — cloning the repo only downloads metadata, not the actual EEG files. After cloning, you must fetch the data:

```bash
# Install DataLad (if not already installed)
pip install datalad

# Clone the dataset
datalad install https://github.com/OpenNeuroDatasets/ds006547.git

# Download all files (this fetches the actual EEG data)
cd ds006547
datalad get .
```

Alternatively, using git-annex directly:

```bash
git clone https://github.com/OpenNeuroDatasets/ds006547.git
cd ds006547
git annex get .
```

Without running `datalad get .` or `git annex get .`, the `.vhdr`, `.eeg`, and `.vmrk` files will be empty symlinks and the pipelines will fail to load any data.

---

## Running the Pipelines

### Custom Pipeline

**Mac / Linux:**
```bash
bash Custom_pipeline_Dataset/Scripts/Phase_1.sh
bash Custom_pipeline_Dataset/Scripts/Phase_2.sh
```

**Windows:**
```bat
Custom_pipeline_Dataset\Scripts\Phase_1.bat
Custom_pipeline_Dataset\Scripts\Phase_2.bat
```

### BIDS Pipeline

**Mac / Linux:**
```bash
bash Bids_pipeline_Dataset/Scripts/Phase_1.sh
bash Bids_pipeline_Dataset/Scripts/Phase_2.sh
```

**Windows:**
```bat
Bids_pipeline_Dataset\Scripts\Phase_1.bat
Bids_pipeline_Dataset\Scripts\Phase_2.bat
```

---

## Running the Notebook

The submission notebook (`EEG_Final_Version.ipynb`) loads pre-computed results — no raw EEG processing needed.

**Locally:** Open and run all cells. Paths resolve automatically.

**Google Colab:** Upload `Custom_pipeline_Dataset/outputs/` and `Bids_pipeline_Dataset/outputs/` to Google Drive, then run the notebook.

---

## Condition Mapping

- **Retinotopy (codes 1–20):** Full field, hemifields, quadrants, octants, fovea/periphery, blank
- **Orientation (codes 41–56):** 16 orientations, 0°–337.5° in 22.5° steps

---

## Caveats

- Divisive normalization uses a fixed σ = 0.5 (matching the paper)
- ERSP clusters are threshold-based, not permutation-based
- Gamma effects show high inter-subject variability (CV = +0.929 Custom / +1.734 BIDS)
- One subject excluded in BIDS Pipeline (sub-30): per-subject preprocessing completed but sub-30 was excluded during pipeline configuration and not included in group analysis
- Alpha-gamma correlation not computed for BIDS Pipeline

---

## Conclusion

The dissociable properties of gamma and alpha rhythms in human visual cortex — broad alpha suppression, focal gamma enhancement, subadditive alpha summation via divisive normalization, and gamma-selective orientation tuning — are reliably observed across two independent analysis pipelines applied to the same raw EEG data.
