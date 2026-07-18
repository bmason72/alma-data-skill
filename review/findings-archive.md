# Archive-query, identifier, and packaging review findings

Review date: 2026-07-18 UTC  
Reviewer scope: `SKILL.md`, `doc-review-plan.md`,
`references/archive-query.md`, and `references/identifiers-and-packaging.md`  
Evidence scope: current official ALMA manuals/policies/knowledge-base articles,
official ALMA archive notebooks, current IVOA Recommendations, Astroquery's
primary documentation, and read-only probes of the EU ALMA TAP/DataLink
services. See `sources-archive.md` for the full source ledger and PDF hashes.

Resolution note: this is the pre-edit evidence ledger. Its “current text” and
edit-map language records what was reviewed, not an instruction to reapply a
patch. The resulting rules are in `SKILL.md`/`references/`; cross-domain
resolution and remaining advice are in `final-audit.md`.

## Executive result

The references have the right operational center of gravity: query ObsCore,
group on `member_ous_uid`, enumerate deliverables through DataLink, keep raw
identifiers, treat package layout as versioned, and search by evidence rather
than one fixed path. Several statements nevertheless need correction before
the skill should be treated as authoritative:

1. The live ObsCore table has **73 columns**, not approximately 63. A number is
   inherently brittle; retain only the instruction to introspect
   `TAP_SCHEMA.columns`.
2. One representative `frequency_support` value per MOUS is unsafe. In a live
   MOUS it changed across EBs/fields, including shifted ranges and different
   sensitivities. Preserve all distinct values with row context.
3. The current controlling policy gives DDT projects **no proprietary period
   by default**, with an exceptional period of up to six months only if
   granted at proposal submission. The current Archive Primer contains both
   the obsolete six-month generalization and a footnote with the new policy;
   the Users' Policies document controls.
4. `uid://A002/...` is not a safe entity-type test. A live Cycle 1
   `member_ous_uid` is `uid://A002/X5d7935/X11b`. Use the ObsCore column,
   DataLink context, and ASDM filename/role to identify an EB.
5. Not every archive repeats the full ASA tree. Top-level delivery tarballs
   normally reconstruct a common tree when unpacked together; nested
   `*.auxproducts.tgz`, `*.caltables.tgz`, `*.flagversions.tgz`, and weblog
   archives have their own local payloads. Nested DataLink file enumeration
   can also flatten path context.
6. `frequency_support` currently includes both `@10km/s` and `@native`
   sensitivities. It is human-readable/versioned text, so a fixed-arity parser
   is inappropriate. The claim that its resolution is always a post-Hanning
   value was not substantiated by current documentation.
7. Current ALMA TAP capabilities advertise **ADQL 2.0**, not a generic 2.x,
   and advertise more geometry functions than just `CONTAINS` and
   `INTERSECTS`. Code should inspect capabilities instead of assuming ADQL
   2.1 support merely because 2.1 is the latest IVOA Recommendation.
8. The live archive repackages old data into present-day DataLink containers.
   Historical PI-delivery layout and currently downloadable layout are
   different questions and must not be conflated.

## Claim findings

Status meanings:

- **CONFIRMED**: supported by current primary evidence.
- **CONTRADICTED**: current primary evidence directly disagrees.
- **NEW**: an operational trap absent from the references.
- **STALE-RISK**: plausible or historically documented, but too volatile,
  weakly sourced, or overgeneralized to state as a current invariant.

### Endpoints, protocols, and service capabilities

