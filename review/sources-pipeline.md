# Pipeline-domain source registry

Review date: 2026-07-18. Scope: ALMA pipeline releases from the Cycle 4
era through the current PL2025 patches.

## Acquisition and evidence policy

All documents below are primary ALMA/CASA documents except the refereed
pipeline paper, which is an official retrospective synthesis written by the
pipeline team. PDF sources were downloaded into one isolated `mktemp`
directory and converted to text exactly once. Review was performed on those
text conversions. The temporary PDFs, text, and converter dependency were
deleted together after the notes in this directory were written.

Orientation labels used here:

- **operational-retrospective**: records software actually deployed or
  describes an already released pipeline;
- **living-retrospective**: continuously edited current page; valid for the
  present state, but not a frozen record of when an issue was first known;
- **technical reference**: proves that a task and parameters exist in a
  release, but does not by itself prove recipe inclusion, default enablement,
  successful execution, or archive delivery;
- **retrospective synthesis**: later history assembled by pipeline authors;
- **prospective subsection**: a future plan in an otherwise retrospective
  document. It is not evidence that the feature was deployed.

## Release and operations sources

| ID | Official source | Issued / observed scope | Orientation | Use in this review |
|---|---|---|---|---|
| OPS | [Overview and Pipeline: CASA versions accepted for ALMA data processing](https://almascience.nrao.edu/processing/science-pipeline) | live page read 2026-07-18; operations table from 2014 to present | operational-retrospective, but living | Primary source for exact operations date ranges and current restore compatibility. It conflicts with several contemporaneous guides; see findings. |
| KI | [ALMA Pipeline Known Issues](https://casaguides.nrao.edu/index.php?title=ALMA_Pipeline_Known_Issues) | live page read 2026-07-18; PL2020.1 through PL2025 sections | living-retrospective | Release-specific failure modes; especially the PL2025.1.0.36 export loss fixed in .37 and the CASA `<6.6.6.18` `linearPD` phase-unwrapping defect. |
| Z24 | [Zenodo record 14502284: PL2024 User's Guide](https://zenodo.org/records/14502284) | issued 2024-10 | frozen software-documentation deposit | Persistent snapshot and bibliographic metadata for the PL2024 guide. The ALMA portal copy was used for content. |

## Pipeline User's Guides

These are release-specific, operational-retrospective documents. “What's
New” establishes a feature's release-family introduction and often its recipe
status. It does not prove that a particular MOUS successfully exercised the
feature or that every runtime artifact was archived.

| ID | Guide | Document date | Release / CASA stated by guide | Relevant sections |
|---|---|---|---|---|
| UG47 | [ALMA Science Pipeline User's Guide for CASA 4.7.0](https://almascience.org/documents-and-tools/alma-science-pipeline-users-guide-casa-4.7.0) | 2016-10, Doc 4.13 v1 | Cycle 4; CASA 4.7.0 | §2 What's New; §3 versions; §6 helper files |
| UG472 | [ALMA Science Pipeline User's Guide for CASA 4.7.2](https://almascience.org/processing/documents-and-tools/alma-science-pipeline-users-guide-casa-4-7.2) | 2017-07, Doc 4.13 v2 | Cycle 4 patch; CASA 4.7.2 | §2.1–2.2 explicitly separates 4.7.0 and 4.7.2 |
| UG51 | [ALMA Science Pipeline User's Guide for CASA 5.1.1](https://almascience.org/processing/documents-and-tools/alma-science-pipeline-users-guide-casa-5-1.1) | 2017-11, Doc 5.13 v1 | Pipeline 40896 / CASA 5.1.1-5; also says it applies to 40833 / CASA 5.1.0 | §3 What's New; §7 helper files |
| UG54 | [ALMA Science Pipeline User's Guide for CASA 5.4](https://almascience.org/processing/documents-and-tools/alma-science-pipeline-users-guide-casa-5-4) | 2018-10, Doc 6.13 v1 | Pipeline 42030 / CASA 5.4.0-68 | §3 What's New; auxproducts export statement |
| UG56 | [ALMA Science Pipeline User's Guide for CASA 5.6.1](https://almascience.org/processing/documents-and-tools/alma-science-pipeline-users-guide-casa-5-6.1) | 2019-10, Doc 7.13 v1 | Pipeline 42833 / CASA 5.6.1-8 | §3 What's New |
| UG20 | [ALMA Science Pipeline User's Guide for Release 2020.1](https://almascience.nrao.edu/documents-and-tools/alma-science-pipeline-users-guide-casa-6-1.1) | 2020-10, Doc 7.13 v2 | 2020.1.0.36 / CASA 6.1.1-10; says deployed 2020-11 | §3 What's New |
| UG21 | [ALMA Science Pipeline User's Guide for Release 2021.2](https://almascience.nrao.edu/documents-and-tools/alma-science-pipeline-users-guide-casa-6-2.1) | 2021-10, Doc 2021.2 v1 | 2021.2.0.128 / CASA 6.2.1-7 | §3 What's New; §7 helper files |
| UG22 | [ALMA Science Pipeline User's Guide for Release 2022.2.0.64](https://almascience.nrao.edu/documents-and-tools/cycle9/alma-science-pipeline-users-guide-casa-6-4.1) | 2022-10, Doc 2022 v1 | 2022.2.0.64 / CASA 6.4.1-12 | §3 What's New |
| UG22P | [ALMA Science Pipeline User's Guide for Release 2022.2.0.68](https://almascience.nrao.edu/documents-and-tools/cycle9/alma_pipeline_users_guide_2022_1) | Doc 2022 v1.1 | .64 and .68 / CASA 6.4.1-12 | §3.1 is the authoritative .68 patch delta |
| UG23 | [ALMA Science Pipeline User's Guide for Release 2023.1.0.124](https://almascience.nrao.edu/documents-and-tools/cycle10/alma_pipeline_users_guide_2023) | 2023-10, Doc 2023 v1 | 2023.1.0.124 / CASA 6.5.4-9 | §3 What's New; selfcal/polcal and manifest changes |
| UG24 | [ALMA Science Pipeline User's Guide for Release 2024.1.0.8](https://almascience.nrao.edu/processing/alma_pipeline_user_s_guide_for_release_2024-1.pdf) | 2024-10, v1 | 2024.1.0.8 / CASA 6.6.1-17 | §3 What's New; §5 helper files |
| UG25 | [ALMA Science Pipeline User's Guide for Release 2025.1.0.35](https://almascience.org/processing/documents-and-tools/cycle12/alma_pipeline_users_guide_2025) | 2025-10, v1.1 | 2025.1.0.35 / CASA 6.6.6-17 | §3 What's New; §5.2 documents per-EB `uid*antennapos.json` |

## Pipeline Reference Manuals

The manuals were compared for task presence. The nine-release comparison
confirmed these first appearances in the sampled references:
`hif_checkproductsize` at 4.7.2; `hifa_targetflag` at 2020.1;
`hifa_renorm` at 2021.2; `hif_selfcal` and `hifa_polcal` at 2023.1; and
`hifa_diffgaincal` plus `hifa_tsysflagcontamination` at 2024.1.

| ID | Official reference manual | Release represented |
|---|---|---|
| RM47 | [CASA 4.7.0 Reference Manual](https://almascience.org/documents-and-tools/alma-science-pipeline-reference-manual-casa-4.7.0) | 4.7.0, Doc 4.14 v1 |
| RM472 | [CASA 4.7.2 Reference Manual](https://almascience.org/processing/documents-and-tools/alma-science-pipeline-reference-manual-4-7.2) | 4.7.2, Doc 4.14 v2 |
| RM54 | [CASA 5.4 Reference Manual](https://almascience.org/processing/documents-and-tools/alma-science-pipeline-reference-manual-5-4) | Cycle 6, initial Pipeline-CASA54 build |
| RM20 | [Pipeline 2020.1 Reference Manual](https://almascience.org/documents-and-tools/reference-manual-2020.1) | 2020.1.0.36 |
| RM21 | [Pipeline 2021.2 Reference Manual](https://almascience.org/documents-and-tools/reference-manual-2021.2) | 2021.2.0.128 |
| RM22 | [Pipeline 2022.2 Reference Manual](https://almascience.org/documents-and-tools/cycle9/reference-manual-2022.2) | 2022.2.0.64 |
| RM23 | [Pipeline 2023 Reference Manual](https://almascience.org/documents-and-tools/cycle10/reference-manual-2023) | 2023.1.0 |
| RM24 | [Pipeline 2024 Reference Manual](https://almascience.nrao.edu/processing/reference-manual-2024.pdf) | 2024.1.0.8 |
| RM25 | [Pipeline 2025 Reference Manual](https://almascience.org/processing/documents-and-tools/cycle12/reference-manual-2025) | 2025.1.0 family |

No separate reference manual was linked by the operations table for CASA
5.1.1 or 5.6.1. The contemporaneous User's Guides were used for those
release-family deltas.

## Historical synthesis

| ID | Source | Scope and orientation | Important boundary |
|---|---|---|---|
| H23 | [Hunter et al. 2023, *The ALMA Interferometric Pipeline Heuristics*](https://doi.org/10.1088/1538-3873/ace216); [author manuscript](https://arxiv.org/abs/2306.07420) | retrospective synthesis through 2022.2.0.64 / Cycle 9; §§1–8 retrospective, §9 prospective | Calibration pipeline entered operations in 2014-10; imaging entered operations in Cycle 4 and first automated images were delivered in 2017. Its §9 statements about selfcal, complete polcal, and combined imaging were plans at writing, not deployments. |

## Sources intentionally not used as operational proof

- Proposer's Guides and Technical Handbooks describe offered observing
  capabilities prospectively. They are useful for observing-mode context but
  cannot establish which pipeline release processed or packaged a MOUS.
- Search snippets, helpdesk answers, and third-party tutorials were not used
  to establish release milestones.
- A release-family User's Guide and its own “What's New” section count as one
  source, not two independent confirmations.
