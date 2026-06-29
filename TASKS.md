# ewing-fusion-detection-benchmark — Task Backlog

**Status:** Draft · **Version:** 0.1.0 · **Last updated:** 2026-06-28 ·
**Owner:** TBD (maintainer) · **Lane:** donated (funded-lane fallback only for heavy compute,
with a hard budget cap) · **Companion:** see `PLAN.md` for the 17-section project plan.

> **Honesty flags carried by every task:** `verifiedNeed: false` (no partner/steward secured —
> see PLAN §2/§11). `requestor` is TBD. Any task that would touch controlled-access or
> identifiable patient data is **refused and flagged**, not scheduled (PLAN §7).

---

## How these tasks map to Elyos

Each task below becomes an Elyos **Task JSON** validated against
`packages/schema/src/schemas.ts`. Field mapping:

- **`id`** — stable slug `ewing-fdb-<area>-NNN`.
- **`title`** — the task name in the milestone table.
- **`project`** — always `"ewing-fusion-detection-benchmark"`.
- **`type`** — one of `code | research | writing | data | design-spec | maintenance`.
- **`lane`** — `donated` (default). Heavy full-sweep compute may use `funded` **with
  `fundedBudgetUsd` (hard cap)**.
- **`priority`** — `high | medium | low`.
- **`domain`** — array, e.g. `["cancer-research","bioinformatics","ewing-sarcoma"]`.
- **`riskTier`** — `medium` baseline; `low` for pure scaffolding/docs; `high` only for any
  patient-facing artefact (none in this backlog — gated, see PLAN §8).
