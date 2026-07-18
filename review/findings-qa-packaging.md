# Findings: QA, products, packaging, restores, MSs, mosaics, and TP

Review date: 2026-07-18. Source IDs refer to
[`sources-qa-packaging.md`](sources-qa-packaging.md). Proposed edits are written
for the existing concise, agent-trap-oriented style. No existing skill or
reference file was edited in this review lane.

Resolution note: this file preserves the pre-edit evidence and recommendations
from that lane. The root task subsequently applied/adjudicated them in the
current skill references; do not reapply them mechanically. See
`final-audit.md` for the post-edit audit.

## Executive adjudication

Most core guardrails are sound. The highest-value fixes are:

1. Separate **archive tar bundling era** from **unpacked member-tree layout**.
   Cycle 1–4 products were mostly bundled in numbered tars; the split FITS plus
   auxiliary pattern is Cycle 5+, even though a standardized member tree and
   five directory roles span a broader period.
2. Replace the apparently positional FITS-name “grammar” with a token-family
   grammar. The official Cycle 12 prose and its own examples disagree on where
   `selfcal` sits, proving that parser logic must not depend on one exact token
   order.
3. Add the official MS-view/column table: Cycle 4–8 `_target.ms`, Cycle 9+
   `_targets.ms`, Cycle 9+ `_targets_line.ms`, and their different `DATA` /
   `CORRECTED_DATA` meanings.
4. Add `cont.dat`'s PL2023→PL2024 format break and the crucial distinction
   between an omitted SPW and a present-but-empty SPW.
5. Add legacy restore traps (pre-2017 missing manifests and `.tar.gz` versus
   `.tgz`) and ARI-L's status as an external, non-QA2 Cycle 2–4 product family.
6. Soften several highly detailed JSON/XML assertions that are supported by a
   local package corpus but not by a stable official contract.

## CONFIRMED

### C01 — MOUS is the processing/delivery/QA2 unit; EBs remain raw units

**Claim checked:** `SKILL.md` hierarchy and `products-and-qa.md` QA ladder.

**Evidence:** Every QA2 Products edition reviewed states that QA2 is conducted
on one MOUS, comprising executions of one SB; raw data are separate per-EB
ASDMs. The current Cycle 12 document states that a GOUS contains the 12-m, 7-m,
and TP MOUSs intended for combination (S01–S07, sections 1–3).

**Adjudication:** CONFIRMED. Keep the MOUS/EB distinction prominent.

### C02 — Standard delivery does not normally contain a calibrated MS

**Claim checked:** `SKILL.md` guardrail 4; `products-and-qa.md` restore section;
`identifiers-and-packaging.md` early-cycle exception.

**Evidence:** S01–S07 describe products/supporting material plus optional raw
ASDMs, with calibrated MS regeneration via `scriptForPI.py`. S09 and S01/S06
state that Cycle 0 and early Cycle 1 packages could include ready-to-use MSs.
S16 describes both ARC-provided calibrated MSs and local regeneration.

**Adjudication:** CONFIRMED with the already-stated Cycle 0/early Cycle 1 and
external-service exceptions.

### C03 — Five role directories and README are the standard member-level model

**Claim checked:** `SKILL.md` and `identifiers-and-packaging.md` tree.

**Evidence:** S01–S06 and S08 give `calibration`, `log`, `product`, `qa`,
`script`, and README; `raw` appears only when separately retrieved. S07 still
has a README section and README tar description, although its overview diagram
counts only the five directories and raw. S08 warns that `product/` is present
only if image products were selected for download.

**Adjudication:** CONFIRMED as a **logical full-delivery model**, not as a
guarantee that every partial Request Handler selection creates every directory.

**Proposed edit — `identifiers-and-packaging.md`, current-style tree:**

> This is the logical full QA2 delivery tree. A Request Handler selection can
> omit `product/` (or other unselected material), so absence of a directory in a
> partial download is not evidence that processing did not produce it.

### C04 — README authority changed around Cycle 5

**Claim checked:** `products-and-qa.md` QA prohibitions and packaging reference.

**Evidence:** S01–S07 section 4.1 say that through Cycle 4 README summarized QA2
and package structure; since Cycle 5 it normally points to the QA2 report. S31
uses README for Cycle 1–4 and QA2 report for Cycle 5+ when locating the CASA
version.

