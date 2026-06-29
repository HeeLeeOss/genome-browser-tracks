# PLAN — genome-browser-tracks

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) · Lane: donated

## Executive summary

Cancer genomics knowledge is published across dozens of open resources — gene models (GENCODE,
RefSeq, Ensembl), curated variant interpretations (CIViC, ClinVar), driver-gene and target
evidence (Open Targets), recurrent mutation hotspots, and pathway membership (Reactome) — but a
researcher, bioinformatician, educator, or research-literate patient advocate who simply wants to
*see* these cancer features laid out along the genome in a browser must hand-assemble them: find
the source, check its license, normalise coordinates to the right assembly, convert to a track
format, write a browser config, and validate that it renders. This is repetitive, error-prone
(coordinate off-by-one and wrong-assembly mistakes are the classic silent failures of genomics),
and most people never do it — so the open knowledge stays locked in flat files and web portals
instead of being explorable in context.

This project produces **curated, license-clean, provenance-complete cancer feature "track packs"
and the browser configurations to load them** — standard track files (BED/bigBed/bigWig/GFF3/VCF)
plus matching **UCSC track-hub**, **JBrowse 2**, and **IGV** configurations — built from
**open-access annotations only** and emitted from a single canonical manifest so all three browser
targets stay in sync. The deliverable is the *derived track files + configs + provenance*, made
freely loadable by URL so anyone can drop a cancer feature layer into the browser they already use.

Two hard constraints define the project and lead every design decision:

1. **The cancer data guardrail (binding).** ONLY open-access / aggregate / de-identified data is in
   scope. Controlled-access sources (dbGaP, EGA, individual-level biobanks) and any identifiable
   patient data are **out of scope** — they require authorised access and IRB oversight we neither
   have nor seek. Every source is license-verified before any track is built; non-commercial
   sources (COSMIC, the Cancer Gene Census, OncoKB) **cannot** be redistributed as open tracks and
   are excluded from open packs. Provenance is recorded on every feature.

2. **No medical advice.** Track packs are **research and education artifacts, not clinical
   decision support.** Anything patient-facing is education-only, carries an explicit "not medical
   advice" notice, and requires **oncologist + patient-advocate review** (`riskTier: high`). The
   core researcher/educator-facing packs are `riskTier: low`; the moment a pack frames variants as
   clinically actionable for an individual, it leaves scope or escalates to high.

Risk tier is **low** for the core engine and packs, with a clearly-fenced **high** boundary for any
patient-facing educational layer. The dominant risks are not novelty but *correctness and
licensing*: a wrong-assembly track, a non-commercial source laundered into an "open" file, or a
research track misread as clinical advice. The plan front-loads a license/data-tier gate, a
coordinate/assembly validation gate, and explicit "not medical advice" framing.

## Problem & beneficiaries

**Who is helped.**
- **Cancer researchers & bioinformaticians** who want to view cancer-relevant features (driver
  genes, hotspots, curated variants, pathways) alongside their own data in UCSC/JBrowse/IGV without
  rebuilding the annotation plumbing each time.
- **Educators & students** in genomics, oncology, and bioinformatics who need ready-to-load,
  correctly-sourced teaching tracks (e.g. "the canonical cancer driver genes on GRCh38").
- **Research-literate patient advocates** who read the primary science and want to understand where
  in the genome the features they read about actually sit — strictly as education, never as advice.
- **Open-science maintainers** (genome-browser projects, annotation hubs) who benefit from
  well-documented, license-clean community track hubs they can list.

**The verified need.** The general need — open cancer annotations are hard to view in-browser and
people re-do the assembly/format/license work repeatedly — is well established from the existence of
ad-hoc, often license-unclear track hubs and the recurring "how do I get X into the browser"
questions in genomics forums. We treat the *general* need as real but the **per-pack, per-partner
need as TO BE SECURED**: we have **not** yet confirmed a named lab, educator, browser project, or
advocacy org who has agreed to adopt specific packs. Until a named beneficiary confirms they will
use/adopt a pack, tasks carry `verifiedNeed: false`. This honesty matters because "delivered, not
merged" requires the output to be *used by a beneficiary*, not merely produced and parked.

**Partner org.** TO BE SECURED. Candidate channels: an academic cancer-genomics lab or core
facility; a bioinformatics course/instructor; a genome-browser project (UCSC public hub registry,
JBrowse plugin/config registry); and — for the high-risk educational layer only — a patient
advocacy organisation that can supply oncologist + advocate review. M0 includes explicit outreach;
no partner is assumed. The cold-start is de-risked by self-serve hosting (below) so a real
*used-in-a-browser* outcome is reachable even before a partner is secured.

## Goals and non-goals

**Goals**
- Produce a reusable **canonical track-pack manifest** and a toolkit that emits, from one source of
  truth, valid **UCSC track-hub**, **JBrowse 2**, and **IGV** configurations plus the underlying
  track files.
- For each in-scope source, deliver a complete, license-verified, provenance-complete cancer track
  pack that is **verified to render correctly on its declared assembly** in the target browser(s).
- Make **source-license + data-tier verification** a non-skippable, auditable gate (open/aggregate/
  de-identified only; non-commercial and controlled-access excluded).
