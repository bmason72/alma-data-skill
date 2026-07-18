---
name: working-with-alma-data
description: "Work safely and accurately with ALMA archive and restored data: query TAP/ObsCore and DataLink; interpret OUS/MOUS/EB identifiers, packages, ASDMs, MeasurementSets, listobs, intents, SPWs, products, QA, pipeline artifacts, and release history; and branch for mosaics, moving targets, Total Power, polarization, band-to-band, or VLBI data. Use when Codex needs to discover, download, unpack, restore, parse, compare, or scientifically interpret ALMA data or metadata."
---

# Working with ALMA data

Systematically reviewed 2026-07-18 against current official Archive, QA2,
CASA, and pipeline-release documentation, plus DataLink/package artifacts from
each public observing Cycle 4--11. Time-sensitive claims carry dates in the
references.

ALMA data pass through several distinct layers, and most agent mistakes come
from blurring them. Before acting, identify which layer you are touching:

> **Archive row ≠ dataset. MOUS ≠ EB. ASDM ≠ MS. One EB ≠ one MS view.
> SPW number ≠ spectral identity. One scan ≠ one intent. Pipeline QA
> score ≠ QA disposition. Delivered FITS ≠ complete science content.
> A `mode='manual'` flag ≠ proven human action.**

## The data hierarchy (memorize this)

```
Project (2021.1.00123.S)
└── Science Goal OUS (one per proposal science goal)       UID (normally A001)
    └── one or more Group OUSs (GOUSs)                     UID (normally A001)
        └── one or more Member OUSs (MOUSs), normally one SB UID (normally A001)
            └── Execution Block ("EB") = one execution,
                stored as one ASDM raw dataset             UID (normally A002)
```

| Layer | What it is | What happens at this level |
|---|---|---|
| EB / ASDM | one telescope execution (~1 h) | raw data; QA0 |
| MOUS | EBs normally associated with one Scheduling Block and one independently calibrated setup | calibration, pipeline run, QA2, archive delivery |
| GOUS | groups sibling MOUSs within a Science Goal | cross-array relationship; combination is a later science step |
| Science Goal OUS | proposal science-goal structure above one or more GOUSs | preserves the proposal hierarchy |

- **The MOUS is the atomic unit** of processing and delivery. For ordinary
  non-solar observing, 12-m, 7-m, and Total Power data are normally separate
  sibling MOUSs grouped for combination (SB-name suffixes `_TM1`/`_TM2`,
  `_7M`, `_TP`). Do not make that an invariant: commissioning/legacy cases can
  span more than one SB, and solar or exceptional heterogeneous interferometric
  EBs can mix 7-m and 12-m antennas. TP remains a separate single-dish stream.
  Determine actual composition from the EB/MS antenna table, not hierarchy or
  suffix alone.
- ObsCore's `asdm_uid` is the EB UID; an ASDM raw package represents an EB.
  Do not identify the entity solely from an A001/A002 prefix.

## UID and project-code grammar

- UIDs: `uid://A001/Xnnn/Xnnn` for project-structure entities and
  `uid://A002/Xnnn/Xnnn` for EBs/ASDMs are normal modern conventions (hex
  after `X`), not entity types. A live Cycle 1 Member OUS has an A002 UID;
  identify MOUS/EB from `member_ous_uid`/`asdm_uid`, paths, or package role.
- Conventional filesystem-sanitized form used in entity-bearing archive
  filenames and directories:
  `uid://A001/X133d/X1d1` → `uid___A001_X133d_X1d1` (`://` → `___`,
  `/` → `_`).
- Project codes: `<year>.<period>.<number>.<type>`, e.g. `2021.1.00123.S`.
  `period`: `1` main call, `2` supplemental call, a letter (`A`, ...) for DDT.
  `type`: `S` standard, `L` large program, `T` ToO, `V` VLBI, `P` phased
  array. Code year ↔ cycle mapping has gaps (no `2014.x`, no `2020.1`) — see
  `references/cycle-capabilities.md`.

## Non-negotiable guardrails

1. **ObsCore rows are finer-grained than datasets** (repeated per EB, per
   source/field, per coverage). Aggregate by `member_ous_uid` for
   MOUS-level work; dedupe EBs by `asdm_uid`; never count SPWs by summing
   `frequency_support` over grouped rows.
2. **Mind ObsCore units**: `t_min`/`t_max` are MJD floats; `obs_release_date`
   is an ISO string; `em_min`/`em_max` are wavelengths in **meters**;
   `frequency` is GHz but `bandwidth` is **Hz**; `spatial_resolution` is
   arcsec.
3. **Parse `band_list` tolerantly** (`6`, `BAND 6`, multi-valued `5 10` for
   band-to-band). Determine the science tuning from `frequency_support`, not
   from `band_list` alone.
4. **The calibrated MS is not in the standard delivery.** The package holds
   caltables + scripts + QA + selected FITS products. Restoring an MS
   requires the raw ASDMs and a CASA version authorized by the package/current
   ALMA compatibility table (use the original for identical reproduction) — see
   `references/products-and-qa.md` (calibrated-MS services at the ARCs and
   NRAO SRDP may spare you the restore).
