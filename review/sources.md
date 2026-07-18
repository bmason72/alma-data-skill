# Documentation review source registry

Review date: 2026-07-18 UTC.

This is the entry point for the source-specific ledgers produced by the
systematic review. The detailed ledgers record document versions, dates,
orientation, URLs, one-time text-extraction method, and the exact claims each
source supports. The archive lane also retained PDF hashes; the other lanes
did not, and none were reconstructed after cleanup:

- [`sources-archive.md`](sources-archive.md): current Archive Manual and
  Primer, Users' Policies, archive notebooks and knowledge-base articles,
  live TAP/DataLink, IVOA specifications, and historical packaging guidance.
- [`sources-pipeline.md`](sources-pipeline.md): operations version table,
  Pipeline User's Guides and Reference Manuals from CASA 4.7 through PL2025,
  known issues, Zenodo releases, and the pipeline paper.
- [`sources-qa-packaging.md`](sources-qa-packaging.md): Cycle-specific QA2
  product documents, restore/reprocessing guidance, CASA task documentation,
  and service-scope pages.

## Evidence policy used

- Current service schema/capabilities and current controlling policy outrank
  stale examples embedded in notebooks or primers.
- Pipeline history is release-first. “Implemented,” “in a recipe,” “enabled,”
  “attempted,” “succeeded,” and “delivered” are separate statuses.
- Historical PI-delivery layout is separate from the current DataLink
  containerization of the same old observation.
- A high-risk operational claim needs an official document plus an
  operational artifact or independent primary evidence where available.
- Package observations are corpus facts, not universal invariants. Their
  exact sample identifiers and inventory are in [`sample-inventory.tsv`](sample-inventory.tsv)
  and [`findings-sample.md`](findings-sample.md).

## PDF handling

Each acquired PDF was downloaded to a unique review scratch directory and
converted to text once. Reviewers worked from that text thereafter. Conversion
details (and hashes where recorded) are in the domain ledgers; all PDFs,
extracted text, temporary conversion libraries, and package payloads were
removed after the review.
