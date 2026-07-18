# Pipeline-domain findings

Review date: 2026-07-18. Sources and orientation are registered in
[`sources-pipeline.md`](sources-pipeline.md); the release matrix is in
[`pipeline-history-draft.md`](pipeline-history-draft.md).

Resolution note: this is the pre-edit evidence ledger. Proposed edits were
adjudicated into `references/pipeline-history.md` and the other current
references; do not reapply them mechanically. See `final-audit.md` for the
post-edit audit.

Findings use the review-plan vocabulary: **CONFIRMED**, **CONTRADICTED**,
**NEW**, and **STALE-RISK**. Page/section references are to the official
release documents, not to temporary text-conversion line numbers.

## Executive result

The skill's central guardrail is correct: pipeline capability follows the
pipeline release used to process a MOUS, not its proposal cycle. The release
history can now be anchored to exact operations intervals from 4.7.0 through
2025.1.0.37, with one unresolved block: official sources disagree about
initial and later operational builds in Cycle 6, Cycle 7, and 2020.1.

The largest substantive correction needed in the existing skill is the
status of regular-calibration image filenames in PL2023+: the PL2023 User's
Guide says both `Regcal` and `selfcal` specifiers were added to filenames.
The present claim that regcal variants are generally not archived is not
supported by that official release document and should be removed or narrowed
to a package-corpus observation with a query.

## CONFIRMED

### C1. Release, not observing cycle, is the primary historical key

- **Claim checked:** `cycle-capabilities.md` says a project's cycle does not
  pin the pipeline version and restore work must read CASA/pipeline versions
  per MOUS.
- **Evidence:** the live operations table gives dated releases; every User's
  Guide reports the versions visible in the weblog; the page explicitly says
  exact reproduction requires the same CASA+pipeline release, while the
  latest accepted pipeline can restore many earlier pipeline products.
- **Verdict:** **CONFIRMED** with one refinement: say “same version for
  identical reproduction”; do not imply that exact version is always required
  merely to restore calibrated data.
- **Provenance:** operational page; UG47–UG25 §1/version sections.

### C2. The pipeline operates on a MOUS, not on an archive row or GOUS

- Every reviewed guide defines the pipeline input as the set of QA-assured EBs
  belonging to one Scheduling Block, collected as a MOUS. Current guides also
  say different array-component MOUSs are not officially combined by the
  pipeline.
- **Verdict:** **CONFIRMED**. This supports the skill's MOUS-as-atomic-unit
  guardrail and the warning not to infer GOUS combination.
- **Provenance:** UG47–UG25 §1.2; H23 §2.

### C3. Pipeline calibration began in 2014; operations imaging and delivery came later

- H23 retrospectively dates the calibration pipeline to 2014-10, after Cycle
  2 began. It dates imaging-pipeline introduction to Cycle 4 and says the first
  automated image products were delivered in 2017.
- **Verdict:** **CONFIRMED**, and sharper than “official pipeline from late
  2014 / imaging from ~Cycle 4.” Keep all three events separate: calibration
  operations, imaging-recipe introduction, and first delivered automated
  images.
- **Provenance:** H23 abstract and §1, retrospective through PL2022.

### C4. Product-size mitigation starts in the 4.7.2 patch

- UG472 §2.2 and RM472 establish `hif_checkproductsize`; the guide gives
  default trigger estimates of 30 GB for a cube and 400 GB total, and lists
  mitigation through channel/cell size, field of view, and number of sources.
- **Verdict:** **CONFIRMED**. This is strong historical backing for the skill's
  “delivered FITS are not complete science content” guardrail.
- **Status:** exists + in recipe + default for the release; the particular
  mitigation selected remains a runtime outcome.

### C5. Helper files and auxiliary packaging are release-dependent

- `flux.csv`, `antennapos.csv`, flag templates, and `cont.dat` are documented
  from the Cycle-4 guides. UG54 says flag templates, including the Tsys
  template, are exported in `auxproducts.tgz`. UG24 adds SPW names to
  `cont.dat`. UG25 changes the default antenna-position path to online-queried,
  per-EB `uid*antennapos.json`, while retaining CSV as a manual override.
- **Verdict:** **CONFIRMED** for helper semantics and format transition.
- **Caveat:** “pipeline uses/writes this in its working directory” is not
  automatically “every archive package delivers it.” Package samples must
  establish prevalence.

### C6. Renormalization changes semantics at PL2024

- UG21 introduces `hifa_renorm` before export. H23 explains its purpose for
  strong target lines in autocorrelation spectra. UG24 states that PL2024 no
  longer edits the MS directly; it creates a Tsys-like caltable applied by
  `hif_applycal` and restore.
- **Verdict:** **CONFIRMED**. A restore/parser should expect a processing and
  calibration-product consequence at the PL2023→PL2024 boundary, not merely
  a renamed stage.
- **Status:** recipe at PL2021; application depends on its heuristic result;
  caltable implementation from PL2024.

