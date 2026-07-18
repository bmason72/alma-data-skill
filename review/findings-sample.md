# Empirical Cycle 4--11 package review

Review date: 2026-07-18

This report is the empirical companion to the documentation review. It
examines the scratch corpus at
`<deleted-scratch>/sample` and the DataLink responses beside it. That
2.95-GB scratch tree was permanently removed after inventorying. Paths below
are historical command-log placeholders, not live evidence paths; redownload
the listed MOUSs into a new scratch root and replace `<deleted-scratch>` before
rerunning commands. File hashes were not retained and were not reconstructed
after cleanup.
The machine-readable row-level inventory is
`working-with-alma-data/review/sample-inventory.tsv`.

## Outcome

The corpus supports the skill's central warnings about MOUS granularity,
partial downloads, nested archives, cycle/version drift, and filename
heuristics. It also supplies several sharper corrections:

1. A Cycle 4 manual package and a Cycle 5+ pipeline package are materially
   different things. Cycle 4 here has 419 outer tar members, expanded
   calibration plots, manual scripts, QA PNG/text, and no PPR/manifest/weblog.
   Each Cycle 5--11 PASS auxiliary tar has only 15--19 outer members, with the
   large content held in nested `*.tgz` archives.
2. Proposal cycle is not package version. The sampled CASA/pipeline builds run
   from CASA 4.7.2 (manual Cycle 4) through CASA 6.6.1.17 / Pipeline
   2024.1.0.8 (Cycle 11). Moreover, the tar member mtimes show that Cycles
   5--9 were repackaged on 2025-07-01, long after their processing runs.
3. The pipeline manifest is not one stable XML schema. The first `<image>` has
   only 2 attributes in Cycles 5--7, 42 in Cycles 8--9, 57 in Cycle 10, and 58
   in Cycle 11. Cycle 11 alone in this corpus has a self-referential
   `<manifest .../>` element.
4. Recipe and filename tokens are not execution/outcome proofs. The Cycle 9
   `hifa_calimage` pipescript calls `hifa_renorm`; the Cycle 11
   `hifa_calimage` pipescript calls `hif_selfcal`; and the Cycle 10 and Cycle
   11 selfcal JSON files both contain `scal_targets: []`.
5. Weblog MS views change at Pipeline 2022.2 in this sample: `.ms` plus
   singular `_target.ms` in Cycles 5--8, then `.ms`, plural `_targets.ms`, and
   `_targets_line.ms` in Cycles 9--11. This should be described as a pipeline
   release-era observation, not a universal Cycle 4--8 claim: the sampled
   Cycle 4 dataset is manual and has no weblog views.
6. `pipeline_aquareport.xml` and `applycalQA_outliers.txt` first appear in the
   sampled Cycle 9 / PL2022 weblog. The three sampled
   `applycalQA_outliers.txt` files contain only four threshold settings, not a
   list of detected outliers. Presence of this filename is not evidence that
   an outlier exists.
7. A QA2 SEMIPASS auxiliary tar can be a QA-only shell. The Cycle 9 and Cycle
   10 screening examples explicitly say that no QA0-PASS executions existed,
   no reduction or QA2 procedures ran, and only raw QA0-SEMIPASS data are
   downloadable. Their auxiliary tars contain QA reports and a 32-byte
   product-rename bookkeeping file, but no calibration restore assets.
8. Product FITS grammar is a token family, not a rigid template. The current
   DataLink inventories range from a two-file manual Cycle 4 product to a
   Cycle 11 MT-MFS continuum family with `alpha`, `alpha.error`, `pb.tt0`,
   `tt0.pbcor`, and `tt1.pbcor`. Mask companions are absent for some sampled
   cube/SPW rows.
9. Product FITS are not required to restore the MS. The generic archive README
   says to download the auxiliary tar plus raw ASDM tar(s), then run
   `scriptForPI.py`. Requiring the numbered product tar would add up to tens of
   GB in this corpus without helping the restore.

## Corpus and review method

Primary QA2-PASS package roots:

- `<deleted-scratch>/sample/cycle4/unpacked`
- `<deleted-scratch>/sample/cycle5/pass/unpacked`
- `<deleted-scratch>/sample/cycle6/unpacked`
- `<deleted-scratch>/sample/cycle7/unpacked`
- `<deleted-scratch>/sample/cycle8/unpacked`
- `<deleted-scratch>/sample/cycle9/pass/unpacked`
- `<deleted-scratch>/sample/cycle10/pass/unpacked`
- `<deleted-scratch>/sample/cycle11/unpacked`

Rejected/screening QA2-SEMIPASS roots, used only as counterexamples:

- `<deleted-scratch>/sample/cycle5/unpacked`
- `<deleted-scratch>/sample/cycle9/unpacked`
- `<deleted-scratch>/sample/cycle10/unpacked`

No PDF was converted or parsed again. QA status and report facts were read only
from the existing `*.pdf.txt` files and these already-converted PASS reports:

- `<deleted-scratch>/sample/cycle5/candidate-pass.qa2_report.pdf.txt`
- `<deleted-scratch>/sample/cycle9/candidate-pass.qa2_report.pdf.txt`
- `<deleted-scratch>/sample/cycle10/candidate-pass.qa2_report.pdf.txt`

Outer and nested archives were inspected with `tar -t` / `tar -xO`; weblogs
were not unpacked. JSON was parsed read-only with Node. Product FITS were not
downloaded; their current file inventories came from
`<deleted-scratch>/dl-product-cycle{4..11}.xml`.

The scratch corpus was deliberately left untouched. Cleanup belongs to the
root task that created it.

At handoff, `du -sb` reports exactly **2,942,565,213 bytes** under the
`sample/` corpus and **2,949,046,063 bytes** for the complete scratch root
(candidate CSVs, DataLink XMLs, existing PDF/text pairs, and the sample).
The evidence layout shows a direct archive workflow: TAP candidate results in
`candidates-*.csv`, per-MOUS DataLink VOTables in `dl-*.xml`, selected
`dataPortal` auxiliary/README/QA files, and ordinary tar extraction/member
listing. No `almaBulkManifest.json` or `almaBulkSummary.json` exists anywhere
under the scratch root, so this was not an `alma_bulk_tools`-managed download.
That direct, auxiliary-first method avoided fetching the numbered product and
raw ASDM payloads.

## Per-cycle primary PASS matrix

| Cycle | MOUS; QA2; mode | CASA / pipeline | Recipe evidence | Outer auxiliary grammar | Weblog/MS and helper evidence |
|---|---|---|---|---|---|
| 4 (`2016.1`) | `uid://A001/X879/Xea`; PASS; TM1 manual | CASA 4.7.2; no pipeline | per-EB `scriptForCalibration.py`, `scriptForImaging.py`; no PPR/manifest | 77,199,872 B; 419 members; `calibration`, `log`, a mask under `product`, `qa`, `script`; expanded calibration plots plus `calibration.tgz` | no weblog/auxproducts/listobs; QA is three PNG parts plus a 26 KB text file |
| 5 (`2017.1`) | `uid://A001/X1289/X1a1`; PASS; TM1 B6 FDM | 5.4.0-68 / 42030M | `hifa_calimage` | 205,346,816 B; 19 members; nested auxproducts, session-2 auxcaltables/caltables, flagversions | `.ms`, `_target.ms`; CSV/`cont.dat`; three flag-template families; no machine AQUA/outlier file |
| 6 (`2018.1`) | `uid://A001/X133d/Xb5a`; PASS; TM1 B6 FDM | 5.4.0-70 / 42254M | `hifa_calimage` | 179,041,280 B; 17 members; session-1 layout | same two MS views and three template families; one active QA2 flag command |
| 7 (`2019.1`) | `uid://A001/X1470/X2ef`; PASS; TM1 B6 Mixed | 6.1.1.15 / 2020.1.0.40 | `hifa_calimage` | 171,313,152 B; 17 members; no `log/` member | `.ms`, `_target.ms`; `casa_commands.log` is under `script/`; first sampled weblog PPR copy |
| 8 (`2021.1`) | `uid://A001/X15b8/X11`; PASS; TM1 B7 TDM | 6.2.1.7 / 2021.2.0.128 | `hifa_calimage_renorm` | 151,677,952 B; 15 members; no auxcaltables | `.ms`, `_target.ms`; no flagtarget template; `PLDriver_report.xml`; no machine AQUA/outlier file |
| 9 (`2022.1`) | `uid://A001/X2d20/X3ee5`; PASS; TM1 B6 FDM | 6.4.1.12 / 2022.2.0.64 | `hifa_calimage`, but pipescript calls `hifa_renorm` | 179,034,112 B; 15 members; session-2 caltables | first plural `.ms`, `_targets.ms`, `_targets_line.ms`; first sampled AQUA/outlier files; two active QA2 flags |
| 10 (`2023.1`) | `uid://A001/X3621/X3d68`; PASS; TM1 B6 FDM | 6.5.4.9 / 2023.1.0.124 | `hifa_calimage_selfcal`; pipescript calls `hif_selfcal` | 271,253,504 B; 15 members; `s1.caltables.tgz` | three views; timetracker + selfcal JSON; `scal_targets: []`; AQUA/outlier files |
| 11 (`2024.1`) | `uid://A001/X3788/X7af3`; PASS; TM1 B4 TDM mosaic | 6.6.1.17 / 2024.1.0.8 | plain `hifa_calimage`, but pipescript calls `hif_selfcal` | 228,995,072 B; 15 members; `s1.caltables.tgz` | three views; selfcal/timetracker + pipeline-stats JSON; `scal_targets: []`; SPW names and `Flags: ALLCONT` in `cont.dat` |