- Make **coordinate/assembly correctness** a non-skippable, auditable gate (assembly recorded,
  0-/1-based convention handled, validated with standard tooling).
- Carry **per-feature provenance** (source, source version/release, source record ID, license) in
  every track so any feature can be traced to its open source.
- Host packs so they are **loadable by URL** (self-serve: GitHub Pages / Zenodo) and, where useful,
  listed in public hub registries.
- Keep the engine **agent-neutral and vendor-neutral** per Elyos architecture rules: no
  agent-specific logic in the core; browser-specific projection logic isolated in adapters.

**Non-goals**
- We do **not** redistribute or build tracks from **non-commercial** sources (COSMIC, Cancer Gene
  Census, OncoKB) or any **controlled-access / individual-level** data — excluded, never "best-guessed."
- We do **not** generate primary data, call variants, run pipelines, or re-analyse raw sequencing.
- We do **not** provide clinical interpretation, diagnosis, prognosis, treatment guidance, or any
  per-individual actionability claim. Packs are research/education only.
- We do **not** invent or infer annotations; every feature traces to an open source record.
- We do **not** auto-publish to public hub registries; a human submits after review.
- We do **not** host or proxy the *source* portals' bulk data; we publish only our license-clean
  derived track packs with attribution.

## Success metrics (outcomes)

Outcome-based and beneficiary-centric. Vanity metrics ("tracks generated") are explicitly excluded —
a track nobody loads is not an outcome.

| Metric | Baseline | Target (first 6 months) |
| --- | --- | --- |
| Track packs published **and verified-loadable** in ≥1 target browser on the declared assembly | 0 | 6 |
| **External adoption events** (third party loads/registers/forks/cites a pack, or uses it in a lesson) — verifiable | 0 | ≥ 3 with externally verifiable evidence |
| License + data-tier gate integrity: published packs with a recorded **redistributable** license + complete provenance | n/a | 100% (and **0** non-commercial / controlled-data leaks) |
| Coordinate/assembly correctness: delivered packs with **0** assembly-mismatch or off-by-one defects found in review/validation | n/a | 100% pass (target 0 defects) |
| Confirmed adopting partner(s) (lab / educator / browser project / advocacy org) | 0 | ≥ 1 secured |
| Multi-browser parity: packs emitting **all three** targets (UCSC hub + JBrowse2 + IGV) from one manifest | n/a | 100% of delivered packs |
| Provenance completeness score (per pack, see below) | n/a | every delivered pack ≥ 90/100 |

**Attribution of outcomes.** An "adoption event" must be externally verifiable: a track-hub URL
referenced in a third party's repo/notebook/paper, acceptance into the UCSC public hub list or a
JBrowse config registry, a fork/issue/PR referencing the pack, or a course syllabus/lesson that
loads it. Self-reported use does not count.

**Quantifying provenance completeness (so DoDs are checkable).** Per pack, a **provenance
completeness score (0–100)** = fraction of required provenance fields populated and source-verified:
source name, source release/version, source URL, source record IDs at feature level, license id +
URL + snapshot reference, assembly id, coordinate convention, retrieval timestamp, attribution
string, and "not medical advice"/intended-use statement. Target: every delivered pack ≥ **90/100**;
the score is stored in the pack's manifest and gate artifact.

