# ewing-fusion-detection-benchmark — Project Plan

> **For the families.** Ewing Sarcoma is a rare bone and soft-tissue cancer that strikes
> children, teenagers, and young adults. When a tumour is suspected, the single most
> consequential laboratory question is often: *is the EWSR1 fusion there, and which one?* The
> answer routes diagnosis, prognosis, and clinical-trial eligibility. The software that calls
> those fusions from sequencing data is not, today, benchmarked openly or reproducibly for this
> specific disease. This project builds that missing, honest, public yardstick — carefully,
> conservatively, and with no shortcuts that could let a wrong number reach a clinic. We treat
> every claim as if a family were reading it, because one day one will.

---

**Status:** Draft · **Version:** 0.1.0 · **Last updated:** 2026-06-28 ·
**Owner:** TBD (maintainer) · **Lane:** donated · **Risk tier:** medium (with high-risk
carve-outs for any patient-facing text) · **Domain:** cancer-research, bioinformatics, Ewing
Sarcoma · **Registry slug:** `ewing-fusion-detection-benchmark`

---

## 1. Executive summary

Ewing Sarcoma (ES) is defined at the molecular level by a recurrent gene fusion in which the
*EWSR1* gene (chromosome 22) joins an ETS-family transcription-factor gene — most commonly
*FLI1* (the classic t(11;22) EWSR1-FLI1, ~85% of cases), then *ERG* (~10%), with rarer
*ETV1*, *ETV4 (E1AF)*, and *FEV* partners. Detecting that fusion — and distinguishing the
true ES-defining fusion from the *EWSR1*-family fusions that define **different** tumours
(e.g. EWSR1-WT1 in desmoplastic small round cell tumour, EWSR1-NR4A3 in extraskeletal myxoid
chondrosarcoma, EWSR1-ATF1/CREB1 in clear-cell sarcoma / angiomatoid fibrous histiocytoma) —
is a routine but error-prone computational task. A growing zoo of RNA-seq and DNA-seq fusion
callers (STAR-Fusion, Arriba, FusionCatcher, JAFFA, deFuse, EricScript, pizzly, STAR-SEQR;
DNA structural-variant callers Manta, GRIDSS, SvABA) each make different trade-offs, yet there
is **no open, reproducible, ES-specific benchmark** that tells a researcher or a diagnostic lab
which caller, with which settings, reliably recovers EWSR1 fusions and correctly resolves the
partner — including the hard cases (low tumour purity, low read depth, FFPE-degraded RNA,
alternative breakpoints, and the look-alike non-ES *EWSR1* fusions).

This project delivers exactly that benchmark, built **only on open-access, aggregate, simulated,
or de-identified cell-line data** — never on controlled-access or identifiable patient data
(see §7, which leads with the binding cancer guardrails). The primary deliverables are: (1) an
open, license-clean **truth set** combining realistically simulated EWSR1-fusion RNA-seq reads
(with ground-truth breakpoints) and publicly available **cell-line** RNA-seq with
literature-documented fusions; (2) a **containerised, reproducible benchmarking harness**
(Nextflow/Snakemake + Docker/Singularity) that runs each caller identically and scores
precision, recall, breakpoint accuracy, partner-resolution accuracy, runtime, and memory; (3)
an **open results report and leaderboard** with full provenance on every number; and (4)
**datasheets and model/tool cards** documenting each caller's behaviour, failure modes, and the
exact conditions under which it should and should not be trusted.

The output is a research artefact for the ES research and methods-development community and for
diagnostic-lab method-validation teams. It is **explicitly not a diagnostic device, not clinical
guidance, and not medical advice** (§3). Success is measured by adoption and reuse by real
researchers and by the benchmark's reproducibility — not by repository stars.

---

## 2. Problem & beneficiaries

### The problem
- **A real, recurring computational task with no honest open yardstick.** Labs and methods
  developers must choose a fusion caller and parameters. Published comparisons exist for fusion
  detection *in general*, but they are not ES-specific, are often not reproducible (no pinned
  versions, no containers, no released truth data), and rarely test the cases that actually
  matter for ES: resolving the *correct ETS partner*, distinguishing ES fusions from non-ES
  *EWSR1* look-alikes, and degrading gracefully on low-input / FFPE-like data.
- **Silent failure modes have real cost.** A missed EWSR1-FLI1 call can delay a correct
  diagnosis; a mis-resolved partner can mis-assign a tumour type; an over-confident caller can
  generate false fusions that waste scarce research and clinical-confirmation effort. Today a
  researcher cannot easily find out, from open evidence, which tool fails where.
- **Reproducibility crisis in fusion benchmarking.** Many comparisons cannot be re-run because
  the inputs are private or the environment is unspecified. We make the *method itself* the
  public good: anyone can re-run our benchmark bit-for-bit and extend it.

### Who is helped (beneficiaries)
1. **ES methods developers & computational biologists** — get a reusable, extensible benchmark
   to evaluate new callers and tune existing ones for ES.
2. **Diagnostic / molecular-pathology method-validation teams** — get open, citable evidence to
   inform (not replace) their own validation of fusion-calling pipelines. (We provide evidence;
   they own clinical validation.)
