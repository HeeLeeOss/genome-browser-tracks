# TASKS — genome-browser-tracks

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) · Lane: donated

## How these tasks map to Hee-Lee Oss

Each task below becomes a Hee-Lee Oss **Task JSON** validated against
`packages/schema/src/schemas.ts`. Field mapping:

- `id` — stable slug ID from the tables (e.g. `genome-browser-tracks-manifest-001`).
- `title` — the table's Title.
- `project` — `genome-browser-tracks`.
- `type` — one of `code | research | writing | data | design-spec | maintenance` (per table).
- `lane` — `donated` for all tasks here (no funded escrow). A funded task would add `fundedBudgetUsd`.
- `priority` — `high | medium | low`.
- `domain` — array, e.g. `["cancer-research","genomics","open-science","education"]`.
- `riskTier` — `low | medium | high`. Core packs/tooling `low`; license/data-tier or coordinate
  judgement `medium`; any patient-facing/clinical-framing pack `high` (oncologist + advocate sign-off).
- `urgent` — boolean; `false` for all current tasks.
- `deliverable` — `pr | dataset | document | translation`. **Track packs (BED/bigBed/bigWig/VCF data
  files) → `dataset`**; browser configs / docs / gate artifacts → `document`; code (generators,
  validators, adapters) → `pr`. We do not use `translation` (no translated packs yet).
- `tokenEstimate` — `small | medium | large` (the Size column).
- `status` — `open | in-progress | review | delivered | done`; all start `open`.
- `context`, `objective`, `acceptanceCriteria[]`, `resources[]`, `output` — per task.
- `requestor` — **TO BE SECURED** until a named adopter (lab/educator/browser project/advocacy org)
  confirms they will use a pack.
- `verifiedNeed` — **`false`** for all tasks until a named adopter confirms use (the general need is
  real; per-pack delivery need is unproven).
- `outputLicense` — **`CC-BY-4.0`** for track packs/configs/docs; **`MIT`** for code. Never more
  permissive than a source allows; never an open license on non-commercial-sourced content.

---

## Milestone M0 — Foundation & cold-start

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| genome-browser-tracks-reviewers-000 | Name/secure License+provenance and Bioinformatics reviewers (blocking gate roles) | research | small | low | document | — | Maintainer |
| genome-browser-tracks-manifest-001 | Canonical track-pack manifest model + UCSC track-hub generator | code | medium | low | pr | — | Bioinformatics |
| genome-browser-tracks-licensegate-002 | License + data-tier gate checklist (open/aggregate/de-identified only; NC/controlled excluded) | design-spec | small | medium | document | — | License+provenance |
| genome-browser-tracks-coordgate-003 | Coordinate + assembly gate checklist (assembly recorded, 0/1-based, liftOver provenance) | design-spec | small | medium | document | — | Bioinformatics |
| genome-browser-tracks-validate-004 | Track/format + render validation tooling (validateFiles/hubCheck wrappers) | code | small | low | pr | manifest-001 | Bioinformatics |
| genome-browser-tracks-outreach-005 | Adopter outreach + accepted-source shortlist | research | small | low | document | — | Maintainer |
| genome-browser-tracks-pilot-006 | Pilot pack: open cancer driver-gene track (Open Targets CC0 × GENCODE, hg38) | data | medium | medium | dataset | reviewers-000, manifest-001, licensegate-002, coordgate-003, validate-004, outreach-005 | License+provenance, Bioinformatics |

**Acceptance criteria — key tasks**

- **manifest-001 (canonical manifest + UCSC hub generator)**
  - [ ] Manifest model documents every field from PLAN §6: `id, title, description, intendedUse,
        notMedicalAdvice, assembly{id,source}, coordinateConvention, tracks[]{name,type,file,
        source{name,release,url,recordIdScheme}, license{id,url,permitsRedistribution,nonCommercial,
        snapshotRef}, dataTier, attribution, liftOver{from,to,chain,unmappedCount}},
        provenanceCompleteness{score}, riskTier, reviewers{}, hosting{}}`.
  - [ ] Generator emits a valid UCSC track hub (`hub.txt`, `genomes.txt`, `trackDb.txt`) as a
        projection of the manifest; `hubCheck` passes on a golden fixture.
  - [ ] Track description embeds the source attribution + "not medical advice"/intended-use string.
  - [ ] Golden manifest → expected hub fixtures committed and diffed in CI; synthetic fixtures only.
  - [ ] Code MIT-licensed; `pnpm build && pnpm test && pnpm lint` green; DCO signed-off.