### C7. The three target-MS views are a 2022 processing-layout change

- UG22 says `_targets.ms` contains calibrated target data without continuum
  subtraction and `_targets_line.ms` contains continuum-subtracted data. H23
  §9.2 explains that this freed `CORRECTED` for future selfcal.
- **Verdict:** **CONFIRMED** for processing workspace semantics.
- **Important prohibition:** these are not standard archive-delivered
  calibrated MSs. The same guides say calibrated MSs are produced during a
  pipeline run but not stored in the archive.

### C8. Selfcal is conditional, not a blanket Cycle-10 product property

- UG23 introduces `hif_selfcal` for single-field, non-ephemeris targets and
  says it attempts each source. Gains are used only if S/N improves without an
  unacceptable beam change; only successful sources are imaged as selfcal.
  UG24 adds outcome QA. UG25 expands support to mosaics and long-baseline
  fallback.
- **Verdict:** **CONFIRMED with required wording refinement**. “Selfcal in the
  pipeline from Cycle 10” is acceptable shorthand only if followed by:
  attempted ≠ succeeded ≠ applied ≠ delivered.
- **Agent trap:** a stage, script call, or selfcal sidecar proves an attempt,
  not a successful application.

### C9. Polarization support has three distinct milestones

1. PL2020: polarization-friendly total-intensity calibration, flagging, fixed
   reference antenna; instrumental polarization still solved manually.
2. PL2023: `hifa_polcal` solves instrumental polarization and reports QA.
3. PL2025: IQUV science-target imaging enters polarization recipes.

- **Verdict:** **CONFIRMED**. Any single claim that “full polarization was
  supported from year X” is underspecified; calibration-friendly,
  instrumental-pol solving, and delivered IQUV imaging are different states.
- **Provenance:** UG20 §3.1; UG23 §3.1; UG25 §3.1–3.2.2.

### C10. B2B pipeline calibration is a PL2024 milestone

- UG24 adds `hifa_diffgaincal` and new `hifa_cal_diffgain` /
  `hifa_calimage_diffgain` recipes, with explicit LF/HF SPW mapping. RM24 is
  the first sampled Reference Manual containing the task.
- **Verdict:** **CONFIRMED** for correct-metadata B2B data. This explains
  `DIFFGAIN` intents and two-band calibrator/product evidence, but does not say
  every older B2B MOUS can be rerun automatically.

### C11. The .36→.37 patch has a package-visible omission

- The living PL2025 known-issues page says 2025.1.0.36 could export only one
  of the repBW and full-resolution cubes for the same source/SPW; .37 fixes
  it. The operations table dates .36 to 2026-03-05–2026-05-01 and .37 from
  2026-05-01.
- **Verdict:** **NEW, high-value confirmation** for future package diagnosis:
  a .36 package missing one of those two cubes can reflect the pipeline export
  defect, not an interrupted download.
- **Provenance:** OPS + KI, independent official provenance types.

## CONTRADICTED

### X1. “Regular `regcal` variants are generally not archived”

- **Existing text:** `products-and-qa.md` says Cycle-10+ selfcal adds a
  `selfcal` token and “the internal `regcal` variants are generally NOT
  archived for regular products.”
- **Contrary evidence:** UG23 §3.2.2 says the specifier `Regcal` or `selfcal`
  is now in filenames and, more importantly, the image datatype is in
  `manifest.xml` entries. UG24 says the datatype `(regcal or selfcal)` is
  displayed next to each source.
- **Verdict:** **CONTRADICTED / at minimum materially overbroad**.
- **Proposed fix:** say: “PL2023+ distinguishes regular-calibration and
  successful-selfcal images in filenames/manifest datatype. Do not assume
  both variants are delivered for every target; inspect the manifest and
  package.” If a corpus finds current omission of regcal counterparts, record
  the query, releases, and fraction rather than making it a timeless rule.

### X2. Current official history conflicts with contemporaneous deployment records

- OPS says 5.6.1 ran until 2021-05-10 and lists only 2020.1.0-40 thereafter.
  UG20 says 2020.1.0.36 / CASA 6.1.1-10 was deployed in operations in
  2020-11; H23 independently describes the Python-3/second Cycle-7 release as
  2020-10.
- UG54 says initial operations build 42030 / CASA 5.4.0-68; OPS collapses the
  interval under CASA 5.4.0-70 with r42254 and r42866. UG56 names r42833;
  OPS names r42866.
- **Verdict:** **CONTRADICTED official sources; unresolved**. The skill must
  not manufacture exact transition dates for these internal builds. Use the
  version in each package as authoritative and retain the conflict note in the
  history.

### X3. CASA 6.4.2 in the 2022 package string

- UG22/UG22P print `casa-6.4.2-12-pipeline-...` in one package-string line,
  but both call the bundled and weblog CASA version 6.4.1-12; OPS and RM22
  agree on 6.4.1.
