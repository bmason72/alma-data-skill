# Source registry: QA, packaging, products, restores, MSs, and special cases

Review date: 2026-07-18. All sources are official ALMA, ESO/ARC, NRAO/CASA,
or NAOJ/EA-ARC material. “Retrospective” means the source describes data that
were processed or delivered. “Prospective” means the source described offered
observing capabilities and must not be used by itself to assert what a package
actually contains.

## Versioned QA2 Data Products documents (retrospective)

- **S01 — Cycles 4–6 family / Cycle 5 edition.** *ALMA QA2 Data Products for
  Cycle 5*, Doc 5.12, version 2.0, May 2018. Retrospective delivery and restore
  contract. URL:
  <http://almascience.org/documents-and-tools/cycle5/ALMAQA2Productsv5.1.pdf>
- **S02 — Cycle 7.** *ALMA QA2 Data Products for Cycle 7*, Doc 7.12,
  version 1.0, February 2021. Retrospective. URL:
  <https://almascience.org/documents-and-tools/cycle7/alma-qa2-data-products-for-cycle-7>
- **S03 — Cycle 8.** *ALMA QA2 Data Products for Cycle 8*, Doc 8.2,
  version 1.0, April 2022. Retrospective. URL:
  <https://almascience.org/documents-and-tools/cycle8/alma-qa2-data-products-for-cycle-8>
- **S04 — Cycle 9.** *ALMA QA2 Data Products for Cycle 9*, Doc 9.10,
  version 1.0, October 2022. Retrospective. URL:
  <https://almascience.org/documents-and-tools/cycle9/alma-qa2-data-products-for-cycle-9>
- **S05 — Cycle 10.** *ALMA QA2 Data Products for Cycle 10*, Doc 10.9,
  version 1.1, October 2023. Retrospective. URL:
  <https://almascience.org/documents-and-tools/cycle10/alma-qa2-data-products-for-cycle-10>
- **S06 — Cycle 11.** *ALMA QA2 Data Products for Cycle 11*, Doc 11.3,
  version 1.2, October 2024. Retrospective. URL:
  <https://almascience.org/documents-and-tools/alma-qa2-products-cycle11.pdf>
- **S07 — Cycle 12.** *ALMA QA2 Data Products for Cycle 12*, Doc 12.12,
  version 1.0, October 2025. Retrospective. URL:
  <https://almascience.nrao.edu/documents-and-tools/cycle12/alma-qa2-data-products-for-cycle-12>
- **S08 — Current official index of the preceding documents.** ALMA
  Knowledgebase, “What Calibration and Imaging products will be delivered to
  me?”, updated 2026-03-18. Retrospective operational summary, but rolling and
  therefore volatile. URL:
  <https://help.almascience.org/kb/articles/what-calibration-and-imaging-products-will-be-delivered-to-me>

Local acquisition note: S01–S07 were downloaded to one unique temporary
directory, `/tmp/alma-qa-packaging.ss0c4f`, totaling about 7.3 MB. Each PDF was
converted to text exactly once with pypdf 6.14.2 loaded only from that temporary
directory. Findings were made from the resulting text files, not by repeatedly
converting the PDFs. The temporary directory was removed at the end of this
review.

## Archive packaging and archive behavior (retrospective/current service)

- **S09 — Archive package eras.** ALMA Knowledgebase, “How are ALMA data
  products packaged?”, updated 2023-04-18. Retrospective. It explicitly says
  Cycle 1–4 processed products were mostly in the same numbered tar and Cycle 0
  packages included raw and calibrated MSs. URL:
  <https://help.almascience.org/kb/articles/how-are-alma-data-products-packaged>
- **S10 — Archive documentation / known issues.** Official Archive page,
  rolling current service page. It records the June 2018 Cycle 5+ split into a
  FITS tar plus auxiliary tar, individual-file availability, and the current
  TP-footprint limitation. URL:
  <https://almascience.eso.org/alma-data/archive/archive-documentation>