- **licensegate-002 (license + data-tier gate)**
  - [ ] Encodes the cancer guardrail: PASS only if data tier ∈ {open, aggregate, de-identified}
        AND license permits redistribution of derivatives (`permitsRedistribution: true`,
        `nonCommercial: false`) with a cited clause/URL; otherwise FLAG/EXCLUDE (no default-allow).
  - [ ] Excluded list explicit: COSMIC, Cancer Gene Census, OncoKB (non-commercial) and all
        controlled-access (dbGaP/EGA/individual-level) — never used as sources.
  - [ ] Records license id+URL+snapshot (committed copy + SHA-256 + Wayback) and attribution
        (incl. CC-BY sources like Reactome).
  - [ ] Includes a **re-identification check** for small-cohort variant tracks; any signal = EXCLUDE.
  - [ ] Produces a committed PASS/FLAG/EXCLUDE artifact per source recording which checks ran.

- **coordgate-003 (coordinate + assembly gate)**
  - [ ] Requires the assembly to be recorded (hg38 primary / hg19 secondary) and the source
        coordinate convention (0-based half-open vs 1-based) captured and normalised correctly.
  - [ ] Requires `validateFiles`/format validation to pass and an assembly-mismatch sanity check.
  - [ ] When liftOver is used, requires recorded chain provenance + a mapped/unmapped count report.
  - [ ] Produces a committed PASS/FAIL artifact per pack.

- **pilot-006 (pilot driver-gene pack)**
  - [ ] Source is Priority-A (Open Targets target list, **CC0**) projected onto GENCODE gene models
        on **hg38**; licensegate-002 + coordgate-003 artifacts committed and PASS.
  - [ ] Track files built (bigBed) with per-feature provenance (source, release, record ID, license);
        provenance completeness ≥ 90/100 recorded in the manifest.
  - [ ] UCSC hub emitted, `hubCheck` clean, and **render-verified** (loaded into UCSC/JBrowse;
        features appear at expected loci on hg38; evidence recorded).
  - [ ] Hosted at a loadable URL via self-serve (GitHub Pages / Zenodo); attribution + "not medical
        advice" present.
  - [ ] **Used** by a third party (loaded/referenced) with the Steward's `outcomes/<pack-id>.json`
        recorded — or hosted with the adoption blocker surfaced. `verifiedNeed` flips to `true` only
        on confirmed adoption.

**M0 Definition of Done:** both mandatory reviewer roles named (blocking, before pilot review);
canonical manifest + UCSC-hub generator + validation tooling green in CI with golden fixtures;
license+data-tier gate and coordinate+assembly gate checklists published and applied to the pilot;
one pilot pack built end-to-end, render-verified on hg38, hosted at a loadable URL, provenance
≥ 90/100, and used (evidence recorded) — or hosted with the adoption blocker surfaced; ≥ 1
adopter-outreach thread opened.

---

## Milestone M1 — Gates hardened + multi-browser parity + first adoption

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| genome-browser-tracks-jbrowse-007 | JBrowse 2 config.json projector | code | medium | low | pr | manifest-001 | Bioinformatics |
| genome-browser-tracks-igv-008 | IGV session/registry projector | code | small | low | pr | manifest-001 | Bioinformatics |
| genome-browser-tracks-snapshot-009 | License-text snapshot capture in provenance flow | code | small | low | pr | licensegate-002 | License+provenance |
| genome-browser-tracks-liftover-010 | hg19 liftOver path + unmapped report | code | small | medium | pr | coordgate-003, manifest-001 | Bioinformatics |
| genome-browser-tracks-pack-011 | Curated cancer-gene variant pack from CIViC (CC0), hg38+hg19 | data | medium | medium | dataset | jbrowse-007, igv-008, liftover-010, pilot-006 | License+provenance, Bioinformatics |
| genome-browser-tracks-pack-012 | ClinVar (public-domain) variants-in-cancer-genes pack | data | medium | medium | dataset | jbrowse-007, igv-008, pilot-006 | License+provenance, Bioinformatics |
| genome-browser-tracks-partner-013 | Secure first confirmed adopter | research | small | low | document | outreach-005 | Steward |