| Status | Claim/result | Evidence and required action |
|---|---|---|
| CONFIRMED | EU, NA, and EA expose ALMA archive services, with TAP at `/tap`; the current EU ObsCore `access_url` is a DataLink URL and direct files use `/dataPortal/`. | Current Archive Manual and official notebooks describe the three regional copies and programmatic VO access. Live EU rows returned `https://almascience.org/datalink/sync?ID=...`, and DataLink file rows returned `/dataPortal/...`. Keep the endpoints, but treat returned URLs as authoritative. |
| STALE-RISK | “Three synchronized mirrors” overstates operational equivalence. | The Archive Manual calls them complete copies but also warns that requests are not transferable and that update timing can differ. Say “three regional archive copies; choose one and stay on it for a request.” Do not rewrite a returned host to another mirror. |
| CONFIRMED | A raw TAP sync POST with `REQUEST=doQuery`, `LANG=ADQL`, and `QUERY` is a safe interoperable form. | TAP 1.1 also permits sync GET, but specifies that GET may be cached while POST is expected when current results matter. Keep POST as the recommended raw form without implying GET is invalid. |
| CONTRADICTED | “Standard ADQL 2.x; geometry support is limited to common `CONTAINS`/`INTERSECTS` forms.” | The live EU capability document advertises ADQL **2.0** and `POINT`, `CIRCLE`, `BOX`, `POLYGON`, `REGION`, `CONTAINS`, `INTERSECTS`, `AREA`, `CENTROID`, `COORDSYS`, `COORD1`, and `COORD2`. Replace this with “inspect `/tap/capabilities`; as checked 2026-07-18 the service advertises ADQL 2.0 and these geometry features.” Latest IVOA ADQL is 2.1, but that does not make it an advertised ALMA feature. |
| NEW | Resource limits are discoverable and matter for broad queries. | On 2026-07-18 the EU capabilities advertised a 134,217,728-byte output limit, 600-second execution limit, 100,000-row upload limit, and 604,800-second retention. Do not freeze these numbers into general code; add a note to inspect capabilities, project only necessary columns, and shard broad queries. |
| STALE-RISK | The exact shape of ObsCore `access_url` is volatile. | Current rows are DataLink; the static output embedded in official notebook 0 still shows an older `http://almascience.org/aq?...` HTML URL. Determine behavior from `access_format` and the returned URL, not from a synthesized template. |

### ObsCore schema, row grain, and aggregation

| Status | Claim/result | Evidence and required action |
|---|---|---|
| CONFIRMED | ObsCore rows are finer than a MOUS and repeat across EBs, fields/sources, and spectral coverage. MOUS-level work groups on `member_ous_uid`; executions are distinct `asdm_uid` values. | Live recent-cycle rows and the Archive Primer's UI description support this. Retain the warning. Add that the archive web UI presents a grouped source/MOUS result, whereas raw TAP rows have a finer grain. |
| CONTRADICTED | “Parse one representative `frequency_support` row per MOUS.” | For MOUS `uid://A001/X3788/X62cc` (project `2024.1.00148.S`), distinct rows had shifted SPW ranges across EBs/fields and different sensitivities. Replace with: retain all distinct supports, keyed at least by `asdm_uid` and field/target/science context when available; de-duplicate identical strings; never sum repeated strings. |
| CONTRADICTED | “Currently ~63 columns.” | `SELECT COUNT(*) ... WHERE table_name='ivoa.obscore'` returned **73** on the EU service. The official notebook's static result still says 63, demonstrating why the number should be removed. Retain only runtime schema introspection. |
| CONFIRMED | There is no `science_goal_uid` column in live `ivoa.obscore`; `group_ous_uid`, `member_ous_uid`, and `asdm_uid` are present. | Live `TAP_SCHEMA.columns` inspection. Retain the package-path recovery warning, but phrase it as “not exposed by this current ObsCore schema,” not an eternal protocol guarantee. |
| CONFIRMED | Current key units/types include `t_min`/`t_max` in MJD days, `t_exptime` seconds, `em_min`/`em_max` meters, `frequency` GHz, `frequency_support` GHz text, `bandwidth` Hz, `s_resolution` and `spatial_resolution` arcsec, and `velocity_resolution` m/s. | Live `TAP_SCHEMA.columns`; Astroquery documentation independently labels `bandwidth` in Hz. The old official notebook table displays some stale units, so the live schema is the operational authority. |
| NEW | `access_estsize` and DataLink `content_length` use different contracts. | Live schema defines `access_estsize` as integer kbyte, and it was null for the sample MOUS. DataLink 1.1 defines `content_length` in bytes per link. Plan downloads only after DataLink enumeration; never treat `access_estsize` as the authoritative per-file byte count. |
| CONFIRMED | `qa2_passed` is only `T`/`F` and cannot represent PASS/SEMIPASS/FAIL. | A public `qa2_passed='F'` MOUS (`uid://A001/X3788/Xc582`) still exposed product, auxiliary, and raw deliverables. Keep the warning that `F` does not by itself mean “failed and unavailable.” |
| CONFIRMED | `science_observation='F'` denotes calibration-intent rows. | Live examples carried `BANDPASS FLUX WVR` and `PHASE WVR` scan intents. Keep the statement while treating `scan_intent` as multi-valued text. |
| CONFIRMED | `band_list` can be multi-band. | Live values included `4 9` and `3 7`. Current live values observed in the probe were numeric/space-separated; no `BAND N` value was found. Keep tolerant parsing of historical/display variants, but label numeric strings as the current observed representation and never use `band_list` as the sole science-tuning record. |
| STALE-RISK | `schedblock_name` suffixes and antenna name prefixes are useful array heuristics, not normative identity fields. | No current normative statement was found making those suffixes a complete classifier. Preserve them as best-effort heuristics and prefer actual antenna metadata, array-specific MOUS grouping, and observation context. |

