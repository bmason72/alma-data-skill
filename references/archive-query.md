# Querying the ALMA Science Archive (TAP / ObsCore)

Reviewed 2026-07-18 against the Cycle 13 Archive Manual and Users' Policies,
the live EU TAP/DataLink services, official archive notebooks, and IVOA TAP,
ObsCore, DataLink, and ADQL recommendations. Live schema and capability facts
below are date-stamped because the service evolves.

## Contents

- Endpoints
- Row granularity
- High-value columns
- `frequency_support` grammar
- Download preflight
- Derived quantities
- ADQL patterns

## Endpoints

Three regional complete archive copies — pick by geography and stay on that
copy for a request (updates and request state need not be simultaneous):

- `https://almascience.eso.org` (EU) · `https://almascience.nrao.edu` (NA) ·
  `https://almascience.nao.ac.jp` (EA)

Services on each mirror:

- **TAP**: `<mirror>/tap`, table `ivoa.obscore`. Use
  `pyvo.dal.TAPService("<mirror>/tap")` or `astroquery.alma`
  (`Alma.query_tap(...)`); the raw sync endpoint is a POST to
  `<mirror>/tap/sync` with `REQUEST=doQuery&LANG=ADQL&FORMAT=csv&QUERY=...`.
- **DataLink** (enumerate a MOUS's downloadable files):
  `<mirror>/datalink/sync?ID=<uid>` — accepts `uid://A001/...` or sanitized
  `uid___A001_...`.
- **Direct file download**: `<mirror>/dataPortal/<filename>`.
- ObsCore `access_url` currently returns the observation's **DataLink URL**,
  not a direct file link — inspect `access_format` and follow the URL the
  service returned (or call DataLink yourself) to enumerate actual files.
  Do not rewrite a returned URL onto another mirror.
- **Capabilities**: `<mirror>/tap/capabilities`. Inspect this before depending
  on a language feature or launching a broad query; advertised language,
  output, execution, upload, and retention limits are operational settings.

## Row granularity — the #1 query mistake

`ivoa.obscore` rows are coverage records finer than a dataset: repeated
across EBs, sources/fields, and spectral coverage (do not assume a fixed
Cartesian grain). Consequences:

- MOUS-level work: aggregate by `member_ous_uid`.
- Count executions as distinct `asdm_uid` within the MOUS.
- `frequency_support` can differ across EBs, fields, and science/calibrator
  rows within one MOUS. Preserve every distinct value with at least
  `asdm_uid`, target/field, and science context when available; de-duplicate
  identical strings, and never sum values merely because rows repeat them.
- Per-MOUS values (e.g. release date) can differ across rows; take min/max
  deliberately.

## High-value columns

Column inventory evolves. Introspect rather than memorize: query
`TAP_SCHEMA.columns WHERE table_name='ivoa.obscore'` (the EU service exposed
73 columns on 2026-07-18, while older notebook output showed 63).
The load-bearing ones and their traps:

| Column | Type/unit | Trap |
|---|---|---|
| `proposal_id` | string | project code, e.g. `2021.1.00123.S` |
| `member_ous_uid` | UID | THE grouping key; normally A001 in modern data, but live early-cycle A002 counterexamples exist |
| `group_ous_uid` | UID | parent GOUS. **No `science_goal_uid` column exists in the reviewed schema** — the SG UID appears in delivery paths; recover it from the package tree. |
| `asdm_uid` | UID | the EB identifier. Use this column role, not the A002 prefix alone: at least one live early-cycle Member OUS also has an A002 UID. |
| `schedblock_name` | string | `_TM1`/`_TM2`, `_7M`, and `_TP` are useful array heuristics, not a normative complete classifier. |
| `target_name` | string | proposer-entered free text, not resolver-normalized. Match fixed targets primarily by position; moving/ToO cases require name/time plus executed directions. |
| `s_ra`, `s_dec` | deg ICRS | representative position only; inadequate for mosaics. Moving/ToO catalog positions can be placeholders or differ from executed directions, so verify execution metadata. |
| `s_region` | STC-S footprint | the correct column for spatial intersection. Current TP known issue (checked 2026-07-18): it can show one antenna pointing rather than the full TP map; verify execution pointings and recheck the live issue page. |
| `t_min`, `t_max` | **MJD (days, float)** | vs `obs_release_date` which is an **ISO timestamp string** |
| `t_exptime` | seconds | |
| `em_min`, `em_max` | **wavelength, METERS** | to filter by frequency ν[GHz]: λ = 0.299792458/ν m |
| `frequency` | **GHz** | representative frequency |
| `bandwidth` | **Hz** | nasty: some clients display a GHz unit label; official notebooks divide by 1e9 |
| `frequency_support` | string | see grammar below |
| `velocity_resolution` | m/s | aggregate archive estimate derived across SPWs, not a raw per-SPW channel width |
| `spatial_resolution`, `s_resolution` | arcsec | estimate of synthesized beam |
| `spatial_scale_max` | arcsec | Maximum Recoverable Scale — flux on scales approaching/exceeding this is progressively under-recovered (not a step cutoff) |
| `antenna_arrays` | string | entries are `station:antenna` pairs, e.g. `A004:DV07 A025:CM03`; `DV`/`DA`, `CM`, and `PM` are useful 12-m/7-m/TP heuristics, not an identity contract. No dedicated "array" column exists. |
| `band_list` | string | live values checked in 2026 were numeric (`6`, multi-band `5 10`); historical/display forms can say `BAND 6`. Parse tolerantly and preserve `frequency_support` context. |
| `qa2_passed` | `T`/`F` | boolean convenience; does NOT encode PASS/SEMIPASS/FAIL |
| `data_rights` | `Public`/`Proprietary` | filter `= 'Public'` for anonymous access |
| `obs_release_date` | ISO string | do not recompute access rights. Current policy: regular PASS/SEMIPASS data normally have 12 months from delivery; DDT has no period by default, with at most six months only when exceptionally requested and granted. `3000-01-01...` occurs as a proprietary placeholder, not a literal promise. Check `data_rights` and authorization. |
| `access_estsize` | kbyte, nullable | coarse ObsCore estimate, not a file-download budget; DataLink `content_length` is per-link bytes |
| `science_observation` | `T`/`F` | `F` = calibration-intent rows |
| `scan_intent` | string | `TARGET`, `BANDPASS`, `PHASE`, ... (multi-valued) |
| `is_mosaic` | `T`/`F` | |
| `dataproduct_type` | `cube`/`image` | |
| `calib_level` | 2 or 3 | |
| `pwv` | mm | conditions during observation |
| `sensitivity_10kms` | mJy/beam | line sensitivity per 10 km/s |
| `cont_sensitivity_bandwidth` | mJy/beam | continuum sensitivity over aggregated bandwidth |
| `scientific_category`, `science_keyword` | string | proposal-derived |
| `proposal_abstract`, `proposal_authors`, `pub_title`, `bib_reference`, `first_author` | string | literature linkage; publication joins duplicate rows |

## `frequency_support` grammar

U-joined bracketed segments, normally one per SPW; versioned human-readable
text, not a clean array:

```
[216.90..218.88GHz,31250.00kHz,76.5mJy/beam@10km/s,4.6mJy/beam@native,XX YY] U [...]
```

Fields can include a frequency range (`low..high` + unit), an archive spectral
resolution estimate, one or more sensitivity estimates (`@10km/s`,
`@native`), polarization products, and version-dependent extras. Do not split
on commas into a fixed arity. A tolerant minimum range extractor is
`([\d.]+)\s*\.\.\s*([\d.]+)\s*(GHz|MHz|kHz|Hz)`. Use the archived resolution
as an estimate; do not infer raw channel count or apply a universal Hanning
factor without the ASDM/MS correlator metadata. `velocity_resolution` is also
an aggregate estimate and must not be paired with one SPW range as though it
were that SPW's native channelization.

## Download preflight and DataLink states

- Plan bytes from DataLink `content_length` after enumerating links, not from
  ObsCore `access_estsize` (kbyte, nullable). Allow missing lengths and verify
  the HTTP result.
- A valid proprietary MOUS can return an empty anonymous DataLink table; an
  invalid UID can instead return an explicit `#error`/`NotFoundFault`. Empty
  means “no links visible under this authorization,” not “the MOUS does not
  exist.”
- DataLink semantics are not a full ALMA taxonomy: raw rows have used both
  `#progenitor` and `#package`. Retain the returned semantics/local fields and
  classify conservatively from filename, content type, URL, and context.

## Derived quantities — label them honestly

- `B[m] ≈ 61837 / (spatial_resolution[arcsec] × frequency[GHz])` gives a
  **resolution-equivalent baseline proxy** (θ ≈ λ/B). Useful for binning by
  configuration; it is NOT the physical maximum baseline (weighting, uv
  coverage, and flagging move it by tens of percent). True baseline extrema
  live in the MS / weblog / listobs.

## ADQL patterns

```sql
-- public MOUSs released in a window (group client-side by member_ous_uid)
SELECT DISTINCT member_ous_uid, proposal_id, band_list, obs_release_date
FROM ivoa.obscore
WHERE data_rights = 'Public'
  AND member_ous_uid IS NOT NULL
  AND obs_release_date >= '2024-01-01' AND obs_release_date < '2024-03-01'

-- point-in-footprint test (not s_ra/s_dec!)
SELECT * FROM ivoa.obscore
WHERE 1 = CONTAINS(POINT('ICRS', 52.26, 31.28), s_region)

-- true cone search over footprints: use INTERSECTS with a CIRCLE —
-- the point test above misses observations that overlap the cone
-- without covering its center
WHERE 1 = INTERSECTS(CIRCLE('ICRS', 52.26, 31.28, 0.1), s_region)

-- observation-date window: MJD, not ISO
WHERE t_min >= 60310 AND t_min < 60401

-- frequency overlap via wavelength columns (ν1 < ν2 in GHz)
WHERE em_min <= 0.299792458/[ν1] AND em_max >= 0.299792458/[ν2]
```

Escape single quotes in literals. As checked 2026-07-18 the EU service
advertised ADQL 2.0, not 2.1, plus `POINT`, `CIRCLE`, `BOX`, `POLYGON`,
`REGION`, `CONTAINS`, `INTERSECTS`, `AREA`, `CENTROID`, `COORDSYS`, `COORD1`,
and `COORD2`. Inspect the selected mirror's `/tap/capabilities`; a newer IVOA
standard is not proof that the deployed service implements it.
