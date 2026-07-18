# Mosaics, FIELD vs SOURCE, moving targets, and Total Power

Reviewed 2026-07-18. Current-service statements below are dated because
archive defects and offered observing modes can change.

## FIELD ≠ SOURCE ≠ "a source"

- A proposal "source" can map to one or many telescope **pointings**. In a
  MeasurementSet, mosaic pointings are separate `FIELD` rows — a 50-pointing
  mosaic of one cloud is one science target with 50 FIELDs, not 50 targets.
- MS `FIELD_ID` and `SOURCE_ID` are distinct namespaces, not one-to-one, and
  both are **local to each MS**. Never join EBs or MOUSs on numeric
  FIELD_ID/SOURCE_ID — match on name + position + intent + spectral setup.
- Field/source names are proposer-entered free text and can differ across
  EBs and projects for the same object (calibrator catalogs also evolve:
  `J0423-0120` vs `J0423-013`).

## Mosaics

- `is_mosaic = 'T'` in ObsCore; `s_region` is the union footprint (use it,
  not `s_ra`/`s_dec`, for coverage questions).
- Primary-beam response varies across the mosaic; the delivered `.pb` image
  encodes the combined sensitivity pattern. Edge fields are shallower.
- Pointing centers are in the MS FIELD table; the "phase center" of the
  mosaic product is a synthesis choice recorded in the image header.
- Historical-copy trap: some old 7-m mosaic images made with CASA <5.1.1 and
  Cycle-5-era 12-m/7-m mosaics made with CASA 5.1.1--5.3 had known imaging/PB
  flux-distribution defects. Current archive copies were reimaged or held in
  QA3, but a locally retained old package may still be affected. Read
  `CASAVER`, the QA2 report, and the official issue notice before quantitative
  reuse.

## Ephemeris / moving targets

- Solar-system targets carry time-dependent ephemerides; a single static
  RA/Dec is neither a valid identity nor a valid phase center.
  `importasdm`/pipeline handle ephemeris attachment and (optionally)
  recomputation.
- Archive `s_ra`/`s_dec`/footprints for moving targets may be
  epoch-snapshotted or unreliable; query by `target_name` semantics
  (case/format variants!) and time window instead, and verify against the
  executed data.
- ToO projects may have had placeholder coordinates at proposal time — trust
  executed-data metadata, not proposal metadata.
- Spectral work on moving targets uses source-rest-frame handling
  (ephemeris-aware frames), not LSRK.

## Total Power (TP)

- TP raw data are ASDMs like everything else and restore to per-EB MSs;
  calibration is per EB; imaging combines calibrated data into cubes.
- **Delivered TP products are FITS spectral cubes** (image domain, per SPW).
  There are no interferometric visibilities; autocorrelation single-dish
  data follow different conventions: OFF-position/atmospheric calibration,
  spectral baseline subtraction, gridding and beam (single-dish 12-m beam
  ≈ 58″ × (100 GHz/ν), i.e. ~1.13 λ/D — far larger than synthesized
  beams), and Jy/K flux scaling.
- TP observes spectral-line modes; there is no TP continuum product stream.
- Current Archive known issue (checked 2026-07-18): TP footprints (`s_region`)
  may represent a single antenna pointing rather than the full mapped area.
  Recheck the live archive-known-issues page before relying on TP spatial
  coverage.

## Cross-array combination (12-m + 7-m + TP)

- Sibling MOUSs under one Group OUS are *intended* for combination, but the
  archive delivers them separately, each with its own pipeline run, QA2, and
  products. Combination (e.g. feathering TP with interferometric images,
  joint deconvolution of 12-m+7-m) is a post-delivery science step — never
  assume it has been done.
- Check for siblings via `group_ous_uid` + `schedblock_name` suffixes
  (`_TM1`/`_TM2`, `_7M`, `_TP`). A 12-m-only image progressively loses flux
  on scales approaching and beyond the MRS (see `spatial_scale_max`) — that
  is physics, not a calibration error.