**Adjudication:** CONFIRMED. Prefer “normally” rather than an unconditional
transition because re-deliveries and special/manual packages vary.

### C05 — Delivered FITS images are incomplete/best-effort and mitigation can
omit content

**Claim checked:** `SKILL.md` guardrail 7 and `products-and-qa.md`.

**Evidence:** Every reviewed QA2 document labels imaging best-effort and says
high spatial/spectral-resolution data may be mitigated. S08 and S16 explicitly
say images may omit fields, SPWs, or bandwidth; S12 says products may be binned,
field-limited, or source-limited.

**Adjudication:** CONFIRMED. “Calibrated visibilities contain more” is sound as
an operational warning, but avoid implying every excluded datum is scientifically
usable; flags and QA must still be inspected.

### C06 — PPR presence is the supported pipeline/manual discriminator

**Claim checked:** `products-and-qa.md` recipe discussion.

**Evidence:** S01–S07 state that `*PPR*.xml` or `*pprequest.xml` in `script/`
identifies pipeline-calibrated data. They list per-EB
`*.scriptForCalibration.py` / `*.scriptForSDCalibration.py` for manual
reductions.

**Adjudication:** CONFIRMED. Presence is positive evidence. Absence alone should
remain a heuristic because partial downloads can omit `script/`.

### C07 — `auxproducts` helper files and manual-mode flag caveat

**Claim checked:** `products-and-qa.md` flag templates and
`identifiers-and-packaging.md` nested archives.

**Evidence:** S03–S07 list `flux.csv`, antenna-position files, `cont.dat`, and
the three flag-template families under `auxproducts.tgz`. S12 sections 7.4–7.6
show that templates accept CASA `flagdata` commands, commonly
`mode='manual'`, and are consumed automatically by pipeline tasks. S12 describes
them as manually identified/user/operator inputs.

**Adjudication:** CONFIRMED. `mode='manual'` names the CASA flagging mode; it is
not an authorship field. The skill's prohibition on asserting human intervention
from that token alone is correct.

### C08 — `cont.dat` frequencies are LSRK and feed continuum imaging/subtraction

**Claim checked:** both packaging and product references.

**Evidence:** S12 section 7.6 explicitly says delivered `cont.dat` lists LSRK
frequency ranges used for per-SPW and aggregate continuum imaging and for cube
continuum subtraction. S25 independently describes ALMA's fixed TOPO observing
frequency and common LSRK regridding.

**Adjudication:** CONFIRMED, but see N01 for versioned grammar and empty-entry
semantics.

### C09 — PB-corrected image relation and TP PB difference

**Claim checked:** `products-and-qa.md` FITS section.

**Evidence:** S01–S07 section 4.3 state that interferometric deliveries include
`pbcor` and PB (`pb` or historical `flux`) images, and that the uncorrected image
is recovered by multiplication. They state that TP products already account for
antenna-beam convolution and do not use the same PB correction.

**Adjudication:** CONFIRMED. `flat = pbcor × pb` is correct. “Flat-noise” is a
useful colloquial description but “PB-uncorrected image” is less ambiguous.

### C10 — QA levels must not be conflated with pipeline stage scores

**Claim checked:** `products-and-qa.md` QA ladder and AQUA prohibition.

**Evidence:** S06/S07 distinguish EB-level QA0, observatory QA1, MOUS-level QA2,
and human review/approval of products. S12 describes WebLog QA scores as
at-a-glance stage diagnostics reviewed as part of QA. S24 documents QA0
SemiPass/raw behavior. This supports the inference that a pipeline score is not
the final QA2 disposition.

**Adjudication:** CONFIRMED. Label the conclusion as a synthesis of the two
official contracts rather than claiming a single XML field states it.

### C11 — Restore tree, script selection, CASA version, and disk multipliers

**Claim checked:** `products-and-qa.md` restore steps and `asdm-and-ms.md` size
expectations.

**Evidence:** S01–S07 section 5 require raw ASDMs under the member `raw/` tree
and run `scriptForPI.py` from `script/`. Pipeline packages prefer
`casa_piperestorescript.py`; without it they run `casa_pipescript.py`. The same
documents give up to 14× delivered products+raw with `SPACESAVING=0` and up to
6× with `SPACESAVING=3`. S16/S17 require the appropriate pipeline CASA and S14
warns not to infer it from broad dates.