### Coordinates and spatial querying

| Status | Claim/result | Evidence and required action |
|---|---|---|
| CONFIRMED | Use `s_region` for footprint intersection; a point test and a true cone-overlap test answer different questions. | Official notebooks use ADQL geometry, and the live service advertises the relevant functions. Keep the two query patterns. |
| CONFIRMED | `target_name` is PI-entered text rather than a resolver-normalized identifier. | Official notebook 1 explicitly uses an external name resolver for name-to-coordinate lookup. Keep coordinate-based matching. |
| CONFIRMED (cross-domain correction) | TP footprints have a current special-case defect, but it should remain dated. | The later QA/packaging review located the live official Archive known-issues statement: TP footprints can show one antenna pointing rather than the full mapped area (checked on both official mirrors 2026-07-18; see `sources-qa-packaging.md`, S10). Retain the warning specifically for TP, date it, and recheck before relying on it. Historical interferometric mosaic defects are a separate, remediated issue. |
| STALE-RISK | “`s_ra`/`s_dec` may be wrong for ephemeris/ToO targets” is too broad. | Current ToO guidance says Phase-I targets may use placeholder zero coordinates and executed coordinates can differ. Say that representative catalog coordinates may be insufficient or placeholders for moving/ToO targets; verify execution-level direction metadata before scientific matching. |

### Release dates and access rights

| Status | Claim/result | Evidence and required action |
|---|---|---|
| CONTRADICTED | “DDT ~6 months” as a default proprietary period. | Current ALMA Users' Policies section 9.4.1 says regular Pass/Semipass data have 12 months from delivery; DDT data have no proprietary period unless an exception of up to six months was requested and granted at proposal submission. Replace the parenthetical. The current Archive Primer's prose is internally stale; its footnote agrees with the current policy for Cycle 12 onward. |
| CONFIRMED | Do not locally recompute release eligibility. Use `data_rights`, authentication/authorization, and archive metadata. | Policy exceptions and extensions exist. Keep this operational rule. |
| NEW | `obs_release_date='3000-01-01T00:00:00.000'` occurs as a proprietary sentinel. | A live query for non-20xx release strings returned this value on proprietary rows. Treat it as an indefinite/placeholder sentinel, not a literal scheduled promise. Use `data_rights` and actual authorization state alongside the timestamp. |

### `frequency_support` and spectral-resolution claims