All eight PASS samples have exactly one processed EB. This makes them useful
for era comparisons but weak evidence for multi-session/multi-EB behavior.

## SEMIPASS counterexamples

| Cycle | MOUS | Report evidence | Auxiliary contents |
|---|---|---|---|
| 5 | `uid://A001/X1288/X4dd` | QA2 SEMIPASS; execution count 0.00/1 | 1,085,440 B, 7 members: QA2 PDF+HTML, QA0 reports, `scriptForPI.py`, and a 3-byte product-rename file; no caltables/PPR/weblog |
| 9 | `uid://A001/X2d20/X3f39` | QA2 SEMIPASS; 0.00/5; report explicitly says no QA0-PASS EB and no reduction/QA2 procedures | 3,499,008 B, 4 members: QA2 PDF, two QA0 PDFs, 32-byte product-rename file |
| 10 | `uid://A001/X3645/X30e` | QA2 SEMIPASS; 0.00/24; same explicit no-reduction statement | 2,886,656 B, 4 members: QA2 PDF, two QA0 PDFs, 32-byte product-rename file |

The Cycle 5 shell still contains `scriptForPI.py`; therefore even that
filename is not sufficient evidence that a usable restore path exists. Check
for nonempty processed-EB records plus the caltables/flagversions and the QA2
instructions.

## DataLink and download-volume findings

For every primary sample, the outer DataLink has four actual file roles plus
nested service rows:

- README: `semantics=#documentation`
- numbered product tar: `semantics=#this`
- auxiliary tar: `semantics=#auxiliary`
- raw per-EB ASDM tar: `semantics=#progenitor`

The sampled raw rows are `#progenitor`, not `#package`. Parsers should accept
both known values and classify from filename/content as well as semantics.
Nested auxiliary DataLink rows also classify `casa_commands.log` as
`#documentation`, despite its operational role, so DataLink semantics are not
a physical-directory taxonomy.

The numbered `#this` tars are much larger than the auxiliary tars for several
samples:

| Cycle | Numbered product tar bytes | Auxiliary bytes |
|---|---:|---:|
| 4 | 1,190,912 | 77,199,872 |
| 5 | 18,255,991,808 | 205,346,816 |
| 6 | 1,299,414,016 | 179,041,280 |
| 7 | 2,979,919,872 | 171,313,152 |
| 8 | 364,859,392 | 151,677,952 |
| 9 | 4,354,030,592 | 179,034,112 |
| 10 | 56,813,094,912 | 271,253,504 |
| 11 | 22,187,779,072 | 228,995,072 |

