# Source ledger: archive query, identifiers, and packaging

Review/retrieval date: 2026-07-18 UTC.

This ledger records the primary sources used for
`findings-archive.md`. “Current” means operational/policy guidance intended for
the present archive. “Retrospective” means useful for interpreting historical
delivery eras. Static notebook outputs are treated as dated examples rather
than live schema authority.

## Official ALMA manuals and policies

| Source | Version/date | Orientation | URL | Use in review |
|---|---|---|---|---|
| *ALMA Science Archive User Manual* | Doc. 12.15, v1.0, March 2026 | Current operations with retrospective packaging sections | https://almascience.nrao.edu/documents-and-tools/cycle13/science-archive-manual | Regional archives, data hierarchy, project code grammar, proprietary/access description, programmatic services, package tree, Cycle 0 exceptions, frequency-support description. |
| *ALMA QA2 Data Products for Cycle 12* | Doc. 12.12, v1.0, October 2025 | Current Cycle 12 delivery contract | https://almascience.nrao.edu/documents-and-tools/cycle12/alma-qa2-data-products-for-cycle-12 | MOUS/QA2 unit, SB relationship, product/auxiliary/raw containers, five-directory layout, nested archives, Pass/Semipass and raw-only possibilities. |
| *ALMA QA2 Data Products for Cycle 5* | Document version shown as 5.1 / Doc. 5.12 v2.0, May 2018 | Retrospective, broadly relevant to Cycles 4–6 | https://almascience.org/documents-and-tools/cycle5/ALMAQA2Productsv5.1.pdf | Historical tar naming and tree examples, manual/pipeline-era loose files, “essentially always” one SB per MOUS rather than an absolute invariant. |
| *ALMA Cycle 13 Proposer's Guide / ALMA Archive Primer* | Doc. 13.22, v1.0, January 2026 | Current user primer, but internally inconsistent on DDT policy | https://almascience.nrao.edu/documents-and-tools/cycle13/archive-primer | Web-UI result grain, SG/GOUS/MOUS hierarchy, array separation, commissioning exception, current top-level retrieval categories, raw-semipass explanation, DataLink examples, unpacking instructions. Its old six-month DDT prose was rejected in favor of current Users' Policies; a later footnote in the same primer recognizes the new Cycle 12+ rule. |
| *ALMA Users' Policies* | Doc. 13.16, v1.0, March 2026 | Current controlling policy | https://almascience.nrao.edu/documents-and-tools/cycle13/alma-user-policies | Section 9.4.1: regular 12-month proprietary period from delivery; DDT has no period unless an exceptional period up to six months was granted at submission. This controls over conflicting primer prose. |

### PDF acquisition and one-time text extraction

The PDFs were downloaded into the unique temporary directory
`/tmp/alma-archive-review.DH7lgF`. `pdftotext` was unavailable, so one pinned
temporary installation of Mozilla `pdfjs-dist` 5.4.296 was placed under that
same directory and each downloaded PDF was converted to text exactly once.
The text, PDFs, extractor, and temporary package installation were used only
for this review and then removed together after verification.

| Local review name | SHA-256 | Pages reported by PDF.js | Extracted word count |
|---|---:|---:|---:|
| `science-archive-manual-cycle13.pdf` | `54fbef8c3d8056829072385620bd49641bd1290719d19690f5d3fcafd97f5101` | 32 | 9,521 |
| `alma-qa2-data-products-cycle12.pdf` | `08978fb937c202f209d00c729af232d72dda860b6bcefb8dfb7e4a727e860c62` | 30 | 11,513 |
| `alma-qa2-data-products-cycles4-6-v5.1.pdf` | `5e3a813a6ce8b01b8c4fb9aa59d7477989b1d66b2df0c07e92ecd591f8cfcb69` | 20 | 6,494 |
| `archive-primer-cycle13.pdf` | `661067ecbe22902651b599d52beed8c987d049c9c8a4ada03122852bc3a4c704` | 117 | 47,802 |
| `alma-user-policies-cycle13.pdf` | `7df4d4e3de5898874beb42ba192814b578812a6a2d0a566af98375253a241372` | 8 in the retrieved response | partial response |

The Users' Policies download endpoint returned an anomalous eight-page partial
PDF in the command-line download although the official web renderer exposed
the full 29-page document. The proprietary-period finding was therefore
checked against the official full web-rendered section and the current Archive
Manual, not inferred from the partial local file. No PDF was repeatedly
converted to compensate for that endpoint anomaly.

