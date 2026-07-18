# ALMA identifiers, deliverables, and file trees

Reviewed 2026-07-18 against the official Archive/QA2 packaging guidance and
Cycle 4--11 DataLink/package samples. A proposal cycle does not uniquely
determine packaging: processing release, QA state, re-delivery generation, and
the user's Request Handler selection all matter.

## Contents

- Project codes
- UIDs
- DataLink deliverables
- Packaging eras
- Nesting trap
- Logical QA2 package
- Nested archives
- Weblog
- Primary sources

## Project codes

`<year>.<period>.<number>.<type>`, e.g. `2021.1.00123.S`, `2019.2.00052.S`,
`2022.A.00003.S`.

- `year`: proposal-call year → cycle (gaps: no `2014.x`, no `2020.1`; see
  cycle-capabilities reference).
- `period`: digit for a regular call within that year (`0` historically —
  `2011.0` — usually `1`; `2` has been used for the supplemental call);
  a letter (`A`, ...) marks DDT submission periods.
- `type`: `S` standard, `L` large program, `T` target-of-opportunity,
  `V` VLBI, `P` phased array.

## UID grammar and sanitization

- `uid://A001/Xnnn/Xnnn` for project-structure entities and
  `uid://A002/Xnnn/Xnnn` for EBs/ASDMs are normal modern conventions, not
  reliable entity types. A live Cycle 1 Member OUS is
  `uid://A002/X5d7935/X11b`. Establish an EB from the `asdm_uid` column or
  raw-ASDM role, and a MOUS from `member_ous_uid` or member-path context.
  `Xnnn` segments are normally hex (`X1467`, `Xe1baa0`).
- Sanitized (filesystem) form, conventionally used in entity-bearing archive
  paths and filenames:
  `uid://A001/X1467/X291` ↔ `uid___A001_X1467_X291`
  (`://` → `___`, `/` → `_`). A regex such as
  `uid___A002_X[0-9a-f]+_X[0-9a-f]+` recognizes a common stem but does not
  prove it is an EB.
- The EU DataLink service accepted both forms on 2026-07-18. Preserve the
  canonical URI internally and treat sanitized-form retry as service-specific
  compatibility behavior, not an IVOA contract.

## What DataLink offers per MOUS

`<mirror>/datalink/sync?ID=<mous-uid>` returns a VOTable. Rows are NOT all
downloadable files: exactly one of `access_url`, `service_def`, or
`error_message` identifies the row's access alternative. `content_length`,
when present, is per-link bytes and may be null. Expect direct file rows,
service descriptors, nested DataLink entries to recurse into, error rows, and
documentation links — handle each explicitly. An empty result can mean a
valid proprietary MOUS has no links
visible to the current authorization, while an invalid UID can return an
explicit `#error`/`NotFoundFault`; distinguish both from a valid visible
MOUS with files. `semantics` values include `#this`, `#auxiliary`, and both
`#progenitor` and `#package` on raw/external bundles. Preserve all returned
semantics and ALMA-local columns. Typical deliverable files (current era):

| File | Contents |
|---|---|
| `<project>_<mous-uid>_001_of_00N.tar` | pipeline FITS products; may be split into N parts — you need all N |
| `<project>_<mous-uid>_auxiliary.tar` | supporting calibration/script/log/QA material for a processed delivery. Usually fetch first, but it can be large and a SEMIPASS/unprocessed MOUS can contain only QA reports plus bookkeeping. |
| `member.<mous-uid>.README.txt` | delivery README |
| `*uid___A002_*.asdm.sdm.tar` (one per EB) | raw ASDM — only needed for MS restore / recalibration; large. Official guidance and current holdings show both UID-only and project-prefixed basenames. Preserve the DataLink basename; never synthesize it. |

There is no clean machine-readable taxonomy of file kinds: classification
rests on filename conventions (`weblog`, `README`, `auxiliary`, `asdm.sdm`,
`aquareport`, `auxproducts`, `scriptForPI`, `cube`, `cont`, ...) plus
DataLink `semantics`. Classify conservatively and keep the raw filename.
Following a nested DataLink service can expose individual files without the
original tar directory path. Retain category/path provenance; fetch the
top-level auxiliary tar when reconstructing the delivery tree matters.

## Packaging eras — separate tar grouping from the logical tree

- **Unpacked roles:** `product/`, `calibration/`, `qa/`, `script/`, `log/`
  and README form the logical full-delivery model across the modern QA2 era,
  with file-level drift. A partial request can omit unselected roles.
- **Historical delivery grouping:** Cycles 1--4 processed products were
  *mostly* kept in the same numbered tar. Cycle 5+ normally separates FITS
  product tar part(s), an auxiliary tar, and README.