Downloading auxiliary first was therefore the efficient empirical-review
choice. Product FITS could be inventoried by recursing through DataLink without
transferring 107 GB of product tars.

### Tar prefix and mtime

Every outer auxiliary member repeats:

```
<project>/science_goal.uid___<SG>/group.uid___<GOUS>/member.uid___<MOUS>/<role>/...
```

The prefix-stripping extraction used for the corpus left the logical role
directories at the sample root. In Cycle 4 it also left an empty
`2016.1.00004.S/science_goal.../group...` chain. An unpacker should tolerate
these empty prefix directories and should not mistake them for a second MOUS.

Tar member mtimes are packaging timestamps, not processing timestamps. The
Cycle 5--9 tars all store 2025-07-01 mtimes, whereas their PPR creation dates
and weblog names range from 2018 to 2022. Use the QA2 report, manifest,
PPR/weblog, and archive release metadata for provenance; do not date reduction
from `tar -tvf`.

## Manifest, PPR, and other machine-readable artifacts

### Pipeline manifest drift

All sampled manifests have a `piperesults/ous` shape and stable high-value
orientation attributes:

- top `<ous name="uid___A001_...">`
- `<casaversion name="...">`
- `<pipeline_version name="...">`
- `<procedure_name name="...">`
- `<session name="...">` with caltable/ASDM children

Beyond that orientation layer, schema drift is large:

| Cycles | First `<image>` attribute count | Salient shape |
|---|---:|---|
| 5--7 | 2 | `name`, `imtype` only |
| 8--9 | 42 | WCS, SPW, intent, object, robust, shape metadata |
| 10 | 57 | adds datatype/format/level/ous, `pl_datatype`, RMS/extrema, arrays/polarization/product type |
| 11 | 58 | adds session to image entries; package/level metadata and a self `<manifest>` entry |

Cycle 5 also has the unusual filename
`member.uid___A001_X1289_X1a1.hifa_calimage.calimage.pipeline_manifest.xml`.
Cycles 5--6 mix MOUS-prefixed and EB-prefixed image names. In Cycle 11, the
self `<manifest>` element's `ous` attribute contains
`uid___A001_X3788_X7af3.hifa_calimage`, not the bare MOUS. The authoritative
MOUS orientation field in this file is the outer `<ous name>`, not every
attribute named `ous`.

### PPR

The PPR tag inventory is the same across the seven pipeline samples, but
values still require careful traversal. Cycle 5 and Cycle 9 both have an empty
`SESSION_1` intent and the EB under `SESSION_2`. Do not create a phantom empty
session or stop at the first `SESSION_*` key. Cross-check nonempty session
values against:

```
DataSet/AsdmIdentifier/AsdmRef/ExecBlockId
DataSet/AsdmIdentifier/AsdmDiskName
```

PPR/manifests enumerate the EBs accepted for this processing run. They are not
necessarily the complete raw MOUS membership: QA0 SEMIPASS/FAIL raw ASDMs may
exist outside the processed set.

Cycles 7--11 weblogs contain a byte-identical PPR copy. Its name is
`PPR_<SBStatusUID>.xml` (for example
`pipeline-20250317T153255/html/PPR_uid___A001_X3788_X7af4.xml`), so the UID in
that basename is neither the MOUS UID nor an EB UID. Cycles 5--6 have no PPR
copy inside their sampled weblogs.

### JSON and PLDriver sidecars

- Cycle 10 and 11 selfcal JSON top keys are `datetime`, `pipeline_version`,
  `scal_targets`, `version`; both have an empty target list.
- Timetracker JSON top keys are `results`, `tasks`, `total`, `weblog`, with
  numbered-stage objects below them.
- Cycle 11 `pipeline_stats_uid___A001_X3788_X7af3.json` has top keys `header`
  and the MOUS UID. Its MOUS object contains `EB`, `SPW`, `TARGET`, bands,
  versions, counts, recipe, project, and representative-target fields; each
  scalar metadata item is itself an object with `longdescription`, `origin`,
  and `value`.