## Official ALMA knowledge-base and archive pages

| Source | Page date | Orientation | URL | Use in review |
|---|---|---|---|---|
| *How are ALMA data products packaged?* | last updated 2023-04-18 | Retrospective packaging history | https://help.almascience.org/kb/articles/how-are-alma-data-products-packaged | Cycles 5+ individual-file ingestion/optional packaging; Cycles 1–4 historical product tar behavior; Cycle 0 raw plus calibrated MS; some early Cycle 1 PI deliveries had similar calibrated MSs but those are not retrievable from the archive. |
| *What calibration and imaging products will be delivered to me?* | last updated 2026-03-18 | Current delivery overview | https://help.almascience.org/kb/articles/what-calibration-and-imaging-products-will-be-delivered-to-me | Five member directories plus README, directory purposes, and possibility that product is absent when no images were selected. |
| *ALMA Archive Documentation* | accessed 2026-07-18 | Current documentation portal and update history | https://almascience.nrao.edu/alma-data/archive/archive-documentation | Located current manuals/notebooks and historical archive update notes, including footprint improvements. |
| *What Cycle 5 proposal issues and clarifications should I be aware of before submitting my proposal?* | historical Cycle 5 article | Historical issue, not present invariant | https://help.almascience.org/kb/articles/what-cycle-5-proposal-issues-and-clarifications-should-i-be-aware-of-before-submitting-my-prop | Evidence that at least one mosaic-footprint problem was historical and fixed; supports downgrading an undated present-tense footprint-defect claim. |
| *How do I deal with targets with unspecified coordinates in the OT?* | last updated 2026-03-19 | Current observing guidance | https://help.almascience.org/kb/articles/how-do-i-deal-with-targets-with-unspecified-coordinates-in-the-ot | Phase-I ToO/moving-target coordinates may be zero placeholders; executed coordinates can differ. Supports careful execution-level verification, not a blanket claim that archive coordinates are wrong. |

## Official ALMA archive notebooks

Notebook portal: https://almascience.nrao.edu/alma-data/archive/archive-notebooks

The portal and rendered notebooks do not state a clear package version/date;
they were retrieved 2026-07-18. Their executable patterns are useful, while
embedded outputs can be stale relative to the live service.

| Notebook | URL | Use and caveat |
|---|---|---|
| *Introduction to ALMA archive notebooks* / notebook 0 | https://almascience.nrao.edu/alma-data/archive/archive-notebooks/nb0_alma_notebook_introduction.html | PyVO endpoint and geometry examples. Its embedded table says 63 columns and shows an old `/aq` HTML `access_url`, whereas the live table had 73 columns and DataLink URLs. This is direct evidence to introspect at runtime. |
| *ALMA Query one source* / notebook 1 | https://almascience.nrao.edu/alma-data/archive/archive-notebooks/nb1_ALMA_Query_one_source.html | PI-entered source names, use of external Sesame resolution, and spatial query examples. |
| *ALMA Query by frequency* / notebook 6 | https://almascience.nrao.edu/alma-data/archive/archive-notebooks/nb6_ALMA_Query_by_frequency.html | Shows `frequency` in GHz, converts live `bandwidth` Hz by `1e9`, and treats `frequency_support` as SPW range/sensitivity/polarization text. |
| *ALMA Download data* / notebook 9 | https://almascience.nrao.edu/alma-data/archive/archive-notebooks/nb9_ALMA_Download_data.html | MOUS identifiers, DataLink enumeration, and sanitized-ID example. Live testing additionally established that both URI and sanitized forms worked. |

## IVOA normative specifications

| Specification | Version/date | URL | Use in review |
|---|---|---|---|
| *Table Access Protocol* | IVOA Recommendation 1.1, 2019-09-27 | https://www.ivoa.net/documents/TAP/20190927/REC-TAP-1.1.html | Sync GET/POST behavior, capability discovery, schema/service contracts. Current version landing page: https://www.ivoa.net/documents/TAP/ |
| *DataLink* | IVOA Recommendation 1.1, 2023-12-15 | https://www.ivoa.net/documents/DataLink/ | Result-row alternatives (`access_url`, `service_def`, or `error_message`), `content_length` bytes, controlled semantics, service descriptors. |
| *Observation Data Model Core Components and its Implementation in the Table Access Protocol* | IVOA Recommendation 1.1, 2017-05-09; errata through 2023-11-10 | https://www.ivoa.net/documents/ObsCore/ | Normative ObsCore meaning and the boundary between standard fields and ALMA extensions. |
| *Astronomical Data Query Language* | IVOA Recommendation 2.1, 2023-12-15 | https://www.ivoa.net/documents/ADQL/20231215/REC-ADQL-2.1.html | Establishes latest normative ADQL. It is **not** evidence that ALMA supports 2.1; the live ALMA capability document advertised 2.0. |