- **S11 — Archive documentation portal.** Official index; current page. It
  separately links the Archive Manual, Archive Primer, QA2 products, and notes
  that Cycle 5+ products are split between FITS and auxiliary tar files. URL:
  <https://almascience.eso.org/alma-data/archive/archive-documentation>

## Pipeline and helper-file documentation (release-specific retrospective)

- **S12 — PL2024 Pipeline User's Guide.** *ALMA Pipeline User's Guide for
  Release 2024.1*, version 1.0, October 2024. Release-specific description of
  helper files, `cont.dat`, flag templates, WebLog, QA scores, MS views, and
  limitations. URL:
  <https://almascience.org/processing/alma_pipeline_user_s_guide_for_release_2024-1.pdf>
- **S13 — PL2024 Pipeline Tasks Reference Manual.** Release 2024.1.0.8,
  October 2024. Release-specific task contract. URL:
  <https://almascience.org/processing/reference-manual-2024.pdf>
- **S14 — Historical pipeline/version table.** Official Pipeline Documentation
  Archive page. Retrospective operational/version table and restore-version
  warning. URL:
  <https://almascience.eso.org/documents-and-tools/pipeline-documentation-archive-oldpage>
- **S15 — `applycalQA_outliers.txt`.** S12, section 9.26 (`hif_applycal`): the
  file is linked from the corresponding WebLog page and holds details of
  detected outliers. Same URL as S12.

## Restore and MeasurementSet contracts (retrospective/current guidance)

- **S16 — Calibrated-MS retrieval and local restore.** ALMA Knowledgebase,
  “How do I obtain a file of calibrated visibilities (‘measurement set’) for
  ALMA data?”, updated 2025-05-01. Rolling current service and restore guidance.
  URL:
  <https://help.almascience.org/kb/articles/how-do-i-obtain-a-file-of-calibrated-visibilities-measurement-set-for-alma-data>
- **S17 — Interferometric regeneration.** ALMA Knowledgebase,
  “Interferometric Calibration and Imaging Regeneration”, updated 2024-03-14
  in the indexed copy. Retrospective workflow. URL:
  <https://help.almascience.org/kb/articles/interferometric-calibration-and-imaging-regeneration>
- **S18 — MS naming and columns.** ALMA Knowledgebase, “What MS naming
  conventions does ALMA follow?”, current article published 2025-12-15 and
  updated in 2026. Retrospective, but rolling. It tabulates `uid*.ms`,
  `target.ms`, `targets.ms`, `targets_line.ms`, `ms.split.cal`, and SD products.
  URL:
  <https://help.almascience.org/kb/articles/what-ms-naming-conventions-does-alma-follow>
- **S19 — Pre-manifest restore incompatibility.** ALMA Knowledgebase,
  “Why does scriptForPI.py crash with NameError: name 'hif_restoredata' is not
  defined?”, updated 2025-04-01. Retrospective. It says CASA 5.1.1+ cannot
  restore pre-2017 packages lacking a pipeline manifest. URL:
  <https://help.almascience.org/kb/articles/why-does-scriptforpi-py-crash-with-nameerror-name-hif-restoredata-is-not-defined>
- **S20 — Legacy archive suffix incompatibility.** ALMA Knowledgebase,
  “Why does scriptForPI.py crash with ... `.flagversions.tgz`?”, updated
  2023-11-01. Retrospective. It documents pre-CASA-4.7 `.tar.gz` calibration
  bundles versus newer `.tgz` expectations. URL:
  <https://help.almascience.org/kb/articles/why-does-scriptforpi-py-crash-with-the-error-no-such-file-or-directory-rawdata-uid-a002-x12345>
- **S21 — CASA `split`.** Official CASAdocs stable task documentation. Rolling
  software contract: selecting one input data column always writes it to output
  `DATA`; default input is `corrected`. URL:
  <https://casadocs.readthedocs.io/en/stable/api/tt/casatasks.manipulation.split.html>