3. **Academic ES labs with open RNA-seq** — get a tested, containerised pipeline they can adopt.
4. **The ES patient & advocacy community (indirect)** — benefits when better-validated tooling
   shortens the path to accurate molecular diagnosis. Any *direct* patient-facing material is
   out of this project's core scope and gated at high risk (§3, §8).

### Verified need / partner
- **Verified need: TO BE SECURED.** The need is strongly evidenced by the published literature
  and by the absence of an ES-specific open benchmark, but **no partner organisation has yet
  confirmed they will adopt or steward the output.** All Task JSON in TASKS.md therefore set
  `verifiedNeed: false` until a partner letter of intent is on file.
- **Target partners (to approach):** academic sarcoma genomics groups; pediatric-oncology
  bioinformatics cores; ES foundations and advocacy orgs (e.g. sarcoma/childhood-cancer
  research charities) for steward/advisory roles; an open-science methods venue for peer review.
- **Steward (last-mile owner): TO BE SECURED** — see §11.

---

## 3. Goals and non-goals

### Goals
- **G1.** Publish an open, license-clean **EWSR1-fusion truth set** (simulated + cell-line)
  covering the main partners (FLI1, ERG, ETV1, ETV4, FEV) and the key non-ES *EWSR1* look-alikes
  as negative/confounder controls, with ground-truth breakpoints for all simulated cases.
- **G2.** Publish a **fully reproducible, containerised benchmark harness** that runs ≥6 fusion
  callers identically and is re-runnable by a third party with one command and pinned versions.
- **G3.** Publish an **honest results report + leaderboard** scoring precision, recall, F1,
  breakpoint accuracy, partner-resolution accuracy, runtime, and peak memory — stratified by
  difficulty (depth, purity, FFPE-like degradation, partner rarity), with provenance on every
  number and explicit confidence intervals / known limitations.
- **G4.** Publish **per-caller datasheets / tool cards** documenting configuration, failure
  modes, and "trust boundaries" (when not to rely on a result).
- **G5.** Make the entire artefact **reproducible and reusable** — anyone can re-run, extend
  with a new caller, or add a new difficulty axis.

### Non-goals (binding — see also §5 Scope)
- **NG1. Not a diagnostic device and not clinical guidance.** Nothing here may be used to make,
  confirm, or rule out a diagnosis. The benchmark informs *research and method validation*, full
  stop. This statement appears verbatim in every released artefact.
- **NG2. No medical advice.** No treatment, prognosis, or patient-management content. Any
  educational patient-facing text is out of core scope and, if ever produced, is gated behind
  oncologist **and** patient-advocate review at **high** risk tier, sourced, and labelled "not
  medical advice."
- **NG3. No controlled-access or identifiable patient data.** dbGaP, EGA, TARGET/TCGA
  controlled tiers, individual-level biobanks, and any re-identifiable data are out of scope
  (§7). This is non-negotiable.
- **NG4. We do not rank tools to declare a single "winner" for clinical use.** We report
  evidence and trade-offs; clinical selection is the lab's regulated responsibility.
- **NG5. Not a novel fusion-caller.** We benchmark existing tools; we do not ship a new caller
  in this project (a separate future project could).
- **NG6. No re-distribution of non-redistributable inputs.** Where a license forbids
  redistribution (e.g. some databases), we ship *recipes/accessions*, not the data.

---

## 4. Success metrics (outcomes)

Outcome-based and beneficiary-centric. Vanity metrics (stars, downloads alone) are explicitly
not success.

| # | Outcome | Baseline (today) | Target | How measured |
|---|---|---|---|---|
| O1 | An external party reproduces the full benchmark from scratch | 0 (does not exist) | ≥1 independent successful re-run by M3; ≥3 by M5 | Re-run reports / issues confirming bitwise-or-tolerance match |
| O2 | Callers benchmarked under identical conditions | 0 ES-specific | ≥6 callers in M2; ≥8 by M5 | Harness run logs + leaderboard |
| O3 | Truth-set cases with ground-truth breakpoints | 0 open ES-specific | ≥200 simulated cases across difficulty strata; ≥10 cell-line positives | Dataset manifest |
| O4 | Adoption by a real ES/methods group | none | ≥1 group adopts harness or cites benchmark by M5 | Partner confirmation, citations, forks-with-activity |
| O5 | Provenance completeness | n/a | 100% of reported numbers trace to a pinned input + config hash | Provenance audit (CI-enforced) |
| O6 | Correct non-ES *EWSR1* look-alike handling documented | none | Confounder behaviour reported for 100% of benchmarked callers | Leaderboard "confounder" panel |
| O7 | Expert sign-off on interpretation claims | n/a | 100% of interpretive/biological claims reviewed by domain expert before release | Review log |

**Anti-metric (guardrail):** we will *not* optimise for a flattering headline F1. If results are
mediocre or callers fail on hard cases, we publish that — honesty is the deliverable.

---

## 5. Scope

### In scope
- Simulated EWSR1-fusion RNA-seq read generation with ground-truth breakpoints (multiple
  partners, breakpoint variants, depth/purity/degradation strata).
- Curation of **open** / **cell-line** RNA-seq with literature-documented EWSR1 fusions as
  real-data positives, and non-ES *EWSR1*-fusion or fusion-negative samples as confounders.
