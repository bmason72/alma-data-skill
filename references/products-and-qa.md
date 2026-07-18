# ALMA products, QA, and the restore workflow

Reviewed 2026-07-18 against Cycle 5--12 QA2-product documents, official
restore/CASA guidance, pipeline release guides, and Cycle 4--11 package
artifacts. Treat exact filenames and machine-readable schemas as
release-specific unless a stable contract is cited.

## Contents

- FITS product names
- Product completeness
- QA ladder
- QA artifacts
- Pipeline artifacts
- Calibrated MS restore
- EB cross-check

## Product FITS naming token families

```
member.<mous-uid>.<source>_<intent>.spw<NN[_NN…]>.<versioned tokens>.<layer>.fits[.gz]
e.g. member.uid___A001_X1467_X291.NGC_1333_sci.spw25_27_29_31.cont.I.tt0.pbcor.fits
     member.uid___A001_X1467_X291.J0423-0120_ph.spw25.mfs.I.pbcor.fits
```

- Treat names as a versioned sequence of recognizable token families, not a
  fixed positional grammar: MOUS prefix; source plus intent; `spw...`; image
  mode (`mfs|cont|cube|repBW`); optional calibration state; Stokes; optional
  Taylor/spectral-index/manual/line-comment tokens; layer; extension. Official
  prose and examples do not keep one stable order.
- `<source>` is proposer text and may itself contain `_` and `.`. Parse from
  anchored ends, preserve unknown tokens, and never use naive splitting.
- `<intent>`: `_sci` science target, `_ph` phase cal, `_bp` bandpass, `_chk`
  check source; polarization-cal tokens are era-dependent (Cycle 12 uses
  `pol_leak`, and also defines `amp`). Calibrator images can be delivered —
  do not assume every image is a science target or every calibrator family is
  present.
- `<specmode>`: `mfs` (per-SPW continuum), `cont` (aggregate continuum;
  multi-term products carry `.tt0`/`.tt1`), `cube`.
- `<stokes>` is usually `I`. Full-polarization delivery is release/recipe
  dependent: Cycle 10--11 commonly has pipeline polarization calibration with
  manual science imaging; PL2025/Cycle 12 adds pipeline full-Stokes target
  imaging. Expect IQUV and derived P/A products where that recipe ran, but do
  not assume separate Q/U/V files, a separately delivered Stokes-I target
  image, or stable token order.
- `<layer>`: `pbcor` (primary-beam-corrected), `pb` (PB response; historical
  deliveries used `.flux.fits` instead), `mask` (clean mask), `alpha`
  (spectral index). `pb`/`mask` usually gzipped.
- PL2023+ processing distinguishes regular calibration from successful
  selfcal in internal names and manifest datatype. Archive-renamed filenames
  need not retain a `regcal` token: the sampled PL2023/PL2024 packages used
  manifest `pl_datatype` while regular DataLink names omitted it. A selfcal
  stage or recipe name also does not prove success. Filenames are hints;
  authoritative WCS/frame/beam/units/Stokes live in FITS headers and
  calibration state should be checked in the manifest/weblog.
- The flat-noise (non-pb-corrected) image is not delivered; recover it as
  `flat = pbcor × pb` (pixelwise). Measure noise there — pbcor noise rises
  toward field edges.
- Total Power products are FITS spectral cubes (per SPW).

## Products are not the complete science content

