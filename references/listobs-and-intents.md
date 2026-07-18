# listobs files, scan intents, and SPW naming

`listobs` is CASA's text summary of a MeasurementSet — and you rarely need
CASA to get one: recent pipeline weblogs (PL2024/PL2025 observed) ship a
pre-rendered `listobs.txt` for each MS the pipeline handled, at
`pipeline-*/html/session<name>/<eb-uid>.ms/listobs.txt`. (Directory name =
`session` + the session name, so `sessionsession_1` is normal, not a bug.
Layouts drift — search the weblog tree recursively for `listobs.txt`.)
Reading these is the cheapest route to scans, intents, fields, SPWs, and
antennas for delivered data.

## One EB, several MS views — pick the right listobs

A current-era *imaging*-recipe run (`hifa_calimage` family,
targets.ms/self-cal workflow) carries up to three MSs per EB, each with
its own listobs in the weblog. Calibration-only recipes and manual
reductions do not produce the target views:

| MS | Content |
|---|---|
| `<eb>.ms` | everything: calibrator + target scans; all SPWs incl. WVR/SQLD/CH_AVG |
| `<eb>_targets.ms` | science targets only (post `hif_mstransform`); science SPWs only |
| `<eb>_targets_line.ms` | continuum-subtracted variant; scan/SPW tables often identical to `_targets` |

- Globbing "all listobs of a MOUS" triple-counts every EB and mixes
  incompatible SPW inventories. Choose one view per question: the full MS
  for calibration/intent/time accounting, `_targets` for science-SPW
  tabulation.
- In observed PL2024/25 runs, SPW IDs stay consistent across the three
  views of one EB (science windows keep their full-MS numbers) — treat
  this as a convention to verify, not a guarantee: MS transformations may
  renumber. Cross-EB consistency is a separate question (see below).
- **Split views retain stale parent metadata**: the `Sources:` table of a
  `_targets*` MS can still list calibrator sources and SPW IDs that do not
  exist in that MS's own SPW table. `Fields:` and `Spectral Windows:`
  describe what the MS actually holds; validate `Sources:` rows against
  them before joining.

## Anatomy and parsing

Sections appear in this order in current listobs output. Parse with a
state machine keyed on the section-header lines, and tolerate lines you
don't recognize:

1. Header: `MeasurementSet Name` (the full path exposes the
   `SOUS_*/GOUS_*/MOUS_*/working/` processing tree), `Observer`,
   `Project` (uid), `Data records`, total elapsed time,
   `Observed from <t0> to <t1>`.
2. Scan table: starts at the `Date        Timerange (UTC)` header; columns
   `Scan FldId FieldName nRows SpwIds Average Interval(s) ScanIntent`.
3. `Fields: <n>` table (ID, Code, Name, RA, Decl, Epoch, SrcId, nRows).
4. `Spectral Windows: (<n> unique spectral windows ...)` table (below).
5. `Sources: <n>` table (per-SPW rest frequencies / SysVel).
6. `Antennas: <n>:` table, **at the end** (name, pad/station, diameter,
   Long/Lat, offsets, ITRF XYZ). Its header line is where you read the
   antenna count.

Scan-table traps:

- The `dd-Mon-yyyy/` date prefix appears **only on the first scan row of
  each UTC day**; continuation rows start with whitespace and a bare
  timerange. Carry the date forward while parsing.
- `SpwIds` and `Average Interval(s)` are bracketed comma lists paired
  positionally — validate that the lengths match rather than assuming;
  treat a singleton interval as a logged fallback (broadcast), since a
  length mismatch can also mean your parser broke on a format change.
- `ScanIntent` is a bracketed **list**: scans are routinely multi-intent.

## Scan intents

Grammar: `<ACTION>#<SUBSCAN_STATE>`. Vocabulary in modern interferometric
data (observed corpus-wide across Cycle 11 12-m and 7-m deliveries; other
modes/eras can add more):

```
OBSERVE_TARGET#ON_SOURCE          science target
OBSERVE_CHECK_SOURCE#ON_SOURCE    check source (common in long-baseline)
CALIBRATE_BANDPASS#ON_SOURCE      CALIBRATE_FLUX#ON_SOURCE
CALIBRATE_PHASE#ON_SOURCE         CALIBRATE_POINTING#ON_SOURCE
CALIBRATE_POLARIZATION#ON_SOURCE
CALIBRATE_DIFFGAIN#ON_SOURCE / #REFERENCE      (band-to-band transfer)
CALIBRATE_ATMOSPHERE#{AMBIENT,HOT,OFF_SOURCE,TEST}
CALIBRATE_WVR#{ON_SOURCE,AMBIENT,HOT,OFF_SOURCE,REFERENCE,TEST}
```

Semantics and traps:

- **Multi-intent is the norm.** Bandpass and flux calibration commonly
  share one scan/source
  (`[CALIBRATE_BANDPASS#ON_SOURCE,CALIBRATE_FLUX#ON_SOURCE,...]`); an
  atmosphere scan carries all four ATM sub-states at once.
- **`CALIBRATE_WVR` is a common secondary intent** on pointing,
  atmosphere, and calibrator scans (target scans usually carry only
  `OBSERVE_TARGET#ON_SOURCE`, though WVR *data* flow on every scan) —
  never classify a scan as "WVR" by mere presence of the intent.
- Time/data-volume accounting must assign each scan to exactly **one**
  bucket via an explicit priority (e.g. target > bandpass/flux > phase >
  check > pointing > polarization > diffgain > atmosphere), or per-EB
  fractions will not sum to 1.
- Intents fingerprint the processing recipe: `CALIBRATE_DIFFGAIN` ↔
  `*_diffgain` recipes (band-to-band), `CALIBRATE_POLARIZATION` ↔
  `hifa_polcal*` recipes.

## Spectral Windows table and SPW Name grammar

Columns: `SpwID Name #Chans Frame Ch0(MHz) ChanWid(kHz) TotBW(kHz)
CtrFreq(MHz) BBC Num Corrs`. Real excerpt (Band 4):

```
 0  X973087613#ALMA_RB_04#BB_1#SQLD              1  TOPO  138533.429  2000000.000  2000000.0  138533.4293  1  XX YY
 4  WVR#NOMINAL                                  4  TOPO  184550.000  1500000.000  7500000.0  187550.0000  0  XX
 5  X973087613#ALMA_RB_04#BB_1#SW-01#FULL_RES  128  TOPO  139525.617   -15625.000  2000000.0  138533.4293  1  XX YY
 6  X973087613#ALMA_RB_04#BB_1#SW-01#CH_AVG      1  TOPO  138509.992  1781250.000  1781250.0  138509.9918  1  XX YY
```

Name grammar:
`X<spectral-spec-id>#ALMA_RB_<band>#BB_<baseband>#SW-<nn>#<TYPE>` with

- `#FULL_RES` — a native-resolution correlator window (science *or*
  calibration-only — see the counting trap below);
- `#CH_AVG` — 1-channel channel-averaged companion window (calibration
  aid; do not assume a strict 1:1 pairing with FULL_RES rows);
- `#SQLD` — square-law-detector pseudo-window, one per baseband **per
  spectral spec** (extra tunings duplicate them);
- `WVR#NOMINAL` — 4-channel water-vapor-radiometer window near the 183 GHz
  water line **regardless of receiver band**; single product (`XX`).

Traps:

- **Counting `#FULL_RES` rows in a full MS overcounts the science
  windows**: calibration-only tunings contribute FULL_RES windows too
  (e.g. 7 FULL_RES rows for a 4-SPW project). Count science windows in
  the `_targets.ms` view, or intersect full-MS FULL_RES rows with the
  `SpwIds` of `OBSERVE_TARGET` scans. Never count by SPW-table row count.
- `ChanWid` is **signed**: negative means frequency decreases with channel
  index. Use |width|, and never assume ascending order. Min/max over the
  channel-center endpoints spans the *centers* — full coverage extends
  another half-width past each end; this shortcut suits regular FULL_RES
  windows, not WVR/irregular channelizations (read MS
  `CHAN_FREQ`/`CHAN_WIDTH` for those).
- Receiver band is recoverable from `ALMA_RB_<nn>` in the Name — useful
  when ObsCore metadata is not at hand.
- The listed `Frame` is TOPO for raw/uv data (Doppler setting — see
  `asdm-and-ms.md`).
- Calibration-only SPWs (and WVR) can sit at frequencies **outside the
  nominal receiver-band ranges** (in inter-band gaps such as 116–125 GHz);
  frequency→band lookups must tolerate out-of-band inputs instead of
  failing.

## Which array? Infer it

listobs has no "array type" field. Antenna-name prefixes (`DV`/`DA` 12-m,
`CM` 7-m, `PM` TP) and dish diameters are authoritative. As a fallback for
modern interferometric data, the antenna count separates arrays cleanly:
7-m EBs run ~8–12 antennas, 12-m ~39–50, with an empty gap between
(threshold near 20). Remember TP EBs have ~4 `PM` antennas, and
historical/commissioning data can violate modern counts.

## Cross-EB caution

EBs of one MOUS *usually* share the spectral setup and (by pipeline
convention) the SPW numbering — but verify before merging tabulations:
match nchan, bandwidth, channel width, and center frequency per SPW ID
across EBs, and expect occasional mismatches (re-tunings; Doppler-setting
shifts move TOPO frequencies between EBs).
