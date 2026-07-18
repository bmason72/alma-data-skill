# ALMA products, QA, and the restore workflow

## Product FITS naming grammar

```
member.<mous-uid>.<source>_<intent>.spw<NN[_NN…]>.<specmode>.<stokes>.<layer>.fits[.gz]
e.g. member.uid___A001_X1467_X291.NGC_1333_sci.spw25_27_29_31.cont.I.tt0.pbcor.fits
     member.uid___A001_X1467_X291.J0423-0120_ph.spw25.mfs.I.pbcor.fits
```

- `<source>`: proposer target name — may itself contain `_` and `.`; parse
  filenames from both ends (known prefix `member.uid___…`, known suffix
  tokens), never by naive splitting.
- `<intent>`: `_sci` science target, `_ph` phase cal, `_bp` bandpass, `_chk`
  check source; polarization-cal tokens are era-dependent (Cycle 12 uses
  `pol_leak`, and also defines `amp`). Calibrator images ARE delivered —
  don't mistake them for science targets.
- `<specmode>`: `mfs` (per-SPW continuum), `cont` (aggregate continuum;
  multi-term products carry `.tt0`/`.tt1`), `cube`.
- `<stokes>`: usually `I`; full-polarization projects deliver multi-Stokes
  `IQUV` products plus polarized-intensity `P` and angle `A` maps (not
  separate `Q`/`U`/`V` files).
- `<layer>`: `pbcor` (primary-beam-corrected), `pb` (PB response; historical
  deliveries used `.flux.fits` instead), `mask` (clean mask), `alpha`
  (spectral index). `pb`/`mask` usually gzipped.
- The grammar is **versioned**: Cycle 10+ pipeline self-calibration adds a
  `selfcal` token (the internal `regcal` variants are generally NOT archived
  for regular products); treat unknown tokens as expected. Filenames are
  hints — authoritative WCS/frame/beam/units/Stokes live in FITS headers.
- The flat-noise (non-pb-corrected) image is not delivered; recover it as
  `flat = pbcor × pb` (pixelwise). Measure noise there — pbcor noise rises
  toward field edges.
- Total Power products are FITS spectral cubes (per SPW).

## Products are not the complete science content

Delivered images are QA2-supporting, best-effort products. Imaging
"mitigation" (data-volume limits) may reduce cube sizes, drop channels,
targets, or SPWs. ADMIT products, where present, are added-value and not
QA-assured. The calibrated visibilities always contain more than the
delivered images — for complete or custom imaging, restore the MS.

## The QA ladder

| Level | Scope | When | Values |
|---|---|---|---|
| QA0 | per **EB** | at execution | PASS / SEMIPASS / FAIL (+ QA0+ assessments) |
| QA1 | observatory/array health | ongoing | — |
| QA2 | per **MOUS** | after processing, gates delivery | PASS / SEMIPASS / FAIL |
| QA3 | per delivered dataset | when a problem is found post-delivery | investigation → possible re-delivery |

Prohibitions (agents get these wrong):

- ObsCore `qa2_passed='T'` collapses the three-state QA2 — read the QA2
  report for the true disposition and reasons (since ~Cycle 5 the README is
  often just a pointer; the QA2 report is authoritative).
- Never infer QA0 status from file presence in a package. A QA2-passed MOUS
  is built from QA0-PASS EBs, but raw ASDMs of non-PASS EBs can also sit in
  the archive (QA0 SEMIPASS data may exist *only* as raw).
- AQUA pipeline scores are diagnostics, not the final QA2 disposition.
- QA3 can make data temporarily unavailable and produce re-deliveries — the
  archive can hold more than one product generation for a MOUS.

## Machine-readable QA artifacts

- **AQUA report** (`pipeline_aquareport.xml`): pipeline QA summary with
  per-topic and per-stage score elements (commonly `QaPerTopic`,
  `QaPerStage`/`RepresentativeScore`/`SubScore` with Name/Score/Reason
  attributes — element names, namespaces, and casing vary by pipeline
  version, so match by local tag name, case-insensitively, and validate any
  parser against the reports at hand). Low scores + Reasons are the
  machine-readable trail of what pipeline QA flagged.
- **Flag templates** (per EB: `uid___A002_*.flagtemplate.txt` paired with
  `*.flagtsystemplate.txt` for Tsys flags; plus `*flagtargetstemplate.txt`
  for science-target flags): explicit CASA `flagdata`-style commands
  applied on top of pipeline heuristics. `mode='manual'` lines are evidence
  of *explicit flag selections* — usually reducer-authored, but the pipeline
  machinery also uses templates, so do not report them as proven human
  intervention. `reason='...'` fields say why.
- **`applycalQA_outliers.txt`**: calibration-application outliers found in
  QA (a weblog artifact — find it inside the unpacked weblog hierarchy, not
  as a guaranteed top-level `qa/` file).
- **PPR** (`PPR*.xml` / `*.pprequest.xml`, in `script/`): the pipeline
  processing request. `<Intents>` blocks with keywords `SESSION_1`, ... hold
  the EB UIDs (`uid://A002/...`) grouped into observing sessions — strong
  provenance for what was *requested*; confirm actual processing from the
  weblog/logs.

## Pipeline recipes and run artifacts

The PPR's `ProcedureTitle` — echoed in artifact filename infixes like
`member.<mous>.hifa_calimage.pprequest.xml` — names the **recipe**. In
modern interferometric deliveries (Cycle 11 / PL2024–2025 observed)
`hifa_calimage` dominates; variants you will meet: `hifa_cal`
(calibration only, no imaging), `_diffgain` suffixes (band-to-band; both
`hifa_calimage_diffgain` and `hifa_cal_diffgain` occur), `hifa_polcal` /
`hifa_polcalimage` (+`_totalintensity` variants), and single-dish
`hsd_*`. A small residue of MOUSs is still
**manually calibrated** even in recent cycles — recognizable by per-EB
`*.ms.scriptForCalibration.py` plus `scriptForImaging.py` and the *absence*
of pprequest/manifest/weblog pipeline artifacts.

