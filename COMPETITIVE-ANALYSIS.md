# Competitive & Improvement Analysis — `ewing-fusion-detection-benchmark`

**Analyst pass:** 2026-06-29 · Grounded with web research (sources cited inline).
**Scope:** PLAN.md v0.1.0 + TASKS.md skim. The project proposes an open, reproducible,
Ewing/EWSR1-specific benchmark of RNA-seq gene-fusion callers, built only on open/de-identified
data (simulated reads + open GEO/SRA + cell-line RNA-seq), with standardized truth sets, metrics,
containerized harness, leaderboard, and per-caller tool cards.

---

## 1. Correctness & completeness review of PLAN.md

The plan is unusually mature for a v0.1 draft: guardrails are strong, the canonical `FusionCall`
schema is sound, partner-resolution is correctly separated from raw F1 (D9/§6.2), difficulty
stratification and a confounder panel are first-class, and provenance is CI-enforced. The
governance/compliance posture (controlled-access hard-exclusion, COSMIC/OncoKB flagged, accessions-
only redistribution) is exactly right and better than most academic benchmarks, which is itself a
differentiator (see §4). That said, there are concrete scientific gaps and risks:

**1.1 Truth-set construction — the central scientific risk is under-resolved.**
- **Simulation realism is the load-bearing assumption and the plan acknowledges it (D2, risk row)
  but does not specify *how* realism is validated.** The honest meta-analysis literature is blunt
  that simulated benchmarks systematically over-state real-world accuracy: tools that win on
  simulated data pick up many false positives and miss true fusions on real data (documented
  specifically for JAFFA — high sensitivity on clean simulations, degraded on real data
  [JAFFA, Genome Medicine 2015](https://link.springer.com/article/10.1186/s13073-015-0167-x);
  meta-analysis of benchmark variability
  [bioRxiv 2025](https://www.biorxiv.org/content/10.1101/2025.01.20.633905v1.full)). The plan needs
  a concrete realism-validation protocol: e.g. match simulated error/fragment/coverage profiles to
  a real ES cell-line library, and report a "sim-vs-real concordance" delta per caller rather than
  asserting realism qualitatively.
- **"Open RNA-seq with known fusion truth" is harder to obtain than the plan implies.** The plan's
  real-data truth is "literature-asserted fusion status" on cell lines (A673, SK-N-MC, TC71, EW8,
  CHLA-10). That is *gene-level* truth at best — the literature documents *that* EWSR1-FLI1 exists,
  rarely the *exact validated breakpoint* in the specific deposited RNA-seq run. So the real-data
  arm can score gene-pair recall but generally **cannot** score breakpoint accuracy as "gold." This
  asymmetry (simulation = breakpoint truth; real = gene-level truth) should be stated as an explicit
  design constraint, and the leaderboard must visually separate the two regimes or it will mislead.
- **The strongest available *orthogonally-validated* truth sources are largely closed/commercial and
  do not cover EWSR1.** SeraCare/Horizon Seraseq fusion RNA reference materials provide dPCR-quantified
  "truth" but are proprietary, cover ~18 *druggable* solid-tumor fusions (ALK/RET/ROS1/NTRK/etc.),
  and **do not include EWSR1-FLI1**
  ([SeraCare](https://www.seracare.com/Seraseq-FFPE-Fusion-RNA-RM-v4-0710-0496/)). This is both a gap
  the project fills and a warning that no ready-made open EWSR1 truth standard exists — the project
  must *build* it, raising the realism burden.

**1.2 Breakpoint-level vs gene-level scoring — needs a more careful match model.**
- The ±10 bp tolerance (D3) is reasonable for *simulated* genomic breakpoints, but real callers
  report breakpoints at the **transcript/exon-junction** level and conventions differ (some report the
  last base of the upstream exon, some the first base of the intron; coordinates can legitimately
  differ by the length of a microhomology). A flat ±10 bp genomic window risks unfairly penalizing a
  caller that is *biologically correct* but uses a different reporting convention. Recommend: (a)
  canonicalize breakpoints to exon-junction coordinates before matching; (b) report a tolerance
  *sweep* (exact / ±5 / ±10 / same-exon-junction / gene-pair-only) rather than a single threshold, so
  readers see sensitivity to the choice. The Haas et al. benchmark scored at gene-pair level largely
  for this reason ([Genome Biology 2019](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-019-1842-9)).
- **Partner-resolution scoring needs a defined denominator.** "Did it call the right ETS partner"
  is only meaningful conditioned on having called *a* EWSR1 fusion. The plan separates it (good) but
  should specify it as a conditional metric (partner-accuracy | EWSR1-fusion-detected) plus a
  joint metric, else a caller that detects few fusions can look artificially good on partner accuracy.

**1.3 Read-length / depth / library-prep confounds — partially covered, gaps remain.**
- Depth, purity (spike fraction), and FFPE-degradation are strata (D5) — good. **Missing axes that
  materially change caller ranking:** read length (75 vs 100 vs 150 bp), single- vs paired-end,
  stranded vs unstranded library, poly-A vs total/ribo-depleted RNA (ES clinical samples are often
  FFPE total-RNA), and *insert size*. Arriba's known split-read alignment issues are read-geometry
  dependent ([Arriba, Genome Research 2021](https://genome.cshlp.org/content/31/3/448.full)), so
  omitting read geometry would hide a real effect. Add at least read-length and library-type as axes
  or explicitly scope them out with justification.

**1.4 Tool versioning / containerization — strong, with two gaps.**
- Pinning digests + single GRCh38 + pinned GENCODE (D1, D4) is excellent and exceeds most prior
  benchmarks. **Gap 1: reference *resource bundles*.** STAR-Fusion (CTAT), Arriba, and FusionCatcher
  each ship their *own* large prebuilt reference libraries; results depend on the *bundle version*,
  not just genome build. Pin each caller's resource bundle by checksum, and document that cross-caller
  comparison is confounded by differing built-in blacklists/annotation. **Gap 2: caller-internal
  filters.** Default vs sensitive modes change results by 2-3x in reported fusion counts
  (FusionCatcher ~69 vs Arriba ~39 per sample even after filtering,
  [ESMO Open / comparison](https://www.esmoopen.com/article/S2059-7029(20)30441-5/fulltext)). The
  plan should pre-register the filter setting per caller (default) and ideally report both default and
  a tuned setting, or it invites "you ran my tool wrong" disputes.

**1.5 Statistical power with few Ewing samples — the plan's weakest quantitative spot.**
- O3 targets ≥200 *simulated* cases but only **≥10 cell-line/open positives**. Ten real positives
  cannot support per-stratum confidence intervals or any per-caller significance claim on real data;
  bootstrap CIs on n=10 will be enormous. The plan correctly mandates CIs (D7) but should state
  honestly that the real-data arm is *illustrative/qualitative*, not powered, and avoid per-stratum
  real-data leaderboard cells that imply precision they don't have. Consider augmenting real positives
  with **technical replicates / downsampling** of the few available cell-line runs to characterize
  variance, while being clear that downsamples are not independent samples.

**1.6 Other concrete gaps.**
- **No defined handling of multi-mapping / paralogous / read-through artifacts** (e.g. EWSR1 has
  paralogs; FLI1/ERG are closely related ETS genes — a caller may swap ERG/FLI1). This is exactly the
  ES-specific hard case the project exists for and deserves an explicit scoring rule.
- **DNA-SV secondary track** depends on open WGS/WES cell-line data with documented breakpoints;
  realistically scarce — plan should pre-commit to *defer* unless a named dataset is found (open Q6),
  to avoid scope drift.
- **Versioned, immutable truth set vs "continuously maintained"** are in mild tension: quarterly
  refreshes (§15) must not silently change historical leaderboard numbers. Recommend immutable,
  DOI-pinned truth-set *releases* and an explicit "benchmark version" on every leaderboard.
- **No negative-control simulation of *non-fusion* chimeras** (template switching, trans-splicing
  artifacts) — needed to measure false-positive rate honestly; FUSIM can model read-through
  ([FUSIM, BMC Bioinformatics 2013](https://bmcbioinformatics.biomedcentral.com/articles/10.1186/1471-2105-14-13)).
- **Success metric O4 (adoption by M5)** is the riskiest outcome and entirely depends on the
  unsecured partner/steward — correctly flagged, but it is the project's existential dependency.

---

## 2. Competitive landscape (researched, cited)

No open, ES-specific fusion benchmark exists today — confirmed by absence in the surveyed
literature. The competition is (a) general fusion-detection benchmarks/challenges, (b) the callers
themselves (each ships self-favoring benchmarks), and (c) commercial reference standards.

**Benchmarks & challenges**
- **Haas et al. 2019 — STAR-Fusion accuracy assessment (Genome Biology).** Benchmarked ~23 methods
  on simulated + real cancer RNA-seq; concluded STAR-Fusion, Arriba, and STAR-SEQR are the most
  accurate and fastest. *Strengths:* large, careful, widely cited, released the Fusion Simulator
  Toolkit. *Gaps for us:* pan-cancer/generic, not ES-specific; not continuously maintained;
  simulation-heavy; gene-pair-level scoring; no EWSR1 partner-resolution or confounder analysis.
  [Genome Biology 2019](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-019-1842-9)
- **DREAM SMC-RNA Challenge (Cell Systems 2021).** Crowd-sourced, containerized challenge; 77 fusion
  entries scored on 51 synthetic tumors + 32 cell lines with spiked-in fusion constructs; **Arriba**
  won fusion detection, STAR-Fusion strong. *Strengths:* held-out truth, containerized, objective,
  the gold standard for *methodology*. *Gaps:* concluded ~2018, not maintained; generic; synthetic +
  spike-in truth not ES-focused; infrastructure (Synapse/Seven Bridges) not a lightweight reusable
  harness. [Cell Systems 2021](https://www.cell.com/cell-systems/fulltext/S2405-4712(21)00207-6) ·
  [PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC8376800/)
- **Meta-analysis of RNA-seq fusion tools (bioRxiv 2025).** Aggregates across benchmarks; key finding
  is **high variability of tool rankings across datasets/benchmarks**, and that the field lacks
  widely accepted gold-standard truth sets and reproducibility info. *Strength for us:* independent
  evidence the problem is real. *Gap:* it's a survey, not a runnable harness or truth set.
  [bioRxiv 2025](https://www.biorxiv.org/content/10.1101/2025.01.20.633905v1.full)
- **SEQC2 (MAQC) reference framework.** FDA-led standardized reference samples/spike-ins for
  sequencing QC; strong on DNA variants. *Gap:* fusion/RNA truth coverage limited and not ES-specific;
  not a fusion leaderboard. [SEQC2 / Nat Biotech 2021](https://www.nature.com/articles/s41587-021-01067-3)

**Fusion simulators (truth generators — adjacent, reusable)**
- **Fusion Simulator Toolkit** (Haas) — random exon-fusion transcripts from GENCODE; pairs with the
  2019 benchmark. *Use:* likely our base for `sim/`. *Gap:* random gene pairs, not ES-curated.
- **FUSIM (BMC Bioinformatics 2013)** — simulates inter-chromosomal translocations, trans-splicing,
  read-through; good for negative/artifact controls.
  [FUSIM](https://bmcbioinformatics.biomedcentral.com/articles/10.1186/1471-2105-14-13)
- **SimFuse (2015)** — uses real sequencing data as background and varies supporting-read counts;
  good realism model for depth strata. [SimFuse, PMC](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4709598/)
- **polyester / ART** — general read simulators (named in plan) for the read-generation step.

**Callers (each is a "competitor" via its own benchmark + a subject under test)**
- **STAR-Fusion (CTAT).** Fast, accurate, widely deployed; ships CTAT genome lib. *Weakness:* relies
  on its prebuilt resources (bundle-version confound); chimeric-junction based.
  [Genome Research / STAR-Fusion wiki](https://github.com/STAR-Fusion/STAR-Fusion/wiki)
- **Arriba (DKFZ).** DREAM winner; very sensitive at low read support; clinical features (visualization,
  primer design). *Weakness:* high raw event counts / FP pressure pre-filter; STAR split-read alignment
  edge cases. [Arriba, Genome Research 2021](https://genome.cshlp.org/content/31/3/448.full) ·
  [GitHub](https://github.com/suhrig/arriba)
- **FusionCatcher.** Broad database-driven sensitivity. *Weakness:* high event counts/FP rate
  (~69/sample), heavy reference deps. [comparison](https://www.esmoopen.com/article/S2059-7029(20)30441-5/fulltext)
- **CICERO (St. Jude).** Built *for pediatric driver fusions*; 95% detection on 184 validated driver
  fusions across 170 pediatric tumors; handles complex/non-canonical events (ITDs). **Most ES-relevant
  competitor caller** and a must-include in the panel (plan omits it from the M2 six — see §6).
  [Genome Biology 2020](https://link.springer.com/article/10.1186/s13059-020-02043-x) ·
  [GitHub](https://github.com/stjude/CICERO)
- **JAFFA.** Transcriptome-focused, high sensitivity. *Weakness:* low specificity on real data
  (false positives). [Genome Medicine 2015](https://link.springer.com/article/10.1186/s13073-015-0167-x)
- **pizzly (kallisto-based).** Fast pseudoalignment. *Weakness:* lower precision/outperformed in
  comparisons; ~46 fusions on healthy samples.
  [pizzly preprint](https://www.biorxiv.org/content/10.1101/166322v1.full.pdf)
- **deFuse, EricScript, STAR-SEQR, SOAPfuse, TopHat-Fusion** — older/varied; STAR-SEQR strong in Haas
  2019; useful for breadth.

**Commercial reference standards (closed)**
- **SeraCare/Horizon Seraseq fusion RNA RMs.** dPCR-quantified truth, multi-lab validated, FFPE
  versions. *Gap for us:* proprietary, **no EWSR1-FLI1** (focus on druggable ALK/RET/ROS1/NTRK), not
  redistributable, not a software harness. [SeraCare](https://www.seracare.com/Seraseq-FFPE-Fusion-RNA-RM-v4-0710-0496/) ·
  [multi-lab study](https://blog.seracare.com/ngs/a-multi-lab-study-of-fusion-rna-reference-standards-using-targeted-ngs-panels)

---

## 3. Gaps we can fill

1. **The only open, EWSR1/ES-specific benchmark.** Every existing benchmark is pan-cancer; none
   measures *ETS-partner resolution* (FLI1 vs ERG vs ETV1/4/FEV) or *non-ES EWSR1 look-alike
   specificity* (EWSR1-WT1, -NR4A3, -ATF1/CREB1) — the exact discriminations that change tumor typing.
2. **Continuously-maintained + reproducible.** Haas 2019 and DREAM are frozen snapshots. A versioned,
   DOI-pinned, container-digest-locked harness with a quarterly refresh and a community "add-a-caller"
   path fills the reproducibility-crisis gap the 2025 meta-analysis names directly.
3. **Open, redistributable simulated EWSR1 truth with exact breakpoints** — fills the void left by
   proprietary SeraCare standards that exclude EWSR1.
4. **Honest sim-vs-real and specificity reporting.** Most caller papers report sensitivity on favorable
   data; the field lacks open *specificity/false-positive* characterization on confounders. The
   mandatory confounder panel (D6) is a genuine, citable contribution.
5. **A clean, vendor-neutral, lightweight harness** (Nextflow/Snakemake + uniform adapter) that a
   single ES lab can run — unlike the heavyweight challenge infrastructure of DREAM.
6. **Trust-boundary tool cards** ("when not to rely on this caller for ES") — practical artifacts
   method-validation teams currently have to assemble themselves.

---

## 4. Differentiators to win

1. **Disease-specific depth over pan-cancer breadth** — partner-resolution and look-alike specificity
   as first-class metrics no generic benchmark reports.
2. **Reproducibility as the product** — bit-for-bit re-runnable, provenance-on-every-number,
   CI-enforced; independent reproduction is a *milestone gate* (M3/O1), not an afterthought.
3. **Radical honesty / anti-metric** — publish ugly results, CIs, n-per-stratum, and an explicit
   "simulated ≠ clinical performance" boundary. This is credibility competitors' self-benchmarks lack.
4. **Compliance-first cancer-data posture** — open-only, controlled-access hard-excluded, license-clean,
   accessions-only. This makes the artifact safely reusable and citable where TARGET/dbGaP-derived work
   cannot be freely shared.
5. **Continuously maintained + community-extensible** — quarterly refresh + proven external "add-a-caller"
   PR, so it stays current as Arriba/STAR-Fusion/CICERO evolve.
6. **Includes the pediatric-native caller (CICERO)** that pan-cancer benchmarks under-emphasize, making
   the comparison fair for the ES use case.

---

## 5. Claude API leverage

**Where Claude clearly helps (with human/code verification):**
1. **Harness & adapter scaffolding (highest leverage).** Generate the uniform `run(caller, sample,
   refs, params) -> normalized_calls.tsv` adapters, output-parser normalizers for each caller's
   idiosyncratic TSV/VCF, Nextflow/Snakemake DAG boilerplate, Dockerfiles pinned by digest, and the
   TypeScript scoring-core scaffolding + unit tests. Each caller's output format differs; Claude
   excels at writing/maintaining these parsers — *then humans run them and tests verify*.
2. **Truth-set curation *assistance*.** Triage candidate GEO/SRA accessions and PMC-OA papers,
   draft per-entry provenance records (citation, asserted fusion, evidence strength, suggested
   license read), and extract literature-reported breakpoints into a structured manifest — as
   *drafts for human confirmation*, never as the authoritative label.
3. **Documentation, datasheets, tool cards, and results-narrative.** Draft the dataset datasheets,
   per-caller tool cards, "add-a-caller" contributor guide, external-reproduction guide, and a
   plain-language (non-clinical) summary of leaderboard results — from numbers the code computed.
4. **Code review / reproducibility linting.** Flag un-pinned versions, missing provenance hooks,
   nondeterminism, or guardrail drift (e.g. a PR that adds a controlled-access source).
5. **MCP/query layer** for the leaderboard (see §7) so users can ask "which caller is most specific
   on FFPE-degraded ERG fusions" and get a provenance-linked answer.

**Where Claude must NOT decide (hard lines, mirroring §7/§8 guardrails):**
- **No fabricated or estimated benchmark numbers.** All metrics are computed by the deterministic
  scoring library from real runs; Claude never "estimates" precision/recall or fills gaps.
- **Truth-set labels require orthogonal validation.** Claude may *draft* a fusion label/breakpoint
  from literature, but the authoritative truth comes from simulation ground-truth or human-confirmed,
  citation-backed, orthogonally-validated evidence — never the LLM's say-so.
- **Metrics and match decisions are code, not LLM.** Breakpoint matching, CI computation, and
  aggregation are versioned code; no LLM-in-the-loop scoring.
- **License/redistribution and re-identifiability determinations are human-verified.** Claude may
  surface a license and a recommendation; a human compliance reviewer makes the call (esp.
  COSMIC/OncoKB/controlled-access edge cases).
- **No interpretive/biological or patient-facing claims without domain-expert sign-off** (and, for
  any patient-facing text, oncologist + advocate at high risk). Claude drafts; experts approve.

---

## 6. Ten concrete optimizations

1. **Add CICERO to the M2 caller set** (currently STAR-Fusion, Arriba, FusionCatcher, JAFFA, deFuse,
   pizzly). It is the pediatric-driver-fusion specialist (95% on 184 validated driver fusions) and
   omitting the most ES-relevant tool would be a glaring hole. Swap out deFuse or pizzly if six is
   the cap, or expand to seven.
2. **Replace the single ±10 bp threshold with a tolerance *sweep*** (exact / ±5 / ±10 / exon-junction
   / gene-pair) and canonicalize breakpoints to exon-junction coordinates before matching, to avoid
   penalizing convention differences.
3. **Pin each caller's *reference resource bundle* by checksum** (CTAT lib, Arriba refs, FusionCatcher
   db), not just the genome build — and document the residual cross-caller annotation confound.
4. **Pre-register filter settings** (default per caller) and report a sensitivity-mode variant, to
   pre-empt "you ran it wrong" disputes.
5. **Add read-geometry strata** (read length, paired/single, stranded/unstranded, poly-A vs total/ribo)
   — at minimum read length and library type — since these flip rankings.
6. **Add an artifact/negative simulation arm** (read-through, trans-splicing, template-switching via
   FUSIM) and report false-positive rate, not just sensitivity.
7. **Honestly down-rank the real-data arm to qualitative**: cap per-stratum CI claims, label it
   "illustrative, n-limited," and use downsampling/replicates to show variance rather than implying
   powered comparison.
8. **Add a paralog/partner-swap scoring rule** (ERG↔FLI1, ETV1↔ETV4) — the ES-specific failure mode —
   as a named confusion-matrix panel.
9. **Immutable, versioned benchmark releases**: every leaderboard cell carries a `benchmark-vX` tag and
   DOI; refreshes create new versions and never mutate historical numbers.
10. **Define a "sim-vs-real concordance" delta per caller** as a headline honesty metric (how much a
    tool's simulated accuracy over-states its cell-line accuracy) — directly answers the meta-analysis
    critique and becomes a citable contribution.

---

## 7. Parallel & perpendicular spin-offs

- **Tight ties to sibling Elyos projects:** `ewing-open-data-catalog` should *own* the accession +
  license + datasheet inventory the benchmark consumes (reuse, don't duplicate); `ewing-expression-
  reanalysis` and `ewing-single-cell-atlas` can consume the same curated open RNA-seq accessions and
  the validated alignment containers — share the compliance/provenance tooling across all three.
- **Generalized rare-fusion benchmark template.** The harness + scoring core + adapter pattern is
  disease-agnostic; factor out a `rare-fusion-benchmark-template` so other rare-fusion cancers
  (e.g. DSRCT EWSR1-WT1, synovial sarcoma SS18-SSX, ALK fusions) get a benchmark by swapping the
  truth set. High reuse, low marginal cost.
- **Public leaderboard site** (static, provenance-linked) — already planned; spin it into a small
  standalone that any rare-fusion instance can deploy.
- **MCP server over the benchmark** — expose "query the leaderboard / fetch a tool card / get the
  provenance of number X" as MCP tools so AI assistants (and ES labs' own agents) can answer
  caller-selection questions with cited, code-computed numbers and never hallucinate them.
- **A "fusion-caller tool card" standard** (datasheet-for-datasets analog) that could outlive this
  project and be adopted by `cancer-dataset-datasheets` / `ml-oncology-benchmarks`.
- **Sim-realism validation toolkit** — the protocol that matches simulated to real library profiles is
  reusable infrastructure for any read-simulation benchmark.

---

## 8. Open questions for the maintainer

1. **CICERO inclusion:** confirm CICERO joins the M2 panel (license/containerization feasible?) — it
   is the most ES-relevant caller and currently absent from the named six.
2. **Real-data breakpoint truth:** do any of the candidate open ES cell-line RNA-seq runs have a
   *validated exact breakpoint* (Sanger/dPCR) we can cite, or is real-data truth strictly gene-level?
   This determines whether the real arm can score breakpoints at all.
3. **How is simulation realism validated** without patient data — adopt a sim-vs-real concordance
   protocol against an open cell-line library? Which library?
4. **Breakpoint match model:** accept the tolerance-sweep + exon-junction-canonicalization proposal
   over a single ±10 bp threshold?
5. **Real-data power:** are we comfortable explicitly labeling the ≤~10-positive real arm as
   qualitative/illustrative, not statistically powered?
6. **Caller filter policy:** default-only, or default + sensitivity-mode? Pre-registered how?
7. **Reference-bundle pinning:** can we checksum-pin each caller's prebuilt reference library, and how
   do we communicate the residual annotation confound?
8. **Versioning vs continuous maintenance:** confirm immutable DOI-pinned benchmark releases so
   quarterly refreshes never silently change historical leaderboard numbers.
9. **Partner/steward:** still the existential dependency for O4 adoption — which ES group/foundation is
   the lead candidate, and what would flip `verifiedNeed` to true?
10. **DNA-SV track:** pre-commit to defer unless a *named* open ES WGS/WES dataset with documented
    breakpoints is identified?

---

### Key sources
- Haas et al. 2019, *Genome Biology* — https://genomebiology.biomedcentral.com/articles/10.1186/s13059-019-1842-9
- DREAM SMC-RNA, *Cell Systems* 2021 — https://www.cell.com/cell-systems/fulltext/S2405-4712(21)00207-6 · PMC https://pmc.ncbi.nlm.nih.gov/articles/PMC8376800/
- Meta-analysis of fusion tools, bioRxiv 2025 — https://www.biorxiv.org/content/10.1101/2025.01.20.633905v1.full
- Arriba, *Genome Research* 2021 — https://genome.cshlp.org/content/31/3/448.full · https://github.com/suhrig/arriba
- CICERO, *Genome Biology* 2020 — https://link.springer.com/article/10.1186/s13059-020-02043-x · https://github.com/stjude/CICERO
- JAFFA, *Genome Medicine* 2015 — https://link.springer.com/article/10.1186/s13073-015-0167-x
- pizzly preprint — https://www.biorxiv.org/content/10.1101/166322v1.full.pdf
- FUSIM, *BMC Bioinformatics* 2013 — https://bmcbioinformatics.biomedcentral.com/articles/10.1186/1471-2105-14-13
- SimFuse, PMC — https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4709598/
- SeraCare Seraseq fusion RNA RM — https://www.seracare.com/Seraseq-FFPE-Fusion-RNA-RM-v4-0710-0496/
- SEQC2, *Nat Biotech* 2021 — https://www.nature.com/articles/s41587-021-01067-3
- FusionCatcher/Arriba event-count comparison — https://www.esmoopen.com/article/S2059-7029(20)30441-5/fulltext
