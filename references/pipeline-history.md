# ALMA pipeline release history (release-first backbone)

Review date: 2026-07-18. This matrix records documented release capability; it is not a claim that every feature ran successfully or was delivered for every MOUS in an interval.

## Contents

- Status vocabulary
- Operations matrix
- Unresolved builds
- Cycle projection
- Package cross-check
- Evidence gaps

## How to read status statements

- **exists**: present in a reference manual;
- **recipe**: the User's Guide says it is in an operations recipe;
- **default**: the guide says the heuristic/stage runs by default for the
  supported case;
- **attempted**: the stage runs, but may decide not to apply a result;
- **succeeded**: a per-target/per-MOUS outcome, proven only by runtime
  artifacts;
- **delivered**: the guide explicitly says an output is exported/archived, or
  a real package confirms it.

The distinction matters most for self-calibration: PL2023 adds a stage that
attempts eligible targets, but only successful targets get selfcal images.
Task presence in a Reference Manual is never promoted to recipe/default or
delivery without another statement.

## Operations release matrix

The date ranges below follow the current official operations table unless a
cell is marked **CONFLICT**. End dates use the table's boundary convention;
adjacent rows can repeat a boundary date. “Package consequence” is restricted
to what the reviewed documents actually establish.

| Operations interval | CASA | Pipeline release/build | Capability and status distinctions | Package / archive-visible consequence | Sources | Confidence |
|---|---|---|---|---|---|---|
| 2016-10-25 → 2017-04-17 | 4.7.0 r38355 | Pipeline-Cycle4-R2-B r38377 | Science-target imaging enters the interferometric **recipe**; check-source imaging and `hifa_spwphaseup` low-SNR calibration are in the release; SD processing changes from scantables to MS. The 2023 paper says first automated image products were delivered in 2017, so release availability must not be read as immediate successful delivery for every MOUS. | Export suffix changes from `.tar.gz` to `.tgz`. `flux.csv`, `antennapos.csv`, flag templates, and `cont.dat` are documented helper files. | [OPS](https://almascience.nrao.edu/processing/science-pipeline), [UG47 §2](https://almascience.org/documents-and-tools/alma-science-pipeline-users-guide-casa-4.7.0), [H23 §1](https://arxiv.org/abs/2306.07420) | High for release/dates and feature introduction; medium for exact earliest delivered-package prevalence. |
| 2017-04-17 → 2017-10-01 | 4.7.2 r39762 | Pipeline-Cycle4-R2-B r39732 | `hif_checkproductsize` first **exists** and is in the imaging **recipe/default**; default triggers are predicted cube >30 GB or all products >400 GB. Check-source images are now exported. | Product-size mitigation may coarsen channels/cells, reduce field of view, or reduce the number of imaged sources; delivered images therefore need not cover all science content. | [OPS](https://almascience.nrao.edu/processing/science-pipeline), [UG472 §2.2](https://almascience.org/processing/documents-and-tools/alma-science-pipeline-users-guide-casa-4-7.2), [RM472](https://almascience.org/processing/documents-and-tools/alma-science-pipeline-reference-manual-4-7.2) | High. |
| 2017-10-01 → 2018-10-01 | 5.1.1-5 r40896 | Pipeline-CASA51-P2-B r40896 | Advanced corrected-amplitude flagging replaces `hif_gainflag` in the **recipe**; `hifa_spwphaseup` adds per-SPW mapping before combination; automatic masking is used for target imaging; a representative-bandwidth cube is created when requested resolution is >4× native channel width; SD restore task added. | Guide says the continuum-subtraction calibration table is provided with products. Product-size mitigation remains and is more aggressive. | [OPS](https://almascience.nrao.edu/processing/science-pipeline), [UG51 §3](https://almascience.org/processing/documents-and-tools/alma-science-pipeline-users-guide-casa-5-1.1) | High for r40896; operations status of the guide's additional r40833/CASA 5.1.0 build is unresolved below. |
| 2018-10-01 → 2019-10-01 | 5.4.0-70 | Pipeline-CASA54-P1-B r42254 and r42866 | Parallel imaging is supported when CASA is run under MPI; heterogeneous 7m+12m EBs and ephemeris-source imaging are supported; virtual-SPW IDs are introduced; `hif_makeimages` check-source stage added. | `*flagtsystemplate.txt` and the other flag templates are explicitly exported inside `auxproducts.tgz`. Virtual SPW IDs become archive-visible in weblogs and later product naming logic. | [OPS](https://almascience.nrao.edu/processing/science-pipeline), [UG54 §3](https://almascience.org/processing/documents-and-tools/alma-science-pipeline-users-guide-casa-5-4), [RM54](https://almascience.org/processing/documents-and-tools/alma-science-pipeline-reference-manual-5-4) | Medium: capability family is high; exact internal-build intervals conflict with the guide (42030/CASA 5.4.0-68). |
| 2019-10-01 → 2021-05-10 per OPS (**CONFLICT** from 2020-11 onward) | 5.6.1-8 | Pipeline-CASA56-P1-B r42866 | Spectral scans and multi-target/multi-spectral-spec projects enter supported cases; IF and SD ephemeris support improves; SD gains parallel execution; cube imaging adopts `perchanweightdensity=False`; applycal starts using a saved callibrary. | When `hif_findcont` finds no line ranges, the corresponding continuum-subtracted cube is not cleaned, so absence of a cube can be a recipe decision rather than a download failure. | [OPS](https://almascience.nrao.edu/processing/science-pipeline), [UG56 §3](https://almascience.org/processing/documents-and-tools/alma-science-pipeline-users-guide-casa-5-6.1) | Medium. UG56 names initial r42833, and UG20 says 2020.1.0.36 entered operations in 2020-11, overlapping OPS's end date. |
| from 2020-11, end unknown (**CONFLICT**) | 6.1.1-10 | 2020.1.0.36 | Python-3-era release; full-pol data can go through a polarization-friendly total-intensity calibration recipe, but instrumental-polarization solving is still manual; new `hifa_targetflag` is a default science-target outlier stage; `hif_findcont` gains moment-difference analysis. | Weblog gains the new flagging/findcont evidence. This is **not** complete pipeline polarization calibration and does not imply IQUV science products. | [UG20 §1.1, §3](https://almascience.nrao.edu/documents-and-tools/alma-science-pipeline-users-guide-casa-6-1.1), [RM20](https://almascience.org/documents-and-tools/reference-manual-2020.1), [H23 §1](https://arxiv.org/abs/2306.07420) | High that the guide says deployed in 2020-11; low for exact end date because the current OPS table omits .36. |
| 2021-05-10 → 2021-10-01 | 6.1.1-15 | 2020.1.0-40 | Current operations table calls this the Cycle 7 reprise. Capability family is 2020.1; no separate User's Guide delta was found for .40. | No patch-specific package delta established. | [OPS](https://almascience.nrao.edu/processing/science-pipeline), [UG20](https://almascience.nrao.edu/documents-and-tools/alma-science-pipeline-users-guide-casa-6-1.1) | High for dates/version; low for .36→.40 behavioral delta. |
| 2021-10-01 → 2022-09-26 | 6.2.1-7 | 2021.2.0.128 | `hifa_renorm` first **exists** and is in calibration recipes before export; deterministic BDF flags extend a flag in one correlation to all polarizations; `briggsbwtaper`/`perchanweightdensity=True` used for cubes; ephemeris cube parallelization fixed. | `hif_exportdata` adds manifest elements allowing the archive to rename FITS precisely; check-source QA subscores enter AQUA. Renorm at this release edits calibrated data before export, rather than producing a calibration table. | [OPS](https://almascience.nrao.edu/processing/science-pipeline), [UG21 §3](https://almascience.nrao.edu/documents-and-tools/alma-science-pipeline-users-guide-casa-6-2.1), [RM21](https://almascience.org/documents-and-tools/reference-manual-2021.2) | High. |
| 2022-09-27 → 2023-04-18 | 6.4.1-12 | 2022.2.0.64 | Low-SNR calibration is assessed per phase/check source; phase-offset QA added; `hif_mstransform` creates `_targets.ms` and `hif_uvcontsub` creates `_targets_line.ms` in preparation for selfcal. These are processing views, not proof that calibrated MSs are archived. SD spectral scans and ephemeris cubes are supported. | `applycalQA_outliers.txt` is linked from the weblog. AQUA gains pbcor extrema; run workspace now has separate continuum+line and continuum-subtracted target MS views. | [OPS](https://almascience.nrao.edu/processing/science-pipeline), [UG22 §3](https://almascience.nrao.edu/documents-and-tools/cycle9/alma-science-pipeline-users-guide-casa-6-4.1), [RM22](https://almascience.org/documents-and-tools/cycle9/reference-manual-2022.2), [H23 §9.2](https://arxiv.org/abs/2306.07420) | High. Note the guide's package string says `casa-6.4.2-12` while its prose, weblog label, OPS, and all other evidence say CASA 6.4.1-12; treated as a guide typo, not a second CASA version. |
| 2023-04-18 → 2023-09-30 | 6.4.1-12 | 2022.2.0.68 | Patch fixes SD single-polarization weighting artifacts, SD spline `npiece`, CASA startup on macOS, and unsafe weblog-viewing advice. Core 2022.2 capabilities unchanged. | All SD products affected by the polarization-weight bug were slated for reprocessing; a package may therefore be a later generation despite the same observing cycle. | [OPS](https://almascience.nrao.edu/processing/science-pipeline), [UG22P §3.1](https://almascience.nrao.edu/documents-and-tools/cycle9/alma_pipeline_users_guide_2022_1) | High. |
| 2023-09-30 → 2024-09-29 | 6.5.4-9 | 2023.1.0.124 | `hif_selfcal` first **exists** and **attempts** eligible single-field, non-ephemeris sources; gains are applied only when S/N improves and beam change stays within limit. `hifa_polcal` now solves instrumental polarization, but IQUV science-target imaging is not yet claimed. | Successful selfcal products gain `selfcal`; regular products gain `regcal`; manifest image entries carry datatype, and the image name is added to AQUA sensitivity entries. Failed/ineligible selfcal does not imply a selfcal image. | [OPS](https://almascience.nrao.edu/processing/science-pipeline), [UG23 §3](https://almascience.nrao.edu/documents-and-tools/cycle10/alma_pipeline_users_guide_2023), [RM23](https://almascience.org/documents-and-tools/cycle10/reference-manual-2023) | High. Runtime JSON filenames and delivery prevalence require package evidence. |
| 2024-09-30 → 2025-09-29 | 6.6.1-17 | 2024.1.0.8 | `hifa_diffgaincal` enters new B2B calibration/imaging recipes; `hifa_tsysflagcontamination` added; renorm now makes a Tsys-like caltable applied in `hif_applycal`/restore; SD imaging changes from `sdimaging` to `tsdimaging`. Selfcal remains conditional; new QA distinguishes attempted/not attempted/succeeded but a successful stage still need not improve every target. | `cont.dat` adds SPW names to map virtual to per-EB real IDs. Renorm-as-caltable changes restore/calibration products. Selfcal/regcal datatype is shown in weblog. | [OPS](https://almascience.nrao.edu/processing/science-pipeline), [UG24 §3](https://almascience.nrao.edu/processing/alma_pipeline_user_s_guide_for_release_2024-1.pdf), [RM24](https://almascience.nrao.edu/processing/reference-manual-2024.pdf), [Z24](https://zenodo.org/records/14502284) | High. |
| 2025-09-29 → 2026-03-04 | 6.6.6-17 | 2025.1.0.35 | Selfcal expands to mosaics and improves long-baseline fallback; IQUV target imaging enters polarization recipes; low-SNR/B2B calibration expands; `hifa_antpos` queries online positions by default. | Per-EB `uid*antennapos.json` helper files (absolute positions) replace the normal per-MOUS CSV path, while `antennapos.csv` remains a manual override. Full-pol products add `mfs_fullpol`, `cont_fullpol`, `cube_fullpol`, and `cube_repBW_fullpol` imaging variants. AQUA adopts explicit observed/theoretical sensitivity tags. | [OPS](https://almascience.nrao.edu/processing/science-pipeline), [UG25 §3, §5.2](https://almascience.org/processing/documents-and-tools/cycle12/alma_pipeline_users_guide_2025), [RM25](https://almascience.org/processing/documents-and-tools/cycle12/reference-manual-2025) | High for helper format and capabilities; medium that JSON is always delivered rather than merely present in the working directory. |
| 2026-03-05 → 2026-05-01 | 6.6.6-18 | 2025.1.0.36 | Patch-specific confirmed fix: CASA versions before 6.6.6.18 could unwrap phase incorrectly under `linearPD`, relevant to B2B calibration. No separate User's Guide was published. | Known issue records that `.36` could export only one of a representative-bandwidth cube and full-resolution cube for the same source/SPW. Affected missing images can be requested. | [OPS](https://almascience.nrao.edu/processing/science-pipeline), [KI PL2025](https://casaguides.nrao.edu/index.php?title=ALMA_Pipeline_Known_Issues#PL2025) | High for dates/bugs; medium for the inference that the CASA `linearPD` fix is the principal .35→.36 delta. |
| 2026-05-01 → present (read 2026-07-18) | 6.6.6-18 | 2025.1.0.37 | Cycle 12 patch; no separate capability guide. | Corrects `.36` failure to export both repBW and full-resolution cubes. Packages made by `.36` can be incomplete in exactly this way without a failed download. | [OPS](https://almascience.nrao.edu/processing/science-pipeline), [KI PL2025 item 21](https://casaguides.nrao.edu/index.php?title=ALMA_Pipeline_Known_Issues#PL2025) | High. |

## Other identifiable operational builds and unresolved intervals

These builds are named by contemporaneous User's Guides but are omitted or
collapsed by the current operations table. They should not be silently merged
into the matrix until an operations log or archived version tracker supplies
exact boundaries.

| Build disclosed by guide | Guide's operational statement | Conflict / gap |
|---|---|---|
| Pipeline 40833 / CASA 5.1.0 | UG51 says the document also applies to it; main deployed build is 40896 / CASA 5.1.1-5. | Operations status/date for 40833 not established. It may be a public or short-lived precursor rather than a normal operations release. |
| Pipeline-CASA54-P1-B 42030 / CASA 5.4.0-68 | UG54 says deployed in 2018-10. | OPS instead gives CASA 5.4.0-70 with pipeline r42254 and r42866 for the entire 2018-10-01–2019-10-01 interval. Dates among 42030, 42254, and 42866 are unknown. |
| Pipeline-CASA56-P1-B 42833 / CASA 5.6.1-8 | UG56 says deployed in 2019-10. | OPS lists pipeline r42866 with the same CASA build. Transition date unknown. |
| 2020.1.0.36 / CASA 6.1.1-10 | UG20 says deployed in 2020-11; H23 independently describes the second Cycle-7/Python-3 release as 2020-10. | OPS says 5.6.1 remained the operations version until 2021-05-10 and lists only 2020.1.0-40 thereafter. This is a genuine contradiction, not a rounding issue. |

## Cycle projection (secondary view only)

| Observing cycle | Processing releases an agent should expect | Package-era cues; not guarantees |
|---|---|---|
| Cycle 4 (2016.1) | 4.7.0, 4.7.2, and later releases for late/QA3 work | First operations imaging; `.tgz`; product-size mitigation begins in patch. |
| Cycle 5 (2017.1) | 5.1.1 family and later reprocessing | Automasking, representative-bandwidth cubes, improved calibrated-amplitude flagging. |
| Cycle 6 (2018.1) | 5.4 family and later | Virtual SPWs; flag/Tsys templates explicitly in `auxproducts.tgz`; MPI-capable imaging. |
| Cycle 7 (2019.1/2019.2) | 5.6.1 and 2020.1 builds; exact transition disputed | Callibrary, improved ephemeris/spectral-scan support; 2020 adds target flagging and moment-difference findcont. |
| Cycle 8 (2021.1) | 2021.2, sometimes later | Renorm enters recipe; manifest gains precise archive-renaming elements. |
| Cycle 9 (2022.1) | 2022.2.0.64/.68 | Separate target and line target MS views in processing; applycal outlier text; SD patch may cause re-delivery. |
| Cycle 10 (2023.1) | 2023.1 and later | Conditional single-field selfcal; `regcal`/`selfcal` and manifest datatype; polcal solving. |
| Cycle 11 (2024.1) | 2024.1 and later | B2B recipes, renorm caltable, SPW names in `cont.dat`, `tsdimaging`. |
| Cycle 12 (2025.1) | 2025.1.0.35/.36/.37 | Mosaic selfcal, IQUV target imaging, per-EB antenna-position JSON; `.36` cube-export hole. |

Cycle is never a reliable substitute for reading CASA/pipeline versions from
the MOUS weblog, manifest, QA2 report, or logs. Late processing, QA3, and
re-delivery can put any older project under a newer row.

## Empirical public-package cross-check (2026-07-18)

One QA2-PASS auxiliary package was inspected for every public Cycle 4--11.
These are examples of processing releases, not claims about all data in a
cycle. Top-level tar members, scripts, manifests, nested auxproducts, QA
reports, weblog listobs, and DataLink product inventories were checked; large
FITS product tars and raw ASDMs were not downloaded.

| Cycle | Project / MOUS | Package release evidence | High-value observed cue |
|---|---|---|---|
| 4 | `2016.1.00004.S` / `uid://A001/X879/Xea` | CASA 4.7.2, manual | Large flat auxiliary payload with manual calibration/imaging scripts; no PPR, manifest, or pipeline weblog. |
| 5 | `2017.1.00017.S` / `uid://A001/X1289/X1a1` | CASA 5.4.0-68, pipeline 42030M | Nested modern auxiliary roles; full and singular `_target.ms` listobs views. |
| 6 | `2018.1.00035.L` / `uid://A001/X133d/Xb5a` | CASA 5.4.0-70, pipeline 42254M | `hifa_calimage`; virtual-SPW-era helper bundle. |
| 7 | `2019.1.00003.S` / `uid://A001/X1470/X2ef` | CASA 6.1.1.15, PL2020.1.0.40 | Full and singular `_target.ms` views; older `cont.dat` grammar. |
| 8 | `2021.1.00018.S` / `uid://A001/X15b8/X11` | CASA 6.2.1.7, PL2021.2.0.128 | `hifa_calimage_renorm`; renorm stage and bare-`ALL` `cont.dat`. |
| 9 | `2022.1.00014.S` / `uid://A001/X2d20/X3ee5` | CASA 6.4.1.12, PL2022.2.0.64 | Plural `_targets.ms` and `_targets_line.ms`; AQUA XML and applycal-outlier text in weblog. |
| 10 | `2023.1.00026.S` / `uid://A001/X3621/X3d68` | CASA 6.5.4.9, PL2023.1.0.124 | Selfcal stage/JSON present but `scal_targets` empty; archive product names did not preserve a `regcal` token. |
| 11 | `2024.1.00056.S` / `uid://A001/X3788/X7af3` | CASA 6.6.1.17, PL2024.1.0.8 | SPW names/`Flags:` in `cont.dat`; selfcal/timetracker/stats JSON; empty selfcal targets again. |

The corpus supports the release boundaries for target-MS views, AQUA/outlier
sidecars, `cont.dat`, and runtime JSON, while demonstrating that recipe/stage
or filename presence is not success evidence. Three tiny SEMIPASS screening
packages also showed that ObsCore `qa2_passed='T'` can coexist with a
three-state SEMIPASS report and that a MOUS may have only QA/raw evidence.
See `review/sample-inventory.tsv` and `review/findings-sample.md` for the exact
file and acquisition inventory.

Recipe labels also lag internal stage composition: the sampled PL2022 plain
`hifa_calimage` script calls `hifa_renorm`, and the sampled PL2024 plain
`hifa_calimage` request calls `hif_selfcal`. Keep **stage called**, **target
eligible**, and **result accepted** separate.

## Evidence gaps to ground-truth in packages

1. Sampled PL2018--PL2024 packages confirm delivered pipeline manifests and
   useful version/procedure/image metadata, but do not establish one stable
   filename grammar or XML schema across all releases.
2. Sampled PL2023/PL2024 packages establish selfcal/timetracker JSON delivery,
   and the PL2024 sample adds pipeline-stats JSON. Introduction dates and
   prevalence outside this corpus remain empirical rather than documented
   contracts.
3. UG25 proves the per-EB antenna-position JSON is a pipeline helper and
   default online-query product. It does not say that every archive package
   exports it; package samples must establish delivery prevalence.
4. The Cycle 4 sample has detailed README/manual QA content while sampled
   Cycle 5+ packages use a generic README pointer plus QA2 report, but this
   small corpus does not establish an exact universal transition date or all
   historical directory placements.
5. No guide proves that calibrated `_targets.ms`/`_targets_line.ms` are archive
   deliverables. The guides repeatedly state calibrated MSs are produced by a
   run but not stored in the archive.
6. Exact initial/later build boundaries in Cycle 6, Cycle 7, and the 2020.1
   reprise need a frozen operations version tracker or package-manifest corpus.