**Adjudication:** CONFIRMED. Clarify that 14×/6× is an upper-bound estimate of
**additional working disk relative to delivered products plus raw**, not a
prediction for every MOUS.

### C12 — MS directories and DATA/CORRECTED_DATA semantics

**Claim checked:** `asdm-and-ms.md`.

**Evidence:** S18 states that `applycal` writes calibrated data to
`CORRECTED_DATA`; `split`/`mstransform` normally select corrected and put the
single selected column into output `DATA`. S21 is the normative CASA task
contract for the latter. S18 also confirms MS naming is era- and workflow-
dependent.

**Adjudication:** CONFIRMED. Add the precise view table in N02.

### C13 — ALMA Doppler setting, TOPO input, and regridding

**Claim checked:** `asdm-and-ms.md` spectral frames.

**Evidence:** S25 says the online system computes a fixed TOPO sky frequency at
execution, unregridded ALMA data remain TOPO, and users commonly select LSRK.
S23 explains that repeated executions of a tuning appear as separate SPWs with
different sky frequencies until regridded. S22 documents ephemeris-conversion
options during import.

**Adjudication:** CONFIRMED. “Different EBs are shifted” should be understood as
the usual consequence of different execution times, not a promise that every
pair has a numerically distinct tuning.

### C14 — TP is a separate branch with per-EB calibration and joint imaging

**Claim checked:** `mosaics-ephemeris-and-tp.md`.

**Evidence:** S27 says TP MOUSs often have many EBs, each calibrated
individually, with all calibrated data imaged together. S07 lists `jyperk` helper
files and TP cubes. S28, explicitly prospective for its current cycle, says TP
is offered for spectral-line rather than continuum observations.

**Adjudication:** CONFIRMED. Keep the timeless statement at the level “ALMA's TP
offering is spectral-line, not a TP continuum product stream”; band availability
must carry a cycle label.

### C15 — TP archive footprints are incomplete

**Claim checked:** `mosaics-ephemeris-and-tp.md` known defect.

**Evidence:** S10's current official known-issues section says TP footprints do
not show the full mapped extent and are represented by one TP antenna pointing.

**Adjudication:** CONFIRMED as of 2026-07-18. Because it is a live service bug,
stamp it “current Archive known issue; recheck before relying on it.”

### C16 — Calibrated-MS service statement is correct but needs limits

**Claim checked:** `products-and-qa.md` option 1.

**Evidence:** S16 says each ARC provides a calibrated-MS route. S30 documents EU
Helpdesk CalMS; S31 documents EA service. S32 documents NRAO SRDP and limits
automatic restore to pipeline-reduced Cycle 5+ 12-m/ACA, excluding manual and
TP data and noting selected Cycle 4 exceptions.

**Adjudication:** CONFIRMED. “All three ARCs provide services” is sound; do not
imply the three workflows, retention rules, automation, or coverage are the
same.

### C17 — FIELD/source/pointing distinction and mosaics

**Claim checked:** `mosaics-ephemeris-and-tp.md`.

**Evidence:** Official ALMA Knowledgebase guidance on source versus pointing
states that one source can contain one or multiple pointings and that overlapping
pointings are combined into one pipeline image, whereas different sources are
imaged separately. S12 WebLog documentation exposes mosaic pointings in each MS
overview. This is consistent with the MS-local FIELD/SOURCE warning.

**Adjudication:** CONFIRMED at the conceptual level. Numeric MS table keys being
local is a MeasurementSet schema fact; retain it.

## CONTRADICTED or materially misleading

### X01 — Archive bundling is not “roughly Cycle 4/5 onward current-style”

**Current text:** `identifiers-and-packaging.md` says roughly Cycle 4/5 onward
has the current-style QA2 package and that official documents group Cycles 4–6.

**Evidence:** S09 says Cycles 1–4 processed products were mostly packaged in the
same numbered tar. S10 dates the archive split into a FITS tar plus auxiliary
tar to Cycle 5+ (deployed June 2018). S01 describes the standardized member
directory roles but its May 2018 package filename is still a generic numbered
tar. The unpacked logical tree and archive tar grouping therefore changed on
different timelines.

**Adjudication:** CONTRADICTED as written because it conflates two dimensions.

**Proposed replacement — `identifiers-and-packaging.md`, “Packaging eras”:**