- **`urgent`** — boolean (all `false`; no time-critical beneficiary deadline yet).
- **`deliverable`** — `pr | dataset | document | translation`.
- **`tokenEstimate`** — `small | medium | large` (maps to the table's **Size**).
- **`status`** — `open | in-progress | review | delivered | done` (all start `open`).
- **`context` / `objective` / `acceptanceCriteria[]` / `output`** — written per task.
- **`resources[]`** — links/paths the worker needs.
- **`requestor`** — `"TBD"` until partner secured.
- **`verifiedNeed`** — `false` (no partner yet).
- **`outputLicense`** — `MIT` (code), `CC-BY-4.0` (docs/datasheets), `CC-BY-4.0`/`CC0-1.0`
  (our simulated data); real-data tasks ship **recipes/accessions**, not re-hosted data.
- **`fundedBudgetUsd`** — only on funded tasks (none scheduled; documented in backlog).

Reviewer column: **bio-rev** = bioinformatics/genomics reviewer; **expert** = ES-genomics
domain expert (required for interpretive claims); **maint** = maintainer; **comp** = compliance
reviewer (license/PII gate).

---

## Milestone M0 — Foundation / cold-start

Goal: repo + governance + compliance scaffolding, and one simulated sample through one caller
end-to-end with full provenance.

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| ewing-fdb-repo-001 | Initialise repo, licenses, CONTRIBUTING, compliance README | maintenance | small | low | pr | — | maint, comp |
| ewing-fdb-compliance-002 | Write data-source compliance policy (open-only; controlled-access exclusion; COSMIC/OncoKB flagged) | writing | small | medium | document | repo-001 | comp, expert |
| ewing-fdb-schema-003 | Define canonical FusionCall + ground-truth data model + match-rule spec v1 | design-spec | medium | medium | document | repo-001 | bio-rev, expert |
| ewing-fdb-sim-004 | Simulate one EWSR1-FLI1 RNA-seq sample with ground-truth breakpoints | data | medium | medium | dataset | schema-003 | bio-rev |
| ewing-fdb-harness-005 | Smoke harness: 1 sample → 1 caller (STAR-Fusion) → normalised calls → scored, with provenance | code | medium | medium | pr | sim-004 | bio-rev, maint |
| ewing-fdb-ci-006 | CI: smoke benchmark on every PR + license/provenance gate | code | small | low | pr | harness-005 | maint |

**Acceptance criteria (key tasks)**

- **ewing-fdb-compliance-002**
  - [ ] Policy explicitly lists dbGaP, EGA, TARGET/TCGA controlled tiers, and individual-level
        biobanks as **out of scope**, with the refuse-and-flag rule.
  - [ ] TCGA/GDC open-tier and GEO documented as open; COSMIC and OncoKB documented as
        non-commercial/custom and **reference-only, never redistributed/depended-on**.
  - [ ] Defines the per-source license record fields and the re-identifiability check gate.
  - [ ] Reviewed by compliance reviewer **and** domain expert; CI license gate references it.
- **ewing-fdb-schema-003**
  - [ ] Canonical `FusionCall` and ground-truth records specified (fields per PLAN §6.2).
  - [ ] Match rule v1 published: gene-pair match + breakpoint tolerance (default ±10 bp,
        parameterised) + **partner-resolution scored separately**.
  - [ ] Versioned (`matchrule-v1`) and unit-testable; reviewed by bio-rev + expert.
- **ewing-fdb-harness-005**
  - [ ] One simulated EWSR1-FLI1 sample runs through STAR-Fusion (pinned digest) end-to-end.
  - [ ] Output normalised to `FusionCall`; scored against ground truth; metrics emitted.
  - [ ] Provenance ledger binds the score to input hash + container digest + params + commit.
  - [ ] Re-running produces identical results (deterministic or seeded).

**Definition of Done (M0):** repo public with MIT + CC-BY licenses, CONTRIBUTING, and compliance
README; one simulated sample scored end-to-end through one pinned caller with complete provenance;
CI smoke + license/provenance gate green; all M0 PRs reviewed and signed off; nothing redistributes
controlled/flagged data.

---

## Milestone M1 — Truth set v1

Goal: build the simulated truth set across partners + difficulty strata; ship the scoring library.

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| ewing-fdb-sim-101 | Simulate EWSR1 fusions across partners (FLI1/ERG/ETV1/ETV4/FEV) + breakpoint variants | data | large | medium | dataset | M0 | bio-rev, expert |
| ewing-fdb-sim-102 | Add difficulty strata: depth, tumour-purity (spike-in), FFPE-like degradation | data | large | medium | dataset | sim-101 | bio-rev, expert |
| ewing-fdb-score-103 | Scoring library: precision/recall/F1, breakpoint distance, partner-resolution, CIs | code | large | medium | pr | schema-003 | bio-rev |
| ewing-fdb-datasheet-104 | Dataset datasheet for the simulated truth set (provenance, realism limits) | writing | medium | medium | document | sim-102 | expert, comp |

**Acceptance criteria (key tasks)**

- **ewing-fdb-sim-101**
  - [ ] ≥200 simulated cases spanning the five ES partners with documented breakpoints.
  - [ ] Each case has a machine-readable ground-truth manifest (genes, exact breakpoints,
        support counts, spike fraction); license CC-BY-4.0/CC0; provenance recorded.
  - [ ] Domain expert confirms partner/breakpoint biology is plausible.
- **ewing-fdb-score-103**
  - [ ] Implements the versioned match rule; partner-resolution scored separately from F1.
  - [ ] Reports n-per-stratum and bootstrap confidence intervals; unit tests cover edge cases
        (no-call, multi-call, off-by-tolerance breakpoints).
  - [ ] No point estimate emitted without CI + n (enforced by tests).
- **ewing-fdb-datasheet-104**
  - [ ] Datasheet documents generation method, realism assumptions/limits, and explicit
        "simulated ≠ clinical performance" statement; sources cited; license stated.

**Definition of Done (M1):** ≥200-case stratified simulated truth set released (CC-BY/CC0) with
manifests + datasheet; scoring library released, tested, and producing CI-bounded metrics; match
rule v1 frozen; all reviewed by bio-rev (+ expert for biology).

---

## Milestone M2 — Multi-caller harness

Goal: containerise ≥6 callers behind the uniform adapter; full simulated benchmark + leaderboard.

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| ewing-fdb-callers-201 | Containerise + adapt 6 callers (STAR-Fusion, Arriba, FusionCatcher, JAFFA, deFuse, pizzly) | code | large | medium | pr | M1 | bio-rev, maint |
| ewing-fdb-harness-202 | Full Nextflow/Snakemake DAG: all samples × all callers → normalised → scored → aggregated | code | large | medium | pr | callers-201 | bio-rev, maint |
| ewing-fdb-report-203 | Leaderboard + results report v0 (per-stratum, CIs, runtime/memory) | writing | medium | medium | document | harness-202 | expert, maint |
| ewing-fdb-toolcards-204 | Per-caller tool cards (config, failure modes, trust boundaries) | writing | medium | medium | document | report-203 | expert |

**Acceptance criteria (key tasks)**

- **ewing-fdb-callers-201**
  - [ ] ≥6 callers pinned by container digest behind the uniform `run(...)→normalized_calls`
        adapter; each emits the canonical `FusionCall`.
  - [ ] All use the single pinned GRCh38 reference + annotation (no GRCh37).
  - [ ] Smoke test passes for each caller in CI.
- **ewing-fdb-harness-202**
  - [ ] Full simulated benchmark runs reproducibly via one command; provenance complete for
        every cell; maintainer re-run matches.
  - [ ] Resource usage (runtime, peak memory) captured per caller per sample.
- **ewing-fdb-report-203**
  - [ ] Leaderboard stratified by difficulty + partner; every number has CI + n + provenance link.
  - [ ] Includes a limitations section; no single "winner" declared (trade-offs only).

**Definition of Done (M2):** ≥6 callers benchmarked identically on the full simulated truth set;
reproducible full run; leaderboard + report v0 + tool cards published with provenance and CIs;
reviewed by bio-rev/expert/maint.

---

## Milestone M3 — Real-data + confounders

Goal: add open/cell-line real positives and the non-ES *EWSR1* confounder panel; specificity;
first external reproduction.

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| ewing-fdb-realdata-301 | Curate open/cell-line ES RNA-seq positives (accessions + evidence + license) | research | large | medium | dataset | M2 | bio-rev, expert, comp |
| ewing-fdb-confounder-302 | Curate non-ES *EWSR1* look-alike + fusion-negative confounder panel | research | medium | medium | dataset | realdata-301 | expert, comp |
| ewing-fdb-specificity-303 | Run callers on real + confounder panel; report sensitivity + specificity | code | medium | medium | pr | confounder-302 | bio-rev, expert |
| ewing-fdb-repro-304 | External-reproduction guide + verify ≥1 independent re-run | writing | medium | medium | document | harness-202 | maint |

**Acceptance criteria (key tasks)**

- **ewing-fdb-realdata-301**
  - [ ] ≥10 cell-line/open positives recorded as **accessions + fetch recipe** (no re-hosting
        unless license clearly permits); each with citation + evidence-strength + license +
        re-identifiability check = pass.
  - [ ] Compliance reviewer confirms **no controlled-access/identifiable data**.
- **ewing-fdb-confounder-302**
  - [ ] Includes non-ES *EWSR1* fusions (e.g. EWSR1-WT1, EWSR1-NR4A3, EWSR1-ATF1/CREB1) and
        fusion-negative samples; each sourced + licensed.
  - [ ] Documents the diagnostic importance of distinguishing these from ES fusions.
- **ewing-fdb-repro-304**
  - [ ] Step-by-step external re-run guide; ≥1 independent party confirms a matching run
        (within documented tolerance); discrepancies logged.

**Definition of Done (M3):** real-data positives + confounder panel curated compliantly
(accessions/recipes only); specificity reported alongside sensitivity; ≥1 verified independent
reproduction; all license/PII gates green.

---

## Milestone M4 — Hardening + report

Goal: full honest report, expert sign-off on all interpretive claims, robustness deep-dive,
citable archived release.

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| ewing-fdb-robust-401 | Robustness deep-dive: FFPE/low-input/low-purity failure analysis | research | medium | medium | document | M3 | expert, bio-rev |
| ewing-fdb-report-402 | Results report v1.0 + expert sign-off log on all interpretive claims | writing | medium | medium | document | robust-401 | expert, maint |
| ewing-fdb-release-403 | CITATION.cff + Zenodo DOI + archived reproducible release | maintenance | small | low | pr | report-402 | maint |

**Acceptance criteria (key tasks)**

- **ewing-fdb-report-402**
  - [ ] Every interpretive/biological claim carries a citation and an expert sign-off entry.
  - [ ] 100% of numbers provenanced (CI audit passes); limitations + CIs throughout.
  - [ ] "Not a diagnostic device / not medical advice" on the artefact.
- **ewing-fdb-release-403**
  - [ ] Tagged release archived with DOI; old releases remain reproducible from pinned inputs.

**Definition of Done (M4):** report v1.0 released with complete expert sign-off log and 100%
provenance; robustness analysis published; citable DOI-archived reproducible release.

---

## Milestone M5 — Adoption + sustainability

Goal: drive reuse; secure steward/partner; enable community caller contributions; optional DNA-SV
track.

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| ewing-fdb-contrib-501 | "Add/update a caller" contributor guide + template (proven by ≥1 external PR) | writing | medium | low | document | M4 | maint, bio-rev |
| ewing-fdb-adoption-502 | Outreach + secure partner/steward; flip verifiedNeed; track outcomes O1–O7 | research | medium | low | document | M4 | maint |
| ewing-fdb-dnasv-503 | (Conditional) DNA structural-variant track (Manta/GRIDSS/SvABA) if open data exists | code | large | medium | pr | M4 | bio-rev, comp |

**Acceptance criteria (key tasks)**

- **ewing-fdb-contrib-501**
  - [ ] Documented adapter contract + template; ≥1 external contributor lands a new caller via it.
- **ewing-fdb-adoption-502**
  - [ ] ≥1 real ES/methods group adopts or cites; partner/steward named; `verifiedNeed` flipped
        to true with a recorded letter of intent; outcome ledger live.
- **ewing-fdb-dnasv-503**
  - [ ] Only proceeds if redistributable/open ES WGS/WES with documented fusions exists
        (compliance-confirmed); otherwise formally deferred with rationale.

**Definition of Done (M5):** community contribution path proven by an external PR; partner/steward
secured and `verifiedNeed` true; outcome tracking live; DNA-SV track shipped or explicitly deferred
with documented rationale; maintenance plan ratified.

---

## Backlog / future (sized, unscheduled)

| ID | Title | Type | Size | Risk | Deliverable | Notes |
|---|---|---|---|---|---|---|
| ewing-fdb-longread-601 | Long-read (ONT/PacBio) simulated fusion track | data | large | medium | dataset | Needs long-read simulator + caller adapters |
| ewing-fdb-fullsweep-602 | Funded-lane full parameter sweep (heavy compute, hard budget cap) | code | large | medium | dataset | **funded**; `fundedBudgetUsd` cap required (PLAN §15) |
| ewing-fdb-newcaller-603 | Add emerging callers as they release | maintenance | medium | medium | pr | Quarterly refresh cadence |
| ewing-fdb-viz-604 | Interactive leaderboard explorer (static site) | code | medium | low | pr | UX for researchers; no PII |
| ewing-fdb-crosslink-605 | Integrate datasheets with `ewing-open-data-catalog` | data | small | low | dataset | Reuse sibling project inventory |

---

## Example task JSON (first M0 task, schema-valid)

```json
{
  "id": "ewing-fdb-repo-001",
  "title": "Initialise repo, licenses, CONTRIBUTING, and compliance README",
  "project": "ewing-fusion-detection-benchmark",
  "type": "maintenance",
  "lane": "donated",
  "priority": "high",
  "domain": ["cancer-research", "bioinformatics", "ewing-sarcoma", "reproducibility"],
  "riskTier": "low",
  "urgent": false,
  "deliverable": "pr",
  "tokenEstimate": "small",
  "status": "open",
  "context": "Cold-start (M0) for the EWSR1-fusion detection benchmark. No open, reproducible, Ewing-Sarcoma-specific fusion-caller benchmark exists. Before any data or code, the repo must encode the binding cancer guardrails: open-access/aggregate/de-identified/simulated data ONLY; controlled-access (dbGaP, EGA, TARGET/TCGA controlled tiers, biobanks) and identifiable patient data are OUT OF SCOPE; COSMIC/OncoKB are non-commercial and reference-only; no medical advice; provenance on every assertion. No partner is yet secured.",
  "objective": "Create the public repository skeleton with dual licensing (MIT for code, CC-BY-4.0 for docs/datasheets), a CONTRIBUTING guide, a CITATION stub, and a compliance README that states the data-source guardrails and the refuse-and-flag rule, so all later work inherits the constraints.",
  "acceptanceCriteria": [
    "Repo created with LICENSE (MIT) and LICENSES/CC-BY-4.0 for docs/data; README states scope and the 'not a diagnostic device / not medical advice' disclaimer.",
    "CONTRIBUTING.md describes the donated-lane flow, DCO sign-off (git commit -s), and the review/expert sign-off gates.",
    "Compliance README lists controlled-access sources as OUT OF SCOPE and flags COSMIC/OncoKB as non-commercial reference-only, with the refuse-and-flag rule.",
    "No secrets, tokens, controlled-access data, or re-hosted flagged data are present in the repo.",
    "Reviewed and signed off by the maintainer and the compliance reviewer."
  ],
  "resources": [
    "C:/code/elyos/CLAUDE.md",
    "C:/code/elyos/docs/good-deed-definition.md",
    "C:/code/elyos/planning/ROADMAP.md (Track 8 cancer guardrails)",
    "PLAN.md (sections 7 and 8)"
  ],
  "output": "A pull request adding the repository skeleton: MIT + CC-BY-4.0 licenses, README, CONTRIBUTING.md, CITATION.cff stub, and compliance README encoding the cancer data guardrails.",
  "requestor": "TBD",
  "verifiedNeed": false,
  "outputLicense": "MIT"
}
```

---

## Backlog totals

- **Milestones:** 6 (M0–M5).
- **Scheduled tasks:** 22 across M0–M5.
- **Backlog (unscheduled):** 5.
- **Total tasks:** 27.
- **Risk:** all `low`/`medium`; **no `high` tasks scheduled** — any patient-facing artefact is
  gated behind oncologist + patient-advocate sign-off (PLAN §8) and out of this backlog.
- **`verifiedNeed`:** `false` on every task until a partner/steward is secured (PLAN §2/§11).
- **Funded tasks:** none scheduled; `ewing-fdb-fullsweep-602` is funded-lane and requires a
  `fundedBudgetUsd` hard cap if activated.