| Status | Claim/result | Evidence and required action |
|---|---|---|
| CONFIRMED | `frequency_support` is U-joined human-readable bracketed text describing SPWs, not a relational array. | Current Archive Manual and live values agree. Keep only a tolerant range extractor as a minimum parser. |
| CONTRADICTED | The shown segment grammar is complete enough to describe current output. | A live segment was `[216.90..218.88GHz,31250.00kHz,76.5mJy/beam@10km/s,4.6mJy/beam@native, XX YY]`. Add the native-resolution sensitivity and explicitly allow extra/versioned fields. The current manual also describes an observing-type element, even though no `continuum`/`line` token appeared in sampled live strings. Never split into a fixed number of comma fields without bracket-aware, suffix-tolerant handling. |
| STALE-RISK | The resolution is categorically “post-Hanning” and must never receive another Hanning factor. | Current sources say spectral resolution versus channel spacing depends on correlator mode and averaging; they do not substantiate the absolute post-Hanning statement. Replace with: use the archived value as an archive estimate; do not infer raw channel count or apply a universal correction without ASDM/MS correlator metadata. |
| NEW | `velocity_resolution` is an aggregate estimate, not safely a per-row SPW channel width. | The live schema describes it as estimated “from all spectral windows, from frequency resolution”; the same value repeated across rows with different SPWs. Do not combine it with a per-segment range as though both described one raw channelization. |

### Identifiers and entity classification

| Status | Claim/result | Evidence and required action |
|---|---|---|
| CONFIRMED | Project codes follow `YYYY.C.NNNNN.Z`; regular/DDT call indicator and final type are structured. | Archive Manual appendix. Keep the examples and type list, while describing historical gaps separately. |
| CONFIRMED | Normal modern OUS identifiers commonly use A001 and ASDM/EB identifiers commonly use A002; sanitized form replaces `://` with `___` and `/` with `_`. | Documentation and live paths support this as a naming convention. Keep it only as a recognition aid. |
| CONTRADICTED | An A002-prefix regex is sufficient to identify an EB stem. | Live Cycle 1 project `2012.1.00350.S` has `member_ous_uid='uid://A002/X5d7935/X11b'`. Promote the existing early-cycle caveat into an operational prohibition: never infer entity class solely from A001/A002. An EB is established by the `asdm_uid` column, raw-ASDM role/filename, or equivalent package context. |
| CONFIRMED | DataLink accepts both URI and sanitized forms for the tested MOUS. | Responses for `uid://A001/X3788/X62cc` and `uid___A001_X3788_X62cc` were equivalent apart from the echoed identifier/description. Keep both forms, but preserve the canonical URI internally when available. |
| STALE-RISK | Sanitized form is used in “every path and filename.” | It is overwhelmingly used for entity-bearing archive path/file components, but a universal “every” is unnecessary and vulnerable to exceptions. Say “the conventional filesystem-safe form used in archive entity path/file components.” |

### DataLink semantics and error handling

| Status | Claim/result | Evidence and required action |
|---|---|---|
| CONFIRMED | A DataLink table mixes direct file rows, service descriptors, nested links, documentation, and errors; only rows with a usable `access_url` are direct fetches. | Live response and DataLink 1.1. Retain the row-shape warning. DataLink requires exactly one of `access_url`, `service_def`, or `error_message` for a result row. |
| NEW | Valid-but-unauthorized and invalid identifiers can look different. | An invalid ID returned an explicit `NotFoundFault` row with `#error`; an anonymous query for a valid proprietary MOUS returned an empty table. Treat an empty response as “no links visible under current authorization,” not proof that the MOUS does not exist or has no files. Consult ObsCore `data_rights` and authenticated access. |
| NEW | Raw rows used both `#progenitor` and `#package`; semantics are not a complete ALMA product taxonomy. | In the live sample, three raw ASDM tar rows were `#progenitor` and another was `#package`. Preserve `semantics` and ALMA-local fields, but classify conservatively from URL/filename, content type/qualifier, and context. The distinction likely reflects used versus extra/semipass raw data, but that interpretation was not proven and must be labelled as inference if used. |
| NEW | Nested DataLink enumeration can expose flat individual files without the tar's original directory path. | Following the auxiliary service link returned individual caltables, auxproducts, scripts, QA reports, PPR, manifest, weblog, and log files. A downloader choosing those links must retain category/path provenance or accept that it is not reconstructing the delivery tree. Fetching the top-level auxiliary tar is the simplest structure-preserving route. |
| CONFIRMED | `content_length` is per-link bytes. | DataLink 1.1 and live response. Use it for preflight size accounting, subject to absent values and normal HTTP verification. |