> Distinguish the **unpacked member-tree schema** from the **archive tar
> grouping**. The `product/`, `calibration/`, `qa/`, `script/`, `log/` roles and
> README are documented for Cycle 5 and nearby eras, with file-level drift.
> Archive bundling is different: Cycles 1–4 processed products are mostly in the
> same numbered tar; Cycle 5+ normally separates FITS product tar part(s), an
> auxiliary tar, and README. Cycle 0/early Cycle 1 can include ready-to-use MSs.

**Also change `SKILL.md` orientation label** from “roughly Cycle 4/5 onward” to
“logical Cycle 5+ full-delivery tree; Cycle 4 and re-deliveries may resemble it,
but inspect artifacts.”

### X02 — The FITS “grammar” is too rigid to be a parser contract

**Current text:** `products-and-qa.md` presents
`...<specmode>.<stokes>.<layer>...` and then explains optional `selfcal` and
Taylor terms.

**Evidence:** S05–S07 define more token families (`imagetype`, calibration
state, Stokes, comment, layer, extension). More importantly, S07's prose places
calibration state before Stokes, while its own product examples place
`selfcal` after `I` and before `tt0`/`alpha`. Manual comments and molecular names
add further tokens. The official contract itself demonstrates order drift.

**Adjudication:** CONTRADICTED as a strict positional grammar, though the
examples and “parse from both ends” warning are good.

**Proposed replacement — `products-and-qa.md`:**

> Treat product names as a versioned sequence of recognizable token families,
> not a fixed positional grammar: MOUS prefix; target plus intent; `spw...`;
> image mode (`mfs|cont|cube|repBW`); optional calibration state
> (`selfcal`, archived regular products often omit `regcal`); Stokes; optional
> Taylor/spectral-index/manual/line comment tokens; layer
> (`pbcor|pb|flux|mask|sd`); extension. Parse anchored ends and validate against
> FITS headers. Official prose and examples do not maintain one stable token
> order.

### X03 — QA0 wording is too absolute at MOUS-membership level

**Current text:** `products-and-qa.md` says “A QA2-passed MOUS is built from
QA0-PASS EBs.”

**Evidence:** S06/S07 say QA2 is carried out across pass EBs, but S24 explicitly
warns that a MOUS can have QA2 PASS even when not all ASDMs it contains have
QA0 PASS. The likely resolution is processed-versus-contained raw membership.

**Adjudication:** MATERIALLY MISLEADING without that distinction.

**Proposed replacement:**

> QA2 processing normally uses the QA0-PASS EBs, but the archive/MOUS can also
> contain raw ASDMs with other QA0 states; therefore MOUS QA2 PASS does not imply
> every associated ASDM is QA0 PASS. QA0-SemiPass material may be raw-only.

### X04 — `_targets_line.ms` description needs column-level qualification

**Current text:** `asdm-and-ms.md` calls `_targets_line.ms` a
“continuum-subtracted view,” which invites an agent to use whichever data column
CASA chooses.

**Evidence:** S18 says Cycle 9+ `_targets_line.ms` has calibrated,
continuum-subtracted data in `DATA` and successful self-cal results in
`CORRECTED_DATA`; `_targets.ms` has calibrated data in `DATA` and successful
self-cal in `CORRECTED_DATA`. If no selfcal succeeds, `CORRECTED_DATA` may be
absent. These semantics are not recoverable from the suffix alone.

**Adjudication:** INCOMPLETE/MISLEADING. Apply N02.

## NEW durable agent traps

### N01 — `cont.dat`: omitted SPW and empty SPW are different, and the format is
release-versioned

**Evidence:** S12 section 7.6 says the format changed between PL2023 and PL2024;
PL2024+ can read the older format but not vice versa. In PL2024, a listed SPW
with ranges bypasses `hif_findcont`; a listed SPW with no ranges also bypasses
heuristics and is currently treated downstream as all-continuum; an omitted SPW
allows heuristics to run. This is a serious empty-versus-absent trap.

**Adjudication:** NEW.

**Proposed addition — `identifiers-and-packaging.md`, `cont.dat`:**

> `cont.dat` is release-versioned (notably PL2023→PL2024; newer reads older, not
> vice versa). In PL2024 semantics, **SPW omitted** means run `hif_findcont`,
> while **SPW present with no ranges** suppresses heuristics and can be treated
> downstream as all-continuum. Do not normalize empty and absent entries, and
> use the matching pipeline guide before editing/replaying the file.

### N02 — Add an MS-view/column decision table

**Evidence:** S18 and S23.