- A containerised, version-pinned benchmark harness (workflow manager + per-caller containers).
- Scoring library (precision/recall/F1, breakpoint distance, partner-resolution accuracy,
  resource usage) with a defined, documented "match" definition.
- Results report, leaderboard, per-caller datasheets/tool cards, and dataset datasheets.
- DNA-based structural-variant fusion detection as a **secondary** track (Manta/GRIDSS/SvABA)
  if open WGS/WES cell-line data with documented fusions is available — otherwise deferred.

### Out of scope (explicit)
- **Any controlled-access or identifiable patient data** (dbGaP/EGA/TARGET controlled/biobanks).
- **Clinical validation, diagnostic use, regulatory submission** (CLIA/CAP/IVDR) — we provide
  research evidence only.
- **Medical advice / patient guidance / prognosis** (NG2).
- **Shipping a new fusion caller** (NG5).
- **Wet-lab work**, sequencing, or generating new biological samples.
- **Pan-cancer fusion benchmarking** — we are ES/EWSR1-focused by design (a strength, not a gap).
- **Re-distributing data whose license forbids it** — recipes/accessions only (§7).

---

## 6. Solution approach & architecture

A reproducible data + pipeline + scoring + reporting system. Open-source (MIT for code; CC-BY-4.0
for docs/datasheets; data under the most permissive license compatible with each input — see §7).

### 6.1 Components
1. **`sim/` — Simulated truth generator.** Builds chimeric EWSR1-fusion transcripts from an open
   reference (GENCODE/Ensembl) for each partner and breakpoint variant, then generates reads
   with an established read simulator (e.g. `polyester` / `ART` for short-read Illumina-like;
   optionally `BadRead`/`pbsim` for long-read later). Emits paired FASTQ **plus** a
   machine-readable ground-truth manifest (fusion, gene partners, exact breakpoint coordinates,
   supporting-read counts, spike-in fraction).
2. **`realdata/` — Open/cell-line curation.** Records **accessions** (GEO/SRA open series, and
   cell-line RNA-seq such as A673, SK-N-MC, TC71, EW8, CHLA-10 where openly available) and
   literature-asserted fusion status with citations. Ships a *fetch recipe*, never bulk
   re-hosted controlled data. Each entry carries a license + provenance record.
3. **`callers/` — Pinned caller containers.** One Docker/Singularity image per caller at a fixed
   version with a fixed reference build, wrapped by a uniform adapter interface
   (`run(caller, sample, refs, params) -> normalized_calls.tsv`).
4. **`harness/` — Workflow.** Nextflow (nf-core conventions) or Snakemake DAG: stage inputs →
   run each caller on each sample → normalise outputs to a common fusion-call schema → score →
   aggregate. Every step content-hashes its inputs/config for provenance.
5. **`score/` — Scoring library (TypeScript core + Python adapters where caller I/O demands).**
   Defines the canonical fusion-call record, the **match rule** (gene-pair match; breakpoint
   within a tolerance window; partner correctness), and computes metrics with confidence
   intervals (bootstrap) per difficulty stratum.
6. **`report/` — Reporting + leaderboard.** Generates the static results site/report from scored
   outputs; every cell links to the input hash, caller version, and config that produced it.
7. **`provenance/` — Ledger.** A signed, append-only manifest binding each reported number to
   (input hash, container digest, params, harness commit). CI fails if any number lacks lineage.