### Packaging eras and tree structure

| Status | Claim/result | Evidence and required action |
|---|---|---|
| CONTRADICTED | “Roughly Cycle 4/5 onward” describes the present live container boundary, while earlier archive retrievals remain mixed. | The live archive currently returns separate product, auxiliary, README, and raw links even for a Cycle 1 MOUS and Cycle 4 MOUS. The official packaging KB says historical storage/delivery changed by cycle, but the current service can repackage old holdings. Split this section into (a) current DataLink containerization and (b) historical internal payload/delivery-era differences. |
| CONTRADICTED | Cycle 0 and early Cycle 1 “may include ready-to-use calibrated MSs directly” without qualification. | Current Archive Manual confirms calibrated MS delivery for Cycle 0. The packaging KB says some early Cycle 1 **original PI deliveries** used a similar arrangement but those calibrated MSs are not retrievable from the archive. State those two cases separately; never promise a current Cycle 1 calibrated-MS download. |
| CONFIRMED | Current deliveries normally organize member contents into `product`, `calibration`, `script`, `qa`, and `log`, with README; raw is separately selected. | Current Archive Manual, Cycle 12 QA2 products guide, current QA-products KB, and live DataLink. Keep “strong convention, not guarantee”; product can be absent when images were not selected, and raw-only/QA0-semipass situations exist. |
| CONTRADICTED | “Every tarball internally repeats the ASA tree.” | Official instructions tell users to unpack **top-level delivery tarballs** in one parent so they merge into a tree. Nested second-level archives do not repeat that full prefix, and individual nested DataLink downloads may be flat. Limit the prefix-stripping warning to top-level archives whose member names actually carry an ASA prefix. Inspect archive members before choosing the destination. |
| CONFIRMED | Unpacking related top-level product/auxiliary/raw archives into one controlled parent reconstructs their common project/science-goal/group/member tree. | Current manual and QA2 guide. This supports the skill's controlled-unpack logic, but prefix stripping must be based on the member list rather than an unconditional assumption. |
| CONFIRMED | Present raw filenames are project-prefixed and include the sanitized ASDM UID plus `.asdm.sdm.tar`. | Live example: `<project>_uid___A002_...asdm.sdm.tar`. Keep the broad glob and never synthesize the exact name from a documentation example; use the returned DataLink URL. |
| CONFIRMED | A direct plain-text `member.<MOUS>.README.txt` is currently exposed. | Live DataLink. Some normative delivery documents describe a README tar/container, so downloader logic should classify by the actual response and content rather than requiring an archive suffix. |
| CONFIRMED | MOUS is the QA2/processing/delivery unit and normally represents executions of one SB; array components are normally separate. | Current QA2 guide and Primer. Avoid making “one SB” absolute: the Primer notes commissioning-era exceptions in which a MOUS can contain more than one SB. |
| STALE-RISK | A science goal maps exactly to one GOUS. | Current documents conflict: the Cycle 12 QA2 guide says exact one-to-one, whereas the Cycle 13 Primer says one science goal can contain one or more GOUS. Preserve the actual hierarchy (SG → one or more GOUS → one or more MOUS) and identifiers; do not collapse it based on either prose sentence. |

## Reproducible live probes

These are observations of the EU service on 2026-07-18, not frozen service
contracts. Re-run schema/capability checks when implementing against another
mirror or at a later date.

### Schema count and selected contracts

```sql
SELECT COUNT(*) AS column_count
FROM TAP_SCHEMA.columns
WHERE table_name = 'ivoa.obscore'
```

Result: `73`.