- **Current retrieval grouping:** live DataLink can repackage old holdings
  into current product/auxiliary/README/raw containers. Inspect actual rows;
  a project cycle does not determine the containers downloadable today.
- **Reduction style:** manual-calibration scripts
  (`*.scriptForCalibration.py`) and `scriptForImaging.py` are common in older
  data, but pipeline versus manual is dataset- and mode-dependent, not a cycle
  switch. PPR presence is positive pipeline evidence; PPR absence is
  conclusive only when `script/` was actually retrieved.
- **Cycle 0 and early Cycle 1**: Cycle 0 calibrated MSs remain a documented
  archive delivery case. Some original early-Cycle-1 PI deliveries also
  included calibrated MSs, but those MSs are not retrievable from the current
  archive; restore from the available package/raw data instead.

For Cycles 2--4 also inspect the ASA UI **Collection** label `ari_l` and
DataLink's externally delivered products. Do not write ADQL against a presumed
`collection` column: the ObsCore field is `obs_collection`, and its live values
must be introspected before filtering programmatically. ARI-L reprocessed many
pipeline-compatible MOUSs to add more uniform cubes and continuum images.
These are not the original QA2 delivery: do not use them to infer
original-cycle filenames, pipeline version, or QA2 package completeness.

## The nesting trap

Top-level delivery tarballs commonly share the ASA tree, with or without the
project code as top level:

```
[2021.1.00123.S/]science_goal.uid___A001_.../group.uid___A001_.../member.uid___A001_.../...
```

Inspect member names before stripping anything. When this prefix is present,
unpacking several tarballs "in place" inside an already ASA-shaped tree can
produce nested duplicate trees
(`member.../2021.1.00123.S/science_goal.../...`). Either unpack from the tree
root or detect and strip the redundant prefix.

