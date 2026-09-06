# MIT-BIH ST Change Database (STDB)

[![License: ODC-By 1.0](https://img.shields.io/badge/License-ODC--By%201.0-green)](https://opendatacommons.org/licenses/by/1-0/)
[![Access: public](https://img.shields.io/badge/access-public-0e8a16.svg)](https://physionet.org/content/stdb/1.0.0/)

**MIT-BIH ST Change Database (STDB)** — ECG recordings with transient ST depression (exercise stress tests) or ST elevation (long-term excerpts); PhysioNet open access.

- **Dataset repository**: https://github.com/biometric-community/MIT-BIH-ST-Change-Database
- **Upstream source**: https://physionet.org/content/stdb/1.0.0/
- **DOI**: https://doi.org/10.13026/C2ZW2H
- **Original format**: PhysioNet WFDB (`.dat` / `.hea` / `.atr`) under `data/`
- **License (data)**: [Open Data Commons Attribution License v1.0](https://opendatacommons.org/licenses/by/1-0/) (PhysioNet)
- **License (helpers / docs)**: CC BY 4.0 (see [`LICENSE`](LICENSE))

| Field | Value |
|-------|-------|
| Catalog id (tbiom) | `stdb` |
| Category | `physio` |
| Access | `public` |
| PhysioNet / WFDB slug | `stdb` |
| Upstream homepage | https://physionet.org/content/stdb/1.0.0/ |
| Paper alias | STDB (e.g. Abdeldayem & Bourlai, TBIOM 2019 spectral-correlation ECG ID) |

## TL;DR

- **Task**: ST-segment change analysis / ECG identity or pathology benchmarking
- **Modality**: one- or two-channel ECG (WFDB `.dat` / `.hea` / `.atr`)
- **Platform**: MIT-BIH ST change recordings (exercise stress + long-term excerpts)
- **Real/Synthetic**: real
- **Subjects / records**: **28** recordings (IDs `300`–`327`)
- **Sampling**: 360 Hz, format 212; variable length (~13–67 min; 282 341–1 451 857 samples)
- **Channels**: 18 two-channel + 10 one-channel records
- **Annotations**: **beat labels only** in `.atr` (no ST-change annotations; contrast European ST-T)
- **Content**: most records show transient ST depression during exercise; `323`–`327` are long-term excerpts with ST elevation
- **Size**: ~42.0 MiB uncompressed (PhysioNet); ~42 MiB under `data/` locally
- **Citation**: Albrecht 1983 M.S. thesis (+ PhysioNet citation)

## Table of contents

- [Download](#download)
- [Dataset structure](#dataset-structure)
- [Annotation schema](#annotation-schema)
- [Stats and splits](#stats-and-splits)
- [Quick start](#quick-start)
- [Evaluation and baselines](#evaluation-and-baselines)
- [Datasheet (data card)](#datasheet-data-card)
- [Known issues and caveats](#known-issues-and-caveats)
- [License](#license)
- [Citation](#citation)
- [Contact](#contact)

## Download

- **This repository**: WFDB records under [`data/`](data/) (28 × `.hea` / `.dat` / `.atr`).
- **Upstream**: https://physionet.org/content/stdb/1.0.0/ (ZIP ≈ 42.0 MiB uncompressed).
- **Helper script** (from tbiom monorepo root):

```bash
bash projects/datasets/scripts/download_stdb.sh
```

Preferred (after `pip install wfdb`):

```bash
python -c "import wfdb; wfdb.dl_database('stdb', r'projects/datasets/stdb/data')"
```

Or with wget:

```bash
wget -r -N -c -np https://physionet.org/files/stdb/1.0.0/ -P projects/datasets/stdb/data
```

## Dataset structure

```text
stdb/
├── README.md
├── LICENSE
└── data/
    ├── 300.hea / 300.dat / 300.atr
    ├── 301.hea / 301.dat / 301.atr
    ├── ...
    └── 327.hea / 327.dat / 327.atr
```

**Record IDs** (28): `300`–`327` (contiguous).

- **Splits**: no official ML train/val/test partition.
- **Layout notes**: flat WFDB naming — each record `NNN` has header, signal, and annotation files.

## Annotation schema

### WFDB records (`data/*.hea`, `.dat`, `.atr`)

- **`.hea`**: channel count (1 or 2), sampling rate (360 Hz), sample count, ADC/gain fields
- **`.dat`**: binary ECG samples (format 212 in headers checked locally)
- **`.atr`**: **beat labels only** — upstream states there are **no** ST-change annotations (unlike the European ST-T Database)
- **Time base**: sample index at 360 Hz
- **Example** (record `300` header):

```text
300 2 360 536976
300.dat 212 296 12 0 40 3141 0 ECG
300.dat 212 300 12 0 -5 427 0 ECG
```

Use [WFDB](https://physionet.org/content/wfdb/) / `wfdb` (Python) to read signals and annotations.

### Channel layout (verified from local headers)

| Channels | Records | Count |
|----------|---------|------:|
| 2 | `300`–`312`, `318`, `324`–`327` | 18 |
| 1 | `313`–`317`, `319`–`323` | 10 |

### Clinical content (upstream)

| Records | Description |
|---------|-------------|
| `300`–`322` | Mostly exercise stress tests with transient ST depression |
| `323`–`327` | Excerpts of long-term ECG recordings with ST elevation |

## Stats and splits

Counts verified from local `data/`:

| Measure | Count |
|---------|------:|
| Records (`.hea` / `.dat` / `.atr`) | 28 |
| Sample rate | 360 Hz |
| Samples per record | 282 341 – 1 451 857 |
| Approx. duration per record | ~13 – 67 min |
| Two-channel / one-channel | 18 / 10 |
| Local `data/` size | ~42 MiB |

No official subject-disjoint train/test split — define folds carefully if using for identity or segment-level ML.

## Quick start

```bash
cd projects/datasets/stdb
pip install wfdb
```

```python
from pathlib import Path
import wfdb

root = Path("data")
records = sorted(p.stem for p in root.glob("*.hea"))
print(len(records), "records:", records[:5], "...")

rec = root / "300"
sig, fields = wfdb.rdsamp(str(rec))
ann = wfdb.rdann(str(rec), "atr")
print(sig.shape, fields["fs"], "Hz;", len(ann.sample), "beat annotations")
print("symbol sample:", ann.symbol[:8])
```

**Dependencies**: optional `wfdb` for loading; bash + network for re-download helpers.

## Evaluation and baselines

- **Primary metrics**: ST-change detection / characterization metrics from the original thesis context; ECG identity or pathology metrics when used as a secondary corpus
- **Suggested baselines**: classical ST-segment analyzers; modern ECG deep-learning papers citing STDB
- **Baseline numbers**: not reproduced here — see citing literature

## Datasheet (data card)

### Motivation

Provide ECG recordings that exhibit clinically relevant ST-segment changes (depression during exercise stress, elevation in long-term excerpts) for automated ST analysis and related ECG research.

### Composition

28 one- or two-channel ECG records at 360 Hz. Annotation files contain beat labels only; ST-change events are **not** marked in the distributed `.atr` files.

### Collection process

Prepared as part of MIT work on long-term automated ECG / ST-segment characterization (Albrecht, 1983); distributed via PhysioNet as `stdb` 1.0.0.

### Preprocessing

Distributed in PhysioNet WFDB format at 360 Hz (format 212 in local headers). Record lengths vary; channel count is not uniform.

### Distribution

- **Signal / annotation files**: ODC-By 1.0 via PhysioNet
- **Helpers / docs in this folder**: CC BY 4.0 (`LICENSE`)
- **GitHub mirror**: https://github.com/biometric-community/MIT-BIH-ST-Change-Database

### Maintenance

Re-download with `wfdb.dl_database('stdb', ...)` or `download_stdb.sh` if needed. Keep record IDs and WFDB triples intact.

## Known issues and caveats

- **No ST-change annotations** in `.atr` — only beat labels; do not treat this like European ST-T for ST event scoring
- Channel count is **mixed** (10 single-channel, 18 two-channel); loaders must not assume a fixed lead count
- Record durations vary widely (~13–67 min); pad/crop carefully for fixed-length models
- No official ML split — avoid leakage when segmenting from the same record
- Catalog id `stdb` is the PhysioNet / WFDB database name

## License

**Data files** are licensed under **[ODC-By 1.0](https://opendatacommons.org/licenses/by/1-0/)** as published by PhysioNet.

**Packaging helpers / docs** in this folder are **[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)**. See [`LICENSE`](LICENSE).

## Citation

When using this resource, cite the original publication and PhysioNet:

```bibtex
@mastersthesis{Albrecht1983ST,
  title  = {S-T segment characterization for long-term automated {ECG} analysis},
  author = {Albrecht, Paul},
  school = {MIT Dept. of Electrical Engineering and Computer Science},
  year   = {1983}
}

@misc{stdb100,
  title        = {{MIT-BIH} {ST} Change Database},
  author       = {{MIT-BIH} and {PhysioNet}},
  howpublished = {PhysioNet},
  year         = {1999},
  note         = {Version 1.0.0},
  doi          = {10.13026/C2ZW2H},
  url          = {https://physionet.org/content/stdb/1.0.0/}
}
```

Also include the current PhysioNet platform citation required on the project page.

## Contact

- **Upstream**: https://physionet.org/content/stdb/1.0.0/
- **Dataset repository**: https://github.com/biometric-community/MIT-BIH-ST-Change-Database
- **tbiom catalog**: `projects/datasets/stdb/`
