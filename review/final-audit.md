# Final adversarial audit

Snapshot note: this file is the first-review post-edit audit. The subsequent
review2 implementation made further corrections to the current skill and
references; treat this file as historical provenance, not current readiness.

Audit date: 2026-07-18 UTC  
Snapshot: current `SKILL.md`, every `references/*.md`, `review/claims.tsv`,
`review/sample-inventory.tsv`, all source ledgers, and the four domain/sample
findings reports after the empirical edits were applied. This audit did not
edit the skill or its reference files.

Post-audit resolution: the root task applied S1--S8, replaced deleted scratch
paths with an explicit historical placeholder, extended `claims.tsv`, added
reference contents lists, and generated `agents/openai.yaml` using the
skill-creator helper. The canonical validator was invoked but could not import
its unavailable environment dependency PyYAML; an equivalent local structural
check of frontmatter, naming, metadata, length, and local links passed. Thus S9
remains an environment-specific follow-up, not a known content defect.

## BLOCKER

None found after the final empirical edits and verified scratch cleanup.

## SHOULD

### S1 — Say CASA-compatible, not categorically matching

`SKILL.md:76` says restore requires the “matching CASA version,” whereas
`references/products-and-qa.md:197-200` correctly distinguishes the exact
original version for identical reproduction from currently accepted newer
restore versions. Use “a version authorized by the package/current ALMA
compatibility table; use the original for identical reproduction.”

### S2 — Condition the correlator signal-path summary by receiver type

`references/cycle-capabilities.md:67` states an unconditional “2 sidebands”
signal path, but the same reference lists SSB (Band 1), 2SB, and DSB receiver
types. Recast the arrow as the common 2SB case and direct SSB/DSB work to the
matching Technical Handbook. The following correlator numbers are already
appropriately labelled era-specific.

### S3 — Correct “observed PL2025” and absence-based reduction recognition

`references/products-and-qa.md:116` says PL2024–2025 were observed in modern
deliveries, while `review/sample-inventory.tsv` and claim `PIPE-005` explicitly
say no public Cycle 12/PL2025 package was available. Say “PL2024 observed;
PL2025 documented.”

At lines 122-125, absence of PPR/manifest/weblog is useful manual-reduction
evidence only after a complete relevant auxiliary inventory. In a partial
Request Handler selection, absence proves nothing; repeat the guardrail here.

### S4 — Keep `applycalQA_outliers.txt` wording consistent everywhere

`references/products-and-qa.md` now correctly says sampled files contained
thresholds only. `references/identifiers-and-packaging.md:206-207` still calls
the weblog a source of “applycal outliers” without the distinction. Prefer
“applycal-QA configuration/outlier evidence; inspect contents before claiming
an outlier.”

### S5 — Make empirical provenance survive required cleanup

The empirical report is unusually good about its limitations: all eight PASS
samples are TM1, exactly one processed EB, mostly Band 6, and exclude ACA, TP,
multi-EB, B2B, full-pol, ephemeris, solar, VLBI, and phased-array packages.
The release/table conclusions are generally scoped accordingly.

However, `sample-inventory.tsv` stores now-deleted absolute `/tmp/...` paths
and the “reproducible commands” in `findings-sample.md` require that removed
scratch. Prefer scratch-relative archive names, explicitly say hashes were not
retained if that is the case, label the commands as the command log used for
the review, and explain that the listed MOUS must be redownloaded before
rerunning them. Do not fabricate hashes after cleanup.

Do not broaden the empirical claims: the corpus supports release-era package
and MS-view cues, not array/mode prevalence or multi-EB behavior.

### S6 — Fix the source registry's hash promise

`review/sources.md:6-8,36-39` says every domain ledger preserves PDF hashes.
Only `sources-archive.md` actually lists hashes; the pipeline and QA/packaging
ledgers record conversion method but no hashes. Either add the recorded hashes
or narrow the registry statement. Do not fabricate hashes after the temporary
PDFs are removed.

### S7 — Mark findings reports as pre-edit review records

The findings files contain “current text” and “proposed edit” language even
after many edits landed. Add a short resolution banner or resolution column so
future maintainers do not reapply historical recommendations. The TP-footprint
cross-domain correction is now handled correctly in
`findings-archive.md:92`, and should be the model for resolving other
cross-review conflicts.

### S8 — Extend the claim registry for high-risk empirical corrections

`claims.tsv` captures the main archive/pipeline corrections, but it does not
yet register several high-risk final rules established by the sample pass:

- numbered FITS product tars are not required for a normal local MS restore;
- `applycalQA_outliers.txt` presence does not prove an outlier;
- PPR/manifest EB lists describe the processed set, not necessarily all raw
  MOUS membership;
- top-level tar mtimes can be repackaging rather than processing dates;
- the current TP single-pointing footprint limitation is dated and must be
  rechecked.

Add rows or explicitly define `claims.tsv` as a non-exhaustive priority set.

### S9 — Run the canonical validator in a prepared environment

Manual inspection found valid two-key frontmatter, a valid hyphenated name,
and a 145-line `SKILL.md` below the 500-line guidance. The canonical
`quick_validate.py` could not run in this workspace because PyYAML is absent
(`ModuleNotFoundError: yaml`); this is an environment dependency failure, not
a demonstrated skill failure. Run it where its declared dependencies are
available. Consider generating `agents/openai.yaml`, which skill-creator
recommends for UI metadata but the skill currently lacks.

## OK

- The dangerous Cycle 9 versus PL2023+ `DATA`/`CORRECTED_DATA` distinction is
  now correctly split in both `asdm-and-ms.md` and
  `listobs-and-intents.md`; both instruct readers to inspect real columns or
  producing commands.
- Restore guidance now correctly requires auxiliary plus raw ASDMs and makes
  the potentially 18–57 GB numbered FITS product tars optional unless the
  package itself says otherwise.
- The truncated extraction sentence in `identifiers-and-packaging.md` is
  repaired, and top-level versus nested archive prefixes are distinguished.
- The current TP footprint warning is supported by the live official Archive
  known-issues page, which on 2026-07-18 still says TP footprints show one
  antenna pointing rather than the full extent:
  https://almascience.eso.org/alma-data/archive/archive-documentation
- Local review links now resolve, including `findings-sample.md`; every row in
  `sample-inventory.tsv` has the advertised 24 tab-separated fields.
- The exact empirical scratch path `/tmp/alma-skill-review.o4OJKJ` was removed
  by the root task and independently verified absent; the roughly 2.95 GB of
  temporary package/PDF material is no longer recoverable from that path.
- Spot checks opened the Cycle 11 QA2 PDF, current archive page, current
  pipeline page, old pipeline archive, EA ARC support, NRAO SRDP, the legacy
  suffix KB article, and PL2025 reference manual. No broken external URL was
  confirmed in that high-risk set. This was a spot check, not an assertion
  that every rolling URL is permanent.
- The skill stays compact and uses progressive disclosure well: the core
  guardrails are in `SKILL.md`, release/schema detail is routed to one-level
  references, and empirical observations are generally labelled as corpus
  facts rather than universal contracts.