**Quantifying "verified-loadable".** A pack is *verified-loadable* only when it passes (a) format
validation with the standard tool for the format, (b) an assembly/coordinate sanity check, and (c) a
**render check** — the hub/config URL is actually loaded into the target browser and the expected
features appear at expected loci on the declared assembly (evidence: a recorded screenshot/log
reference in the pack's `outcomes/<pack-id>.json`).

## Scope

**In scope**
- **Track files** derived from open cancer annotations: gene/region features (BED/bigBed/GFF3),
  quantitative tracks (bigWig), and variant tracks (VCF/bigBed) — all carrying per-feature
  provenance.
- **Browser configurations** from one canonical manifest: UCSC **track hub** (`hub.txt`,
  `genomes.txt`, `trackDb.txt`), **JBrowse 2** `config.json`, and **IGV** session/registry entries.
- **Source license + data-tier gate** and **coordinate/assembly gate** artifacts per pack.
- **Assembly support:** GRCh38/hg38 (primary) and GRCh37/hg19 (secondary, still common in cancer),
  with documented liftOver provenance when a source is only available on one assembly.
- **Self-serve hosting** of packs (GitHub Pages / Zenodo) with loadable URLs, and (human-submitted)
  registration into public hub registries.
- A clearly-fenced, **education-only patient-facing layer** (e.g. "where are the common cancer
  genes?") — gated `riskTier: high` with oncologist + advocate review and "not medical advice".

**Source backlog.** The pool of annotation sources we *might* turn into packs is maintained as a
license-classified **source catalog** (embedded in §7 and the scaling table in `TASKS.md`). It is a
*candidate backlog only*: no source becomes a pack until it passes the per-source license+data-tier
gate. The catalog biases toward clearly-redistributable sources (CC0, public-domain, CC-BY) and
honestly flags non-commercial and controlled-access sources as **excluded**.

**Out of scope**
- Non-commercial sources (COSMIC / Cancer Gene Census / OncoKB) and **all** controlled-access /
  individual-level data.
- Variant calling, pipelines, raw-data reanalysis, or generating new annotations.
- Clinical interpretation, diagnosis, prognosis, treatment/actionability guidance for individuals.
- Hosting/mirroring source portals' bulk data, or any pack primarily serving a for-profit's private data.
- Automated, unattended publishing to public registries.

## Solution approach & architecture

This is a **content/data project with light software**: small TypeScript tools that wrap standard,
trusted bioinformatics CLIs to build and validate tracks, plus a canonical manifest that projects to
three browser formats. It is **not** a data pipeline that moves or re-analyses primary data.

**Pipeline (per track pack)**
1. **Source & license + data-tier gate.** Identify the annotation source, its license, and its data
   tier. PASS only if the data is **open-access / aggregate / de-identified** *and* the license
   permits **redistribution of derivative track files** (CC0 / public domain / CC-BY-style).
   Non-commercial, controlled-access, or unclear → FLAG/EXCLUDE. Record a committed gate artifact.
2. **Provenance capture.** Source name, release/version, source URL, license id+URL+snapshot,
   retrieval timestamp, attribution string, assembly, coordinate convention, and the source record
   ID schema used for per-feature provenance.
3. **Coordinate & assembly normalisation.** Pin the reference assembly (hg38 primary), record the
   source's coordinate convention (0-based half-open vs 1-based), normalise to the track format's
   convention, and (only if the source is single-assembly) liftOver with recorded chain provenance
   and a mapped/unmapped report.
4. **Track generation.** Build the standard track file(s) (`bedToBigBed`/`bedSort`, `bgzip`+`tabix`,
   `bigWig` tooling), embedding per-feature provenance fields and the intended-use/"not medical
   advice" statement in the track description.
5. **Config generation.** Emit UCSC hub, JBrowse 2 `config.json`, and IGV session from the canonical
   manifest — each a *projection* of the manifest, never hand-maintained in parallel.
6. **Validation.** Format validation (`validateFiles`/`hubCheck`, JBrowse config lint, VCF/tabix
   checks), coordinate/assembly sanity checks, and a **render check** in the target browser.
7. **Review.** License/provenance reviewer (mandatory) + bioinformatics reviewer (mandatory).
   Oncologist + patient-advocate reviewer for any patient-facing/clinical-framing content (high).
8. **Publish & adopt.** Host the pack (self-serve URL), and a human submits to a registry / hands it
   to the adopting beneficiary. The Steward records the adoption/render evidence artifact.

**Canonical track-pack manifest (source of truth).** One manifest object per pack; all browser
configs are projections of it. Fields include:
`id`, `title`, `description`, `intendedUse` (research/education), `notMedicalAdvice: true`,
`assembly {id, source}`, `coordinateConvention`, `tracks[] {name, type(bed|bigBed|bigWig|vcf|gff3),
file, source {name, release, url, recordIdScheme}, license {id, url, permitsRedistribution:boolean,
nonCommercial:boolean, snapshotRef}, dataTier(open|aggregate|deidentified), attribution, liftOver
{from,to,chain,unmappedCount}}`, `provenanceCompleteness {score}`, `riskTier`,
`reviewers {license, bioinformatics, oncologist?, advocate?}`, `hosting {hubUrl, jbrowseUrl, igvUrl}`.

**Tech stack.** TypeScript, ESM, pnpm workspaces (Elyos convention). Tools are thin Node wrappers
over **standard, trusted external CLIs** (UCSC userApps: `bedToBigBed`, `bedSort`, `fetchChromSizes`,
`liftOver`, `hubCheck`, `validateFiles`; htslib: `bgzip`, `tabix`) — we do not reimplement genomics
primitives. Manifests/configs are JSON; tracks are standard formats. No runtime service; everything
runs locally or in CI. External CLI versions are **pinned and recorded** per pack (see Dependencies).

**Agent-neutrality (Elyos rule).** The canonical manifest, gate, and validators are vendor-neutral
core; each browser target (UCSC/JBrowse/IGV) is an isolated **adapter**. No coding-agent-specific
logic anywhere. Donated lane only — the CLI prepares the workspace and opens PRs; a human runs their
agent and submits.

**Key decisions.**
- **Manifest-first**, so we never hand-maintain three parallel browser configs.
- The **license+data-tier gate** and the **coordinate/assembly gate** are *blocking* committed
  checklist artifacts, not informal judgement.
- We **wrap, not reinvent**, the canonical bioinformatics CLIs (correctness + community trust).
- Packs are **versioned to the source release**; a source bump is a deliberate refresh task, never a
  silent overwrite.

## Data, licensing & compliance

**THIS IS THE CRITICAL SECTION. The cancer data guardrails below are binding and lead it.**

### Cancer data guardrails (binding)
- **Open-access / aggregate / de-identified data ONLY.** Controlled-access (dbGaP, EGA,
  individual-level biobanks) and any identifiable patient data are **out of scope** — they require
  authorised access + IRB and are never used, downloaded, or referenced as track sources.
- **Per-source license verification before any track is built.** A source is usable only if its
  license permits **redistribution of derivative works** under an open license. Open imaging tiers
  (open TCGA/CPTAC) are usable per their open-access terms; **COSMIC, the Cancer Gene Census, and
  OncoKB are NON-COMMERCIAL and are EXCLUDED** from open packs (we cannot redistribute derivative
  tracks under an open license, and Elyos output must be openly licensed).
- **No medical advice.** Tracks are research/education artifacts. Every pack carries an explicit
  intended-use + "not medical advice" statement. Patient-facing content is education-only and
  requires **oncologist + patient-advocate sign-off** (`riskTier: high`).
- **Provenance on every assertion.** Every feature in every track is traceable to an open source
  record (source, release, record ID, license).

### Sources we accept (must be open-tier AND permit redistribution of derivatives)
| Source | Content | License (verify at gate) | Disposition |
| --- | --- | --- | --- |
| GENCODE | Gene/transcript models | Open, no redistribution restriction | **Accept (A)** |
| RefSeq (NCBI) | Gene models | US-gov public domain | **Accept (A)** |
| Ensembl | Gene models/annotation | Open (no restriction; verify per dataset) | **Accept (A)** |
| CIViC | Curated clinical-evidence variants | **CC0** | **Accept (A)** (frame as research evidence) |
| ClinVar (NCBI) | Variant–condition aggregate | US-gov public domain | **Accept (A)** |
| Open Targets Platform | Target/driver/association evidence | **CC0** | **Accept (A)** |
| Reactome | Pathway membership | **CC-BY 4.0** | **Accept (A)** (attribution required) |
| gnomAD | Population allele frequencies (context) | Open (free, no restriction; verify) | **Accept (A)** |
| Pfam / InterPro | Protein domains | **CC0** | **Accept (A)** |
| dbSNP (NCBI) | Variant identifiers | US-gov public domain | **Accept (A)** |
| GDC/TCGA open-access somatic (MAF) | Aggregate somatic mutations (open tier) | Open tier; verify per study/file | **Verify (B)** → gate per file |
| cBioPortal study data | Aggregated study results | Per-study (CC-BY / ODC / custom) | **Verify (B)** → gate per study |
| Cancer hotspots (e.g. MSKCC) | Recurrent hotspot positions | Verify terms | **Verify (B)** → gate |
| DepMap | Dependencies/cell-line data | CC-BY / partly restricted; verify | **Verify (B)** → gate |
| COSMIC / Cancer Gene Census | Mutations / curated cancer genes | **Non-commercial academic** | **EXCLUDE (C)** |
| OncoKB | Variant actionability | **Non-commercial / restricted** | **EXCLUDE (C)** |
| Cancer Genome Interpreter | Interpretation | Verify (NC suspected) | **EXCLUDE pending verification** |
| Any dbGaP/EGA/individual-level | Controlled-access | Controlled | **EXCLUDE (out of scope)** |

The matrix is a *first-pass triage signal*; **no row becomes a pack until the per-source gate
records `permitsRedistribution: true` with a cited clause/URL.** Missing/unparseable evidence =
EXCLUDE, never default-allow.

**Objective "permits redistribution" criterion.** A source PASSes only if (1) data tier ∈
{open, aggregate, de-identified}, AND (2) license is on the accepted list (or verified equivalent)
with `permitsRedistribution: true` and `nonCommercial: false`, evidenced by a cited clause/URL, AND
(3) any attribution requirement (CC-BY/Reactome) is captured. Otherwise EXCLUDE/FLAG.

**Provenance model.** Per pack and per feature: source name, source release/version, source URL,
source record ID, license id + URL + **snapshot reference** (committed copy of the license text +
SHA-256 hash + Wayback save URL), retrieval timestamp, assembly id, coordinate convention,
attribution string. Provenance is part of the committed deliverable and embedded in track metadata.

**Privacy/PII & re-identification stance.** We use only aggregate/open-tier/de-identified data; we
never use individual-level genotypes. The gate explicitly checks that a variant track is derived
from aggregate/open-tier sources and is **not** a small-cohort extract that could be re-identifying
(e.g. a rare variant + locus combination tied to few individuals). Any re-identification signal =
EXCLUDE. We never de-anonymise, link to identities, or aggregate up from controlled data.

**Attribution & output license.** Every pack attributes each source per its license, links the
source, and states clearly that the **derived track files/configs** — not the source data — are our
contribution. Derived **track files/configs/docs are licensed CC-BY-4.0** (compatible with CC0/
public-domain/CC-BY sources; CC-BY sources' attribution preserved). **Code** (manifest tooling,
generators, validators, adapters) is **MIT**. We never relicense a source's data more permissively
than its terms allow and never apply an open license to non-commercial-sourced content.

## Quality, review & risk gates

**Risk tier: low** for the core engine and researcher/educator-facing packs; **high** for any
patient-facing/clinical-framing content (oncologist + patient-advocate sign-off required before
publish). A pack defaults to higher tier on any doubt.

**Required review before a pack is "done":**
- **License + provenance reviewer (mandatory, every pack):** confirms data tier is open/aggregate/
  de-identified, license permits redistribution of derivatives, no non-commercial/controlled source,
  attribution captured, provenance complete. **Hard gate.**
- **Bioinformatics reviewer (mandatory, every pack):** confirms assembly is correct and recorded,
  coordinate convention handled (no off-by-one), liftOver provenance present where used, formats
  validate, and the **render check** shows expected features at expected loci.
- **Oncologist + patient-advocate reviewer (for any patient-facing/clinical-framing pack):**
  required sign-off; verifies education-only framing, accuracy, and the "not medical advice" notice.
  Escalates the pack to `riskTier: high`.

**Test fixtures & golden files (so "CI green" means something).** Each tool ships committed,
CI-exercised assets using **only tiny synthetic or trivially-public fixtures** (never bulk source
data):
- **Manifest → config projectors** — golden manifest → expected UCSC `trackDb` / JBrowse `config.json`
  / IGV session pairs, diffed in CI; outputs validated against the target spec.
- **Track builders** — golden tiny BED/GFF3 → expected bigBed/sorted/tabixed output, with
  deliberate malformed inputs that must fail (bad coordinate, wrong chrom name, out-of-bounds).
- **Coordinate/assembly checks** — fixtures asserting 0-/1-based conversions and an assembly-mismatch
  case that must be caught.
- **Render check** — at least one real load of the hub/config URL into the target browser
  (UCSC public/JBrowse local) per delivered pack, evidence recorded.

**Definition of Shipped.** A track pack is **Shipped** when it is: license+data-tier gate PASSed
(committed artifact); provenance complete (score ≥ 90/100); coordinate/assembly verified; format-
and render-verified in ≥1 target browser on the declared assembly; published at a loadable URL with
attribution and "not medical advice" notice; **and adopted/used by a beneficiary** (third-party
load/registration/citation/fork/lesson use) with the Steward's `outcomes/<pack-id>.json` evidence
artifact recorded. Producing files is **not** Shipped; recorded use is.

## Roadmap & milestones

**M0 — Foundation & cold-start (thin)**
- Goal: build the manifest + UCSC-hub generator + the two gates, and prove the end-to-end flow with
  one pilot pack from a clearly-permissive source; begin partner outreach.
- **Cold-start de-risking.** To avoid producing packs nobody loads, the pilot uses **self-serve
  hosting** (GitHub Pages or Zenodo) so the hub URL is loadable into UCSC/JBrowse by anyone — a real
  *rendered/used* outcome is reachable without a third party. The pilot source must be Priority-A
  (CC0/public-domain/CC-BY): e.g. an **open cancer driver-gene region track** built from **Open
  Targets (CC0)** target lists projected onto **GENCODE** gene models on hg38. Outreach to a lab/
  educator runs in parallel.
- Exit criteria: (1) canonical manifest model + UCSC track-hub generator published; (2)
  license+data-tier gate **and** coordinate/assembly gate checklists exist and are applied to the
  pilot; (3) license/provenance reviewer **and** bioinformatics reviewer roles named (blocking);
  (4) one pilot pack built end-to-end, format- **and render-verified** on hg38, hosted at a loadable
  URL, with provenance ≥ 90/100 — and **used** (loaded by a third party / referenced) with evidence
  recorded, or hosted with the adoption blocker surfaced; (5) ≥ 1 partner-outreach thread opened.

**M1 — Gates hardened + multi-browser parity + first adoption**
- Goal: add JBrowse 2 and IGV projectors, harden the gates, and get real packs adopted.
- Exit criteria: (1) JBrowse 2 + IGV projectors emit valid configs from the manifest, golden-tested;
  (2) license-snapshot capture (committed copy + SHA-256 + Wayback) wired into provenance; (3) ≥ 3
  packs published, each emitting all three targets and render-verified; (4) ≥ 1 external adoption
  event recorded; (5) hg19 liftOver path with recorded chain provenance + unmapped report.

**M2 — Source-catalog scale & multi-source packs**
- Goal: triage more accepted sources through the gate and deliver richer packs (curated variants
  from CIViC/ClinVar in cancer genes, recurrent hotspots, Reactome pathway tracks), with measurable
  effort reduction.
- Exit criteria: (1) ≥ 8 sources triaged with committed gate artifacts (Priority-A first;
  B verified per-source); (2) ≥ 6 packs published+adopted cumulatively; (3) at least one
  multi-source pack (e.g. driver genes + curated variants + pathway membership) on hg38 and hg19;
  (4) median per-pack build effort (AI-session minutes + review cycles, from the outcome ledger)
  measurably reduced vs. the recorded M0/M1 baseline.

**M3 — Adoption, education & sustainability**
- Goal: demonstrate real downstream use, ship the (high-risk) education-only layer with expert
  review, and stand up a refresh model tied to source releases.
- Exit criteria: (1) ≥ 3 verifiable external adoption events; (2) one **education-only** patient-
  facing pack/tutorial shipped with oncologist + patient-advocate sign-off and "not medical advice";
  (3) ≥ 1 pack listed in a public hub/config registry; (4) documented source-release refresh process
  + named steward for ongoing liaison and re-validation.

Dependencies: M1 projectors/snapshot depend on M0 manifest+gates; M2 packs depend on M1 multi-browser
parity + hardened gate; M3 adoption/education depend on a body of M1–M2 delivered packs and the
secured oncologist+advocate reviewers.

## Work breakdown

The itemized, schema-mapped backlog lives in `TASKS.md`, organized by the milestones above. Each
milestone has a task table (`ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer`),
acceptance criteria for the most important tasks, and a milestone Definition of Done. `TASKS.md` also
carries a **source-catalog triage task** and concrete **example per-source pack tasks** drawn from
real accepted sources, a backlog of sized-but-unscheduled tasks, and one complete, schema-valid
example Task JSON.

## Governance, roles & stakeholders

- **Maintainer (Owner):** TBD — owns the manifest, toolkit, source catalog, and backlog.
- **License + provenance reviewer:** TBD (TO BE SECURED) — mandatory, **non-skippable** gatekeeper
  who can read open-data/bioinformatics licenses (CC0/CC-BY/public-domain/NC) and apply the data-tier
  rule. Must be filled **before the M0 pilot is reviewed**. Until named, all tasks stay
  `verifiedNeed: false` and no source can pass the gate. May rotate among ≥ 2 qualified reviewers,
  but ≥ 1 must always exist or work halts.
- **Bioinformatics reviewer:** TBD (TO BE SECURED) — mandatory; verifies assembly/coordinate
  correctness, liftOver provenance, format validity, and the render check. Genomics-literate.
- **Oncologist + patient-advocate reviewers:** TO BE SECURED — required for any patient-facing/
  clinical-framing pack (`riskTier: high`); no such pack ships without both sign-offs.
- **Steward (last-mile owner):** TBD — owns adopter relationships and records adoption/render
  evidence (the "delivered" signal). Critical because Definition of Shipped is *use*, not production.
- **Partner / requestor:** TO BE SECURED — named lab / educator / browser project / advocacy org.

## Dependencies & integrations

- **External standards/specs (pinned, recorded per pack):** UCSC track-hub spec (`hub.txt`/
  `genomes.txt`/`trackDb.txt`), bigBed/bigWig/BED/GFF3 formats, JBrowse 2 `config.json` schema, IGV
  session format, VCF 4.x + tabix, reference assemblies GRCh38/hg38 and GRCh37/hg19, liftOver chain
  files, SPDX license identifiers.
- **External CLIs (pinned versions recorded):** UCSC userApps (`bedToBigBed`, `bedSort`,
  `fetchChromSizes`, `liftOver`, `hubCheck`, `validateFiles`), htslib (`bgzip`, `tabix`). Version
  drift handled by a deliberate bump task, never silent.
- **Annotation sources:** the accepted open sources in §7 (GENCODE, RefSeq, Ensembl, CIViC, ClinVar,
  Open Targets, Reactome, gnomAD, Pfam/InterPro, dbSNP; GDC-open/cBioPortal/hotspots/DepMap per-gate).
  COSMIC/CGC/OncoKB and controlled-access excluded.
- **Hosting/registries:** GitHub Pages / Zenodo (self-serve); UCSC public hub registry, JBrowse
  config registry (human-submitted). Output-only; no automated upload.
- **Elyos pieces:** Task JSON schema (`packages/schema`), donated-lane CLI workspace/PR flow
  (`packages/cli`), good-deed definition + refusal guardrails. No funded-lane/runner dependency.

## Risks & mitigations

| Risk | Likelihood | Impact | Mitigation | Owner |
| --- | --- | --- | --- | --- |
| Wrong-assembly or off-by-one coordinates → track silently points to wrong loci | Medium | High | Coordinate/assembly gate; record assembly + convention; validateFiles + render check; liftOver provenance + unmapped report | Bioinformatics reviewer |
| Non-commercial source (COSMIC/CGC/OncoKB) laundered into an "open" track | Medium | High | License+data-tier gate; excluded list enforced; `permitsRedistribution:true` only with cited clause; reviewers reject on doubt | License+provenance reviewer |
| Controlled-access / individual-level data used as a source | Low | High | Data-tier gate (open/aggregate/de-identified only); excluded by policy; re-identification check | License+provenance reviewer |
| Research track misread as clinical/medical advice | Medium | High | "Not medical advice" + intended-use in every pack; education-only patient layer gated high (oncologist+advocate); no actionability claims | Maintainer / oncologist reviewer |
| Pack produced but never loaded (fails "delivered") | Medium | High | Self-serve loadable hosting; render check; Steward + adoption evidence; `verifiedNeed:false` until adopter secured | Steward |
| Source release changes annotations → packs stale/incorrect | Medium | Medium | Version pack to source release; refresh process; re-validation as `maintenance` tasks | Maintainer |
| Browser-spec / CLI drift breaks configs | Medium | Low | Manifest-first; pinned spec/CLI versions; isolated adapters; golden fixtures in CI | Maintainer |
| Re-identification from a small-cohort variant track | Low | High | Aggregate/open-tier only; re-identification check in gate; exclude on signal | License+provenance reviewer |
| CC-BY attribution (Reactome etc.) dropped from derived tracks | Medium | Low | Attribution captured in manifest + embedded in track metadata; reviewer checks | License+provenance reviewer |
| Scope creep into variant calling / reanalysis | Low | Medium | Explicit non-goal; reviewers reject any primary-data processing | Maintainer |

## Security & privacy

- **Threat surface is small** (no runtime service; tracks/configs are static files). Main surfaces:
  CI, the external CLIs we wrap, and the published packs.
- **No PII/individual-level data** is ever ingested — only aggregate/open-tier/de-identified sources;
  the data-tier gate and re-identification check enforce this. We never de-anonymise or link to
  identities.
- **Secrets handling:** no credentials needed by default. If a registry submission ever needs a
  token, the human submitting supplies it; tokens are never written into logs, receipts, configs, or
  committed files (Elyos rule).
- **Supply-chain:** external CLIs are pinned and checksummed; provenance/license snapshots are hashed
  (SHA-256). We verify source downloads against published checksums where available.
- **Abuse/misuse prevention:** refuse and flag any task steering tracks toward clinical decision
  support for individuals, de-anonymisation, linking variants to identities, laundering a
  non-commercial/controlled source as open, or primarily serving a for-profit. Tracks remain
  descriptive, source-verified, education/research-framed.

## Sustainability & maintenance

- **Post-delivery ownership:** the maintainer keeps the manifest/tooling/adapters current with
  browser-spec and CLI changes; the steward maintains adopter relationships and records outcomes.
- **Refresh:** packs are versioned to their source release; when a source publishes a new release,
  re-validation/refresh is a `maintenance` task. The render check and validators detect breakage.
- **Outcome tracking:** the steward records adoption/render evidence and external reuse signals
  against the success metrics, reviewed each milestone.
- **Bus-factor:** ≥ 2 qualified reviewers per mandatory role; everything (manifest, gates, fixtures)
  is committed and reproducible so a new contributor can rebuild any pack from its manifest.

## Open questions

- Which named lab / educator / browser project / advocacy org will be the first confirmed adopter?
- Self-serve hosting: GitHub Pages (simple, hub-by-URL) vs Zenodo (citable DOI, versioned) as the
  default — or both per pack? (Leaning: Zenodo for citable packs, Pages for live hubs.)
- For Priority-B sources (GDC-open/cBioPortal/hotspots/DepMap), is redistribution of *derivative*
  tracks permitted per-study, or only the upstream portal's own distribution? (Default: gate
  per-source; exclude on doubt.)
- Do we attempt T2T-CHM13 assembly support, or stay hg38/hg19 for now? (Default: hg38/hg19 first.)
- For the education-only patient layer: which advocacy org supplies oncologist + advocate review, and
  what is the standing "not medical advice" template they approve?
- License-snapshot storage format: confirm committed copy + SHA-256 + Wayback (as in
  open-data-datasheets) is the project standard. (Default: yes, reuse it.)

## References

- Elyos work rules — `C:\code\elyos\CLAUDE.md`
- Good Deed Definition + risk tiers — `C:\code\elyos\docs\good-deed-definition.md`
- Task JSON schema — `C:\code\elyos\packages\schema\src\schemas.ts`
- Portfolio roadmap (Track 8 cancer guardrails) — `C:\code\elyos\planning\ROADMAP.md`
- Sibling plan (license/provenance gate pattern) — `C:\code\elyos\planning\projects\open-data-datasheets\PLAN.md`
- UCSC Genome Browser track hubs / bigBed / bigWig formats; UCSC userApps (`bedToBigBed`, `liftOver`, `hubCheck`, `validateFiles`)
- JBrowse 2 configuration; IGV session format
- Reference assemblies GRCh38/hg38, GRCh37/hg19; liftOver chain files; VCF 4.x + htslib/tabix
- Accepted sources: GENCODE, RefSeq, Ensembl, CIViC (CC0), ClinVar (PD), Open Targets (CC0), Reactome (CC-BY 4.0), gnomAD, Pfam/InterPro (CC0), dbSNP (PD)
- Excluded (non-commercial): COSMIC, Cancer Gene Census, OncoKB — and all controlled-access (dbGaP/EGA)

---

## Appendix A — Improvements applied

The following 25 specific improvements were identified during drafting and **applied** to the plan
above (and to `TASKS.md`). Each notes where it landed.

1. **Two distinct blocking gates, not one.** Split the single "gate" into a **license+data-tier
   gate** and a separate **coordinate/assembly gate** — applied in §6 pipeline, §8, and as two
   separate M0 tasks in `TASKS.md`. (Coordinate errors are a different failure class from licensing.)
2. **Cancer guardrails lead §7** as a binding sub-block (open/aggregate/de-identified only;
   NC/controlled excluded; no medical advice; provenance per feature) — per the prompt's mandate.
3. **Explicit excluded-source list with named sources** (COSMIC, Cancer Gene Census, OncoKB,
   dbGaP/EGA) in §7's matrix, so "non-commercial out" is concrete, not abstract.
4. **Objective "permits redistribution" criterion** (data-tier ∈ open/aggregate/de-identified AND
   license accepted AND cited clause AND attribution captured) added to §7 — no default-allow.
5. **"Not medical advice" + intended-use embedded in the manifest and every track's metadata**, not
   just docs — §6 manifest fields, §7, §8, and acceptance criteria in `TASKS.md`.
6. **Render check as a first-class verification** (load the hub/config URL into the real browser and
   confirm features at expected loci) — §4 success metric "verified-loadable", §8 fixtures, and DoDs.
7. **Manifest-first multi-browser parity** so UCSC/JBrowse/IGV are projections of one source of
   truth — §6 key decisions; success-metric row for 3-target parity.
8. **Self-serve hosting cold-start** (GitHub Pages / Zenodo) so a *used-in-browser* outcome is
   reachable before a partner is secured — §2, §9 M0.
9. **Per-feature provenance** (source/release/record-ID/license) elevated to a goal and a
   manifest/track field, with a **provenance completeness score (0–100, ≥90 target)** — §3, §4, §6.
10. **Re-identification check** for small-cohort variant tracks added to the gate and §14 — closes a
    subtle cancer-genomics privacy gap beyond "no PII".
11. **liftOver provenance + unmapped report** required whenever an assembly conversion is done — §6,
    M1 exit criteria, risk table.
12. **Pinned external-CLI versions** (UCSC userApps, htslib) recorded per pack, with a deliberate
    bump task — §6, §12, risk table.
13. **Wrap-not-reinvent decision** for genomics primitives (use trusted UCSC/htslib CLIs) — §6 key
    decisions and tech stack, reducing correctness risk.
14. **Agent-neutral / vendor-neutral architecture** explicitly mapped to Elyos rules (core vs
    adapters; donated lane; no headless agent) — §6.
15. **Deliverable type `dataset`** chosen for track packs (they *are* derived data files), distinct
    from `document` (configs/docs) and `pr` (code) — codified in `TASKS.md` field mapping; a
    deliberate, honest divergence from open-data-datasheets (which never republishes data).
16. **High-risk boundary fenced precisely:** core packs `low`; patient-facing/clinical-framing packs
    `high` with oncologist + patient-advocate sign-off — §1, §8, §11, M3.
17. **Outcome-based success metrics** (adoption events, verified-loadable, 0 leaks, 0 coordinate
    defects) with baselines/targets and externally-verifiable evidence rule — §4.
18. **Two mandatory reviewer roles named as blocking prerequisites** (license/provenance +
    bioinformatics) before the M0 pilot is reviewed, with rotation/bus-factor — §11, M0 exit.
19. **License-text snapshot standard reused** (committed copy + SHA-256 + Wayback) from the sibling
    project for consistency — §7 provenance model, open question confirmed default.
20. **Source-release versioning + refresh process** so stale annotations are caught and re-validated
    as `maintenance` tasks — §6, §15, risk table, M3.
21. **Assembly scope stated** (hg38 primary, hg19 secondary; T2T deferred) so packs declare and
    validate against a known assembly — §5, open questions.
22. **Priority A/B/C source classification** (clearly-permissive / verify-per-source / excluded) in
    §7 and the scaling table — mirrors the sibling project's funnel-and-filter model.
23. **Definition of Shipped = adopted/used, not produced**, with a recorded `outcomes/<pack-id>.json`
    evidence artifact — §8, §4 attribution rule, DoDs throughout `TASKS.md`.
24. **Effort-reduction metric** (median per-pack build minutes + review cycles vs M0/M1 baseline) so
    M2 "scale" is measurable, not asserted — §4 note, M2 exit.
25. **Refusal/abuse surface tailored to genomics** (no clinical decision support, no
    de-anonymisation, no NC/controlled laundering, no for-profit primary benefit) — §5, §14.

## Review sign-off

A completeness + correctness review of this plan and `TASKS.md` was performed; issues found were
fixed in place.

**Completeness.** All 17 required PLAN sections are present in the spec order. `TASKS.md` includes the
field-mapping, milestone tables, per-task acceptance criteria, milestone DoDs, a backlog, a
source-catalog scaling section, and a schema-valid example Task JSON. The cancer guardrails lead §7
as required.

**Correctness checks performed and resolved.**
- **Schema validity of the example Task JSON:** verified every `required` field is present
  (`id, title, project, type, lane, priority, domain, riskTier, urgent, deliverable, tokenEstimate,
  status, context, objective, acceptanceCriteria, output, verifiedNeed`); every enum value is legal
  (`type:writing`, `lane:donated`, `priority:high`, `riskTier:low`, `deliverable:document`,
  `tokenEstimate:small`, `status:open`); `acceptanceCriteria` has ≥1 item; `output` is non-empty; and
  **no `additionalProperties`** beyond the schema (only the allowed optional `resources`, `requestor`,
  `outputLicense` are used; no `fundedBudgetUsd` since lane is donated). PASS.
- **Deliverable-type consistency:** track packs → `dataset`; configs/docs → `document`; code → `pr`;
  no `translation` used. Confirmed consistent across `TASKS.md`.
- **Guardrail consistency:** no accepted source is non-commercial; COSMIC/CGC/OncoKB appear only on
  the excluded list; controlled-access is excluded everywhere; "not medical advice" + high-tier
  patient-facing fencing is consistent across §1/§7/§8/§11/M3.
- **`verifiedNeed: false`** everywhere (no partner secured) and `requestor: TO BE SECURED` —
  consistent with honest cold-start framing.
- **Dependency sanity:** task `Depends on` edges form a DAG (no cycles); pilot depends on manifest +
  both gates + both reviewer roles; M1+ projectors depend on the M0 manifest.

**Residual items for a human decision** (tracked in §16 Open questions): first confirmed adopter;
hosting default (Pages vs Zenodo); per-study redistribution rights for Priority-B sources; T2T
support; and the advocacy partner for the high-risk education layer. None block M0.

Sign-off: Draft v0.1.0 is internally consistent and schema-aligned; ready for maintainer review and
partner outreach.