**Adjudication:** NEW and high value.

**Proposed addition — `asdm-and-ms.md`, after DATA/CORRECTED_DATA:**

| Restored IF view | Era | `DATA` | `CORRECTED_DATA` |
|---|---|---|---|
| `uid*.ms` | all pipeline eras | raw | calibrated |
| `uid*target.ms` | Cycles 4–8 | calibrated | continuum-subtracted |
| `uid*targets.ms` | Cycle 9+ | calibrated | selfcal, per successful field; may be absent |
| `uid*targets_line.ms` | Cycle 9+ | calibrated + continuum-subtracted | selfcal, per successful field; may be absent |
| `uid*.ms.split.cal` | manual or `DOSPLIT=True` | calibrated | normally absent |

> This table applies to restored pipeline IF data, not manual SD, solar, VLBI,
> or phased-array modes. Inspect columns and scripts. `casa_piperestorescript`
> alone restores the full `uid*.ms`; imaging views require later stages or
> `scriptForPI` options such as `DOCONTSUB=True`.

### N03 — Pre-2017 pipeline restores can fail under newer CASA because the
manifest did not exist

**Evidence:** S19 says CASA 5.1.1+ requires the pipeline manifest and cannot
restore packages from before 2017-10-01 that lack it; same-version CASA is the
recommended fallback.

**Adjudication:** NEW.

**Proposed addition — `products-and-qa.md`, restore version bullet:**

> Legacy hard stop: packages processed before 2017-10-01 may lack the pipeline
> manifest required by CASA 5.1.1+, so “try a newer CASA” can fail even when the
> scripts look compatible. Use the package's recorded CASA/pipeline and current
> official restore table.

### N04 — Pre-CASA-4.7 calibration bundles may use `.tar.gz`, not `.tgz`

**Evidence:** S20 documents `hif_restoredata` looking for `.tgz` while older
archives store caltables/flagversions as `.tar.gz`.

**Adjudication:** NEW.

**Proposed addition — packaging nested archives:**

> Legacy restore bundles may end in `.tar.gz` rather than `.tgz`; do not
> classify by one suffix. Old scripts run under newer CASA can fail solely on
> this naming change.

### N05 — ARI-L is an external product family, not the original QA2 package

**Evidence:** S33 says ARI-L generated uniform full cubes and continuum images
for 86% of pipeline-processable Cycle 2–4 data, ingested under collection
`ari_l` as “Externally delivered products”; calibrated MSs are separately
requestable. It complements sparse original QA2 images.

**Adjudication:** NEW and important for interpreting older-cycle samples.

**Proposed addition — `identifiers-and-packaging.md`, after packaging eras:**

> For Cycles 2–4, check `collection='ari_l'` / externally delivered products.
> ARI-L reprocessed many pipeline-compatible MOUSs to add fuller cubes and
> continuum images. These are **not** the original QA2 delivery and should not
> be used to infer original-cycle filenames, pipeline version, or QA2 package
> completeness. ARI-L calibrated MSs follow a separate request route.

### N06 — Historical mosaic products need CASA-version caution even if current
archive copies were remediated

**Evidence:** S26 documents 7-m mosaics imaged with CASA <5.1.1 and 12-m/7-m
mosaics in CASA 5.1.1–5.3 as affected by distinct flux-distribution defects;
the official article says current downloadable affected products were reimaged
or held in QA3.

**Adjudication:** NEW, useful for local old downloads and literature packages.

**Proposed addition — `mosaics-ephemeris-and-tp.md`:**

> Historical-copy trap: some old 7-m mosaic images (CASA <5.1.1) and Cycle 5
> 12-m/7-m mosaics (CASA 5.1.1–5.3) had known PB/mosaic flux errors. The
> Observatory remediated current archive products, but locally retained old
> packages may still be affected; read `CASAVER`/QA2 and the official issue
> notice before quantitative reuse.

### N07 — Full-polarization product expectations changed materially

**Evidence:** S05/S06 say Cycle 10/11 pipeline performed polarization
calibration while science full-Stokes imaging remained manual. S07 says Cycle 12
pipeline performs full-Stokes imaging; it delivers IQUV plus derived P/A and may
not separately deliver the Stokes-I target image used internally for masking.

**Adjudication:** NEW/version-history clarification.

**Proposed edit — `products-and-qa.md` Stokes bullet:**