- `PLDriver_report.xml` appears in Cycle 8 with capitalized spelling and in
  Cycles 9--11 as `*.pldriver_report.xml`. All four sampled files contain the
  MOUS, the one ASDM, and an empty `applied_legacy_fixes` element. Presence is
  not proof that a legacy fix was applied.

These are useful inventories, not enough evidence for one universal JSON/XML
schema.

## Weblog and MS-view findings

All Cycle 5--11 weblogs contain both `html/index.html` and `html/t1-1.html`.
The skill should not describe `t1-1.html` simply as an older alternative;
prefer `index.html`, but accept both in the same archive.

| Cycles | listobs views per EB | `pipeline_aquareport.xml` | `applycalQA_outliers.txt` |
|---|---|---:|---:|
| 5--8 | full `.ms`; singular `_target.ms` | absent | absent |
| 9--11 | full `.ms`; plural `_targets.ms`; `_targets_line.ms` | one | one |

The `sessionsession_1` / `sessionsession_2` directory duplication is real.
Listobs is metadata for a particular view, but it does not list MS table
columns. The CASA command logs supply the column evidence:

- Cycle 5 and Cycle 8 `mstransform` read the full `.ms` with
  `datacolumn='corrected'` and write `_target.ms`.
- Cycle 9--11 do the same into `_targets.ms`.
- The written calibrated values therefore land in output `DATA`, per CASA's
  `mstransform` contract.
- In Cycle 9, `hif_uvcontfit`/`applycal` writes the continuum-subtracted result
  to `_targets.ms` `CORRECTED_DATA`, then `mstransform(...,
  datacolumn='corrected')` copies it into `_targets_line.ms` `DATA`. That
  Cycle 9 target-view `CORRECTED_DATA` is not a selfcal result.
- In Cycles 10--11, `uvcontsub(..., datacolumn='data')` directly creates
  `_targets_line.ms` `DATA`; optional later selfcal may add a corrected column.

Do not infer actual column presence from listobs or the suffix alone; inspect
the MS or the command log/script.

The three sampled AQUA XML files (Cycles 9--11) share the same observed tag
set, including `QaPerStage`, `QaPerTopic`, `FinalScore`, `RepresentativeScore`,
and `SensitivityEstimates`. That consistency across three fixtures does not
establish a public stable schema.

The three `applycalQA_outliers.txt` files contain exactly:

```
AMPLITUDE_SLOPE_THRESHOLD: 25.0
AMPLITUDE_INTERCEPT_THRESHOLD: 53.0
PHASE_SLOPE_THRESHOLD: 40.0
PHASE_INTERCEPT_THRESHOLD: 60.5
```

The filename is best treated as applycal-QA configuration/evidence, not a
boolean "outliers found" marker.

## Auxproducts and calibration-package evolution

All Cycle 5--11 auxproducts contain `antennapos.csv`, `cont.dat`, `flux.csv`,
`flagtemplate.txt`, and `flagtsystemplate.txt`. Sample-specific evolution:

- `flagtargetstemplate.txt`, `*.auxcaltables.tgz`, and
  `_target.ms.auxcalapply.txt` occur in Cycles 5--7 and are absent in Cycles
  8--11.
- Cycle 8 onward uses only the main caltables archive in this corpus.
- Caltable filenames change from `session_1`/`session_2` through Cycle 9 to
  compact `s1.caltables.tgz` in Cycles 10--11.
- JSON first appears in Cycle 10; pipeline-stats JSON appears in Cycle 11.
- Cycle 11 still contains `antennapos.csv`; this sample cannot validate the
  PL2025 per-EB antenna-position JSON delivery change because Cycle 12 had no
  public sample.

`cont.dat` also evolves empirically:

- Cycles 5--6 use `Field`, numeric `SpectralWindow`, and frequency ranges.
- Cycles 7--10 also use bare `ALL` where appropriate.
- Cycle 11 adds the full SPW name after the ID and `Flags: ALLCONT`.

Every sampled `cont.dat` is nonempty, so the empirical corpus cannot test
present-but-empty versus omitted-SPW behavior. That distinction remains a
documentation-based rule rather than a package-fixture result.