Treat every downloaded archive as untrusted input. Preflight selected bytes
and free space, list all members, and reject absolute paths, `..` traversal,
device nodes, and symlink/hardlink targets that escape the extraction root.
Extract into a unique empty staging directory rather than over an existing ASA
or user tree. Validate the staged hierarchy before promotion and retain the
archive plus request inventory until validation succeeds. When using Python,
use an appropriate
[tar extraction filter](https://docs.python.org/3/library/tarfile.html#extraction-filters)
in addition to explicit member checks.

Nested `auxproducts`, `caltables`, `flagversions`, and weblog archives have
local payloads and do not generally repeat the full ASA tree. Individual
nested DataLink downloads may also flatten their original path context.

Top-level tar member mtimes can be **repackaging** times, not observation,
processing, or QA2 dates. Sampled Cycle 5--9 auxiliary tars visible in 2026
all carried a 2025-07-01 mtime despite much older PPR/weblog runs. Use archive
metadata plus QA2/manifest/PPR/weblog provenance instead.

Note: the Science Goal OUS UID is NOT available from ObsCore — the delivery
path (`science_goal.uid___*`) is where you learn it.

Use the non-lossy hierarchy Science Goal → one or more GOUSs → one or more
MOUSs. A MOUS normally maps to one Scheduling Block, with commissioning and
legacy exceptions; do not force a one-to-one hierarchy when ingesting data.

## The logical Cycle-5+ full QA2 package

Five role directories + README under `member.uid___*/`:

```
member.uid___A001_X1467_X291/
├── product/
│   ├── member.uid___*.<src>_sci.spw25_27.cont.I.tt0.pbcor.fits
│   ├── member.uid___*.<src>_sci.spw25.cube.I.pbcor.fits
│   └── ...pb.fits.gz, ...mask.fits.gz
├── calibration/
│   ├── member.uid___*.caltables.tgz       → per-EB calibration tables
│   ├── uid___A002_*.ms.flagversions.tgz   → saved flag states
│   └── member.uid___*.auxproducts.tgz     → cont.dat, flag templates, aux tables
├── script/
│   ├── member.uid___*.scriptForPI.py      → restore entry point
│   ├── casa_pipescript.py                 → full pipeline re-run
│   ├── casa_piperestorescript.py          → fast restore
│   └── PPR*.xml / *.pprequest.xml         → pipeline processing request
├── qa/
│   ├── member.uid___*.weblog.tgz          → pipeline-*/html/...
│   ├── uid___A002_*.qa0_report.pdf        (per EB)
│   └── member.uid___*.qa2_report.pdf      (per MOUS)
├── log/                                    CASA/pipeline logs
└── README
```

`raw/` appears beside these only when you separately download and unpack the
per-EB ASDM tarballs. File placement drifts across cycles (e.g. flag
templates sat loose in `calibration/` before moving into `auxproducts.tgz`;
`auxproducts.tgz` itself has appeared under `product/` in some eras;
`applycalQA_outliers.txt` lives inside the weblog hierarchy). Directory
roles are strong conventions, not guarantees — search by filename pattern,
not fixed path. More importantly, distinguish **not downloaded** from **not
produced**: absence in a partial request is not pipeline or QA evidence. Keep
the DataLink/request inventory.

## Nested second-level archives

Unpack deliberately:

- `*.auxproducts.tgz` — small and high-value; usually worth unpacking first.
  Contains **`cont.dat`**: the pipeline's per-source/per-SPW continuum
  (line-free) frequency selections, in LSRK — the primary portable record of
  what the pipeline considered continuum (the weblog encodes the same
  decisions less conveniently). Also flag templates, flux/antenna-position
  tables, self-cal restore metadata.

  `cont.dat` is release-versioned. Older sampled files have
  `SpectralWindow: <id>` plus an optional bare `ALL`; PL2024 adds SPW names
  and `Flags: ALLCONT|LOWBANDWIDTH|LOWSPREAD`. Frequency ranges use
  `<lo>~<hi>GHz LSRK`. Do not normalize three distinct states: **SPW omitted**,
  **SPW listed with no ranges**, and **SPW listed with ranges**. In PL2024
  semantics omission allows `hif_findcont` to run, while a present-but-empty
  entry suppresses the heuristic and can be treated downstream as
  all-continuum. Newer pipelines can read the older format, not necessarily
  vice versa; use the matching Pipeline User's Guide before replaying or
  editing it. SPW IDs can also fail to match a particular MS/listobs view, so
  join defensively by spectral setup.
- `*.caltables.tgz` — calibration tables; needed for restore.
- `*weblog*.tgz` — unpacks to `pipeline-YYYYMMDDTHHMMSS/html/`.
- `*.flagversions.tgz` — per-EB saved flag states; primarily needed by the
  CASA restore (also useful for inspecting/reverting flag versions); many
  small files, so leave packed unless needed.

Legacy bundles may use `.tar.gz` rather than `.tgz`; classify both. A
pre-CASA-4.7 package restored under a newer task can fail solely because the
task searches for the newer suffix.

## Weblog layout

```
pipeline-YYYYMMDDTHHMMSS/html/
├── index.html          ← preferred landing page
├── t1-1.html           ← compatible alternate; often coexists
├── t2-*.html           per-stage pages
├── stage<N>/           per-stage plots and QA detail
└── session<name>/      per-session, per-MS metadata, incl.
    └── <eb-uid>.ms/listobs.txt   ← pre-rendered listobs per MS view
```

The landing page states the CASA and pipeline versions (needed for restore;
`pipeline_manifest.xml` carries the same machine-readably). The weblog is
the richest QA record: per-stage scores, flagging fractions, applycal-QA
configuration/outlier evidence (inspect contents before claiming an outlier),
and per-EB/per-SPW diagnostics. The per-MS `listobs.txt` files (also
for `_targets.ms` / `_targets_line.ms` views) give scans, intents, SPWs,
and antennas without CASA — see `references/listobs-and-intents.md`,
including why session directories are named like `sessionsession_1`.

Sampled Cycle 7--11 weblogs contain a byte-identical PPR copy under
`html/PPR_<SBStatusUID>.xml`; the filename UID is the SB-status entity, not the
MOUS, and this is not a second processing request. Search XML content and
roles instead of classifying by UID-looking filename alone.

Search rather than assuming role paths: the sampled Cycle 7
`casa_commands.log` sits under `script/`, and nested DataLink can label a
command log `#documentation`. Neither path nor semantics alone is a physical
role contract.

## Primary sources

- [ALMA Science Archive User Manual, Cycle 13](https://almascience.nrao.edu/documents-and-tools/cycle13/science-archive-manual)
- [ALMA QA2 Data Products for Cycle 12](https://almascience.nrao.edu/documents-and-tools/cycle12/alma-qa2-data-products-for-cycle-12)
- [How ALMA data products are packaged](https://help.almascience.org/kb/articles/how-are-alma-data-products-packaged)
- [ARI-L project and retrieval guidance](https://almascience.eso.org/alma-data/aril)
- [IVOA DataLink 1.1](https://www.ivoa.net/documents/DataLink/20231215/REC-DataLink-1.1.html)

Package layouts, filenames, semantics, and tar mtimes described from the
Cycle 4--11 corpus are empirical observations dated 2026-07-18, not archive
schema guarantees.