5. **Never treat filenames as authoritative metadata** — read FITS headers /
   weblog / PPR. Filename SPW numbers are pipeline "virtual" SPW IDs (not
   the PI's ordinal windows), and SPW IDs are not stable across
   import/split/regrid.
6. **Inspect archive member names before choosing an extraction root.**
   Top-level delivery tarballs commonly repeat a shared ASA prefix
   (`[project/]science_goal.uid___*/group.uid___*/member.uid___*/...`), so
   strip it or unpack from the tree root when present. Nested archives have
   local payloads and do not generally repeat the full prefix. Before
   extraction, preflight selected DataLink bytes and free space; list every
   member; reject absolute paths, `..` traversal, device nodes, and links that
   escape the staging root. Extract into a unique empty staging directory,
   never over an existing archive/user tree. Validate the resulting ASA tree
   before promoting it, and retain the tar plus request inventory until then.
   Unpack deliberately (`*.flagversions.tgz` mainly matters for restores).
   Directory/file absence in a partial Request Handler download is not
   processing evidence; retain the DataLink/request inventory.
7. **Delivered FITS products may not represent the complete science content** —
   imaging mitigation can drop targets/SPWs/channels. Inspect the product
   inventory; restore calibrated visibilities when omitted content or custom
   imaging matters.
8. **QA discipline**: `qa2_passed` (`T`/`F`) cannot reconstruct the
   three-state QA2 (PASS/SEMIPASS/FAIL) — read the README/QA2 report. Never
   infer QA0 status from file presence, final QA2 from AQUA pipeline scores,
   or human intervention solely from `mode='manual'` flag commands.
9. **Spectral frames**: ALMA uses Doppler *setting* per EB (native frame
   TOPO), not continuous tracking — different EBs are shifted in sky
   frequency and must be regridded (products are typically LSRK).
10. **Branch to special handling** for TP, mosaics, ephemeris/solar targets,
    full-polarization, band-to-band, and VLBI/phased-array data.
11. **Scans are multi-intent, and pipeline imaging recipes create multiple MS
    views per EB**: sampled PL2018--PL2021 packages have full + singular
    `_target`; PL2022+ have full + plural `_targets` + `_targets_line`.
    Manual/calibration-only packages can have neither. Each can have its own
    listobs and different DATA/CORRECTED_DATA meaning. Intent bookkeeping needs
    single-bucket priority rules; listobs harvesting needs deliberate view
    selection — see `references/listobs-and-intents.md`.

## The delivered MOUS package (orientation)

Logical full QA2 delivery tree (normally Cycle 5+): five role directories +
README under `member.uid___*/`:

```
product/      FITS image/cube families (pbcor, pb, and masks where exported)
calibration/  caltables.tgz, flagversions.tgz, auxproducts.tgz → cont.dat, flag templates
script/       scriptForPI.py, casa_pipescript.py, PPR*.xml
qa/           weblog.tgz, QA0/QA2 report PDFs
log/          CASA/pipeline logs
```

Placement of individual files varies by cycle — search by filename pattern,
never by fixed path. Raw EB tar basenames can be UID-only or project-prefixed;
preserve the returned DataLink basename and match
`*uid___A002_*.asdm.sdm.tar` rather than synthesizing it. They are separate
per-EB downloads, not part of the package. Earlier cycles are
packaged materially differently, and a SEMIPASS MOUS with no QA0-PASS EBs can
have a QA-only auxiliary tar rather than a restore package. Current DataLink
can also expose old/re-delivered data in newer tar groupings — see
`references/identifiers-and-packaging.md`.
Even `scriptForPI.py` can occur in a SEMIPASS QA shell; confirm processed EBs,
caltables/flagversions, and QA instructions before calling it restorable.

## Where to look next

| Task touches... | Read |
|---|---|
| TAP/ADQL queries, ObsCore columns, footprints, release dates | [archive-query.md](references/archive-query.md) |
| project codes, UIDs, datalink deliverables, tarballs, unpacking, tree layout | [identifiers-and-packaging.md](references/identifiers-and-packaging.md) |
| raw data, importasdm, MS structure, SPWs, spectral frames | [asdm-and-ms.md](references/asdm-and-ms.md) |
| listobs parsing, scan intents, SPW names (FULL_RES/CH_AVG/SQLD/WVR), array inference | [listobs-and-intents.md](references/listobs-and-intents.md) |
| FITS products, naming, weblog, AQUA/PPR, QA levels, recipes, manifest/selfcal/stats files, MS restore | [products-and-qa.md](references/products-and-qa.md) |
| mosaics, FIELD vs SOURCE, moving targets, Total Power, array combination | [mosaics-ephemeris-and-tp.md](references/mosaics-ephemeris-and-tp.md) |
| bands, arrays/configurations, correlator modes, cycle capabilities | [cycle-capabilities.md](references/cycle-capabilities.md) |
| pipeline/CASA operations dates, capability milestones, patch-specific package traps | [pipeline-history.md](references/pipeline-history.md) |