**Acceptance criteria — key tasks**

- **jbrowse-007 (JBrowse 2 projector)** *(pattern also applies to igv-008)*
  - [ ] Emits a valid JBrowse 2 `config.json` as a projection of the manifest (igv-008 emits a valid
        IGV session); golden manifest → expected config fixtures diffed in CI.
  - [ ] Output validated against the target spec; attribution + "not medical advice" carried through.
  - [ ] Code MIT-licensed; tests + CI green; no credentials embedded.

- **snapshot-009 (license-text snapshot)**
  - [ ] Implements committed local copy of the license text/page + SHA-256 + Wayback save URL;
        `license.snapshotRef` records committed path + hash + Wayback timestamp. Bare URL insufficient.
  - [ ] Code MIT-licensed; tests + CI green.

- **pack-011 (CIViC curated-variant pack, hg38+hg19)**
  - [ ] Source = CIViC (**CC0**); licensegate-002 PASS with `permitsRedistribution: true`; framed as
        **research evidence**, not clinical advice; "not medical advice" present.
  - [ ] Built on hg38 and lifted to hg19 with recorded chain provenance + unmapped report.
  - [ ] All three browser targets emitted (UCSC + JBrowse2 + IGV) and render-verified.
  - [ ] Per-feature provenance to CIViC record IDs; provenance completeness ≥ 90/100.
  - [ ] Hosted at a loadable URL; adoption evidence recorded or blocker surfaced.

- **partner-013 (first confirmed adopter)**
  - [ ] A named lab/educator/browser project/advocacy org confirms they will use/adopt a pack.
  - [ ] Adoption mechanism documented (hub registry / lesson / repo reference).
  - [ ] Tasks for that adopter updated to `verifiedNeed: true` with `requestor` set.

**M1 Definition of Done:** JBrowse2 + IGV projectors green with golden fixtures; license-snapshot
capture wired in; hg19 liftOver path with unmapped report; ≥ 3 packs published (each emitting all
three targets, render-verified); ≥ 1 external adoption event recorded; ≥ 1 confirmed adopter.

---

## Milestone M2 — Source-catalog scale & multi-source packs

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| genome-browser-tracks-catalog-014 | Triage the source catalog → approved source shortlist | research | medium | medium | document | licensegate-002 | License+provenance |
| genome-browser-tracks-pack-015 | Recurrent cancer hotspots pack (gated source, hg38+hg19) | data | medium | medium | dataset | catalog-014, liftover-010 | License+provenance, Bioinformatics |
| genome-browser-tracks-pack-016 | Reactome (CC-BY) cancer-pathway membership track | data | medium | low | dataset | catalog-014, jbrowse-007 | License+provenance, Bioinformatics |
| genome-browser-tracks-pack-017 | Multi-source driver pack: genes + curated variants + pathway | data | large | medium | dataset | pack-011, pack-016 | License+provenance, Bioinformatics |
| genome-browser-tracks-effort-018 | Per-pack build-effort instrumentation + outcome ledger | code | small | low | pr | manifest-001 | Maintainer |

**Acceptance criteria — key tasks**

- **catalog-014 (source-catalog triage → shortlist)**
  - [ ] Every catalog source assigned a disposition: `APPROVED` (open tier + license verified +
        `permitsRedistribution: true` with cited clause/URL), `VERIFY-LATER`, or `EXCLUDED`
        (non-commercial / controlled-access / non-redistributable).
  - [ ] COSMIC/CGC/OncoKB recorded as EXCLUDED (non-commercial); controlled-access EXCLUDED.
  - [ ] Priority-B sources (GDC-open/cBioPortal/hotspots/DepMap) resolved per-source with cited terms;
        unresolved stay VERIFY-LATER (not promoted to packs).
  - [ ] Output = committed shortlist (≥ 6 APPROVED sources) with one gate artifact per decision.

