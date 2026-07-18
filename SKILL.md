---
name: working-with-alma-data
description: Essential peculiarities of ALMA data for agents — OUS/MOUS/EB hierarchy and UID grammar, ALMA Science Archive (TAP/ObsCore) metadata quirks, deliverable packages and unpacked file trees, ASDM/MeasurementSet semantics, listobs parsing, scan intents, SPW naming, pipeline run artifacts, QA levels, product naming. Use when querying the ALMA archive, parsing ALMA metadata or listobs/pipeline files, or navigating/downloading/restoring ALMA data.
---

# Working with ALMA data

ALMA data pass through several distinct layers, and most agent mistakes come
from blurring them. Before acting, identify which layer you are touching:

> **Archive row ≠ dataset. MOUS ≠ EB. ASDM ≠ MS. One EB ≠ one MS view.
> SPW number ≠ spectral identity. One scan ≠ one intent. Pipeline QA
> score ≠ QA disposition. Delivered FITS ≠ complete science content.
> A `mode='manual'` flag ≠ proven human action.**

## The data hierarchy (memorize this)

```
Project (2021.1.00123.S)
└── Science Goal OUS (one per proposal science goal)      uid://A001/Xxxx/Xxxx
    └── Group OUS ("GOUS": groups MOUSs for combination)  uid://A001/Xxxx/Xxxx
        └── Member OUS ("MOUS") ↔ one Scheduling Block    uid://A001/Xxxx/Xxxx
            └── Execution Block ("EB") = one execution,
                stored as one ASDM raw dataset            uid://A002/Xxxx/Xxxx
```

| Layer | What it is | What happens at this level |
|---|---|---|
| EB / ASDM | one telescope execution (~1 h) | raw data; QA0 |
| MOUS | all EBs of one Scheduling Block, one array | calibration, pipeline run, QA2, archive delivery |
| GOUS / Science Goal | sibling 12-m / 7-m / TP MOUSs of one goal | (data combination is a later science step) |

- **The MOUS is the atomic unit** of processing and delivery. 12-m, 7-m, and
  Total Power observations of the same target are *separate sibling MOUSs*
  under one Group OUS (SB-name suffixes `_TM1`/`_TM2`, `_7M`, `_TP`) — never
  three array types inside one MOUS.
- ObsCore's `asdm_uid` is the EB UID; "EB", "ASDM", and one `uid://A002/...`
  are the same thing in three vocabularies.

## UID and project-code grammar

- UIDs: `uid://A001/Xnnn/Xnnn` for project-structure entities (OUSs, SBs);
  `uid://A002/Xnnn/Xnnn` for EBs/ASDMs (hex after `X`). This is the normal
  operational pattern — treat it as a recognition aid, not a validator
  (legacy / Science Verification data can violate it).
- Filesystem-sanitized form used in all filenames and directories:
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
   requires the raw ASDMs and the *matching CASA version* — see
   `references/products-and-qa.md` (calibrated-MS services at the ARCs and
   NRAO SRDP may spare you the restore).
5. **Never treat filenames as authoritative metadata** — read FITS headers /
   weblog / PPR. Filename SPW numbers are pipeline "virtual" SPW IDs (not
   the PI's ordinal windows), and SPW IDs are not stable across
   import/split/regrid.
6. **Tarballs internally repeat the ASA tree prefix**
   (`[project/]science_goal.uid___*/group.uid___*/member.uid___*/...`);
   unpack from the tree root or strip the prefix, or you nest duplicate
   trees. Deliverables also contain second-level archives — unpack
   deliberately (`*.flagversions.tgz` mainly matters for restores).
7. **Delivered FITS products are not the complete science content** —
   imaging mitigation may drop targets/SPWs/channels. The calibrated
   visibilities always contain more than the delivered images.
8. **QA discipline**: `qa2_passed` (`T`/`F`) cannot reconstruct the
   three-state QA2 (PASS/SEMIPASS/FAIL) — read the README/QA2 report. Never
   infer QA0 status from file presence, final QA2 from AQUA pipeline scores,
   or human intervention solely from `mode='manual'` flag commands.
9. **Spectral frames**: ALMA uses Doppler *setting* per EB (native frame
   TOPO), not continuous tracking — different EBs are shifted in sky
   frequency and must be regridded (products are typically LSRK).
10. **Branch to special handling** for TP, mosaics, ephemeris/solar targets,
    full-polarization, band-to-band, and VLBI/phased-array data.
11. **Scans are multi-intent, and modern imaging-recipe processing yields
    up to three MS views per EB** (full / `_targets` / `_targets_line`),
    each with its own listobs. Intent bookkeeping needs single-bucket
    priority rules; listobs harvesting needs deliberate view selection —
    see `references/listobs-and-intents.md`.

## The delivered MOUS package (orientation)

Current-era QA2 package (roughly Cycle 4/5 onward): five directories +
README under `member.uid___*/`:

```
product/      FITS images/cubes (pbcor + pb + mask)
calibration/  caltables.tgz, flagversions.tgz, auxproducts.tgz → cont.dat, flag templates
script/       scriptForPI.py, casa_pipescript.py, PPR*.xml
qa/           weblog.tgz, QA0/QA2 report PDFs
log/          CASA/pipeline logs
```

Placement of individual files varies by cycle — search by filename pattern,
never by fixed path. Raw EB tarballs (`<project>_uid___A002_*.asdm.sdm.tar`)
are separate per-EB downloads, not part of the package. Earlier cycles are
packaged materially differently — see
`references/identifiers-and-packaging.md`.

## Where to look next

| Task touches... | Read |
|---|---|
| TAP/ADQL queries, ObsCore columns, footprints, release dates | `references/archive-query.md` |
| project codes, UIDs, datalink deliverables, tarballs, unpacking, tree layout | `references/identifiers-and-packaging.md` |
| raw data, importasdm, MS structure, SPWs, spectral frames | `references/asdm-and-ms.md` |
| listobs parsing, scan intents, SPW names (FULL_RES/CH_AVG/SQLD/WVR), array inference | `references/listobs-and-intents.md` |
| FITS products, naming, weblog, AQUA/PPR, QA levels, recipes, manifest/selfcal/stats files, MS restore | `references/products-and-qa.md` |
| mosaics, FIELD vs SOURCE, moving targets, Total Power, array combination | `references/mosaics-ephemeris-and-tp.md` |
| bands, arrays/configurations, correlator modes, cycle capabilities | `references/cycle-capabilities.md` |
