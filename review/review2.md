Conduct an independent, evidence-driven review of the `working-with-alma-data` skill in `/workspace/working-with-alma-
  data`.

  First read `/workspace/AGENTS.md`, the complete `SKILL.md`, every file under `references/`, and the source/sample
  registries under `review/`. Treat the existing findings, claims register, and final audit only as leads—not as
  authoritative conclusions. Do not assume the prior implementation or review was correct.

  Your task is review-only: do not modify repository files unless I explicitly ask after seeing your report.

  Evaluate:

  1. Scientific and operational accuracy
     - Verify high-risk claims against current primary sources: official ALMA manuals, policies, pipeline guides/
     reference manuals, known-issues pages, CASA documentation, live TAP/DataLink behavior, and applicable IVOA
     specifications.
     - Check hierarchy and UID semantics, ObsCore row granularity and units, DataLink behavior, packaging eras, restore
     requirements, ASDM/MS column semantics, spectral frames, QA interpretation, product naming, mosaics/TP, and
     pipeline-release history.
     - Distinguish observing cycle from processing release and separate implemented, recipe-enabled, attempted,
     successful, and delivered capabilities.
     - Preserve genuine conflicts between official sources rather than manufacturing a resolution.

  2. Empirical claims
     - Audit `review/sample-inventory.tsv` and `review/findings-sample.md` for unsupported generalization, internal
     inconsistency, incorrect arithmetic, or claims stronger than an eight-MOUS, TM1-heavy, single-EB corpus permits.
     - The original package scratch was intentionally deleted. Do not treat historical paths as live evidence.
     - Use small read-only TAP/DataLink probes if needed. Do not download large product tars or raw ASDMs merely to
     repeat the prior exercise.
     - If a small package download is essential, preflight its exact byte count, use a unique `/tmp` directory, inspect
     tar members before extraction, and permanently clean up only that exact directory when finished.

  3. Skill quality
     - Review triggering metadata, `agents/openai.yaml`, progressive disclosure, reference routing, clarity,
     duplication, imperative guidance, and whether another agent could apply the skill safely.
     - Check that long references are navigable, local links resolve, frontmatter is valid, and the skill remains
     concise enough to load effectively.
     - Run the skill-creator validator if its dependencies are available. Report dependency failures separately from
     content failures.

  4. Provenance and maintainability
     - Check that time-sensitive claims are dated.
     - Verify that citations directly support nearby claims.
     - Review `review/claims.tsv` for important omissions or misleading statuses.
     - Confirm that pre-edit findings are clearly distinguishable from the current implemented guidance.
     - Check whether current retrieval behavior is incorrectly presented as historical original-delivery behavior, or
     vice versa.

  When obtaining a PDF, download it once, convert it to text once, and perform all subsequent review from that extracted
  text. Record title, version/date, URL, orientation, and any hash actually calculated. Do not fabricate missing hashes.

  Deliver a self-contained report organized as:

  - BLOCKERS: facts or instructions likely to cause incorrect science, unsafe downloads/restores, data loss, or serious
  archive misuse.
  - SHOULD FIX: material accuracy, provenance, maintainability, or usability issues.
  - OPTIONAL: lower-impact improvements.
  - CONFIRMED: important claims independently supported.
  - VALIDATION: commands/checks run and their outcomes.
  - LIMITATIONS: evidence you could not independently verify.

  For every issue, provide:

  - exact file and line;
  - current claim;
  - verdict;
  - primary evidence with a direct link;
  - precise recommended replacement or action.

  Prioritize primary official sources. If browsing technical material, do not rely on search snippets, secondary
  summaries, or the prior review’s paraphrases. Clearly label inferences and empirical observations. End with an
  explicit recommendation: ready as-is, ready after specified fixes, or not ready.