Active, non-comment flag-template lines occur in Cycle 6 (one), Cycle 8 (one
each in flag and Tsys templates), and Cycle 9 (two); all other primary sample
templates contain comments/examples only. Count nonblank, non-comment lines;
do not count example `mode='manual'` text in comments.

## Product DataLink grammar

The product DataLink file-row counts are 2, 66, 51, 62, 70, 54, 49, and 51
for Cycles 4 through 11 respectively. These are inventories of current
archive products, not downloaded FITS or proof of original-cycle packaging.

Key grammar observations:

- Cycle 4 uses
  `Mira_sci.spw0_1_2_3.mfs.I.manual.B6.image.pbcor.fits` and a PB file; no
  calibrator images occur in this two-row inventory.
- Cycles 5--10 show the familiar `_sci`, `_bp`, `_ph`, `_chk`, `cube`, `mfs`,
  aggregate `cont`, optional `repBW`, and `mask`/`pb`/`pbcor` families.
- Missing companion masks are real in the DataLink inventories. For example,
  several Cycle 8, Cycle 10, and Cycle 11 science cubes have PB and pbcor rows
  but no corresponding mask row. A downloader must not require a strict
  three-file set.
- Cycle 11's aggregate continuum uses MT-MFS variants:
  `cont.I.alpha.fits`, `cont.I.alpha.error.fits`, `cont.I.pb.tt0.fits`,
  `cont.I.tt0.pbcor.fits`, and `cont.I.tt1.pbcor.fits`. The layer/token order
  differs from simple `.pbcor`/`.pb` positional assumptions.
- Archive-renamed Cycle 10/11 product names omit `regcal`, while manifest
  `pl_datatype` retains regular-calibration state.

FITS headers were not inspected, so this review does not claim anything
about WCS, BUNIT, beam keywords, or data axes for these specific products.

## Concrete edits recommended for the skill

These are edit targets for the root implementer; this subtask did not edit the
skill/reference files.

### `SKILL.md`

1. In the MS-view guardrail, replace the categorical "Cycles 4--8" wording
   with a release-aware statement: sampled pipeline packages PL2018--PL2021
   use `_target.ms`; PL2022+ uses `_targets.ms` and `_targets_line.ms`; manual
   packages may have neither.
2. In the logical package tree, change `pbcor + pb + mask` from an implied
   required triple to common product families whose companions can be omitted.
3. Add the SEMIPASS counterexample: QA reports and even `scriptForPI.py` do
   not prove a restorable/calibrated package.

### `references/identifiers-and-packaging.md`

1. Add DataLink `#progenitor` as the observed raw-ASDM semantics in all eight
   primary fixtures. Continue accepting `#package` and filename-based
   classification.
2. State that current archive tar mtimes can record repackaging, not pipeline
   execution or observation time.
3. Update weblog landing guidance: every sampled Cycle 5--11 weblog has both
   `index.html` and `t1-1.html`; prefer `index.html`, fall back to either.
4. Mention that `casa_commands.log` can be in `script/` (Cycle 7 fixture) and
   can carry DataLink `#documentation`; path/semantics are not fixed role
   contracts.
5. Add the empirically observed helper transition and caltable filename drift,
   but label it as sample evidence rather than a guaranteed cycle boundary.

### `references/products-and-qa.md`

1. Correct `applycalQA_outliers.txt`: it may contain thresholds only; file
   presence does not prove outliers.
2. In restore instructions, require auxiliary + raw ASDM(s); make numbered
   FITS product tar(s) optional/reference products. This avoids unnecessary
   18--57 GB downloads in the Cycle 5/10/11 examples.
3. Say that manifest/PPR EB lists are the processed/accepted EBs, not complete
   raw membership. Cross-check against ObsCore/QA0 rather than expecting
   unconditional equality.
4. Add the manifest schema counts and advise parsing top `<ous name>` plus
   version/procedure/session orientation fields first, preserving unknown
   elements/attributes.
5. Add PPR empty-session handling and the weblog PPR filename warning.
6. Record that PLDriver report presence is not legacy-fix evidence and casing
   differs.
