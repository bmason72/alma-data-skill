# ALMA cycle capabilities (Cycle 12 operations / accepted Cycle 13 capabilities)

Reviewed 2026-07-18. This file is the observing-cycle projection; processing
history is release-first in `pipeline-history.md`.

Capabilities are **cycle-dependent** — every table here carries an "as of"
label; verify against the current Proposer's Guide / Technical Handbook
(almascience.org → Documents & Tools) for anything decision-critical.

Primary sources: [Cycle 13 Proposer's Guide](https://almascience.nrao.edu/proposing/proposers-guide),
[Cycle 13 Technical Handbook](https://almascience.nrao.edu/documents-and-tools/cycle13/alma-technical-handbook),
and [Cycle 13 Archive Manual](https://almascience.nrao.edu/documents-and-tools/cycle13/science-archive-manual).
Cycle 13 entries are accepted/prospective capabilities; no public Cycle 12
(`2025.1`) ObsCore rows existed in the checked EU service on 2026-07-18.

## Contents

- Cycle and project-code year
- Receiver bands
- Arrays and configurations
- Correlator and spectral setup
- Pipeline/CASA coupling

## Cycle ↔ project-code year

| Cycle | Code year | Cycle | Code year |
|---|---|---|---|
| 0 | 2011.0 | 7 | 2019.1, 2019.2 |
| 1 | 2012.1 | — | (no 2020.1 — COVID) |
| 2 | 2013.1 | 8 | 2021.1 |
| 3 | 2015.1 (no 2014) | 9 | 2022.1 |
| 4 | 2016.1 | 10 | 2023.1 |
| 5 | 2017.1 | 11 | 2024.1 |
| 6 | 2018.1 | 12 | 2025.1 |
| — | — | 13 | 2026.1 (observations anticipated from 2026-10-01) |

Cycles 0–2 predate many current conventions: expect different packaging,
manual calibration, sparser metadata.

## Receiver bands

| Band | Freq (GHz) | λ | Availability notes (as of mid-2026) |
|---|---|---|---|
| 1 | 35–50 | ~7 mm | offered from Cycle 10 (2023) |
| 2 | 67–116 | ~4 mm | offered in accepted Cycle 13 capabilities on the 12-m Array only; no Cycle 13 observations existed on the review date |
| 3 | 84–116 | 3 mm | workhorse |
| 4 | 125–163 | 2 mm | |
| 5 | 158–211 | 1.8 mm | offered from Cycle 5 (2017) |
| 6 | 211–275 | 1.3 mm | workhorse |
| 7 | 275–373 | 0.85 mm | |
| 8 | 385–500 | 0.65 mm | |
| 9 | 602–720 | 0.45 mm | DSB receiver |
| 10 | 787–950 | 0.35 mm | DSB receiver |

Band-to-band (B2B) phase transfer pairs a high band with a low one for
calibration — the reason `band_list` sometimes holds two values (e.g.
`5 10`; the higher is usually the science band, but confirm from
`frequency_support`).

Real datasets contain frequencies **outside** these nominal ranges: WVR
windows sit near 183 GHz in every band, and calibration-only SPWs can fall
in inter-band gaps (e.g. 116–125 GHz). Frequency→band lookups must
tolerate out-of-band inputs, not assume every SPW maps to a science band.

## Arrays and configurations

| Array | Antennas | Antenna-name prefixes | Baselines |
|---|---|---|---|
| 12-m main array | ~43–50 × 12 m | `DV`, `DA` | 15 m – 16.2 km |
| ACA / Morita 7-m | ~10–12 × 7 m | `CM` | ~9–50 m |
| Total Power | 4 × 12 m | `PM` | single dish |

- 12-m configurations `C-1` … `C-10` (a.k.a. `C43-N`), approximate maximum
  baselines: 0.16, 0.31, 0.50, 0.78, 1.4, 2.5, 3.6, 8.5, 13.9, 16.2 km.
- Configuration is not an ObsCore column; the resolution-proxy formula in
  `archive-query.md` can help with coarse binning, but validate configuration
  or baseline requirements from the MS/weblog.
- Ordinary non-solar 12-m, 7-m, and TP observations are normally separate;
  solar and exceptional heterogeneous interferometric EBs can mix 7-m and
  12-m antennas. Inspect the EB/MS antenna table before classifying.
- Configuration definitions and allowed band combinations are cycle-specific.
  Cycle 13 calls C-7--C-10 long-baseline and offers Bands 1--10 across
  C-1--C-10 subject to its detailed constraints; use the matching-cycle table.

## Correlator / spectral setup

- Common 2SB-receiver signal path: 2 sidebands → up to **4 basebands**
  (~2 GHz each) → each baseband hosts SPWs. SSB/DSB receivers differ; use
  the matching-cycle Technical Handbook for receiver-specific interpretation.
- **TDM** ("time division", continuum): nominal 2 GHz / 128 channels
  (dual-pol; 256 single-pol, 64 full-pol); current documentation quotes
  1.875 GHz usable. Older datasets and raw descriptions use the 2 GHz/128
  numbers — don't hard-code either.
- **FDM** ("frequency division", spectral line): up to 3840 channels;
  usable bandwidths 58.6 MHz – 1.875 GHz; channel spacings down to a few
  kHz; online channel-averaging factors (2, 4, 8, 16) are common.
- Accepted Cycle 13 capabilities add 4×4-bit FDM modes alongside 2×2-bit:
  at matched spectral resolution, 4×4-bit uses 960 channels and one quarter
  of the bandwidth. Treat this as prospective until Cycle 13 data exist.
- **Hanning smoothing**: effective resolution ≈ 2× channel spacing unless
  averaged.
- Typical continuum setup: 4 × ~2 GHz SPWs, one per baseband. Typical line
  setups mix widths per baseband.
- Polarization: default dual linear (`XX YY`); single-pol and full-pol
  (`XX XY YX YY`) modes exist — check `pol_states`.
- Treat all correlator numbers as era-specific; read the matching-cycle
  Technical Handbook for decision-critical interpretation.

## Pipeline / CASA coupling

Each delivery is tied to the CASA + pipeline release that processed it, not to
its proposal cycle. Read the QA2 report/README, weblog, or manifest per MOUS;
late processing, QA3, and re-delivery put one cycle under multiple releases.
The exact original version is the identical-reproduction baseline, while the
current official compatibility table authorizes some newer restore versions.

See `pipeline-history.md` for the dated operations matrix, official-source
conflicts, patch traps, and the separate statuses **exists / in recipe /
default / attempted / succeeded / delivered**. In particular, PL2023
self-calibration attempts eligible targets; task/script/JSON presence is not
proof that selfcal succeeded or was delivered.