```sql
SELECT column_name, datatype, unit, description
FROM TAP_SCHEMA.columns
WHERE table_name = 'ivoa.obscore'
  AND column_name IN (
    'member_ous_uid', 'group_ous_uid', 'asdm_uid', 'science_goal_uid',
    'frequency', 'bandwidth', 'frequency_support', 'velocity_resolution',
    's_resolution', 'spatial_resolution', 'access_url', 'access_format',
    'access_estsize', 'obs_release_date'
  )
ORDER BY column_name
```

Selected result summary:

- no `science_goal_uid` row;
- `bandwidth`: `double`, Hz;
- `frequency`: `double`, GHz;
- `frequency_support`: character, GHz;
- `velocity_resolution`: `double`, m/s;
- `s_resolution` and `spatial_resolution`: `double`, arcsec;
- `access_estsize`: integer, kbyte;
- `obs_release_date`: character timestamp.

### A row-grain and frequency-support sample

```sql
SELECT member_ous_uid, proposal_id, asdm_uid, target_name,
       science_observation, frequency_support, velocity_resolution
FROM ivoa.obscore
WHERE member_ous_uid = 'uid://A001/X3788/X62cc'
ORDER BY asdm_uid, target_name
```

The MOUS (`2024.1.00148.S`) had four distinct ASDM UIDs and repeated coverage
rows. Distinct `frequency_support` strings varied. One current segment was:

```text
[216.90..218.88GHz,31250.00kHz,76.5mJy/beam@10km/s,
 4.6mJy/beam@native, XX YY]
```

The same aggregate `velocity_resolution` appeared with different SPW rows.
This is direct evidence against choosing one arbitrary MOUS row or treating
the aggregate velocity estimate as one SPW's channel width.

### Release sentinel

```sql
SELECT TOP 20 proposal_id, member_ous_uid, data_rights, obs_release_date
FROM ivoa.obscore
WHERE obs_release_date IS NOT NULL
  AND obs_release_date NOT LIKE '20%'
```

Result rows included proprietary records with
`3000-01-01T00:00:00.000`.

### Early-cycle identifier counterexample

```sql
SELECT TOP 20 proposal_id, member_ous_uid, asdm_uid
FROM ivoa.obscore
WHERE proposal_id = '2012.1.00350.S'
```

Result includes member OUS `uid://A002/X5d7935/X11b`. Its current DataLink
response exposes a direct README, a separate product tar, an auxiliary tar,
and raw ASDM tars. This one row invalidates any unconditional “A002 means EB”
or “Cycle 1 is still retrieved only as a mixed container” rule.

### DataLink behavior

Queries tested:

```text
https://almascience.eso.org/datalink/sync?ID=uid://A001/X3788/X62cc
https://almascience.eso.org/datalink/sync?ID=uid___A001_X3788_X62cc
```

Both forms worked. The top-level VOTable identified DataLink 1.1 and included:

- a direct documentation/README row;
- product `#this` plus a related service-descriptor row;
- auxiliary `#auxiliary` plus a related service-descriptor row;
- raw rows using both `#progenitor` and `#package`;
- ALMA-local columns including `local_semantics`, `content_qualifier`,
  `link_auth`, and `link_authorized`.

An invalid `uid___A001_X0_X0` yielded an explicit `NotFoundFault`/`#error`
row. Anonymous DataLink for valid proprietary MOUS
`uid://A001/X3845/X42e` yielded an empty table. Callers must distinguish these
states and should not translate either into “no data exist.”

## Concrete edit map

### `references/archive-query.md`

1. **Lines 5–22 (endpoints):** change “synchronized mirrors” to regional
   complete archive copies; advise staying with one mirror and following
   returned URLs. Keep POST as recommended, mention that TAP sync GET exists,
   and add runtime capability inspection.
2. **Lines 30–35 (row granularity):** replace the representative-frequency
   rule with contextual preservation/de-duplication of every distinct
   `frequency_support` string.
