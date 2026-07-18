# Plan: systematic documentation review to further update `working-with-alma-data`

Status: PLAN ONLY (not yet executed). Drafted 2026-07-18; revised same day
after external (codex gpt-5.6-sol) critique.
Scope: bring the skill up to date against current official ALMA
documentation, and add a verified history of ALMA pipeline capabilities
from Cycle 4 onward. The skill covers ALMA data as it exists in the
archive — WSU forecast material stays out of scope.

## Principles (from the critique)

- **Organize the history by pipeline release, not by cycle.** Software
  capability, recipe inclusion, default enablement, operational
  deployment, and delivery are different events with different dates. The
  primary object is a *release/patch matrix* with operational date ranges;
  the cycle-oriented view is a projection of it. Never write "Cycle N
  gained X" when the truth is "PL-YYYY.R, in operations from date D,
  gained X" — one cycle's MOUSs get processed under multiple releases
  (late processing, QA3, re-deliveries).
- **Label every source** by observing cycle, operations interval,
  publication date, and whether it is prospective (Proposer's Guide,
  Technical Handbook — describe *future* observations) or retrospective
  (pipeline guides, QA2 docs — describe processing/delivery). A current
  Proposer's Guide cannot validate current delivered products.
- **Typed provenance instead of a blanket two-source rule.** Ordinary
  claims: one authoritative primary source. High-risk claims (milestones,
  defaults, deployment dates): an operational artifact (real package,
  manifest, weblog) or a genuinely independent second source — a User
  Guide and its own release notes are NOT independent. Valid provenance
  types: official document §, live service schema, reproducible corpus
  query, package artifact. Empirical corpus findings are admissible with
  the query recorded.
- **Track capability status separately**: implemented / in recipe /
  enabled by default / attempted / succeeded / delivered.

## Phase 0 — Claim inventory (cheap, local-only)

Mechanically seed `review/claims.tsv` from `SKILL.md` + `references/*.md`
(claim, file:line, feeding source, risk tier, status). Triage by impact ×
volatility; do NOT aim for exhaustive extraction — low-risk stylistic
claims get spot checks only.

## Phase 1 — Source acquisition and registry

Fetch current versions and record title/version/date/URL/orientation
(prospective vs retrospective) in `review/sources.md`. Document families:

1. **Pipeline** (almascience.org science-pipeline page): (a) the
   operations/version table (which pipeline+CASA ran operations, from
   when); (b) per-release Pipeline User's Guide incl. its "What's New"
   section; (c) Pipeline Reference Manual (per-task hifa_*/hif_*/hsd_*);
   (d) pipeline known-issues pages. Include patch releases. Historical
   snapshots: the Pipeline Documentation Archive / Zenodo deposits; the
   2023 ALMA pipeline paper (Hunter et al., PASP) as historical synthesis.
2. **Archive**: ASA Manual and Primer (distinct official documents),
   archive-documentation portal pages, the official archive Jupyter
   notebooks, live `TAP_SCHEMA` introspection, and the normative IVOA
   standards the ASA implements (ObsCore, TAP, DataLink).
3. **QA2/products**: current QA2 Data Products document (+ earlier-cycle
   versions for the history phase), ALMA Knowledgebase articles
   (packaging, MS naming, restores, weblog navigation).
4. **Observing-side** (prospective; capability tables only): Proposer's
   Guide + Technical Handbook of the relevant cycles.
5. **Reprocessing ecosystem**: CASA Guides (interferometric + TP
   reprocessing), SRDP/ARC calibrated-MS service pages, ARI-L (Cycles 2–4
   add-on imaging products) documentation, relevant CASA task docs
   (importasdm, mstransform, listobs).

Many are PDF-only: download and read page ranges; archive local copies
with version stamps.

## Phase 2 — Confirm / contradict / harvest

Reviewers work **by domain across sources** (archive-query; packaging;
pipeline-calibration; imaging/products; QA), not one worker per source —
this avoids source-siloed adjudication of the same claim. Output per
domain in `review/findings-<domain>.md`: CONFIRMED / CONTRADICTED (quote +
proposed fix) / NEW facts (must be ALMA-peculiar, agent-trap-shaped,
durable) / STALE-RISK (needs "as of" label). Every finding carries typed
provenance (see Principles). Contradictions between sources escalate to a
high-capability critic pass (currently codex gpt-5.6-sol high) with both
quotes.

## Phase 3 — Pipeline capability history (releases first, then cycles)

Deliverable: `references/pipeline-history.md`.

1. **Release matrix** (the backbone): one row per pipeline release/patch
   from ~2016 (Cycle 4 era, CASA 4.7) to current — operations start/end
   dates, CASA version(s), recipes available, stage additions/removals
   (renormalization, findcont evolution, checkproductsize/mitigation
   rules, mstransform/uvcontsub target views, selfcal, polcal, SD
   stages), product/packaging consequences (auxproducts.tgz, cont.dat
   placement, selfcal products, manifest/stats files, per-EB vs per-MOUS
   artifacts, aux-format changes like antennapos.csv→json), QA-visible
   changes (AQUA format, README→QA2-report shift). Sources: family 1
   above, per release.
2. **Cycle projection**: map operational date ranges onto cycles for the
   user-facing view ("data observed in Cycle N were mostly processed
   under releases X/Y; expect Z in the package"), with explicit caveats
   for late/QA3 processing.
3. **Ground truthing**: sample real packages near each release boundary —
   across 12-m, 7-m, TP, manual-calibration, polarization, and diffgain
   cases where available (the local Cycle 11 corpus covers the newest
   boundary; older samples need archive downloads) — and check the matrix
   row against artifacts.
4. Status vocabulary per capability (implemented / in recipe / default /
   delivered), single-source entries explicitly marked.

`cycle-capabilities.md`'s "Pipeline / CASA coupling" section then shrinks
to a pointer at the history file.

## Phase 4 — Adjudication and application

Merge findings; apply edits file-by-file in the skill's existing style
(dense, trap-oriented, "as of" labels). Final adversarial pass over the
full diff by an independent high-capability model: "what would an agent
still get wrong; what is now redundant or contradictory".

## Phase 5 — Regression hooks

Commit `review/claims.tsv` (updated statuses + provenance) so the next
review is a diff, not a restart. Stamp each touched reference file with
review date + doc versions.

## Sequencing

Phase 0 local-only; Phases 1–2 need web access; Phase 3 is the largest
(fan out per release, synthesize once) and can run as its own session;
Phases 4–5 close out. Rough order of effort: 3 > 2 > 1 > 0 ≈ 4 ≈ 5.