- **Verdict:** **internal document contradiction, almost certainly a typo**.
  Record CASA 6.4.1-12 and mention the typo; do not create a phantom CASA
  6.4.2 operations row.

## NEW facts worth integrating

### N1. Manifest schema gained precise archive renaming in PL2021

- UG21 says `hif_exportdata` added manifest elements allowing the archive to
  rename FITS images precisely. UG23 later adds `regcal`/`selfcal` datatype
  and image name to manifest/AQUA output.
- **Why useful:** filename grammar is explicitly versioned, and manifest
  parsing should be release-aware. This is better evidence than treating the
  filename as a timeless grammar.

### N2. `cont.dat` becomes more explicitly cross-EB in PL2024

- UG24 says SPW names were added to `cont.dat` to match per-EB real SPW IDs.
- **Why useful:** parsers should tolerate older files without SPW names and
  newer files with virtual/real-SPW mapping information; this reinforces the
  skill's “SPW number is not spectral identity” guardrail.

### N3. Renorm restore is faster and semantically different from PL2024

- Because renorm is a caltable rather than an in-place MS edit, it can be
  applied by `hifa_restoredata`. Restore workflows and calibration-product
  inventories should branch on pipeline version before interpreting renorm
  evidence.

### N4. PL2025 full-pol products add explicit imaging-stage variants

- UG25 names `mfs_fullpol`, `cont_fullpol`, `cube_fullpol`, and
  `cube_repBW_fullpol` variants and says `hif_checkproductsize` accounts for
  the added volume.
- **Why useful:** an IQUV MOUS can be more aggressively mitigated; missing
  cubes/targets must be checked against the product-size stage and manifest.

### N5. PL2025 AQUA sensitivity tags are undergoing a schema transition

- UG25 says both IF and SD now report
  `ObservedSensitivityJyPerBeam` and `TheoreticalSensitivityJyPerBeam`; the
  ambiguous `SensitivityJyPerBeam` is announced for removal in PL2026.
- **Why useful:** AQUA parsers should accept both old and new local tag names,
  prefer the explicit pair, and not require the deprecated field.
- **Orientation:** the PL2025 tags are deployed; PL2026 removal is
  **prospective** and must be rechecked after release.

## STALE-RISK / evidence gaps

### S1. Runtime JSON and report sidecars

No reviewed User's Guide or Reference Manual establishes the archive delivery
or first release of:

- `pipeline-*.selfcal.json`;
- `pipeline-*.timetracker.json`;
- `pipeline_stats_*.json`;
- `*.pldriver_report.xml`.

The corresponding passages in `products-and-qa.md` may be accurate package
observations, but they need typed corpus provenance. Until then, label them
“observed in sampled PL20xx packages,” not universal release facts. In
particular, file presence must not be equated with selfcal success.

### S2. Exact `pipeline_manifest.xml` filename and schema stability

The guides consistently discuss `*manifest.xml` and its growing content, but
do not guarantee the exact archive filename or a stable XML schema across all
releases. Local-name, namespace-tolerant parsing remains appropriate; package
samples should establish filename prevalence.

### S3. Antenna-position JSON delivery prevalence

UG25 proves per-EB JSON helper semantics and default online acquisition, but
not that every archive delivery contains the JSON. The current skill's
“expect one or the other” should be qualified as an observed/current pipeline
pattern until the package sample confirms it across representative MOUSs.

### S4. README → QA2-report transition

Pipeline guides progressively refer to the QA2 report, but this review did not
establish a precise release at which README became only a pointer. Use the
cycle-specific QA2 Data Products documents and sampled packages for this
history; do not date the transition from guide wording alone.

### S5. Known-issues pages are living records

The current page includes old releases but is continuously edited. It proves
that an issue is recognized now, not when it was first recognized. Historical
claims such as “users knew this during Cycle N” require a frozen page or
release guide.

### S6. Patch deltas are incomplete where no patch guide exists

There are explicit patch notes for 4.7.2 and 2022.2.0.68, and a known-issues
statement for 2025.1.0.37. No separate official delta document was found for
2020.1.0.40, 2025.1.0.36, or the Cycle-6/7 internal build transitions. The
matrix deliberately labels those gaps.

## Proposed application to the skill

1. Replace the dense “Pipeline / CASA coupling” paragraph in
   `cycle-capabilities.md` with a short pointer to a release-first pipeline
   history and retain only the read-the-version-per-MOUS guardrail.
2. Add the operations matrix from `pipeline-history-draft.md`, preserving the
   initial-build conflicts and status vocabulary.
3. Rewrite the `regcal`/`selfcal` filename paragraph as described in X1.
4. Mark runtime JSON/statistics claims empirical unless package sampling adds
   a reproducible inventory.
5. Add PL2025 AQUA explicit sensitivity tags and prospective PL2026 deprecation
   to the tolerant-parser guidance.
6. Add the PL2025.1.0.36 repBW/full-resolution export omission as a narrow,
   versioned package diagnostic.