Delivered images are QA2-supporting, best-effort products. Imaging
"mitigation" (data-volume limits) may reduce cube sizes, drop channels,
targets, or SPWs. ADMIT products, where present, are added-value and not
QA-assured. Calibrated visibilities may contain fields, SPWs, or channels not
represented by the delivered images. Inspect the product/mitigation inventory
and restore the MS when omitted content or custom imaging matters. See the
[official delivered-products guidance](https://help.almascience.org/kb/articles/what-calibration-and-imaging-products-will-be-delivered-to-me).

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
- QA2 processing normally uses QA0-PASS EBs, but the archive/MOUS can also
  contain raw ASDMs in other QA0 states. MOUS QA2 PASS therefore does not
  imply every associated ASDM is QA0 PASS; QA0 SEMIPASS material may be
  raw-only. Never infer QA0 from package-file presence.
- AQUA pipeline scores are diagnostics, not the final QA2 disposition.
- QA3 can make data temporarily unavailable and produce re-deliveries — the
  archive can hold more than one product generation for a MOUS.

## Machine-readable QA artifacts

- **AQUA report** (`pipeline_aquareport.xml` in sampled PL2022+ weblogs):
  pipeline QA summary with per-topic/per-stage scores and reasons. Its XML is
  not a stable public schema; observed local tag names, namespaces, casing,
  and sensitivity fields vary. Parse defensively by local name and keep
  release-tagged fixtures. PL2025 adds explicit observed/theoretical
  sensitivity tags while older ambiguous tags can coexist.
  Sampled PL2018--PL2021 manifests referenced an AQUA report even though the
  file was absent from their weblog payload; PL2022--PL2024 carried it as bare
  `html/pipeline_aquareport.xml`. A manifest reference is not local-presence
  evidence.
- **Flag templates** (per EB: `uid___A002_*.flagtemplate.txt` paired with
  `*.flagtsystemplate.txt` for Tsys flags; plus `*flagtargetstemplate.txt`
  for science-target flags when supplied — present in sampled Cycles 5--7,
  absent in 8--11): explicit CASA `flagdata`-style commands
  applied on top of pipeline heuristics. `mode='manual'` lines are evidence
  of *explicit flag selections* — usually reducer-authored, but the pipeline
  machinery also uses templates, so do not report them as proven human
  intervention. `reason='...'` fields say why.
- **`applycalQA_outliers.txt`**: output from calibration-application QA (a
  weblog artifact — find it inside the unpacked weblog hierarchy, not as a
  guaranteed top-level `qa/` file). Sampled PL2022--PL2024 files contained
  threshold settings only, so presence does **not** prove an outlier was
  detected.
- **PPR** (`PPR*.xml` / `*.pprequest.xml`, in `script/`): the pipeline
  processing request and a strong source for recipe/session/EB intent. Exact
  XML traversal is release-specific; parse defensively, and confirm what
  actually ran from the weblog/logs. Sampled Cycle 7--11 weblogs also carry a
  byte-identical copy named `html/PPR_<SBStatusUID>.xml`; that UID is not the
  MOUS and the copy is not a second request.

## Pipeline recipes and run artifacts

The PPR's procedure title — commonly echoed in artifact filename infixes like
`member.<mous>.hifa_calimage.pprequest.xml` — names the **recipe**. In
modern interferometric deliveries (PL2024 observed; PL2025 documented)
`hifa_calimage` dominates; variants you will meet: `hifa_cal`
(calibration only, no imaging), `_diffgain` suffixes (band-to-band; both
`hifa_calimage_diffgain` and `hifa_cal_diffgain` occur), `hifa_polcal` /
`hifa_polcalimage` (+`_totalintensity` variants), and single-dish
`hsd_*`. Release history and capability-status distinctions are in
`references/pipeline-history.md`. A small residue of MOUSs is still
**manually calibrated** even in recent cycles — positively recognizable by
per-EB `*.ms.scriptForCalibration.py` plus `scriptForImaging.py`. Absence of
pprequest/manifest/weblog supports that conclusion only after a complete
relevant auxiliary inventory; it proves nothing in a partial selection.

Machine-readable run artifacts (in/near the auxiliary products; exact set and
schema vary by release — sample before assuming):

- `*.pipeline_manifest.xml` — the pipeline export/restore manifest and a
  strong source for CASA/pipeline versions, procedure, associated EBs, and
  packaged-product metadata. Element and attribute sets grow across releases;
  introspect the file and test against fixtures instead of coding one universal
  schema. In the sampled PL2018--PL2024 packages, version/procedure attributes
  and per-image metadata were present. The observed schema grows in tiers:
  early samples have a small core plus `aux_asdm`/`aux_caltables`, PL2021 adds
  rich beam/WCS image attributes, PL2023 adds datatype/package/level fields,
  and PL2024 adds a manifest self-entry. Many new attributes can be `N/A`.
  Match by suffix and XML content: a sampled Cycle 5 filename duplicated
  `.calimage.calimage`, and early image names mixed `member.uid___*` with bare
  `uid___*`.
- `casa_pipescript.py` — the ordered stage sequence as *requested*
  (completion is proven by the weblog/logs, not by this script).
  Fingerprints of the modern flow worth recognizing: `hifa_renorm`
  (corrects amplitude-normalization errors caused by bright lines in
  target autocorrelations), `hif_findcont` (writes `cont.dat`),
  `hif_checkproductsize` (the cube/product-size **mitigation gate**, with
  explicit GB thresholds in the call), `hif_mstransform` (creates
  `_targets.ms`), `hif_uvcontsub` (`_targets_line.ms`), `hif_selfcal`, and
  repeated `hif_makeimlist`/`hif_makeimages` rounds with
  `datatype='selfcal'`/`'best'`. An ordinary sampled PL2024 `hifa_calimage`
  request includes `hif_selfcal`, so neither a `_selfcal` recipe suffix nor a
  stage call proves that any target was eligible or succeeded.
- `pipeline-<timestamp>.selfcal.json` and timetracker JSON were delivered in
  the sampled PL2023/PL2024 packages; `pipeline_stats_<mous>.json` appeared in
  the sampled PL2024 package and is documented for Cycle 12 deliveries.
  These are release-specific sidecars, not timeless contracts. **Selfcal JSON
  presence is only attempt evidence**: the sampled PL2023 file had an empty
  `scal_targets` list. Inspect per-target outcomes when present. Multiple
  timestamped files prove multiple recorded invocations, not their cause;
  correlate with logs and the manifest.
- The sampled PL2024 `pipeline_stats_*.json` uses dynamic MOUS/EB/SPW/target
  keys. `bands.value` can include `WVR`, and
  `flagdata_percentage.value.qa2` is a newly-QA2-flagged percentage, **not**
  the QA2 disposition. Parse by documented meaning and release, not key name
  alone.
- `flux.csv` — flux-model/control rows by EB, field, and SPW, not the final
  end-to-end transferred flux scale. Its comment commonly records model
  origin and may add age/query provenance; treat qualifiers as optional.
- Antenna-position helpers may be a per-MOUS `antennapos.csv` or newer per-EB
  JSON. PL2025 changes the default helper path, but delivery prevalence,
  coordinate semantics, and coexistence are release-specific: read the file
  and matching guide rather than inferring from the suffix.
- `*.pldriver_report.xml` occurs with release-dependent spelling/casing in
  sampled packages (`PLDriver_report.xml` also occurs). Treat its content as
  empirical and optional; report presence is not proof a legacy fix ran.

## Getting a calibrated MeasurementSet

The standard archive package does NOT include calibrated visibilities. Your
options, in order of effort:

1. **Ask the archive/ARC** (service status checked 2026-07-18): the three ARCs
   provide calibrated-MS routes, but coverage and output state differ. NRAO
   SRDP's automated path covers much public pipeline-reduced Cycle 5+ 12-m/ACA
   data, excludes manual/TP data, and returns calibrators+targets without
   selfcal or continuum subtraction. Check the
   [current official calibrated-MS guidance](https://help.almascience.org/kb/articles/how-do-i-obtain-a-file-of-calibrated-visibilities-measurement-set-for-alma-data)
   before relying on coverage or retention.
2. **Restore locally with scriptForPI**:
   - Download the auxiliary package plus the **raw** ASDMs required by the
     package's restore script/manifest/PPR. Preserve the complete associated-
     raw inventory separately; do not feed every archive-associated QA0
     SEMIPASS/FAIL ASDM into a rerun. Numbered FITS product tars are
     science/reference products, not required to reconstruct the calibrated MS
     unless the package's own script/README explicitly says otherwise.
   - Work in an isolated, release-compatible CASA environment on a staged MOUS
     copy. Preserve the downloaded raw/package holdings. Keep the staged ASA
     tree intact: `raw/` must sit beside `script/`,
     `calibration/`, `qa/` in the member directory (scriptForPI resolves
     relative paths).
   - Match the CASA + pipeline version stated in the QA2 report / README /
     weblog / manifest. The exact original is the identical-reproduction
     baseline; ALMA's
     [current compatibility table](https://almascience.nrao.edu/processing/science-pipeline)
     authorizes newer releases for many restores. Do not infer the version
     from proposal cycle.
     Packages processed before 2017-10-01 may lack the manifest required by
     CASA 5.1.1+, and legacy `.tar.gz` versus `.tgz` names can independently
     break newer restore tasks.
   - Treat package Python as executable code: inspect it, verify that exactly
     one `scriptForPI.py` belongs to the staged MOUS, then from that staged
     `script/` directory run `casa --pipeline -c <exact-scriptForPI-path>` —
     never select the script with a wildcard.
     Internally uses `casa_piperestorescript.py` (fast: importasdm + apply
     stored caltables + flagversions) or falls back to `casa_pipescript.py`
     (full pipeline re-run).
   - Output lands under the member dir in `calibrated/` (per-EB
     `uid___*.ms` with CORRECTED_DATA, or `calibrated/working/`; older
     cycles: `uid___*.ms.split.cal`).
   - Disk: official guidance gives upper-bound working-space estimates of
     about 14× delivered products+raw at `SPACESAVING=0`, down to about 6× at
     3; actual MOUSs vary.
3. **Manually calibrated datasets** (common through ~Cycle 3,
   dataset-dependent in later cycles): `script/` contains per-EB
   `*.scriptForCalibration.py`; scriptForPI drives them under the
   era-appropriate CASA; expect longer runtimes and more babysitting.

## Cross-checking "which EBs made it"

Treat these as different scopes: ObsCore `asdm_uid` rows/raw DataLink describe
associated raw membership, while PPR sessions and calibration filenames
describe EBs accepted for that processing run. They often agree for a simple
PASS delivery, but QA0-failed/SEMIPASS raw EBs and partial downloads can make a
legitimate difference. Preserve both lists and investigate before combining.
