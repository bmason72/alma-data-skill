# ALMA cycle capabilities (as of Cycle 12, mid-2026)

Capabilities are **cycle-dependent** — every table here carries an "as of"
label; verify against the current Proposer's Guide / Technical Handbook
(almascience.org → Documents & Tools) for anything decision-critical.

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

Cycles 0–2 predate many current conventions: expect different packaging,
manual calibration, sparser metadata.

## Receiver bands

| Band | Freq (GHz) | λ | Availability notes (as of mid-2026) |
|---|---|---|---|
| 1 | 35–50 | ~7 mm | offered from Cycle 10 (2023) |
| 2 | 67–116 | ~4 mm | NOT generally offered yet; 12-m introduction announced for Cycle 13 (obs. from Oct 2026) |
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
  `archive-query.md` bins datasets adequately.
- Long-baseline configurations (C-8+) exclude high bands in some cycles;
  7-m/TP accompany 12-m only when short-spacing coverage was requested.

## Correlator / spectral setup

- Signal path: 2 sidebands → up to **4 basebands** (~2 GHz each) → each
  baseband hosts SPWs.
- **TDM** ("time division", continuum): nominal 2 GHz / 128 channels
  (dual-pol; 256 single-pol, 64 full-pol); current documentation quotes
  1.875 GHz usable. Older datasets and raw descriptions use the 2 GHz/128
  numbers — don't hard-code either.
- **FDM** ("frequency division", spectral line): up to 3840 channels;
  usable bandwidths 58.6 MHz – 1.875 GHz; channel spacings down to a few
  kHz; online channel-averaging factors (2, 4, 8, 16) are common.
- **Hanning smoothing**: effective resolution ≈ 2× channel spacing unless
  averaged.
- Typical continuum setup: 4 × ~2 GHz SPWs, one per baseband. Typical line
  setups mix widths per baseband.
- Polarization: default dual linear (`XX YY`); single-pol and full-pol
  (`XX XY YX YY`) modes exist — check `pol_states`.
- The Wideband Sensitivity Upgrade (WSU) will overhaul bandwidths and the
  correlator in coming cycles — treat all of the above as era-specific.

## Pipeline / CASA coupling

Each delivery is tied to the CASA + pipeline version that produced it
(stated in README and weblog landing page, and machine-readably in
`pipeline_manifest.xml`). Restores use that version (`casa --pipeline`;
some newer versions are officially blessed for restores — check the
science-pipeline page's compatibility guidance). A project's cycle does
**not** pin the pipeline version: processing follows the *operations
pipeline at processing time*, so late processing, QA3, and re-deliveries
put one cycle's projects under multiple releases (e.g. Cycle 11 MOUSs
delivered under both PL2024.1/CASA 6.6.1 and PL2025.1/CASA 6.6.6), and
auxiliary-product formats change across releases (per-MOUS
`antennapos.csv` offsets → per-EB `antennapos.json` absolute positions at
PL2024→PL2025). Read the versions per MOUS; never infer them from the
proposal cycle. Pipeline capability milestones that affect what you
find in a package: official pipeline from late 2014, phased in through
Cycles 3–5 (manual reduction remains dataset-dependent in later cycles);
pipeline imaging from ~Cycle 4, maturing through Cycles 5–9;
**self-calibration in the pipeline from Cycle 10** (adds `selfcal` product
variants); ARC/SRDP calibrated-MS services expanding from ~2023 onward.
