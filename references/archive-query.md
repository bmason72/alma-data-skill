# Querying the ALMA Science Archive (TAP / ObsCore)

## Endpoints

Three synchronized mirrors — pick by geography:

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
  not a direct file link — follow it (or call DataLink yourself) to enumerate
  actual files.

## Row granularity — the #1 query mistake

`ivoa.obscore` rows are coverage records finer than a dataset: repeated
across EBs, sources/fields, and spectral coverage (do not assume a fixed
Cartesian grain). Consequences:

- MOUS-level work: aggregate by `member_ous_uid`.
- Count executions as distinct `asdm_uid` within the MOUS.
- `frequency_support` repeats the spectral setup on many rows: parse ONE
  representative row per MOUS, never sum over grouped rows.
- Per-MOUS values (e.g. release date) can differ across rows; take min/max
  deliberately.

## High-value columns

Column inventory evolves (currently ~63 columns). Introspect rather than
memorize: query `TAP_SCHEMA.columns WHERE table_name='ivoa.obscore'`.
The load-bearing ones and their traps:

| Column | Type/unit | Trap |
|---|---|---|
| `proposal_id` | string | project code, e.g. `2021.1.00123.S` |
| `member_ous_uid` | `uid://A001/...` | THE grouping key |
| `group_ous_uid` | `uid://A001/...` | parent GOUS. **No `science_goal_uid` column exists** — the SG UID appears only in delivery paths; recover it from the package tree. |
| `asdm_uid` | `uid://A002/...` | the EB |
| `schedblock_name` | string | suffix encodes array: `_TM1`/`_TM2` 12-m, `_7M`, `_TP` |
| `target_name` | string | proposer-entered free text; not resolver-normalized; join on coordinates, never on name |
| `s_ra`, `s_dec` | deg ICRS | representative position only; inadequate for mosaics; may be wrong for ephemeris/ToO targets |
| `s_region` | STC-S footprint | the correct column for spatial intersection; TP footprints have a known defect (may cover one pointing, not the mapped area) |
| `t_min`, `t_max` | **MJD (days, float)** | vs `obs_release_date` which is an **ISO timestamp string** |
| `t_exptime` | seconds | |
| `em_min`, `em_max` | **wavelength, METERS** | to filter by frequency ν[GHz]: λ = 0.299792458/ν m |
| `frequency` | **GHz** | representative frequency |
| `bandwidth` | **Hz** | nasty: some clients display a GHz unit label; official notebooks divide by 1e9 |
| `frequency_support` | string | see grammar below |
| `velocity_resolution` | m/s | includes Hanning smoothing; ≠ naive channel-width conversion |
| `spatial_resolution`, `s_resolution` | arcsec | estimate of synthesized beam |
| `spatial_scale_max` | arcsec | Maximum Recoverable Scale — flux on scales approaching/exceeding this is progressively under-recovered (not a step cutoff) |
| `antenna_arrays` | string | entries are `station:antenna` pairs, e.g. `A004:DV07 A025:CM03`; classify by antenna prefix: `DV`/`DA` 12-m, `CM` 7-m, `PM` TP. No dedicated "array" column exists. |
| `band_list` | string | formats vary: `6`, `BAND 6`, multi-band `5 10` (band-to-band). Parse tolerantly; resolve science tuning from `frequency_support`. |
| `qa2_passed` | `T`/`F` | boolean convenience; does NOT encode PASS/SEMIPASS/FAIL |
| `data_rights` | `Public`/`Proprietary` | filter `= 'Public'` for anonymous access |
| `obs_release_date` | ISO string | trust it; do not recompute proprietary periods (12-month default, but DDT ~6 months, extensions and early-QA0-access policies exist) |
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

U-joined bracketed segments, one per SPW; versioned human-readable text, not
a clean array:

```
[86.21..88.09GHz,976.56kHz,1.2mJy/beam@10km/s,XX YY] U [98.10..99.98GHz,...]
```

Per segment: frequency range (`low..high` + unit), **effective spectral
resolution** (+ unit — this is the post-Hanning resolution, NOT the raw
channel spacing; do not apply another Hanning factor to it), per-SPW
sensitivity, polarization products. Parse with a tolerant regex like
`([\d.]+)\s*\.\.\s*([\d.]+)\s*(GHz|MHz|kHz|Hz)`. nchan ≈
round(range/resolution) only approximates the channel count (can differ by
~2× from the raw channel number).

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

Standard ADQL 2.x; geometry support is limited to the common
CONTAINS/INTERSECTS forms. Escape single quotes in literals.
