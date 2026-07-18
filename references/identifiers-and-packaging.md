# ALMA identifiers, deliverables, and file trees

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

- `uid://A001/Xnnn/Xnnn` — project-structure entities (OUSs, SBs).
- `uid://A002/Xnnn/Xnnn` — EBs/ASDMs. `Xnnn` segments are hex (`X1467`,
  `Xe1baa0`). Recognition aid only — early-cycle/Science-Verification data
  can deviate.
- Sanitized (filesystem) form, used in every path and filename:
  `uid://A001/X1467/X291` ↔ `uid___A001_X1467_X291`
  (`://` → `___`, `/` → `_`). Regex for EB stems (case-insensitive hex):
  `uid___A002_X[0-9a-f]+_X[0-9a-f]+`.
- DataLink accepts both forms.

## What DataLink offers per MOUS

`<mirror>/datalink/sync?ID=<mous-uid>` returns a VOTable. Rows are NOT all
downloadable files: expect file rows (`access_url` + `content_length`),
service-descriptor rows (`service_def`, empty `access_url`), nested DataLink
entries to recurse into, error rows, and documentation links — handle each
explicitly. `semantics` values include `#this`, `#auxiliary`, and commonly
`#package` for raw/external bundles. Typical deliverable files (current era):

| File | Contents |
|---|---|
| `<project>_<mous-uid>_001_of_00N.tar` | pipeline FITS products; may be split into N parts — you need all N |
| `<project>_<mous-uid>_auxiliary.tar` | everything-but-images: scripts, caltables, weblog, QA reports, README. Small — fetch first. |
| `member.<mous-uid>.README.txt` | delivery README |
| `<project>_uid___A002_*.asdm.sdm.tar` (one per EB) | raw ASDM — only needed for MS restore / recalibration; large. Basenames are project-prefixed: glob `*uid___A002_*.asdm.sdm.tar`, not `uid___A002_*`. |

There is no clean machine-readable taxonomy of file kinds: classification
rests on filename conventions (`weblog`, `README`, `auxiliary`, `asdm.sdm`,
`aquareport`, `auxproducts`, `scriptForPI`, `cube`, `cont`, ...) plus
DataLink `semantics`. Classify conservatively and keep the raw filename.

## Packaging eras — check the cycle before assuming layout

- **Roughly Cycle 4/5 onward**: the current-style QA2 package described
  below (official docs group Cycles 4–6 together; details still drift
  per cycle).
- **Earlier cycles**: materially different packaging; products mixed
  together; manual-calibration scripts (`*.scriptForCalibration.py`) and
  `scriptForImaging.py` instead of pipeline scripts. The pipeline became
  official in late 2014 and phased in through Cycles 3–5 — but manual
  reduction is dataset-dependent and still occurs in later cycles for
  special modes.
- **Cycle 0 and early Cycle 1**: legacy packages; may include ready-to-use
  calibrated MSs directly.

## The nesting trap

Every tarball internally repeats the ASA tree, with or without the project
code as top level:

```
[2021.1.00123.S/]science_goal.uid___A001_.../group.uid___A001_.../member.uid___A001_.../...
```

Unpacking several tarballs "in place" inside an already ASA-shaped tree
without stripping this prefix produces nested duplicate trees
(`member.../2021.1.00123.S/science_goal.../...`). Either unpack from the tree
root or detect and strip the redundant prefix.

Note: the Science Goal OUS UID is NOT available from ObsCore — the delivery
path (`science_goal.uid___*`) is where you learn it.

## The current-style QA2 package

Five directories + README under `member.uid___*/`:

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
not fixed path.

## Nested second-level archives

Unpack deliberately:

- `*.auxproducts.tgz` — small and high-value; usually worth unpacking first.
  Contains **`cont.dat`**: the pipeline's per-source/per-SPW continuum
  (line-free) frequency selections, in LSRK — the primary portable record of
  what the pipeline considered continuum (the weblog encodes the same
  decisions less conveniently). Also flag templates, flux/antenna-position
  tables, self-cal restore metadata.

  `cont.dat` grammar: `Field: <name>` blocks holding
  `SpectralWindow: <id> <name>` sub-blocks, each with an optional
  `Flags: ALLCONT|LOWBANDWIDTH|LOWSPREAD` line and one or more
  `<lo>~<hi>GHz LSRK` range lines. The **frequency ranges are the
  authoritative content**; the flags are findcont diagnostic labels:
  `ALLCONT` = all usable bandwidth accepted as continuum (by far the
  common case; ranges may still trim SPW edges), `LOW*` = warnings that
  the selection is narrow or poorly spread. No Flags line just means an
  explicit partial selection was recorded. `cont.dat` can be absent for a
  MOUS, and its SPW IDs can fail to match a given listobs/MS view — join
  defensively.
- `*.caltables.tgz` — calibration tables; needed for restore.
- `*weblog*.tgz` — unpacks to `pipeline-YYYYMMDDTHHMMSS/html/`.
- `*.flagversions.tgz` — per-EB saved flag states; primarily needed by the
  CASA restore (also useful for inspecting/reverting flag versions); many
  small files, so leave packed unless needed.

## Weblog layout

```
pipeline-YYYYMMDDTHHMMSS/html/
├── index.html          ← landing page (older pipelines: t1-1.html)
├── t2-*.html           per-stage pages
├── stage<N>/           per-stage plots and QA detail
└── session<name>/      per-session, per-MS metadata, incl.
    └── <eb-uid>.ms/listobs.txt   ← pre-rendered listobs per MS view
```

The landing page states the CASA and pipeline versions (needed for restore;
`pipeline_manifest.xml` carries the same machine-readably). The weblog is
the richest QA record: per-stage scores, flagging fractions, applycal
outliers, per-EB/per-SPW diagnostics. The per-MS `listobs.txt` files (also
for `_targets.ms` / `_targets_line.ms` views) give scans, intents, SPWs,
and antennas without CASA — see `references/listobs-and-intents.md`,
including why session directories are named like `sessionsession_1`.