- **pack-016 (Reactome pathway track)**
  - [ ] Source = Reactome (**CC-BY 4.0**); attribution captured in manifest and embedded in track.
  - [ ] Pathway-membership features mapped to gene coordinates on hg38; coordgate PASS.
  - [ ] All three targets emitted + render-verified; provenance completeness ≥ 90/100.

- **pack-017 (multi-source driver pack)**
  - [ ] Combines gene models + curated variants (CC0) + pathway membership (CC-BY) in one pack with
        each track carrying its own source license + attribution; no NC/controlled source present.
  - [ ] hg38 and hg19; all three browser targets; render-verified; "not medical advice" present.

- **effort-018 (effort instrumentation)**
  - [ ] Records per-pack build minutes + human-review cycles into the outcome ledger so M2 effort
        reduction vs the M0/M1 baseline median is measurable.

**M2 Definition of Done:** ≥ 8 sources triaged with committed gate artifacts; ≥ 6 packs
published+adopted cumulatively; ≥ 1 multi-source pack on hg38+hg19; measurable median per-pack
effort reduction vs the recorded M0/M1 baseline.

---

## Milestone M3 — Adoption, education & sustainability

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| genome-browser-tracks-edu-019 | Education-only "where are the cancer genes?" pack + tutorial | writing | medium | high | document | pack-017, partner-013 | Oncologist, Patient-advocate, Bioinformatics |
| genome-browser-tracks-registry-020 | Submit a pack to a public hub/config registry | research | small | low | document | pack-011, pack-016 | Steward |
| genome-browser-tracks-reuse-021 | Track and verify external adoption events | research | small | low | document | pack-011, pack-012, pack-016 | Steward |
| genome-browser-tracks-refresh-022 | Source-release refresh + re-validation process | maintenance | small | low | document | validate-004 | Maintainer |

**Acceptance criteria — key tasks**

- **edu-019 (education-only patient-facing pack + tutorial)** — `riskTier: high`
  - [ ] Education-only framing; prominent "not medical advice"; no per-individual actionability,
        diagnosis, prognosis, or treatment guidance.
  - [ ] **Oncologist sign-off AND patient-advocate sign-off** recorded before publish (both required).
  - [ ] Every assertion traces to an open source record; bioinformatics reviewer confirms accuracy
        and correct loci on the declared assembly.
  - [ ] Built only from APPROVED open sources; no NC/controlled content.

- **reuse-021 (adoption tracking)**
  - [ ] ≥ 3 verifiable external adoption events recorded in the outcome ledger (third-party load/
        registry acceptance/citation/fork/lesson use), each with externally verifiable evidence.

- **refresh-022 (refresh process)**
  - [ ] Documented process to detect a source's new release and re-validate/refresh affected packs.
  - [ ] Stale packs become `maintenance` tasks; render check + validators detect breakage.

**M3 Definition of Done:** ≥ 3 verifiable adoption events; one education-only patient-facing pack
shipped with oncologist + patient-advocate sign-off and "not medical advice"; ≥ 1 pack listed in a
public registry; documented source-release refresh process + named steward.

---

## Source-catalog (candidate backlog → packs)

These tasks operationalize the accepted-source matrix in PLAN §7. `catalog-014` triages sources into
an approved shortlist; the rows below are **concrete example per-source pack tasks** drawn from real
accepted sources. All are `type: data`, `deliverable: dataset`, `lane: donated`, `verifiedNeed: false`,
`requestor: TO BE SECURED`. Each still requires its own committed `licensegate-002` + `coordgate-003`
artifacts before work proceeds — listing here does not pre-approve a source.

**Priority:** A = clearly-redistributable (CC0 / public-domain / CC-BY) + open tier. B = verify
per-source (GDC-open/cBioPortal/hotspots/DepMap). C = EXCLUDED (non-commercial / controlled-access).