7. Expand the product token examples with the observed Cycle 11 MT-MFS
   family, and explicitly allow missing masks.
8. Keep selfcal wording outcome-based: recipe/pipescript/JSON presence is only
   attempt/request evidence, and an empty `scal_targets` is a concrete no-target
   outcome.

### `references/asdm-and-ms.md` and `references/listobs-and-intents.md`

1. Qualify the singular/plural view era by pipeline release and manual versus
   pipeline reduction, not observation cycle alone.
2. Cite CASA command-log evidence for the output `DATA` conclusion; listobs
   itself cannot inventory columns.
3. Split Cycle 9 column semantics from Cycle 10+: Cycle 9 `_targets.ms`
   `CORRECTED_DATA` can be the continuum-fit result, while later optional
   selfcal may use corrected columns differently.
4. Do not say `_targets.ms` may be absent merely because selfcal has no
   successful targets: both Cycle 10 and 11 fixtures have a target view/listobs
   while selfcal JSON is empty. Separate target-view creation from selfcal
   outcome.

### `references/pipeline-history.md`

1. Add empirical anchors for the exact sampled build strings and their
   manifests.
2. Note the observed divergence between recipe label and internal stage list:
   PL2022 plain `hifa_calimage` includes `hifa_renorm`; PL2024 plain
   `hifa_calimage` includes `hif_selfcal`.
3. Preserve the distinction between "stage exists/called", "target eligible",
   and "result accepted".

## Sample-selection limitations

- One PASS MOUS per Cycle 4--11 is a small convenience sample, not a census.
- Every primary PASS sample is TM1 and has exactly one processed EB. No ACA,
  TP, multi-EB, band-to-band, full-polarization, ephemeris, solar, VLBI, or
  phased-array package is represented.
- Bands are concentrated in Band 6: B6 in Cycles 4--7, 9, and 10; B7 in Cycle
  8; B4 in Cycle 11. Bands 1--3, 5, and 8--10 are absent.
- Spectral modes are mostly FDM; Cycle 7 is Mixed and Cycles 8/11 are TDM.
- Cycle 4 is the only manual PASS package. Cycles 5--11 are pipeline
  calibration+imaging examples. This cannot determine the within-cycle
  prevalence of manual processing.
- Cycle 11 is a mosaic according to manifest `obspatt="mos"`, providing one
  mosaic example but not a comparison set.
- Product evidence is DataLink filename/size metadata only. No product FITS
  payload or header was downloaded.
- The current archive can repackage or re-deliver old observations. These
  fixtures compare the packages visible on 2026-07-18, not immutable original
  delivery snapshots.
- The three screening SEMIPASS examples were selected because they were bad
  primary candidates; they should not be treated as representative of every
  SEMIPASS mode.
- `<deleted-scratch>/candidates-2025.csv` contains only its header
  (mtime 2026-07-18 03:03 UTC). Thus no public `2025.1` / Cycle 12 MOUS was
  available in that TAP result, and PL2025 packaging claims remain
  documentation-only here.

## Historical read-only command log

The following commands record how the core inventories were produced without
unpacking more data or converting PDFs. They require redownloading the listed
MOUSs first and replacing the `<deleted-scratch>` placeholder; the original
scratch no longer exists.

### Validate the TSV

```bash
awk -F '\t' '{print NR, NF}' /workspace/working-with-alma-data/review/sample-inventory.tsv
```

Every line should report 24 fields.

### Outer tar counts, sizes, and role directories

```bash
find <deleted-scratch>/sample -type f -name '*_auxiliary.tar' -print0 |
  sort -z |
  while IFS= read -r -d '' archive; do
    stat -c '%n\t%s' "$archive"
    tar -tf "$archive" | wc -l
    tar -tf "$archive" |
      awk -F/ 'NF >= 5 {print $5}' |
      sort -u
  done
```

### Nested archive member lists without extraction

```bash
find <deleted-scratch>/sample -type f -name '*.tgz' -print0 |
  sort -z |
  while IFS= read -r -d '' archive; do
    printf '%s\t' "$archive"
    tar -tzf "$archive" | wc -l
  done
```