> Full-polarization delivery is era/recipe dependent. Cycle 10–11 commonly has
> pipeline polarization calibration with manual science imaging; Cycle 12 adds
> pipeline full-Stokes imaging. Expect IQUV and derived P/A products, but do not
> assume a separately delivered Stokes-I science image or stable token order.

### N08 — Partial download structure is not processing evidence

**Evidence:** S08 explicitly says `product/` exists only if imaging products are
selected for download; Request Handler exposes individual files for modern
cycles.

**Adjudication:** NEW.

**Proposed guardrail:**

> Distinguish “not selected/downloaded” from “not produced.” Directory or file
> absence in a partial Request Handler download is not pipeline/QA evidence;
> retain the DataLink/request manifest.

### N09 — Current calibrated-MS services return different scientific states

**Evidence:** S30–S32 and S18. NRAO SRDP's automated restore returns full
calibrator+science MSs without selfcal or continuum subtraction and excludes
manual/TP; ARC Helpdesk services have different access and delivery policies.

**Adjudication:** NEW.

**Proposed edit — `products-and-qa.md`, option 1:**

> Check the service's output contract. NRAO SRDP currently restores
> pipeline-reduced Cycle 5+ 12-m/ACA data and returns calibrators+targets without
> selfcal or continuum subtraction; manual and TP reductions are excluded.
> EU/EA/NA Helpdesk delivery policies and retention differ.

### N10 — `applycalQA_outliers.txt` is a linked WebLog artifact, not a generic
top-level QA file

**Evidence:** S15 section 9.26 says it is linked at the bottom of the
`hif_applycal` WebLog page.

**Adjudication:** NEW official confirmation of an existing claim. Keep the
current search-recursively guidance and cite the release-specific nature of the
page layout.

## STALE-RISK or insufficient official contract

### R01 — Detailed `pipeline_aquareport.xml` tag inventory

**Current text:** `products-and-qa.md` names `QaPerTopic`, `QaPerStage`,
`RepresentativeScore`, `SubScore`, and attribute shapes.

**Evidence:** S12 confirms WebLog stage/topic QA and an AQUA report but does not
provide a stable XML schema contract in the reviewed material. The current text
already admits namespaces/casing vary.

**Adjudication:** STALE-RISK / package-artifact provenance. Keep only as a
defensive empirical parser recipe, mark “observed in sampled releases,” and add
fixture tests from each actual release parsed. Never describe the tags as a
normative schema.

### R02 — Detailed `pipeline_manifest.xml` image attributes

**Current text:** calls the manifest authoritative and lists image WCS/beam/rms,
`datatype`, `pl_datatype`, applycal commands, and exact version tags.

**Evidence:** S05–S07 say `*manifest.xml` contains packaged-file metadata and
associated EBs; S18/S19 establish its restore role. The exact element/attribute
inventory is not documented as a stable public schema in the sources reviewed.

**Adjudication:** STALE-RISK / empirical corpus claim. Proposed wording:

> `*pipeline_manifest.xml` is the pipeline export/restore manifest and a strong
> source for versions, recipe, EBs, and packaged-product metadata. Element and
> attribute sets are release-specific; introspect the file and test against
> fixtures rather than coding one universal schema.

### R03 — Selfcal/timetracker/pipeline-stats JSON semantics

**Current text:** gives exact filenames, trigger conditions, leaf shapes, and
interprets multiple timetrackers as retries/continuations/splits.

**Evidence:** S07 confirms that selfcal, timetracker, and `pipeline_stats*.json`
are delivered in Cycle 12. S12 confirms selfcal success must be evaluated
per-target and final images only differ on success. It does not establish all
current filename/leaf invariants or prove why multiple files exist.

**Adjudication:** PARTLY CONFIRMED, otherwise STALE-RISK. Keep “file presence is
not success” strongly. Mark JSON key shapes and multiple-file causality as
observed, release-specific, and do not infer retry versus delivery without logs
and manifest.

### R04 — `antennapos.csv`→per-EB JSON supersession details

**Current text:** says CSV offsets were superseded by per-EB JSON absolute ITRF
positions fetched from an ASA uncertainties service and that one or the other,
not both, should occur.

**Evidence:** S07 supports `antennapos.csv or antennapos.json`; no reviewed
stable contract supports mutual exclusion, exact coordinate semantics, or the
service provenance.

**Adjudication:** STALE-RISK / empirical. Proposed wording:

> Current deliveries may contain CSV or JSON antenna-position helpers. Their
> coordinate semantics and coexistence are pipeline-release-specific; read the
> helper and matching Pipeline User's Guide rather than inferring “offset” or
> “absolute” solely from the suffix.

### R05 — `flux.csv` extended comment fields

**Current text:** promises `origin`, `age`, and `queried_at` in comments.

**Evidence:** S12/S13 normatively document rows by ASDM/field/SPW and Stokes
I/Q/U/V plus a comment, and document `origin=Source.xml` versus `origin=DB`.
They do not promise `age` or `queried_at` in every release.

**Adjudication:** PARTLY CONFIRMED, STALE-RISK for qualifiers. Proposed wording:

> `flux.csv` is the calibration model/control table, not the final transferred
> flux scale. Its comment commonly records origin and may add age/query
> provenance; treat comment qualifiers as optional.

### R06 — PPR `SESSION_1` XML traversal

**Current text:** asserts `<Intents>` blocks with `SESSION_1` hold EB UIDs.

**Evidence:** S01–S07 establish PPR as the processing request and recipe driver,
but the exact XML traversal was not found in a stable public schema.

**Adjudication:** STALE-RISK / observed-package claim. Keep it as a strong
provenance source only with defensive XML parsing and package fixtures; actual
processing must still be checked in logs/WebLog.

### R07 — “Exact CASA version required” versus compatibility tables

**Current text:** says exact version is required for faithful reproduction, then
notes compatibility guidance may bless newer versions.

**Evidence:** S14/S16/S19 and S26 recommend the recorded QA2 version for
calibration restore, especially for old packages, while official compatibility
tables authorize some alternatives. The exact original remains the
reproducibility baseline but is not the only version that can ever work.

**Adjudication:** GOOD BUT HIGH-VOLATILITY. Preserve both clauses, add the
pre-manifest hard stop, and never derive version from observing cycle alone.

### R08 — “All three ARCs” and SRDP coverage

**Evidence:** Confirmed by S16/S30–S32 as of the review date, but these are
services rather than data-format contracts.

**Adjudication:** STALE-RISK. Date-stamp or say “check the current official
calibrated-MS article,” avoiding a permanent promise of turnaround, retention,
or dataset coverage.

### R09 — Current TP bands and archive footprint bug

**Evidence:** S28 is prospective/current-cycle; S10 is a live known-issue page.

**Adjudication:** STALE-RISK. Keep stable principles (TP spectral-line branch;
full footprint cannot currently be trusted) but stamp offered bands and defect
status with cycle/access date.

## Suggested application order

1. Fix X01 in `SKILL.md` and `identifiers-and-packaging.md`; it affects how the
   requested Cycle 4–current package sample is interpreted.
2. Apply N08 before judging missing files in downloaded samples.
3. Replace the FITS grammar per X02 and add N07.
4. Apply N01, N03, N04, and N05 to packaging/restore guidance.
5. Add N02 to `asdm-and-ms.md` and adjust X04.
6. Apply X03 to QA wording and N06 to mosaics.
7. Soften R01–R06 unless the package-sampling phase supplies reproducible
   fixtures and recorded corpus evidence.
8. Date-stamp R08/R09 service claims.

## Package-sampling checks requested from the next phase

For each sampled MOUS, record rather than merely eyeball:

- project code/cycle, MOUS, EB UIDs, observation date, processing date,
  pipeline/CASA versions, recipe, and QA disposition;
- DataLink filename, semantics, content length, and whether it is original QA2,
  ARI-L, large-program external, ADMIT, or ARC-added-value material;
- tar top-level prefix and complete member-relative file inventory before nested
  unpacking;
- presence/absence of the five logical directories and whether absence follows
  download selection;
- numbered product tar versus auxiliary split, and `.tgz` versus `.tar.gz`;
- README and QA2-report formats/authority;
- PPR/manual-script discriminator;
- helper-file grammar (`cont.dat` empty versus omitted SPWs, flag templates,
  flux and antenna-position helpers);
- manifest/AQUA/JSON element or key inventories with release tags;
- full/target(s)/targets_line listobs and MS-column state if an MS is available;
- ARI-L/external-product labels for Cycle 4 samples;
- TP/polarization/mosaic special-mode markers.

This evidence is necessary before promoting R01–R06 from empirical observations
to release-scoped skill guidance.