| Task ID | Source | Content | Priority | Risk | License gate | Depends on |
| --- | --- | --- | --- | --- | --- | --- |
| ds-gencode-001 | GENCODE | Gene/transcript models | A | low | cleared (confirm at gate) | catalog-014 |
| ds-refseq-002 | RefSeq | Gene models | A | low | cleared (confirm at gate) | catalog-014 |
| ds-civic-003 | CIViC | Curated evidence variants (CC0) | A | medium | cleared (confirm at gate) | catalog-014 |
| ds-clinvar-004 | ClinVar | Variant–condition (public domain) | A | medium | cleared (confirm at gate) | catalog-014 |
| ds-opentargets-005 | Open Targets | Driver/target evidence (CC0) | A | low | cleared (confirm at gate) | catalog-014 |
| ds-reactome-006 | Reactome | Cancer-pathway membership (CC-BY) | A | low | cleared (confirm at gate) | catalog-014 |
| ds-gnomad-007 | gnomAD | Population allele frequencies (context) | A | low | verify (confirm at gate) | catalog-014 |
| ds-pfam-008 | Pfam/InterPro | Protein domains (CC0) | A | low | cleared (confirm at gate) | catalog-014 |
| ds-dbsnp-009 | dbSNP | Variant IDs (public domain) | A | low | cleared (confirm at gate) | catalog-014 |
| ds-gdcopen-010 | GDC/TCGA open somatic (MAF) | Aggregate somatic mutations (open tier) | B | medium | Verify per study → gate | catalog-014 |
| ds-cbioportal-011 | cBioPortal study data | Aggregated study results | B | medium | Verify per study → gate | catalog-014 |
| ds-hotspots-012 | Cancer hotspots | Recurrent hotspot positions | B | medium | Verify terms → gate | catalog-014 |
| ds-depmap-013 | DepMap | Dependencies/cell-line data | B | medium | Verify terms → gate | catalog-014 |

**Excluded — no pack task created**

| Source | Reason |
| --- | --- |
| COSMIC / Cancer Gene Census | Non-commercial academic license (cannot redistribute as open) |
| OncoKB | Non-commercial / restricted redistribution |
| Cancer Genome Interpreter | Non-commercial suspected — excluded pending license verification |
| dbGaP / EGA / individual-level biobanks | Controlled-access; out of scope (need authorised access + IRB) |

---

## Backlog / future

| ID | Title | Type | Size | Risk | Deliverable | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| genome-browser-tracks-t2t-023 | T2T-CHM13 assembly support | code | medium | medium | pr | If/when T2T enters scope |
| genome-browser-tracks-dash-024 | Outcome dashboard (adoption + render evidence) | code | medium | low | pr | Reads the outcome ledger |
| genome-browser-tracks-i18n-025 | Translate an education-only pack's docs (reviewed) | writing | small | high | translation | Widens reach; needs language + oncologist/advocate review. type:writing (translation is a deliverable, not a type per Hee-Lee Oss schema) |
| genome-browser-tracks-domains-026 | Protein-domain track from Pfam/InterPro (CC0) | data | medium | low | dataset | Useful overlay for variant interpretation (research) |

---

## Example task JSON