### Weblog paths and view transition

```bash
find <deleted-scratch>/sample -type f -name '*.weblog.tgz' -print0 |
  sort -z |
  while IFS= read -r -d '' weblog; do
    echo "$weblog"
    tar -tzf "$weblog" |
      rg '(index\.html|t1-1\.html|listobs\.txt|pipeline_aquareport\.xml|applycalQA_outliers\.txt)$'
  done
```

### Read the in-weblog AQUA/outlier files without extracting them

```bash
weblog=<deleted-scratch>/sample/cycle11/unpacked/qa/member.uid___A001_X3788_X7af3.hifa_calimage.weblog.tgz
aqua=$(tar -tzf "$weblog" | rg 'pipeline_aquareport\.xml$')
outliers=$(tar -tzf "$weblog" | rg 'applycalQA_outliers\.txt$')
tar -xOzf "$weblog" "$aqua" |
  rg -o '<\/?[A-Za-z_][A-Za-z0-9_:.-]*' |
  sed 's#^</\?##' |
  sort -u
tar -xOzf "$weblog" "$outliers"
```

### PPR sessions and processed EB records

```bash
rg -n -A1 -B1 \
  '<Keyword>SESSION_|<ExecBlockId>|<AsdmDiskName>|<ProcedureTitle>' \
  <deleted-scratch>/sample/cycle{5,6,7,8,9,10,11}/{pass/,}unpacked/script/*.pprequest.xml
```

The brace command also sees a screening tree where one exists; filter to the
primary roots listed above when producing a strict PASS-only table.

### MS output-column evidence

```bash
rg -n -B4 -A8 "outputvis='[^']*_(target|targets).*\.ms'" \
  <deleted-scratch>/sample/cycle5/pass/unpacked/log/*.casa_commands.log \
  <deleted-scratch>/sample/cycle8/unpacked/log/*.casa_commands.log \
  <deleted-scratch>/sample/cycle9/pass/unpacked/log/*.casa_commands.log \
  <deleted-scratch>/sample/cycle10/pass/unpacked/log/*.casa_commands.log \
  <deleted-scratch>/sample/cycle11/unpacked/log/*.casa_commands.log
```

Cycle 7's command log is under `script/`, an intentional path counterexample.

### JSON top-key inventory

```bash
find <deleted-scratch>/sample/cycle{10,11} -type f -name '*.json' -print0 |
  while IFS= read -r -d '' json_file; do
    node -e \
      'const fs=require("fs"); const x=JSON.parse(fs.readFileSync(process.argv[1], "utf8")); console.log(process.argv[1], Object.keys(x).sort())' \
      "$json_file"
  done
```

### DataLink rows without downloading payloads

```bash
node -e '
const fs = require("fs");
for (const file of process.argv.slice(1)) {
  const xml = fs.readFileSync(file, "utf8");
  console.log("FILE", file);
  for (const row of xml.matchAll(/<TR>([\s\S]*?)<\/TR>/g)) {
    const cells = [...row[1].matchAll(/<TD(?:\s*\/|>([\s\S]*?)<\/TD)>/g)]
      .map(x => (x[1] || "").replace(/<[^>]+>/g, "").trim());
    if (cells[1]) console.log(cells[4], cells[7], cells[8], cells[1].split("/").pop());
  }
}
' <deleted-scratch>/dl-short-2024-uid___A001_X3788_X7af3.xml
```

### Confirm Cycle 12 absence in this query result

```bash
wc -l <deleted-scratch>/candidates-2025.csv
sed -n '1,2p' <deleted-scratch>/candidates-2025.csv
stat -c '%y %n' <deleted-scratch>/candidates-2025.csv
```

### Confirm scratch size and that cleanup has not occurred

```bash
du -sb <deleted-scratch>/sample
du -sb <deleted-scratch>
find <deleted-scratch> -type f \
  \( -name 'almaBulkManifest.json' -o -name 'almaBulkSummary.json' \) -print
```

At handoff the first two commands returned 2,942,565,213 and 2,949,046,063
bytes respectively; the manifest search returned no paths. No cleanup command
was run by this reviewer.