- **S22 — CASA `importasdm`.** Official CASAdocs 6.6.0 task documentation.
  Version-specific software contract for online flags, binary flags,
  correlation selection, and ephemeris conversion. URL:
  <https://casadocs.readthedocs.io/en/v6.6.0/api/tt/casatasks.data.importasdm.html>
- **S23 — Imaging Prep CASA Guide 6.5.4.** Official NRAO CASA Guide;
  retrospective workflow for restored ALMA data. It distinguishes Cycle 4–8
  `_target.ms`, Cycle 9+ `_targets.ms`, split-column behavior, EB-dependent
  sky frequencies, and ephemeris handling. URL:
  <https://casaguides.nrao.edu/index.php?title=Imaging_Prep_CASA_6.5.4>

## QA and spectral frames (retrospective/current operational guidance)

- **S24 — QA0 SemiPass.** ALMA Knowledgebase, “What does QA0 ‘SemiPass’ mean
  in Results Table of ALMA Archive Query?”, updated 2023-04-18. Retrospective.
  URL:
  <https://help.almascience.org/kb/articles/what-does-qa0-semipass-mean-in-results-table-of-alma-archive-query>
- **S25 — Frequency frames.** ALMA Knowledgebase, “What are the frequency
  reference frames in CASA?”, updated 2022-03-09. Retrospective ALMA/CASA
  behavior: fixed TOPO Doppler setting and common LSRK/SOURCE regridding. URL:
  <https://help.almascience.org/kb/articles/what-are-the-frequency-reference-frames-in-casa>
- **S26 — Historical mosaic imaging defects.** ALMA Knowledgebase, “I heard
  that images generated with CASA<5.1.1 are affected by CASA imaging issues.
  How should I deal with it?”, updated 2023-11-01. Retrospective. URL:
  <https://help.almascience.org/kb/articles/i-heard-that-images-generated-with-casa-5-1-1-are-affected-by-casa-imaging-issues-how-should-i>

## Total Power and cross-array handling

- **S27 — TP restore and imaging.** ALMA Knowledgebase, “How do I restore and
  image Total Power Data?”, updated 2025-03-28. Retrospective operational
  guidance. URL:
  <https://help.almascience.org/kb/articles/how-do-i-restore-and-image-total-power-data>
- **S28 — Current TP offering.** ALMA Knowledgebase, “Can I make a single dish
  observation?”, updated 2026-03-18. **Prospective for the then-current
  observing cycle**, not proof of historical deliveries. It confirms TP is an
  offered spectral-line, not continuum, capability for the stated cycle. URL:
  <https://help.almascience.org/kb/articles/can-i-make-a-single-dish-observation>
- **S29 — Cycle 12 Technical Handbook.** Doc 12.3, version 1.0, 2025-03-01.
  **Prospective observing-side source.** Consulted only as a capability
  cross-check; not used to validate package contents. URL:
  <https://almascience.eso.org/proposing/documents-and-tools/latest/alma-technical-handbook>

## Calibrated-MS services and legacy products (current services; volatile)

- **S30 — EU ARC CalMS.** Official EU ALMA Science Portal service page; service
  offered since 2020. Current/volatile. URL:
  <https://almascience.eso.org/tools/eu-arc-network/the-european-arc-calms-service>
- **S31 — EA ARC data-reduction support.** Official EA ARC page, current.
  Includes calibrated-MS delivery and the README-versus-QA2-report version
  guidance. URL:
  <https://www2.nao.ac.jp/~eaarc/DATARED/support_data_reduction_en.html>
- **S32 — NRAO ALMA SRDP.** Official NRAO page, current/volatile. It limits
  automated ALMA MS restore to pipeline-reduced Cycle 5+ 12-m/ACA data, with
  explicit exceptions and no current TP restore. URL:
  <https://science.nrao.edu/srdp/science-ready-data-products-srdp-for-alma>
- **S33 — ARI-L.** Official EU ALMA Science Portal page, retrospective project
  description plus current access instructions. It identifies ARI-L as
  externally delivered Cycle 2–4 products and reports 86% coverage of data
  processable by the pipeline. URL:
  <https://almascience.eso.org/alma-data/aril>