```json
{
  "id": "genome-browser-tracks-licensegate-002",
  "title": "License + data-tier gate checklist (open/aggregate/de-identified only; NC/controlled excluded)",
  "project": "genome-browser-tracks",
  "type": "design-spec",
  "lane": "donated",
  "priority": "high",
  "domain": ["cancer-research", "genomics", "open-science", "licensing"],
  "riskTier": "medium",
  "urgent": false,
  "deliverable": "document",
  "tokenEstimate": "small",
  "status": "open",
  "context": "genome-browser-tracks builds cancer feature tracks for genome browsers from open annotations. The binding cancer guardrail allows ONLY open-access / aggregate / de-identified data: controlled-access sources (dbGaP, EGA, individual-level biobanks) and non-commercial sources (COSMIC, Cancer Gene Census, OncoKB) are out of scope because we cannot redistribute derivative tracks under an open license. Before any track is built, each source must pass a blocking license + data-tier gate. This task produces that gate as a committed, reviewable checklist artifact.",
  "objective": "Create the blocking license + data-tier gate checklist that every source must pass before a track pack is built, encoding the cancer guardrails and producing a committed PASS/FLAG/EXCLUDE artifact per source.",
  "acceptanceCriteria": [
    "PASS only if data tier is one of {open, aggregate, de-identified} AND the license permits redistribution of derivative works (permitsRedistribution: true, nonCommercial: false) evidenced by a cited clause/URL; missing or unparseable evidence = FLAG/EXCLUDE (no default-allow).",
    "Explicitly excludes COSMIC, Cancer Gene Census, and OncoKB (non-commercial) and all controlled-access / individual-level sources (dbGaP, EGA, biobanks).",
    "Records license id + URL + snapshot reference (committed copy + SHA-256 hash + Wayback save URL) and any required attribution (e.g. CC-BY sources such as Reactome).",
    "Includes a re-identification check for small-cohort variant tracks; any re-identification signal forces EXCLUDE.",
    "Produces a committed, reviewable PASS/FLAG/EXCLUDE artifact per source recording which checks ran and what fired, signed off by the License+provenance reviewer.",
    "Document output licensed CC-BY-4.0; commit is DCO signed-off."
  ],
  "resources": [
    "C:\\code\\hee-lee-oss\\planning\\projects\\genome-browser-tracks\\PLAN.md",
    "C:\\code\\hee-lee-oss\\planning\\ROADMAP.md",
    "C:\\code\\hee-lee-oss\\docs\\good-deed-definition.md",
    "SPDX license list; CC0 / CC-BY 4.0 / public-domain terms"
  ],
  "output": "A committed license + data-tier gate checklist plus the per-source PASS/FLAG/EXCLUDE artifact template, ready for use by every per-source pack task.",
  "requestor": "TO BE SECURED",
  "verifiedNeed": false,
  "outputLicense": "CC-BY-4.0"
}
```

---

## Generated task index

> Auto-generated by the Hee-Lee Oss task-decomposition pass on 2026-06-29. Every TASKS.md backlog row now has a corresponding schema-valid `tasks/<id>.json`. Run `node validate-tasks.mjs tasks/` to verify all 40 files pass.

