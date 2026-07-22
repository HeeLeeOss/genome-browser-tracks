# Competitive + Improvement Analysis — `genome-browser-tracks`

> Scope: curated OPEN cancer feature tracks/configs for genome browsers (UCSC track hubs / IGV / JBrowse 2),
> built from open annotations only, with per-source license verification, provenance, and genome-build correctness.
> Reviewed against PLAN.md v0.1.0 (2026-06-28). Web-researched June 2026.

---

## 1. Correctness & completeness review of PLAN.md

The plan is unusually strong on the two failure classes that actually sink genome-track projects —
**licensing** and **coordinate/assembly correctness** — and it correctly splits them into two separate
blocking gates (Appendix A#1). The following review confirms what is right and flags gaps.

**Genome-build / coordinate correctness (the dominant silent-failure risk).**
- The plan correctly identifies wrong-assembly and off-by-one as the classic silent failures. This is real:
  hg38 contains assembled sequence absent from hg19, and regions where hg19↔hg38 *both* gap are ~3.5% of the
  genome; liftOver chains can map a region to the **wrong** location when a contig was replaced between builds
  ([UCSC genome list / liftOver issues](https://groups.google.com/a/soe.ucsc.edu/g/genome/c/eTQugD99iKE),
  [Benchmark of liftover tools, NARGAB](https://academic.oup.com/nargab/article/2/3/lqaa054/5881791)).
- **Correct and well-specified:** recording assembly + coordinate convention, requiring a liftOver
  mapped/unmapped report, and a render check. Good.
- **Gap 1 (convention precision):** the plan says "0-based half-open vs 1-based" but does not pin the
  per-format truth table. BED/bigBed are **0-based half-open**; GFF3 and VCF are **1-based, closed**; UCSC
  position search boxes are 1-based; the UCSC API/`bedToBigBed` expect 0-based. The golden fixtures (§8) should
  encode this *per format*, not generically, because the off-by-one almost always happens at a BED↔GFF3↔VCF
  boundary, not within one format.
- **Gap 2 (chrom naming):** `liftOver` and UCSC tooling require `chr1`-style names; Ensembl/RefSeq emit `1`
  / `NC_000001.11`. The plan never makes **chrom-name normalization** an explicit gate step, yet a `1` vs
  `chr1` mismatch silently drops every feature in UCSC. Add it to the coordinate gate
  ([Griffith Lab liftOver](https://genviz.org/module-01-intro/0001/06/02/liftoverTools/)).
- **Gap 3 (liftOver is not idempotent / not the only option):** the plan defaults to UCSC `liftOver`. For some
  features (especially variant/indel coordinates) NCBI Remap-style or assembly-native re-annotation is safer
  than chain liftOver. Recommend: **prefer native-assembly source files over liftOver** whenever the source
  publishes both builds (GENCODE, Ensembl, ClinVar all do), and only liftOver single-assembly sources — the
  plan implies this but should state it as a rule.

**Track-format standards (BED/bigBed/bigWig/GFF3/VCF).**
- Format choices are correct and idiomatic: region features → BED/bigBed/GFF3, quantitative → bigWig, variants
  → VCF+tabix or bigBed. Wrapping UCSC userApps (`bedToBigBed`, `bedSort`, `fetchChromSizes`, `hubCheck`,
  `validateFiles`) and htslib (`bgzip`/`tabix`) rather than reimplementing is the right call and matches what
  UCSC itself documents for hubs ([trackDb v3 spec](https://genome.ucsc.edu/goldenpath/help/trackDb/trackDbHub.html)).
- **Gap 4 (bigBed AutoSql / extra fields):** per-feature provenance (source, record ID, license) is the plan's
  signature feature, but standard BED has no place for it. To carry provenance *in the track* you need a
  **bigBed with a custom AutoSql `.as` schema** (extra fields), or `mouseOver`/`url` trackDb settings. The plan
  asserts "embedding per-feature provenance fields" without naming the mechanism (AutoSql). This is the single
  biggest under-specified technical detail; it should be pinned in M0.
- **Gap 5 (bigBed/bigWig require `chrom.sizes` for the exact assembly):** `fetchChromSizes` is listed, good —
  but the gate should assert the chrom.sizes used matches the *declared* assembly (a subtle way to ship a
  "valid" bigBed pointing at the wrong build).

**Source annotation licenses (the second binding gate).** The plan's accept/exclude matrix is **accurate** as
verified June 2026:
- **Open / redistributable (correctly Accept):** ClinVar = CC0; CIViC curated content = CC0; Open Targets =
  now explicitly **CC0** ([Open Targets licence](https://platform-docs.opentargets.org/licence),
  [CC0 announcement](https://community.opentargets.org/t/open-targets-products-are-now-marked-with-a-cc0-licence/1082));
  Ensembl data "available without restriction," code Apache-2.0 ([Ensembl 2017](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5210575/));
  GENCODE open ([GENCODE data access](https://www.gencodegenes.org/pages/data_access.html)); RefSeq/dbSNP US-gov public domain; Reactome CC-BY-4.0.
- **Correctly Excluded as non-commercial / non-redistributable:** COSMIC + Cancer Gene Census — the COSMIC
  non-commercial terms explicitly forbid redistribution to third parties beyond publication enablement
  ([COSMIC NC license PDF](https://cancer.sanger.ac.uk/cancergenome/assets/COSMIC_non-commercial_license_May2021.pdf),
  [Griffith Lab COSMIC](https://genviz.org/module-01-intro/0001/08/01/cosmic/)); OncoKB requires a license for
  most uses ([OncoKB licensing FAQ](https://faq.oncokb.org/licensing)). The plan's exclusion is correct and
  important — this is exactly the "laundering" risk it guards against.
- **Gap 6 (CC-BY attribution is structural, not cosmetic):** Reactome CC-BY-4.0 attribution must survive into
  the *derived track file's* metadata, not just the pack README — the plan says this (Appendix A#5) but the
  AutoSql/trackDb mechanism (Gap 4) is what makes it enforceable. Also: a **CC-BY source mixed into a CC0/PD
  pack forces the whole pack to CC-BY** with attribution; the plan licenses derived output CC-BY-4.0 uniformly,
  which is the safe choice — confirm this is intentional (it sacrifices CC0 purity for simplicity; fine).
- **Gap 7 (gnomAD nuance):** gnomAD is free but historically had attribution/"don't republish as your own"
  norms and dataset-version license drift; the plan marks it Accept — keep it as **Verify (B)** per release, not
  blanket Accept, to be safe.
- **Gap 8 (Priority-B per-study reality):** cBioPortal/GDC-open/DepMap/MSK cancer-hotspots redistribution rights
  genuinely vary per study/file. The plan handles this correctly (gate per file). Worth noting DepMap is
  CC-BY/partly restricted and MSK hotspots terms are non-trivial — keep them low-priority.

**Provenance, versioning, hosting, validation.**
- Provenance model (source, release, record ID, license id+URL+SHA-256 snapshot+Wayback, retrieval timestamp,
  assembly, convention) and the 0–100 completeness score are excellent and exceed common practice. **Gap 9:**
  add the *exact source file URL + its checksum*, not just the portal URL — annotation portals silently
  re-release files under the same version string.
- Versioning to source release with explicit refresh tasks is correct (annotations like ClinVar update weekly,
  GENCODE ~2×/yr). **Gap 10:** the plan lacks a **stable-vs-rolling** distinction — ClinVar is a moving target;
  a pack should pin to a dated ClinVar release, not "latest," or the render check passes today and the science
  drifts tomorrow.
- Hosting via GitHub Pages / Zenodo with loadable-by-URL hubs is exactly how recount3, and most academic hubs,
  do it ([recount3 create_hub](http://research.libd.org/recount3/reference/create_hub.html)). Sound.
- Validation chain (`validateFiles` + `hubCheck` + JBrowse lint + tabix check + render check) is the right and
  complete set ([hubCheck blog](https://genome-blog.soe.ucsc.edu/blog/2017/03/17/how-portable-is-your-track-hub-use-hubcheck-to-find-out/)).
- **Completeness gaps not covered by the plan:** (a) **no CORS/byte-range hosting requirement** — bigBed/bigWig/
  tabix all rely on HTTP range requests; GitHub Pages supports this but Zenodo's download endpoints historically
  did **not** reliably serve range requests, which would break JBrowse/IGV remote loading. This must be tested in
  M0, not assumed. (b) **No T2T-CHM13** (acknowledged as deferred — fine). (c) **No mention of trackDb
  `compositeTrack`/`superTrack`** grouping, which multi-source packs will need for usability.

**Verdict:** The plan is correct and notably rigorous on its two named gates. The most material *missing*
specifics are (1) the AutoSql/bigBed mechanism for carrying per-feature provenance (Gap 4) and (2) chrom-name
normalization + per-format coordinate truth-table as explicit gate steps (Gaps 1–2), plus the CORS/byte-range
hosting test (completeness gap a). None are conceptual flaws; all are "pin this detail in M0."

---

## 2. Competitive landscape

| Player | What it is | Strengths | Weaknesses (our opening) |
| --- | --- | --- | --- |
| **UCSC Genome Browser + Public Hub list / track hubs** | The canonical browser + hub mechanism + curated public-hub list ([hubs](https://genome.ucsc.edu/goldenpath/help/hgTrackHubHelp.html), [public hub guidelines](https://genome.ucsc.edu/goldenPath/help/publicHubGuidelines.html)) | De-facto standard; huge native annotation set; `hubCheck`/`validateFiles`; trusted | Hubs are *infrastructure*, not curated **cancer** content; making a license-clean cancer hub is left to the user; provenance not standardized |
| **Ensembl Track Hub Registry (THR, EMBL-EBI)** | Central registry of public hubs ([THR](https://github.com/Ensembl/trackhub-registry), [thr](https://github.com/Ensembl/thr)) | Discovery/distribution channel; cross-browser | A *registry*, not a content producer; no license/provenance curation; we can publish **into** it |
| **JBrowse 2** | Modular embeddable browser; `config.json` ([config guide](https://jbrowse.org/jb2/docs/config_guide/), [setup, PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC11412189/)) | Modern, embeddable, plugin ecosystem, same file formats (bigBed/bigWig/tabix) | Ships a *viewer*, not curated cancer tracks; config authoring is manual — exactly our projection target |
| **IGV / IGV-Web / igv.js** | Desktop + web viewer, shareable sessions ([IGV-Web](https://igvteam.github.io/igv-webapp/), [igv.js paper](https://academic.oup.com/bioinformatics/article/39/1/btac830/6958554)) | Ubiquitous in cancer labs; session URLs; native "mutation" track type | Viewer only; user supplies tracks; no curation/licensing/provenance |
| **UCSC Xena / (legacy) Cancer Genomics Browser** | Cancer-specific multi-omic visual portal over TCGA/PCAWG/ICGC/GTEx/GDC, 1500+ datasets ([Xena bioRxiv](https://www.biorxiv.org/content/10.1101/326470v6), [CGB 2015](https://pmc.ncbi.nlm.nih.gov/articles/PMC4383911/)) | The strongest cancer-specific incumbent; deep sample-level matrices; private-hub model | **Matrix/heatmap paradigm, not genome-coordinate tracks**; not designed to drop a license-clean *feature track* into the user's own UCSC/IGV/JBrowse; provenance/redistribution not packaged for reuse |
| **recount2/3 track hubs** | Auto-generated UCSC hubs of RNA-seq bigWig coverage ([recount3](https://rna.recount.bio/), [create_hub](http://research.libd.org/recount3/reference/create_hub.html)) | Proves the "hub-as-data, hosted-by-URL, generated-from-manifest" pattern at scale | Coverage signal, not curated cancer *features*; single source; single browser (UCSC) |
| **UCSC Genome Browser MCP server (hlydecker) / MCPmed** | LLM access to UCSC API/hubs ([repo](https://github.com/hlydecker/ucsc-genome-mcp), [MCPmed, PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC12927880/)) | Early mover on LLM↔genome-browser; 12 UCSC tools | Read-only API wrapper; doesn't *produce* curated, license-clean cancer tracks — complementary to us |

**Net:** No incumbent occupies our exact slot. Browsers (UCSC/IGV/JBrowse) and registries (THR) are
**infrastructure**; Xena is the cancer-specific incumbent but uses a **matrix paradigm**, not portable
coordinate tracks; recount proves the *delivery pattern* but for one non-cancer-feature data type and one
browser. The whitespace is **curated, license-verified, provenance-complete cancer FEATURE track packs that load
into all three browsers from one manifest.**

---

## 3. Gaps we can fill

1. **License-clean, cancer-specific feature packs.** No one ships a vetted "canonical cancer driver genes on
   GRCh38, CC0/CC-BY, with provenance" track that a user can trust to redistribute. Xena has the data but not as
   portable, license-tagged tracks; UCSC has the plumbing but not the curation.
2. **Provenance-in-the-track.** Per-feature source/record-ID/license carried *inside* the bigBed (via AutoSql)
   is essentially absent from existing public hubs — turning a track into an auditable artifact.
3. **One-manifest → three browsers (UCSC + JBrowse2 + IGV) parity.** Today users hand-author each config;
   nobody publishes the *same* curated cancer pack across all three from a single source of truth.
4. **An explicit, auditable license/data-tier gate** that visibly *excludes* COSMIC/CGC/OncoKB and explains why —
   a trust signal no ad-hoc hub provides.
5. **Coordinate/assembly correctness as a published, validated guarantee** (declared build, convention truth
   table, liftOver mapped/unmapped report, render evidence) — the thing most ad-hoc hubs silently get wrong.
6. **Education-only "where are the cancer genes" packs** with explicit "not medical advice" framing — a niche
   between dense research portals and unsourced patient web content.

---

## 4. Differentiators to win

1. **Trust-as-a-feature: the dual gate (license/data-tier + coordinate/assembly) made auditable.** Every pack
   ships its gate artifact, provenance score, and render evidence. This is the moat — not the files, the *proof*.
2. **Per-feature provenance baked into the binary track** (AutoSql bigBed), so the artifact is self-describing
   and citable (Zenodo DOI) — uniquely reusable and audit-ready.
3. **Single canonical manifest → UCSC + JBrowse2 + IGV.** Multi-browser parity from one source of truth; nobody
   else does curated cancer content this way.
4. **Strictly open, explicitly-excludes-NC posture.** "We *cannot* redistribute COSMIC/CGC/OncoKB and here's the
   clause" is a credibility differentiator vs. license-murky community hubs.
5. **Loadable-by-URL + registry-listed** (THR / UCSC public hub) so adoption is one paste, lowering the
   cold-start barrier the plan rightly worries about.
6. **Reproducible from manifest** — anyone can rebuild a pack from its committed manifest + pinned CLI versions,
   which research-software reviewers and educators value.

---

## 5. Claude API leverage (and hard limits)

**Where Claude API adds real leverage (mechanical/text transformation around a verified core):**
1. **Generate/validate browser configs from the manifest** — draft UCSC `trackDb.txt` (with AutoSql `.as`),
   JBrowse2 `config.json`, IGV session JSON as *projections*, then check them against the spec; flag
   non-conforming/deprecated trackDb settings (mirrors what `hubCheck` enforces). High value, low risk.
2. **Author provenance/attribution + "not medical advice" prose and pack documentation** — turn structured
   manifest fields into READMEs, attribution strings, and intended-use statements consistently across packs.
3. **License triage assistant (proposes, never decides)** — read a source's terms page and *draft* a gate
   artifact (data tier, redistribution clause citation, NC flag) for a **human reviewer** to confirm.
4. **Harmonization helper** — propose chrom-name normalization (`1`→`chr1`), per-format coordinate-convention
   mappings, and field crosswalks between source schemas — emitted as *deterministic transforms to be validated
   by tooling*, not free-hand edits.
5. **Coordinate-sanity narration** — explain a liftOver unmapped report or an assembly-mismatch diff in plain
   language to speed bioinformatics review.

**Where Claude must NOT decide (hard guardrails — match PLAN §5/§8):**
- **Coordinate/biology correctness is verified by tools and authoritative sources, never by the LLM.**
  `liftOver`/`validateFiles`/`bedToBigBed`/`tabix` + the render check are ground truth; an LLM-asserted
  coordinate is a defect, not a feature. No locus, breakpoint, or gene model is accepted on the model's say-so.
- **No fabricated or inferred annotations.** Every feature must trace to a real open-source record ID; the model
  may format but never invent oncogene/TSG loci, fusion breakpoints, or regulatory regions.
- **License calls are human-verified.** Claude may draft the gate; a named license/provenance reviewer makes the
  redistributable/NC determination. "Permits redistribution" requires a cited clause, not model confidence.
- **No clinical/actionability framing.** The model must not phrase any track as clinically actionable for an
  individual; patient-facing layers require oncologist + advocate sign-off (riskTier high).
- **Genome build is asserted by source + tooling, not guessed** — the model must never "assume hg38."

---

## 6. Ten concrete optimizations

1. **Pin the per-format coordinate truth table + AutoSql provenance schema in M0.** Make BED/bigBed=0-based-half-open,
   GFF3/VCF=1-based-closed explicit in code and golden fixtures; define the bigBed `.as` extra-field schema
   (source, recordId, license, sourceRelease) that carries per-feature provenance. (Closes Gaps 1, 4.)
2. **Add chrom-name normalization as an explicit, tested gate step** (`1`/`NC_*`↔`chr1`), with a fixture that
   fails on mismatch. (Closes Gap 2 — a top silent-failure.)
3. **Prefer native-assembly source files over liftOver.** Rule: liftOver only single-assembly sources; for
   GENCODE/Ensembl/ClinVar (publish both builds) use the native file. Record the decision in the manifest.
4. **Test CORS / HTTP byte-range serving in M0** for the chosen host (GitHub Pages vs Zenodo) before committing —
   bigBed/bigWig/tabix remote loading depends on range requests; Zenodo historically was unreliable here.
5. **Pin sources to dated releases, never "latest."** Especially ClinVar (weekly). Store source file URL +
   checksum, not just portal URL. (Closes Gaps 9, 10.)
6. **Make gnomAD and all Priority-B sources Verify-per-release, not blanket Accept;** capture the cited clause in
   the gate artifact each time. (Closes Gap 7.)
7. **Ship a public "excluded sources + why" page** (COSMIC/CGC/OncoKB clause citations) as a trust artifact and
   SEO/credibility asset — turns a constraint into a differentiator.
8. **Use trackDb `compositeTrack`/`superTrack` grouping** for multi-source packs so driver-genes + variants +
   pathways render as a usable, collapsible set, not a flat pile.
9. **Auto-register validated packs into the Ensembl Track Hub Registry + apply to the UCSC public-hub list**
   (human-submitted) to manufacture the externally-verifiable adoption events the plan's metrics require.
10. **Add a one-command "verify-pack" CI target** chaining `validateFiles`→`hubCheck`→tabix check→JBrowse lint→
    headless render screenshot, emitting the `outcomes/<pack-id>.json` evidence automatically. Makes
    "verified-loadable" cheap and repeatable.

---

## 7. Parallel & perpendicular spin-offs

- **`ewsr1-fli1-knowledge-graph` ↔ tracks:** project the graph's EWSR1-FLI1 loci, GGAA-microsatellite enhancer
  regions, and known fusion breakpoints into a curated **Ewing-sarcoma track pack** (hg38+hg19) — the graph
  supplies the *what*, this project supplies the *where-on-the-genome* and the license/coordinate guarantee.
- **`ewing-expression-reanalysis` ↔ tracks:** that project produces expression/coverage signal; emit it as
  **bigWig coverage tracks** via the exact recount3 `create_hub` pattern, co-loadable with the Ewing feature
  pack for in-context interpretation. Natural producer→viewer pairing.
- **`oncogene-knowledge-graph` ↔ tracks:** the KG's oncogene/TSG entities and pathway memberships become the
  canonical "cancer driver genes on GRCh38" educational pack and the Reactome pathway-membership track (CC-BY
  attribution preserved). The KG is the curation backbone; tracks are its spatial projection.
- **Reusable "track-hub-as-data" framework (horizontal spin-off):** the manifest→3-browser projector + dual gate
  is **domain-agnostic**. Generalize it into a Hee-Lee Oss building block any open-data deed can use to publish
  license-clean, provenance-tracked, render-verified hubs (rare disease, microbiology, plant genomes). recount3
  proves the demand for generated hubs; we add curation + multi-browser + provenance.
- **MCP server serving tracks (perpendicular):** a Hee-Lee Oss **"cancer-tracks" MCP server** that lets an agent
  query "show me the driver-gene track for KRAS on hg38" and returns the pack URL + provenance + a JBrowse/IGV
  deep-link. Complements (does not duplicate) the existing read-only UCSC MCP server and aligns with the MCPmed
  call for MCP-enabled bioinformatics services. Reuses the manifest as the MCP resource model.

---

## 8. Open questions

1. **AutoSql vs trackDb-only provenance:** carry per-feature provenance as bigBed extra fields (richer, but
   bigBed-specific) or as `mouseOver`/`url` trackDb settings (portable, shallower)? Affects every pack — decide
   in M0. (Plan is silent on the mechanism.)
2. **Hosting byte-range reality:** does the chosen host (Pages vs Zenodo) reliably serve HTTP range requests for
   bigBed/bigWig/tabix? Citable DOI (Zenodo) vs live-hub reliability (Pages) may force "both," with Zenodo for
   archival and Pages for the live URL.
3. **Rolling sources (ClinVar/CIViC):** pin to dated snapshots (reproducible, stale faster) or track latest
   (fresh, render-check drifts)? Governs the refresh cadence and the maintenance burden.
4. **CC-BY contamination policy:** is uniform CC-BY-4.0 output (to safely absorb Reactome) acceptable, or do we
   want CC0-pure packs (excluding CC-BY sources) for maximal downstream freedom? Affects which packs can mix.
5. **Priority-B per-study rights:** can derivative tracks from cBioPortal/GDC-open/DepMap/MSK-hotspots be
   redistributed per-study, or only via the upstream portal? Default exclude-on-doubt may shrink scope.
6. **Adoption channel:** which is the fastest externally-verifiable adoption event — Ensembl Track Hub Registry
   listing, UCSC public-hub acceptance, a course syllabus, or a paper citation? Prioritize outreach accordingly.
7. **First confirmed adopter** (unchanged from plan) — lab, educator, browser project, or advocacy org — remains
   the gating risk for "delivered, not merged."

---
*Research conducted June 2026. All license determinations above are first-pass triage signals for the plan's
human license/provenance reviewer, not authoritative legal calls — consistent with PLAN §5/§7.*