Machine-readable run artifacts (in/near the auxiliary products; exact set
varies by pipeline version — sample before assuming):

- `*.pipeline_manifest.xml` — authoritative CASA + pipeline versions
  (`<casaversion>`, `<pipeline_version>`), recipe name, per-session
  caltable tarballs, per-ASDM final flagversions and applycal-command
  files, and one `<image>` element per delivered FITS carrying
  WCS/beam/rms/intent plus `datatype`/`pl_datatype` attributes (e.g.
  `REGCAL_CONTLINE_SCIENCE` vs self-cal variants) — the machine-readable
  alternative to scraping the weblog.
- `casa_pipescript.py` — the ordered stage sequence as *requested*
  (completion is proven by the weblog/logs, not by this script).
  Fingerprints of the modern flow worth recognizing: `hifa_renorm`
  (corrects amplitude-normalization errors caused by bright lines in
  target autocorrelations), `hif_findcont` (writes `cont.dat`),
  `hif_checkproductsize` (the cube/product-size **mitigation gate**, with
  explicit GB thresholds in the call), `hif_mstransform` (creates
  `_targets.ms`), `hif_uvcontsub` (`_targets_line.ms`), `hif_selfcal`, and
  repeated `hif_makeimlist`/`hif_makeimages` rounds with
  `datatype='selfcal'`/`'best'`.
- `pipeline-<timestamp>.selfcal.json` — written whenever the self-cal
  stage runs, i.e. for essentially every recent calimage run. **File
  presence ≠ self-cal applied**: check per-target `sc_success` (true only
  for a minority of targets); `spwsel_*` keys mirror the `cont.dat`
  continuum selections.
- `pipeline-<timestamp>.timetracker.json` — wall-clock seconds per
  pipeline stage. Multiple timetracker files per MOUS mean multiple
  pipeline invocations (retries, continuations, or deliberate splits) —
  not multiple deliveries; correlate timestamps with logs/manifest.
- `pipeline_stats_<mous-uid>.json` (newer pipelines, ~2024.1 on) — compact
  per-MOUS stats: per-EB antenna/scan counts and flagged fractions, L80
  baseline percentile, per-SPW nchan/width/frequency, target list,
  versions. Leaves have shape `{value, units?, origin?, longdescription?}`
  — the qualifier keys are optional; parse accordingly.
- `flux.csv` — flux-model table consumed and maintained by calibration:
  Stokes I/Q/U/V and spectral index per (EB, field, SPW), with provenance
  in the comment column (`origin` — the ALMA calibrator DB *or* the ASDM
  `Source.xml` — plus `age`, `queried_at`). These are the model inputs to
  `hifa_gfluxscale`, not the end-to-end flux-scale transfer.
- `antennapos.csv` (per-MOUS XYZ *offset* corrections for `hifa_antpos`) —
  superseded in newer pipeline versions by per-EB
  `<eb-uid>.antennapos.json` (absolute ITRF positions fetched from the ASA
  uncertainties service). Expect one or the other, not both.
- `*.pldriver_report.xml` — per-ASDM record of "applied legacy fixes"
  (usually empty).

## Getting a calibrated MeasurementSet

The standard archive package does NOT include calibrated visibilities. Your
options, in order of effort:

1. **Ask the archive/ARC**: all three ARCs (EU, NA, EA) provide
   calibrated-MS services; NRAO SRDP additionally offers calibrated MSs for
   much public Cycle 5+ pipeline data. Check before burning a day on a
   restore.
2. **Restore locally with scriptForPI**:
   - Download products + auxiliary + **raw** ASDM tarballs for the MOUS.
   - Keep the ASA tree intact: `raw/` must sit beside `script/`,
     `calibration/`, `qa/` in the member directory (scriptForPI resolves
     relative paths).
   - Match the CASA + pipeline version stated in the README / weblog
     landing page / `pipeline_manifest.xml`, run `casa --pipeline`. The
     **exact** version is required for faithful reproduction; ALMA's
     compatibility guidance (science-pipeline page) blesses certain newer
     versions for many restores — check it rather than assuming either
     way. Unsupported mismatches → subtle or fatal failures.
   - From `script/`: `casa --pipeline -c member.uid___*.scriptForPI.py`.
     Internally uses `casa_piperestorescript.py` (fast: importasdm + apply
     stored caltables + flagversions) or falls back to `casa_pipescript.py`
     (full pipeline re-run).
   - Output lands under the member dir in `calibrated/` (per-EB
     `uid___*.ms` with CORRECTED_DATA, or `calibrated/working/`; older
     cycles: `uid___*.ms.split.cal`).
   - Disk: up to ~14× the delivered data volume (`SPACESAVING=0`); set
     `SPACESAVING=1..3` to trade intermediates for space (~6× at 3).
3. **Manually calibrated datasets** (common through ~Cycle 3,
   dataset-dependent in later cycles): `script/` contains per-EB
   `*.scriptForCalibration.py`; scriptForPI drives them under the
   era-appropriate CASA; expect longer runtimes and more babysitting.

## Cross-checking "which EBs made it"

The EB lists from (a) ObsCore `asdm_uid` rows, (b) PPR SESSION intents, and
(c) `uid___A002_*` filename stems in `calibration/`/`raw/` should agree for
a clean delivery. Mismatches usually mean QA0-failed/SEMIPASS EBs or a
partial download — investigate before combining data.