| File | Title | Milestone / Section | Type | Deliverable |
| --- | --- | --- | --- | --- |
| tasks/genome-browser-tracks-reviewers-000.json | Name/secure License+provenance and Bioinformatics reviewers | M0 | research | document |
| tasks/genome-browser-tracks-manifest-001.json | Canonical track-pack manifest model + UCSC track-hub generator | M0 | code | pr |
| tasks/genome-browser-tracks-licensegate-002.json | License + data-tier gate checklist | M0 (seed) | design-spec | document |
| tasks/genome-browser-tracks-coordgate-003.json | Coordinate + assembly gate checklist | M0 | design-spec | document |
| tasks/genome-browser-tracks-validate-004.json | Track/format + render validation tooling | M0 | code | pr |
| tasks/genome-browser-tracks-outreach-005.json | Adopter outreach + accepted-source shortlist | M0 | research | document |
| tasks/genome-browser-tracks-pilot-006.json | Pilot pack: open cancer driver-gene track | M0 | data | dataset |
| tasks/genome-browser-tracks-jbrowse-007.json | JBrowse 2 config.json projector | M1 | code | pr |
| tasks/genome-browser-tracks-igv-008.json | IGV session/registry projector | M1 | code | pr |
| tasks/genome-browser-tracks-snapshot-009.json | License-text snapshot capture in provenance flow | M1 | code | pr |
| tasks/genome-browser-tracks-liftover-010.json | hg19 liftOver path + unmapped report | M1 | code | pr |
| tasks/genome-browser-tracks-pack-011.json | Curated cancer-gene variant pack from CIViC (CC0), hg38+hg19 | M1 | data | dataset |
| tasks/genome-browser-tracks-pack-012.json | ClinVar (public-domain) variants-in-cancer-genes pack | M1 | data | dataset |
| tasks/genome-browser-tracks-partner-013.json | Secure first confirmed adopter | M1 | research | document |
| tasks/genome-browser-tracks-catalog-014.json | Triage the source catalog into an approved source shortlist | M2 | research | document |
| tasks/genome-browser-tracks-pack-015.json | Recurrent cancer hotspots pack (gated source, hg38+hg19) | M2 | data | dataset |
| tasks/genome-browser-tracks-pack-016.json | Reactome (CC-BY) cancer-pathway membership track | M2 | data | dataset |
| tasks/genome-browser-tracks-pack-017.json | Multi-source driver pack: genes + curated variants + pathway | M2 | data | dataset |
| tasks/genome-browser-tracks-effort-018.json | Per-pack build-effort instrumentation + outcome ledger | M2 | code | pr |
| tasks/genome-browser-tracks-edu-019.json | Education-only 'where are the cancer genes?' pack + tutorial | M3 | writing | document |
| tasks/genome-browser-tracks-registry-020.json | Submit a pack to a public hub/config registry | M3 | research | document |
| tasks/genome-browser-tracks-reuse-021.json | Track and verify external adoption events | M3 | research | document |
| tasks/genome-browser-tracks-refresh-022.json | Source-release refresh + re-validation process | M3 | maintenance | document |
| tasks/ds-gencode-001.json | GENCODE gene/transcript model track pack | Source catalog (A) | data | dataset |
| tasks/ds-refseq-002.json | RefSeq gene model track pack | Source catalog (A) | data | dataset |
| tasks/ds-civic-003.json | CIViC curated cancer-variant evidence track pack | Source catalog (A) | data | dataset |
| tasks/ds-clinvar-004.json | ClinVar variant-condition aggregate track pack | Source catalog (A) | data | dataset |
| tasks/ds-opentargets-005.json | Open Targets driver/target evidence track pack | Source catalog (A) | data | dataset |
| tasks/ds-reactome-006.json | Reactome cancer-pathway membership track pack | Source catalog (A) | data | dataset |
| tasks/ds-gnomad-007.json | gnomAD population allele frequency context track pack | Source catalog (A) | data | dataset |
| tasks/ds-pfam-008.json | Pfam/InterPro protein domain track pack | Source catalog (A) | data | dataset |
| tasks/ds-dbsnp-009.json | dbSNP variant identifier context track pack | Source catalog (A) | data | dataset |
| tasks/ds-gdcopen-010.json | GDC/TCGA open-access somatic mutation aggregate track pack | Source catalog (B) | data | dataset |
| tasks/ds-cbioportal-011.json | cBioPortal aggregated study results track pack | Source catalog (B) | data | dataset |
| tasks/ds-hotspots-012.json | Cancer recurrent hotspots track pack | Source catalog (B) | data | dataset |
| tasks/ds-depmap-013.json | DepMap cancer dependency/cell-line context track pack | Source catalog (B) | data | dataset |
| tasks/genome-browser-tracks-t2t-023.json | T2T-CHM13 assembly support | Backlog | code | pr |
| tasks/genome-browser-tracks-dash-024.json | Outcome dashboard (adoption + render evidence) | Backlog | code | pr |
| tasks/genome-browser-tracks-i18n-025.json | Translate an education-only pack's docs | Backlog | writing | translation |
| tasks/genome-browser-tracks-domains-026.json | Protein-domain track from Pfam/InterPro (CC0) | Backlog | data | dataset |

**Notes:**
- All 40 tasks carry `lane: donated`, `status: open`, `verifiedNeed: false`, `requestor: TO BE SECURED` (no partner secured yet — consistent with PLAN.md §2).
- `i18n-025` uses `type: writing` + `deliverable: translation` — `translation` is a deliverable not a type in the Hee-Lee Oss task schema.
- Source-catalog Priority-B tasks (ds-gdcopen-010 through ds-depmap-013) require catalog-014 gate confirmation before any pack is built; if the gate FAILS they become EXCLUDED records with no track produced.
- `edu-019` is `riskTier: high` and requires oncologist + patient-advocate sign-off before publication per PLAN.md.
