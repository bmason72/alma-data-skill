# ASDM and MeasurementSet semantics

Reviewed 2026-07-18 against the current ALMA QA2-product/restore guidance,
the official CASA `importasdm`/`split` contracts, and Cycle 4--11 package
artifacts. Naming and column state are release-dependent; the table below is
for restored interferometric data, not TP, solar, VLBI, or phased-array modes.

## Contents

- ASDM raw format
- Extra SPWs
- SPW identity
- Directory-form datasets
- DATA columns
- Spectral frames
- Size and time

## ASDM: the raw format

- An ASDM (ALMA Science Data Model) is a **directory-form dataset**: XML
  metadata tables (`ASDM.xml`, `Main.xml`, `Scan.xml`, `SpectralWindow.xml`,
  ...) plus binary payloads under `ASDMBinary/`. One ASDM = one EB
  (`uid://A002/...`).
- Raw tarballs (project-prefixed basenames — glob
  `*uid___A002_*.asdm.sdm.tar`) unpack to that directory (named after the
  UID, normally an A002 identifier). Identify it from the raw-ASDM role or
  `asdm_uid`, not the prefix alone. It is not directly usable for science —
  convert with CASA `importasdm`.
- `importasdm` options matter scientifically: handling of online flags
  (`Flag.xml` — apply or import as FLAG_CMD), binary flags,
  auto- vs cross-correlation selection, WVR-corrected data streams,
  ephemeris conversion. "Just run importasdm" is not a complete instruction —
  the pipeline/scriptForPI invokes it with the right options; prefer that
  path for reproducing standard calibration.

## The ASDM has more SPWs than the PI asked for

A raw ASDM/imported MS contains, besides the science SPWs: channel-averaged
"companion" SPWs, WVR (water-vapor radiometer) SPWs, pointing/atmospheric/
Tsys calibration SPWs, and square-law-detector windows. A "4-SPW" project
can easily show 25+ SPWs after import. The SPW *Name* string encodes the
taxonomy (`...#FULL_RES` science windows vs `#CH_AVG`/`#SQLD`/
`WVR#NOMINAL`) — grammar and classification traps in
`references/listobs-and-intents.md`. This is why delivered product
filenames carry IDs like `spw25` — they are the pipeline's virtual SPW IDs
(imaged-window identifiers), **not** "the PI's 25th window" and not 0-based
science indices. (They are also not MS `DATA_DESC_ID`s, which map to
SPW × polarization-setup pairs — the numbers may coincide in simple cases,
but the identifiers are distinct.)

## SPW IDs are not durable identities

SPW numbering can change through `importasdm`, `split`/`mstransform`,
concatenation, and pipeline product generation. The same physical window may
be spw 25 in one MS and spw 0 in another. Match SPWs across datasets by
frequency range + channelization + intent, never by numeric ID. (Within one
MOUS the pipeline keeps "virtual" SPW IDs consistent across EBs — that is a
pipeline convention, not an MS guarantee.)

## MeasurementSets and caltables are directories

A CASA MS and every calibration table is a directory tree, not a file.
Copy/checksum/delete them accordingly (`rsync -a`, `shutil.copytree`, du for
sizes); naive file-oriented code (glob for files, open(), file checksums)
silently misbehaves.

## DATA / CORRECTED_DATA semantics

- A freshly imported MS has `DATA` (raw). Applying calibration
  (`applycal`) writes `CORRECTED_DATA`.
- `split`/`mstransform` write the *selected* column into the output's
  `DATA` — a split-out "calibrated" MS has calibrated values in `DATA` and
  usually no `CORRECTED_DATA`.
- Which state you have depends on restore history. For the standard pipeline
  interferometric views, use this as an orientation table, then inspect the
  actual columns and scripts:

| View | Era | `DATA` | `CORRECTED_DATA` |
|---|---|---|---|
| `uid*.ms` | all pipeline eras | raw | calibrated |
| `uid*_target.ms` | pipeline imaging through PL2021 (sampled Cycles 5--8) | calibrated | continuum-subtracted |
| `uid*_targets.ms` | Cycle 9 / PL2022 | calibrated | continuum-fit/subtracted result; may be absent |
| `uid*_targets_line.ms` | Cycle 9 / PL2022 | calibrated + continuum-subtracted (copied from the preceding corrected column) | normally absent |
| `uid*_targets.ms` | Cycle 10+ / PL2023+ | calibrated | successful selfcal result per eligible field; may be absent |
| `uid*_targets_line.ms` | Cycle 10+ / PL2023+ | calibrated + continuum-subtracted | successful selfcal result per eligible field; may be absent |
| `uid*.ms.split.cal` | manual reductions or `DOSPLIT=True` | calibrated | normally absent |

  `casa_piperestorescript.py` restores the full `uid*.ms`; target/imaging
  views require later imaging stages or the corresponding `scriptForPI`
  options. These column semantics come from the producing CASA commands,
  not `listobs.txt` (which does not inventory columns); verify with CASA table
  tools or casacore. A service-supplied calibrated MS may have a different
  output contract. Never choose a data column from the suffix alone.

## Spectral frames: Doppler setting, not tracking

- ALMA does **not** Doppler-track. Sky frequencies are fixed per EB
  ("Doppler setting"); the native uv-data frame is **TOPO**.
- Different EBs of the same MOUS (observed weeks apart) have shifted sky
  frequencies; combining or stacking spectra requires regridding
  (`mstransform`/tclean `outframe`).
- Pipeline image products are typically in **LSRK**; `cont.dat` frequency
  ranges are LSRK. Solar-system work uses topocentric/ephemeris frames
  (REST/SOURCE options).
- Effective spectral resolution is about 2× channel spacing for
  Hanning-smoothed, unaveraged data. Do not assume the ObsCore aggregate
  `velocity_resolution` is a raw per-SPW channel description; use the MS/ASDM
  correlator metadata for exact channelization.

## Practical size/time expectations

- Restores are disk-hungry: official guidance quotes up to ~14× the
  delivered data volume with `SPACESAVING=0`, down to ~6× with
  `SPACESAVING=3` (environment variable read by scriptForPI to control
  intermediate-product cleanup).
- ASDM XML tables are cheap to parse directly (e.g. count scans/intents from
  `Scan.xml`) when a full CASA environment is unavailable — but treat that
  as read-only reconnaissance, not a substitute for CASA.
