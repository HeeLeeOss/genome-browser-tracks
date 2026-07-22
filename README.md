# genome-browser-tracks

> Cancer genomics knowledge is published across dozens of open resources — gene models (GENCODE, RefSeq, Ensembl), curated variant interpretations (CIViC, ClinVar), driver-gene and target evidence (Open  ·  **Risk tier:** low  ·  **Status:** planning

Cancer genomics knowledge is published across dozens of open resources — gene models (GENCODE, RefSeq, Ensembl), curated variant interpretations (CIViC, ClinVar), driver-gene and target evidence (Open Targets), recurrent mutation hotspots, and pathway membership (Reactome) — but a researcher, bioinformatician, educator, or research-literate patient advocate who simply wants to *see* these cancer features laid out along the genome in a browser must hand-assemble them: find the source, check its license, normalise coordinates to the right assembly, convert to a track format, write a browser config, and validate that it renders. This is repetitive, error-prone (coordinate off-by-one and wrong-assembly mistakes are the classic silent failures of genomics), and most people never do it — so the open knowledge stays locked in flat files and web portals instead of being explorable in context.

**Definition of shipped:** (committed artifact); provenance complete (score ≥ 90/100); coordinate/assembly verified; format- and render-verified in ≥1 target browser on the declared assembly; published at a loadable URL with attribution and "not medical advice" notice; **and adopted/used by a beneficiary**

This is an **Hee-Lee Oss** good-deed project. Contributors pull a task, do it with their own coding agent, and open a PR. Platform: https://github.com/jdev1977/hee-lee-oss

## Plan
- [PLAN.md](./PLAN.md) — robust enterprise plan (vision, architecture, roadmap, risks; includes an applied-improvements appendix + review sign-off)
- [TASKS.md](./TASKS.md) — schema-mapped task backlog
- [tasks/](./tasks/) — ready-to-pull task JSON(s)

## Contribute
```bash
hee-lee-oss browse
hee-lee-oss next --repo Hee-Lee-Oss-Projects/genome-browser-tracks --no-fork
```

## Licensing & review
- Open license (see PLAN.md).
- Risk tier **low** — deeds are *delivered, not merged*; standard review applies.

> Planning stage; no adopting partner secured yet (`verifiedNeed: false` on delivery-dependent tasks).