> **Agent-neutral / Hee-Lee Oss-fit note:** this project is delivered as a standalone open repo of
> data + workflow + docs. It does not modify the Hee-Lee Oss core. Caller-specific logic lives behind
> the uniform `callers/` adapter (mirroring Hee-Lee Oss's `adapters/` rule), keeping the harness
> vendor-neutral.

### 6.2 Canonical fusion-call data model (normalised)
```
FusionCall {
  sampleId: string
  geneA: string            // 5' gene (HGNC symbol + Ensembl ID)
  geneB: string            // 3' gene
  breakpointA: {chrom, pos, strand}   // genomic, GRCh38
  breakpointB: {chrom, pos, strand}
  junctionReadSupport: int
  spanningFragmentSupport: int
  callerScore: number      // caller-native confidence (documented per caller)
  caller: string
  callerVersion: string
  refBuild: "GRCh38"       // single pinned build for all callers
}
```
Ground-truth records share the same shape (minus `callerScore`), enabling a clean join.

### 6.3 Key decisions (locked)
- **D1. Reference build: GRCh38 only** across all callers (no GRCh37 to avoid liftover noise);
  one pinned GENCODE annotation release for all.
- **D2. Simulation is the primary truth source; cell-line/open real data is the secondary,
  orthogonally-confirmed source.** Real-data "truth" is only as good as the literature evidence,
  so it is labelled by evidence strength, never treated as gold.
- **D3. Match rule is published and versioned** (gene-pair + breakpoint tolerance, default
  ±10 bp configurable; partner-resolution scored separately) so results are interpretable and
  comparable; the tolerance is a reported parameter, not a hidden choice.
- **D4. Everything pinned + containerised.** No "latest" tags; image digests recorded.
- **D5. Difficulty stratification is first-class:** read depth, tumour-purity (spike-in
  fraction), FFPE-like degradation, partner rarity, and alternative breakpoints are explicit
  axes, because that is where clinically-relevant failures hide.
- **D6. Confounder panel is mandatory:** non-ES *EWSR1* fusions and fusion-negative small-round-
  blue-cell-tumour-like samples are included so we measure *specificity*, not just sensitivity.
- **D7. Honest reporting:** confidence intervals, n per stratum, and known limitations are
  required fields in the report; no point estimate ships without them.

### 6.4 Workflow / tooling
- **Languages:** TypeScript/ESM for the scoring core and tooling (per Hee-Lee Oss conventions); Python
  permitted inside caller adapters and the simulator where the ecosystem requires it.
- **Workflow manager:** Nextflow (nf-core style) preferred for portability; Snakemake fallback.
- **Containers:** Docker for dev, Singularity/Apptainer for HPC.
- **Packaging:** pnpm workspace for TS packages; conda/pip lockfiles inside caller images.
- **CI:** GitHub Actions runs a *tiny* smoke benchmark (1 simulated sample, 1 caller) on every
  PR; the full benchmark runs on tagged releases or manual dispatch.

---

## 7. Data, licensing & compliance

> ### 7.0 BINDING CANCER GUARDRAILS (read first — these override everything below)
> - **Open-access / aggregate / de-identified data ONLY.** Controlled-access sources — **dbGaP,
>   EGA, TARGET/TCGA controlled tiers, individual-level biobanks** — and **ANY identifiable
>   patient data** are **OUT OF SCOPE**. They require authorized access + IRB/DUA, which is *not*
>   something donated AI tasks can or may obtain. Any task that would touch such data is refused
>   and flagged.
> - **Per-source license verification is mandatory.** TCGA/GDC **open-tier** and **GEO** are open
>   and reusable; **COSMIC** and **OncoKB** are **non-commercial / custom-license — FLAGGED**: we
>   reference identifiers/links but do **not** redistribute their data, and we do not depend on
>   them for any redistributable artefact. Every source's license is recorded before use.
> - **No medical advice.** Any patient-facing content is **education only**, sourced, labelled
>   "**not medical advice**", and gated behind **oncologist + patient-advocate review (riskTier
>   high)**. Core project ships none.
> - **Provenance on every assertion.** Every biological claim, every benchmark number, and every
>   dataset entry carries a citation or an input/config hash. No unsourced assertions.
> - **When in doubt, stop and surface.** Ambiguous license or possible re-identifiability →
>   halt and escalate to maintainer + expert reviewer, per CLAUDE.md.

### 7.1 Data sources and their handling
| Source | Type | License / access | Use here | Redistribute? |
|---|---|---|---|---|
| GENCODE / Ensembl reference + annotation | Reference genome/transcriptome | Open (Ensembl/GENCODE terms; reference is freely usable) | Build simulated fusions; align/annotate | Recipe + version pin (no re-host of large refs) |
| Simulated reads (generated by us) | Synthetic FASTQ + truth | We license **CC-BY-4.0 / CC0** (our creation; no patient data) | Primary truth set | **Yes** — fully redistributable |
| GEO open RNA-seq series (ES cell lines / open studies) | Real RNA-seq | Open per GEO; verify each series' stated terms | Real-data positives/confounders | **Accessions + fetch recipe only** |
| SRA (linked to open GEO) | Raw reads | Open per linked study | Fetch by accession | Accessions only |
| Cell-line RNA-seq (e.g. CCLE/DepMap-listed ES lines) | Real RNA-seq | Verify per-portal terms (CCLE generally open; **confirm each file**) | Real-data positives | Accessions/recipe; re-host only if license clearly permits |
| Published fusion annotations (papers, PMC-OA) | Literature evidence | PMC-OA / CC where applicable; cite always | Establish real-data "truth" labels with evidence strength | Cite; no figure/table re-host beyond license |
| COSMIC | Fusion/variant database | **Non-commercial / custom — FLAGGED** | **Cross-reference identifiers only** | **No** redistribution; not a dependency |
| OncoKB | Annotation knowledge base | **Non-commercial / custom — FLAGGED** | Reference only if at all | **No** redistribution; not a dependency |

### 7.2 Controlled-access exclusion (hard line)
- **TARGET** Ewing sarcoma sequencing (the largest pediatric ES cohort) is **controlled-access
  via dbGaP** at the individual level and is therefore **OUT OF SCOPE**. We do **not** download,
  process, or derive from it in this project. If a future, separately-governed project obtains
  authorized access + IRB/DUA, that is a different project with different controls — not this one.
- We rely on **simulation** (no human data at all) as the primary truth, and on **cell-line /
  explicitly-open** RNA-seq for real-data checks. Cell lines are immortalised lab models, not
  identifiable persons.

### 7.3 Privacy / PII stance
- **No human-subject identifiable data is ingested, stored, or produced.** Simulated reads
  contain no human individual's sequence beyond the public reference; cell lines are not
  identifiable persons. We perform a documented **re-identifiability check** before adding any
  real-data accession and reject anything that could be individual-level patient data.
- No secrets, tokens, or access credentials are ever written to logs, manifests, or commits
  (per CLAUDE.md).

### 7.4 Provenance model
- Every dataset entry: `{accession|generator, source, license, retrieved/created date, checksum,
  citation, evidence-strength (for real-data truth), reidentifiability-check: pass}`.
- Every benchmark number: bound to `{input hash, container digest, params, harness commit}` in
  the provenance ledger; CI rejects un-provenanced numbers.

### 7.5 Attribution
- Cite GENCODE/Ensembl, each GEO/SRA study, each cell-line resource, and each literature source
  per its terms. Our own simulated data and code carry clear MIT/CC-BY notices and a CITATION.cff.

---

## 8. Quality, review & risk gates

### 8.1 Risk tier
- **Project baseline: MEDIUM.** It requires domain accuracy (genomics + ES biology) but produces
  research methodology, not patient-facing or diagnostic output.
- **HIGH-risk carve-out:** *any* artefact that interprets results for a clinical/patient audience,
  or that could be read as guidance on diagnosis or care, is **HIGH** and requires **credentialed
  oncologist + patient-advocate sign-off**, "not medical advice"/"not a diagnostic device"
  labelling, and sourcing. The core project deliberately produces none; this gate exists to catch
  scope creep.

### 8.2 Required review before a deed is "done"
- **All `medium` tasks:** review by a reviewer with **bioinformatics / genomics** competence;
  reproducibility check (re-run the relevant step); license + provenance audit; CI green.
- **Any biological/interpretive claim:** **domain-expert (ES genomics) sign-off** that the claim
  is correct and sourced, before release.
- **Any `high` task (patient-facing, should it ever arise):** **oncologist + patient-advocate**
  credentialed sign-off, recorded in the review log, before merge — no exceptions.
- **License gate:** no PR merges if it adds a source without a verified license record, or if it
  would redistribute COSMIC/OncoKB/controlled data.

### 8.3 Definition of Shipped (project-level)
A deed is *shipped* (not merely merged) when: acceptance criteria met **and** CI green **and**
the artefact is independently re-runnable from pinned inputs **and** required reviewer/expert
sign-off recorded **and** provenance complete **and** the output is published in the open repo /
release where the beneficiary can actually use it. (Mirrors CLAUDE.md "delivered, not merged.")

---

## 9. Roadmap & milestones

Phased, each with a measurable **exit criterion**. M0 is a thin cold-start that proves the spine
on one caller and one simulated sample before scaling.

| Phase | Name | Goal | Exit criteria (measurable) |
|---|---|---|---|
| **M0** | Foundation / cold-start | Stand up repo, governance, compliance scaffolding, and a one-sample/one-caller end-to-end smoke benchmark | Repo + LICENSE(s) + CONTRIBUTING + compliance doc live; 1 simulated EWSR1-FLI1 sample runs through 1 caller and produces a scored result with full provenance; CI smoke test green |
| **M1** | Truth set v1 | Build the simulated truth set across partners + difficulty strata; define + publish the match rule and scoring library | ≥200 simulated cases (FLI1/ERG/ETV1/ETV4/FEV × depth/purity/degradation strata) with ground-truth breakpoints; match-rule spec v1 published; scoring lib unit-tested; dataset datasheet written |
| **M2** | Multi-caller harness | Containerise ≥6 callers behind the uniform adapter; run full simulated benchmark; first leaderboard | ≥6 callers pinned + containerised; full simulated run reproducible by maintainer; leaderboard with CIs + per-stratum results published; per-caller tool cards drafted |
| **M3** | Real-data + confounders | Add open/cell-line real positives and the non-ES *EWSR1* confounder panel; specificity analysis; first external reproduction | ≥10 cell-line/open positives + confounder panel curated (accessions + evidence + license); specificity reported; ≥1 independent external re-run confirmed |
| **M4** | Hardening + report | Full honest results report, expert sign-off on all interpretive claims, robustness (FFPE/low-input) deep-dive | Results report v1.0 released with expert sign-off log; all numbers provenanced (100%); robustness section complete; CITATION.cff + Zenodo DOI |
| **M5** | Adoption + sustainability | Drive reuse; secure steward/partner; enable community contribution of new callers | ≥1 real ES/methods group adopts or cites; "add-a-caller" contributor guide proven by ≥1 external caller PR; steward named; maintenance plan ratified; (optional) DNA-SV secondary track if open data exists |

**Sequencing / dependencies:** M1 depends on M0's harness contract; M2 depends on M1's truth set
+ scoring lib + match rule; M3 depends on M2's normalised pipeline; M4 depends on complete M2+M3
results; M5 depends on a stable M4 release. Partner/steward acquisition runs in parallel from M0
and **gates** the `verifiedNeed` flag flipping to true and any high-risk patient-facing work.

---

## 10. Work breakdown

The itemised, schema-mapped backlog lives in **`TASKS.md`**, organised by the M0–M5 milestones
above. Each task maps to a Hee-Lee Oss Task JSON (§ schema), carries acceptance criteria and a
Definition of Done, and is sized (`small`/`medium`/`large`). TASKS.md includes a complete,
schema-valid example Task JSON for the first M0 task, milestone tables
(`ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer`), and a backlog of
sized-but-unscheduled future tasks. All tasks set `verifiedNeed: false` until a partner is
secured (§2, §11).

---

## 11. Governance, roles & stakeholders

| Role | Who | Responsibility |
|---|---|---|
| **Maintainer / Owner** | TBD | Overall direction, merges, release sign-off, guardrail enforcement |
| **Reviewers (rotation)** | TBD pool with genomics/bioinformatics competence | PR review, reproducibility checks, license/provenance audits |
| **Domain expert (ES genomics)** | TO BE SECURED | Sign-off on every biological/interpretive claim before release |
| **Credentialed oncologist** | TO BE SECURED | Required only if any high-risk patient-facing artefact arises |
| **Patient advocate** | TO BE SECURED | Co-reviews any patient-facing artefact; voices family perspective |
| **Steward (last-mile owner)** | TO BE SECURED | Ensures the benchmark actually reaches + is used by beneficiaries; tracks outcomes |
| **Partner / requestor** | TO BE SECURED | Confirms the need; adopts/stewards output; flips `verifiedNeed` to true |
| **Compliance reviewer** | Maintainer + reviewer | Enforces §7 data/license/PII gates on every PR |

Governance follows Hee-Lee Oss rules: COI + veto checklist for edge cases; changes to scope/guardrails
go through documented governance. **No high-risk artefact ships without credentialed sign-off.**

---

## 12. Dependencies & integrations

- **External datasets:** GENCODE/Ensembl reference; open GEO/SRA series; cell-line RNA-seq
  portals (verify each). **No controlled-access source.**
- **External tools:** fusion callers (STAR-Fusion, Arriba, FusionCatcher, JAFFA, deFuse,
  EricScript, pizzly, STAR-SEQR; aligners STAR/HISAT2); simulators (polyester/ART, optionally
  BadRead/pbsim); structural-variant callers (Manta/GRIDSS/SvABA, secondary track).
- **Infrastructure:** Nextflow/Snakemake; Docker + Singularity/Apptainer; GitHub Actions; Zenodo
  (DOI/archival); an HPC or sufficiently large donated-compute environment for full runs.
- **Upstream/related Hee-Lee Oss projects:** `ewing-open-data-catalog` (shares dataset inventory +
  datasheets), `ewsr1-fli1-knowledge-graph` (biological grounding), `cancer-dataset-datasheets`
  (datasheet templates), `ml-oncology-benchmarks` (benchmarking conventions). Reuse, don't fork.
- **Hee-Lee Oss pieces:** Task schema (`packages/schema`), the donated-lane workflow (CLI prepares
  workspace, human runs agent, PRs opened), governance/review process.

---

## 13. Risks & mitigations

| Risk | Likelihood | Impact | Mitigation | Owner |
|---|---|---|---|---|
| Output misused as clinical/diagnostic guidance | Medium | High | Prominent "not a diagnostic device / not medical advice" on every artefact; NG1–NG2; high-risk gate; no patient-facing claims | Maintainer + expert |
| Accidental ingestion of controlled-access / identifiable data | Low | Very High | Hard exclusion list (§7.2); CI/PR license + re-identifiability gate; refuse-and-flag rule; accessions-only for real data | Compliance reviewer |
| License violation (COSMIC/OncoKB/GEO terms) | Medium | High | Per-source license record before use; no redistribution of flagged sources; recipes not data | Compliance reviewer |
| Simulated data not realistic → misleading benchmark | Medium | High | Validate simulator against known cell-line behaviour; document realism limits; label sim vs real; never claim sim == clinical performance | Domain expert |
| Real-data "truth" labels wrong (literature error) | Medium | Medium | Evidence-strength labelling; require ≥1 orthogonal/citation; treat as secondary, not gold (D2) | Domain expert |
| Benchmark not reproducible by others | Medium | High | Pinned versions + container digests; provenance ledger; CI smoke + external re-run as M3 exit gate | Maintainer |
| "Winner" framing pressures a lab into a wrong tool choice | Low | High | NG4: report trade-offs, not a single winner; CIs + per-stratum + limitations required | Maintainer |
| Caller version drift / tool deprecation | Medium | Medium | Pin digests; document tested versions; "add/update a caller" contributor path | Reviewers |
| No partner/steward secured → output unused | Medium | Medium | Outreach from M0; `verifiedNeed:false` until secured; M5 adoption gate | Maintainer/Steward |
| Compute cost of full runs exceeds donated capacity | Medium | Medium | Tiered runs (smoke vs full); downsample strata; document resource needs; consider funded lane with hard cap for heavy runs | Maintainer |
| Optimistic/biased reporting | Low | High | Anti-metric (O), mandatory limitations section, expert + peer review | Expert/Reviewers |

---

## 14. Security & privacy

- **Threat surface:** primarily supply-chain (caller containers, reference downloads) and
  accidental ingestion of sensitive data. No user-facing service, no auth, no PII store.
- **Secrets:** none required for open data; if any API token is ever needed (e.g. SRA), it is
  read from environment/secret store, **never** logged or committed (CLAUDE.md).
- **Supply chain:** pin container digests; checksum all downloaded references/inputs; record
  provenance; prefer official/Bioconda/nf-core images; review third-party adapters.
- **PII / human data:** none ingested or produced (§7.3); documented re-identifiability check
  gates any real-data addition; controlled-access hard-excluded.
- **Abuse/misuse prevention:** explicit non-diagnostic framing; refusal guardrails enforced;
  no content that aids harm; the benchmark cannot be repurposed into a diagnostic claim without
  violating its own published terms.
- **Integrity:** signed/append-only provenance ledger; CI rejects un-provenanced numbers and
  un-licensed sources.

---

## 15. Sustainability & maintenance

- **Post-delivery ownership:** the **steward** (TO BE SECURED) owns last-mile adoption and
  outcome tracking; the **maintainer** owns the repo and release cadence.
- **Maintenance model:** quarterly "refresh" releases re-running the benchmark against current
  caller versions; a documented "add/update a caller" path so the community keeps it current;
  pinned old releases remain reproducible (DOI-archived on Zenodo).
- **Outcome tracking:** track O1–O7 (§4) — independent reproductions, citations, adoptions,
  contributor PRs — in a public, lightweight ledger. If after M5 there is no adoption and no
  steward, the project is honestly marked dormant rather than padded with vanity metrics.
- **Funding lane note:** heavy full-benchmark compute may exceed donated capacity; if so, a
  **funded-lane** task with a **hard per-task budget cap** (never exceeding escrow, per CLAUDE.md)
  runs the expensive sweep, with a public cost ledger.

---

## 16. Open questions

1. **Partner/steward:** which ES research group or foundation will adopt and steward this? (Gates
   `verifiedNeed` and high-risk work.)
2. **Real-data availability:** how many ES cell-line / open RNA-seq sets with *redistributable*
   reads and *credible* fusion annotations actually exist? (Determines M3 size; sim may dominate.)
3. **Cell-line license specifics:** confirm per-file redistribution terms for each CCLE/DepMap ES
   line we cite (re-host vs recipe-only).
4. **Match-rule tolerance:** is ±10 bp the right default for breakpoint accuracy, and should
   partner-resolution be weighted above raw F1 for ES? (Needs domain-expert input.)
5. **Simulator choice & realism:** which simulator best reproduces FFPE-degraded / low-input ES
   RNA, and how do we validate realism without patient data?
6. **DNA-SV track:** is there enough open ES WGS/WES with documented fusions to justify the
   secondary track, or defer entirely?
7. **Compute:** is donated compute sufficient for the full sweep, or is a capped funded-lane run
   required?
8. **Peer review venue:** target an open methods/benchmarking venue for external validation?

---

## 17. References

- Hee-Lee Oss work rules — `C:\code\hee-lee-oss\CLAUDE.md`
- Good Deed Definition (criteria + risk tiers) — `C:\code\hee-lee-oss\docs\good-deed-definition.md`
- Task JSON schema — `C:\code\hee-lee-oss\packages\schema\src\schemas.ts`
- Portfolio roadmap (Track 8 cancer guardrails) — `C:\code\hee-lee-oss\planning\ROADMAP.md`
- Planning spec — `scratchpad/PLAN_SPEC.md`
- Depth exemplar — `C:\code\Ofelia\plan.md`
- Domain background (to cite precisely per-claim in artefacts, not asserted here): WHO
  classification of soft-tissue & bone tumours (Ewing sarcoma molecular definition); published
  fusion-detection benchmark methodology; GENCODE/Ensembl, GEO/SRA, CCLE/DepMap documentation;
  nf-core/Nextflow and Snakemake best-practice guides. *Every such reference is cited at point of
  use in the artefacts with full provenance (§7.4).*

---

## Appendix A — Improvements applied

The following 25 specific improvements were identified against the first draft and **applied**
to the plan above (and to TASKS.md). Each is concrete and reflected in the document.

1. **Led §7 with the binding cancer guardrails** as a boxed "read first" block (§7.0) that
   overrides the rest, per the project mandate — not buried mid-section.
2. **Added an explicit controlled-access exclusion sub-section (§7.2)** naming TARGET/dbGaP ES
   data as out of scope, since TARGET is the obvious tempting source for ES and the most likely
   compliance trap.
3. **Flagged COSMIC and OncoKB by name as non-commercial/custom** and demoted them to
   "cross-reference identifiers only, never a dependency" in the data table (§7.1).
4. **Made the non-ES *EWSR1* look-alike confounder panel a locked decision (D6)** and a success
   metric (O6) — specificity, not just sensitivity, because mis-resolving the partner mis-types
   the tumour. This is the ES-specific subtlety a generic benchmark misses.
5. **Added a "family-first" framing header** to ground the team in who this is really for,
   matching the care the brief demanded, without overclaiming clinical benefit.
6. **Set every Task JSON `verifiedNeed: false`** and added a TO-BE-SECURED partner/steward line
   in §2 and §11, honestly reflecting that no partner is confirmed.
7. **Added an explicit anti-metric** (§4) forbidding optimisation for a flattering F1 — honesty
   as the deliverable, guarding against the classic benchmark-gaming failure.
8. **Pinned a single reference build (GRCh38) decision (D1)** to remove liftover noise as a
   confound — a real source of spurious caller disagreement.
9. **Separated partner-resolution accuracy from raw precision/recall** in the scoring model and
   metrics, because for ES the *partner* is diagnostically meaningful, not just "a fusion."
10. **Made breakpoint match-rule explicit, versioned, and parameterised (D3)** with a published
    tolerance, so results are interpretable and not silently tuned.
11. **Added difficulty stratification as first-class (D5)**: depth, purity, FFPE-degradation,
    partner rarity, alternative breakpoints — the strata where clinically-relevant failures hide.
12. **Required confidence intervals + n-per-stratum + limitations on every number (D7, O5)** and
    enforced provenance in CI — no point estimate ships naked.
13. **Added a provenance ledger component (§6.1 #7)** binding each reported number to input hash
    + container digest + params + commit, with CI enforcement.
14. **Adopted accessions/recipe-only distribution** for any non-redistributable input (NG6,
    §7.1), so we never breach a source license by re-hosting.
15. **Added a documented re-identifiability check** as a gate before adding any real-data
    accession (§7.3, risk table) — defence-in-depth against accidental patient data.
16. **Carved out a HIGH-risk gate for scope creep** (§8.1): any patient-facing/interpretive
    artefact jumps to high risk requiring oncologist + advocate sign-off, even though core ships
    none — catches drift before it ships.
17. **Defined "Definition of Shipped" at project level (§8.3)** aligned to CLAUDE.md "delivered,
    not merged," including independent re-runnability.
18. **Made independent external reproduction a milestone exit gate (M3) and a top metric (O1)** —
    reproducibility is the actual public good, so it must be a hard gate, not a hope.
19. **Added a tiered run strategy (smoke vs full)** in CI and §6.4 / risk table to keep PRs fast
    and bound compute cost; added a funded-lane-with-hard-cap fallback for heavy sweeps (§15).
20. **Kept caller-specific logic behind a uniform adapter** (§6.1 #3, §6.3) mirroring Hee-Lee Oss's
    `adapters/` rule, so the harness stays vendor-neutral and "add-a-caller" is cheap.
21. **Added a contributor "add/update a caller" path** as an M5 exit gate proven by an external
    PR (§9), making community maintenance real rather than aspirational.
22. **Cross-linked sibling Hee-Lee Oss projects** (`ewing-open-data-catalog`,
    `ewsr1-fli1-knowledge-graph`, `cancer-dataset-datasheets`, `ml-oncology-benchmarks`) to reuse
    datasheets/inventory and avoid duplicated effort (§12).
23. **Added DNA structural-variant calling as an explicitly *secondary, conditional* track**
    gated on open-data availability (§5, §9 M5, open Q6) — ambition without overcommitting.
24. **Specified a CITATION.cff + Zenodo DOI + archived reproducible releases** (M4, §15) so the
    benchmark is citable and old versions stay reproducible.
25. **Made simulation the primary truth and labelled real-data truth by evidence strength (D2)**
    with an explicit risk + mitigation that sim realism is validated and never equated to clinical
    performance — the central scientific-integrity risk of any fusion benchmark.

---

## Review sign-off

**Reviewer:** Senior staff engineer + TPM (drafting author, self-review pass) ·
**Date:** 2026-06-28 · **Scope of review:** completeness against PLAN_SPEC.md 17-section
structure; correctness against CLAUDE.md guardrails, the Good Deed Definition, the Track-8 cancer
guardrails, and the Task schema; internal consistency between PLAN.md and TASKS.md.

**Completeness check**
- All 17 required H2 sections present and in order (Executive summary → References). ✓
- §7 leads with the binding cancer guardrails as mandated. ✓
- Outcome-based metrics with baseline + target (§4). ✓
- Roadmap M0–M5 each has a measurable exit criterion; M0 is a thin cold-start. ✓
- Risks table uses the required columns (Risk | Likelihood | Impact | Mitigation | Owner). ✓
- Appendix A lists 25 specific, applied improvements. ✓

**Correctness / guardrail check**
- Controlled-access (dbGaP/EGA/TARGET/biobanks) and identifiable patient data explicitly out of
  scope, with a hard-line sub-section and CI gate. ✓
- COSMIC/OncoKB flagged non-commercial and never redistributed/depended-on. ✓
- TCGA/GDC open-tier + GEO correctly treated as open; redistribution still recipe/accession-only
  where a study's terms require. ✓
- No medical advice; non-diagnostic framing on every artefact; high-risk gate (oncologist +
  patient-advocate) for any patient-facing content. ✓
- Provenance required on every assertion and number; CI-enforced. ✓
- Honest posture: `verifiedNeed:false`, partner/steward TO BE SECURED, anti-metric, mandatory
  limitations. ✓
- Hee-Lee Oss engineering conventions (TS/ESM, pnpm, MIT code / CC-BY docs, donated lane, DCO) honoured;
  funded-lane fallback carries a hard budget cap. ✓

**Fixes applied during review**
- Clarified that simulated reads contain no individual human sequence beyond the public reference
  (§7.3), removing ambiguity about "human data."
- Made the DNA-SV track's *conditional* status consistent across §5, §9, and §16.
- Ensured the high-risk carve-out language is identical in §3, §8.1, and §11.
- Confirmed every milestone in §9 has a corresponding milestone section in TASKS.md.

**Outstanding (require human decision, not blockers to planning):** the eight Open Questions in
§16 — chiefly securing a partner/steward (gates `verifiedNeed`), confirming redistributable
real-data availability and per-file cell-line licenses, and the compute/funding decision for the
full sweep.

**Verdict:** Plan is internally consistent, guardrail-compliant, and ready to seed the backlog in
TASKS.md. No guardrail violations found. Approved as Draft v0.1.0 pending partner acquisition.