3. **Lines 39–40 (column count):** delete `~63`; use only the schema query.
4. **Lines 49, 51–52, 62–63 (array, coordinates, band):** label suffix/prefix
   classification as heuristic; bound moving-target claims and retain the
   dated current TP-footprint known issue from the cross-domain review; say
   live band strings were numeric but retain a tolerant parser.
5. **Line 59 and lines 78–93 (spectral metadata):** document the native
   sensitivity element and extensibility; remove the universal post-Hanning
   assertion; explain aggregate `velocity_resolution`.
6. **Line 66 (release):** add the year-3000 sentinel; replace DDT six-month
   default with no default proprietary period and an exceptional up-to-six
   month grant. Continue to prohibit local policy recomputation.
7. **Lines 129–130 (ADQL):** state the checked advertised version/features,
   link to capabilities, and warn that IVOA ADQL 2.1 availability must not be
   assumed.
8. Add a DataLink preflight note distinguishing null/kbyte `access_estsize`
   from byte-valued DataLink `content_length`.

Suggested replacement for the most dangerous aggregation sentence:

> `frequency_support` can differ across EBs, fields, and science/calibrator
> rows within one MOUS. Preserve all distinct values with `asdm_uid` and
> target/science context, de-duplicate identical strings, and never sum values
> merely because ObsCore repeats them.

### `references/identifiers-and-packaging.md`

1. **Lines 18–25 (UID grammar):** keep A001/A002 as conventions only. Add the
   Cycle 1 A002 member-OUS counterexample and prohibit prefix-only entity
   classification. Qualify “every path and filename.”
2. **Lines 30–47 (DataLink):** add the empty-valid-proprietary versus explicit
   invalid-ID distinction; include `#progenitor`; state that nested individual
   downloads can lose directory context; preserve all returned semantics and
   local columns.
3. **Lines 49–61 (eras):** rewrite as two axes: current live retrieval
   containers versus historical package/payload content. Make Cycle 0
   calibrated MS current-archive behavior distinct from non-retrievable early
   Cycle 1 PI-delivery calibrated MSs.
4. **Lines 63–75 (nesting):** replace “Every tarball” with a member-list-driven
   rule for top-level delivery archives. Explicitly exclude nested archives
   from the full-tree assumption.
5. **Lines 80–113 (current package):** note optional product, raw-only and
   QA0-semipass cases; classify README as text or archive based on the actual
   link; avoid exact generated raw names.
6. Add the hierarchy caveat: SG → one or more GOUS → one or more MOUS; a MOUS
   normally corresponds to one SB, with commissioning exceptions.

Suggested replacement for the UID classifier:

> A001 and A002 are naming conventions, not globally reliable entity types.
> Identify an EB from the `asdm_uid` role or raw-ASDM package context, and a
> MOUS from `member_ous_uid` or its member path. Early data can use A002 for a
> Member OUS (`uid://A002/X5d7935/X11b` is a live Cycle 1 example).

Suggested replacement for the nesting invariant:

> Top-level ALMA delivery tarballs commonly contain a shared
> project/science-goal/group/member prefix and are intended to be unpacked in
> one controlled parent. Inspect member names before stripping that prefix.
> Nested `auxproducts`, `caltables`, `flagversions`, and weblog archives do not
> generally repeat the full ASA tree, and nested DataLink file downloads may
> not preserve their original directory path.

## Review limitations and follow-up tests

- Live probes used the EU archive. Mirror-specific transient differences are
  possible; portable code should inspect the selected mirror.
- The sample establishes counterexamples, not exhaustive frequencies for all
  cycles. Avoid converting observed absence (for example, no sampled `BAND N`
  string) into a parser rejection rule.
- DataLink raw `#progenitor` versus `#package` likely tracks used versus
  additional/semipass EBs, but no current normative definition tying those
  exact ALMA categories was located. Preserve both until package contents and
  README evidence establish meaning.
- A separate empirical multi-cycle package sampling pass should validate
  actual tar member paths, nested archive placement, and current repackaging.
  That pass should use `tar -tf` before extraction and update these rules from
  observed member lists rather than filenames alone.