## Primary client documentation

| Source | Version/date | URL | Use in review |
|---|---|---|---|
| *Astroquery ALMA Queries* | Astroquery 0.4.12.dev documentation, accessed 2026-07-18 | https://astroquery.readthedocs.io/en/latest/alma/alma.html | `Alma.query_tap`, query patterns, and `bandwidth` Hz confirmation. Used only as primary client documentation, subordinate to the live service schema and ALMA docs. |

## Live service evidence

All probes were read-only and used the EU archive root
`https://almascience.eso.org` on 2026-07-18.

### TAP resources

- TAP service: `https://almascience.eso.org/tap`
- Sync query resource: `https://almascience.eso.org/tap/sync`
- Capabilities: `https://almascience.eso.org/tap/capabilities`
- Schema table queried: `TAP_SCHEMA.columns`
- Science table queried: `ivoa.obscore`

Observed capability facts (date-stamped, not permanent constants):

- TAP 1.1 and ObsCore 1.1 advertised;
- ADQL 2.0 advertised;
- geometry features included `POINT`, `CIRCLE`, `BOX`, `POLYGON`, `REGION`,
  `CONTAINS`, `INTERSECTS`, `AREA`, `CENTROID`, `COORDSYS`, `COORD1`, and
  `COORD2`;
- hard/default output limit 134,217,728 bytes;
- execution limit 600 seconds;
- upload limit 100,000 rows;
- retention limit 604,800 seconds.

Key live queries and results are reproduced in `findings-archive.md`. Important
sample identifiers were:

- recent public MOUS `uid://A001/X3788/X62cc`, project `2024.1.00148.S`, used
  for row-grain, frequency-support, DataLink, and identifier-form probes;
- public `qa2_passed='F'` MOUS `uid://A001/X3788/Xc582`, used to show that the
  boolean is not a three-state QA2 verdict or availability flag;
- proprietary MOUS `uid://A001/X3845/X42e`, whose anonymous DataLink response
  was empty;
- Cycle 1 member OUS `uid://A002/X5d7935/X11b`, project
  `2012.1.00350.S`, the counterexample to prefix-only UID classification;
- Cycle 4 member OUS `uid://A001/X87d/Xb`, project `2016.1.01604.S`, whose
  current live retrieval is already separated into product, auxiliary, raw,
  and external links.

### DataLink resources and behavior

Primary sample requests:

- `https://almascience.eso.org/datalink/sync?ID=uid://A001/X3788/X62cc`
- `https://almascience.eso.org/datalink/sync?ID=uid___A001_X3788_X62cc`
- invalid test: `https://almascience.eso.org/datalink/sync?ID=uid___A001_X0_X0`

The sample top-level response identified DataLink links 1.1, and returned
direct documentation, product, auxiliary, and raw links as well as service
descriptors. Following the auxiliary descriptor exposed individual nested
files (caltables, auxproducts, calapply, flagversions, scripts, processing
request/manifest, weblog, QA PDFs, and log material) without enough original
archive path information to reconstruct the tree solely from the link URL.

Live filenames and semantics are observations, not a frozen taxonomy. The
review therefore recommends retaining the raw VOTable metadata and URL,
classifying conservatively, and avoiding filename synthesis.

## Source conflicts resolved

1. **DDT proprietary period:** current Users' Policies controls over stale
   Primer prose. Default is none; exceptional granted period may be up to six
   months.
2. **Column count and access URL:** live TAP schema/rows control over static
   notebook outputs. Count was 73 and current `access_url` was DataLink.
3. **GOUS cardinality:** current ALMA documents conflict. The skill should use
   the non-lossy SG → one-or-more GOUS → one-or-more MOUS model rather than
   assert a one-to-one collapse.
4. **Historical packaging versus current download:** historical manuals/KB
   describe original ingestion/delivery; live DataLink describes what can be
   downloaded now. Record both axes explicitly.
