# Paper Generation Pipeline

This document guides Claude Code through the complete process of generating a scientific paper from a research idea — without depending on any existing codebase. Follow Stage 0 first, then Stages 1-6 in order.
## Setup
### SLR (Systematic Literature Review) Input

If an `SLR/` directory exists in the project root, it contains literature review materials in markdown format (one or more `.md` files). These files may include:

- Summaries of existing papers and their findings
- Research gaps and open questions identified during the review
- Comparisons of methods, datasets, or results across papers
- Key themes, trends, or contradictions in the literature
- Taxonomy of the field, technical patterns, and traceable research ideas

A deep research prompt template is available at `SLR/deepresearch_prompt.md`. Use it with ChatGPT, Claude, or Gemini to generate SLR content, then save the output as `.md` files in `SLR/`.

Once the SLR materials are generated, you can proceed to the following stages.

## Stage 0: SLR Pre-Check (MANDATORY before starting the pipeline)

Before running any stage, check whether the `SLR/` directory contains deep research output:

1. List all files in `SLR/`
2. If the **only** file is `SLR/deepresearch_prompt.md` (no other `.md` files), **STOP** and guide the user:

> **You don't have any deep research output yet.**
>
> Before starting the paper generation pipeline, you should use the deep research prompt to generate a literature review. This will give the pipeline much stronger foundations — better ideas, better citations, and better related work.
>
> **How to generate SLR materials:**
>
> 1. Open `SLR/deepresearch_prompt.md`
> 2. Replace `{topic}` with your research topic
> 3. Submit the prompt to one of these AI deep research tools:
>    - **ChatGPT** → use Deep Research mode
>    - **Claude** → paste into claude.ai
>    - **Gemini** → use Deep Research mode
> 4. Copy the AI's output (in markdown format)
> 5. Save it as a new `.md` file in the `SLR/` directory (e.g., `SLR/my_topic_review.md`)
> 6. Come back and run the pipeline again
>
> **Tip:** You can run multiple sessions with different AI tools or topic angles. Save each output as a separate `.md` file — the pipeline will read all of them.

3. If `SLR/` contains other `.md` files beyond `deepresearch_prompt.md`, proceed to Stage 1.
4. If `SLR/` does not exist at all and the user has provided a research idea directly, proceed to Stage 1 without SLR.

---

## Pipeline Overview

```
[SLR Pre-Check] ─────────────→ (guide user if no deep research output)
                                     │
[SLR Materials (optional)] ─┐       ↓
                             ├→ Stage 1: Setup workspace + initialize memory
[Research Idea] ─────────────┘
                              → Stage 2: Run experiments (4 phases)
                              │           ↕ feedback loop: results update idea
                              │           (research_log.jsonl tracks all changes)
                              → Stage 3: Create publication figures
                              → Stage 4: Gather citations (seeded from SLR)
                              → Stage 5: Write LaTeX paper (reflects evolved idea)
                              → Stage 6: Self-review with scores
                              → [Finished Paper PDF + Review]
```

---

## Long-Term Memory System

Inspired by [Hyperagents](https://github.com/facebookresearch/Hyperagents), this pipeline maintains a **three-layer memory** that tracks how the research idea evolves through experimentation. The idea is not static — it is a living hypothesis that gets updated as evidence accumulates.

### Memory Architecture

```
experiments/{idea_name}/
├── research_log.jsonl       # Layer 1: Append-only event log (machine-readable)
├── idea_evolution.md        # Layer 2: Human-readable idea change history
├── idea.md                  # Layer 3: Current state (always up-to-date)
└── idea.json                # Layer 3: Current metadata (always up-to-date)
```

### Layer 1: Research Log (`research_log.jsonl`)

An **append-only** event journal (one JSON object per line). Every significant event during the pipeline is logged here. This is the single source of truth for what happened and when.

**Event types:**

```jsonl
{"timestamp": "2026-03-31T10:00:00", "event": "idea_initialized", "version": 1, "summary": "Initial hypothesis: X outperforms Y on task Z", "source": "user+SLR"}
{"timestamp": "2026-03-31T10:30:00", "event": "experiment_started", "phase": 1, "description": "Initial implementation of method X"}
{"timestamp": "2026-03-31T11:00:00", "event": "experiment_completed", "phase": 1, "metrics": {"train_loss": 0.34, "test_acc": 0.72}, "supports_hypothesis": true, "analysis": "Basic results look promising"}
{"timestamp": "2026-03-31T12:00:00", "event": "experiment_completed", "phase": 2, "metrics": {"proposed_acc": 0.75, "baseline_acc": 0.78}, "supports_hypothesis": false, "analysis": "Baseline outperforms on accuracy, but proposed method has 2x faster training"}
{"timestamp": "2026-03-31T12:05:00", "event": "idea_updated", "version": 2, "change": "Shifted claim from accuracy improvement to efficiency-accuracy tradeoff", "reason": "Phase 2 results show proposed method is faster but not more accurate", "fields_changed": ["hypothesis", "expected_outcomes", "metrics"]}
{"timestamp": "2026-03-31T14:00:00", "event": "experiment_completed", "phase": 3, "metrics": {"proposed_acc": 0.81, "baseline_acc": 0.80, "proposed_time": 45, "baseline_time": 120}, "supports_hypothesis": true, "analysis": "At full scale, accuracy gap closes while speed advantage holds"}
{"timestamp": "2026-03-31T15:00:00", "event": "idea_updated", "version": 3, "change": "Added scaling analysis as key contribution", "reason": "Phase 3 shows the advantage grows with dataset size", "fields_changed": ["contributions", "experiment_plan"]}
```

### Layer 2: Idea Evolution (`idea_evolution.md`)

A **human-readable changelog** that records each version of the idea with the reasoning behind changes. This feeds directly into the paper's narrative.

```markdown
## Version 1 (Initial)
- **Hypothesis:** Method X outperforms baseline Y on task Z
- **Source:** User idea + SLR gap analysis
- **Key assumptions:** X's attention mechanism captures long-range dependencies better

## Version 2 (After Phase 2)
- **Hypothesis:** Method X achieves comparable accuracy to Y with 2x training speedup
- **What changed:** Shifted primary claim from accuracy to efficiency
- **Why:** Baseline matched accuracy (0.78 vs 0.75), but proposed method trained 2x faster
- **Impact on experiment plan:** Added wall-clock time as primary metric, kept accuracy as secondary

## Version 3 (After Phase 3)
- **Hypothesis:** Method X matches accuracy and scales better than Y on large datasets
- **What changed:** Added scaling behavior as key contribution
- **Why:** At full scale, accuracy gap closed (0.81 vs 0.80) while speed advantage increased (2.7x)
- **Impact on paper:** Reframed Introduction around efficiency-at-scale narrative
```

### Layer 3: Current State (`idea.md` + `idea.json`)

These files always reflect the **latest version** of the idea. When the idea is updated, these files are rewritten in place (the old version is preserved in Layer 2).

`idea.json` includes a version counter and last-modified timestamp:
```json
{
  "Name": "efficient_attention",
  "Title": "...",
  "version": 3,
  "last_updated": "2026-03-31T15:00:00",
  "update_reason": "Phase 3 results confirmed scaling advantage",
  ...
}
```

### When to Update the Idea

After each experiment phase, evaluate results against the current hypothesis:

| Result | Action |
|--------|--------|
| **Strongly supports** hypothesis | Log confirmation, no idea change needed |
| **Partially supports** (some metrics good, others not) | Update hypothesis to emphasize supported aspects, adjust claims |
| **Contradicts** hypothesis | Pivot the idea: change hypothesis, redefine success metrics, or propose new explanation |
| **Reveals unexpected finding** | Expand the idea to incorporate the new finding as a contribution |

**Critical rule:** Never ignore contradictory evidence. If the experiment doesn't support the claim, update the claim — don't cherry-pick results.

### Memory Operations

The pipeline uses these operations throughout:

| Operation | When | What |
|-----------|------|------|
| `log_event()` | Every significant action | Append to `research_log.jsonl` |
| `update_idea()` | After experiment results contradict or expand the hypothesis | Update `idea.md`, `idea.json`, append to `idea_evolution.md`, log to `research_log.jsonl` |
| `read_memory()` | Before each stage | Read `research_log.jsonl` and `idea_evolution.md` to understand current state |
| `get_idea_version()` | Before writing paper | Check version in `idea.json` to ensure paper reflects final idea |

---

## Stage 1: Idea Setup

**Skill:** `idea-setup`

**Input:** A research idea (from the user, a JSON file, or a conversation), optionally informed by SLR materials

**What to do:**

0. **If `SLR/` directory exists:** Read all `.md` files in `SLR/`. Synthesize the key findings, research gaps, and open questions. Use this synthesis to:
   - Generate a research idea (if the user hasn't provided one)
   - Refine and strengthen the user's idea with evidence from the literature
   - Identify appropriate baselines and datasets mentioned in the reviewed papers
   - Write an `slr_synthesis.md` summary in the experiment workspace
1. Clarify the idea — pin down the hypothesis, method, datasets, metrics, and baselines
2. Create the experiment workspace directory structure
3. Write `idea.md` (structured experiment plan with 4 stages, version 1)
4. Write `idea.json` (machine-readable metadata, with `"version": 1`)
5. Write `config.yaml` (experiment hyperparameters)
6. **Initialize long-term memory:**
   - Create `research_log.jsonl` with the first `idea_initialized` event
   - Create `idea_evolution.md` with Version 1 entry (initial hypothesis, source, key assumptions)
7. Optionally scaffold a starter `runfile.py`

**Output:**
```
experiments/{idea_name}/
├── idea.md                 # Current idea (version 1)
├── idea.json               # Machine-readable metadata (version 1)
├── idea_evolution.md        # Idea changelog (initialized)
├── research_log.jsonl       # Event log (initialized)
├── config.yaml
├── slr_synthesis.md         # (if SLR/ exists)
├── data/
├── logs/
├── experiment_results/
└── figures/
```

**Move to Stage 2 when:** The workspace is set up, the experiment plan is clear, and memory is initialized.

---

## Stage 2: Run Experiments

**Skill:** `run-experiments`

**Input:** The workspace from Stage 1 (including initialized memory)

**What to do:** Work through 4 sequential phases. **After each phase, evaluate results against the current hypothesis and update the idea if needed.**

### Phase 1: Initial Implementation
- Read current `idea.md` to understand what to implement
- Write Python code implementing the proposed method
- Run it, debug errors, verify it produces reasonable results
- Save metrics to stdout as `METRIC_NAME: VALUE`
- Save numerical data as `.npy` files in `experiment_results/`
- **→ Log:** Append `experiment_completed` event to `research_log.jsonl`
- **→ Evaluate:** Do initial results support the hypothesis? Log the assessment.

### Phase 2: Baseline Comparison
- **→ Read memory:** Check `research_log.jsonl` for Phase 1 findings before proceeding
- Implement baseline methods
- Run all methods (proposed + baselines) with same conditions
- Use at least 3 random seeds
- Generate comparison data
- **→ Log:** Append `experiment_completed` event with `supports_hypothesis` field
- **→ Evaluate & Update:** Compare proposed method vs baselines. If results contradict the hypothesis (e.g., baseline wins), **update the idea**:
  1. Append new version to `idea_evolution.md` with what changed and why
  2. Rewrite `idea.md` and `idea.json` to reflect the updated hypothesis (increment version)
  3. Log `idea_updated` event to `research_log.jsonl`
  4. Adjust remaining experiment plan if needed

### Phase 3: Full Experiments
- **→ Read memory:** Check `idea_evolution.md` for the latest hypothesis version
- Scale up to full dataset and thorough evaluation
- Run multiple seeds, compute mean +/- std
- Generate learning curves, performance tables
- Run statistical significance tests if applicable
- **→ Log & Evaluate:** Same feedback loop as Phase 2

### Phase 4: Ablation Studies
- **→ Read memory:** Check latest idea version — ablate components of the *current* method (which may have evolved)
- Identify key components of the proposed method
- Systematically remove/modify each one
- Measure and record the impact
- **→ Log & Evaluate:** If ablations reveal a component is unnecessary, update the idea

### Feedback Loop Summary

After each phase, follow this protocol:

```
1. Log experiment results → research_log.jsonl
2. Compare results against current hypothesis in idea.md
3. Classify: supports / partially supports / contradicts / unexpected finding
4. If idea needs updating:
   a. Append new version to idea_evolution.md (what changed + why)
   b. Rewrite idea.md and idea.json (increment version)
   c. Log idea_updated event to research_log.jsonl
5. Proceed to next phase with updated understanding
```

**For each experiment run:**
- Execute the code
- If it errors: read traceback, fix, re-run (max 3 attempts)
- If results look wrong (NaN, zero, exploding): diagnose and fix
- Save results as `.npy` files and write summary JSONs

**Output:**
```
experiments/{idea_name}/
├── idea.md                  # Updated to final version
├── idea.json                # Updated with final version number
├── idea_evolution.md         # Complete history of idea changes
├── research_log.jsonl        # Full event log
├── baseline_summary.json
├── research_summary.json
├── ablation_summary.json
├── experiment_results/
│   ├── *.npy (all numerical results)
│   └── *.png (diagnostic plots)
```

**Move to Stage 3 when:** All 4 phases are complete, results look reasonable, and `idea.md` reflects the final evolved hypothesis.

---

## Stage 3: Aggregate Plots

**Skill:** `aggregate-plots`

**Input:** `.npy` files and summary JSONs from Stage 2

**What to do:**
1. Survey all available data (`.npy` files, summaries)
2. Plan 4-8 publication-quality figures covering:
   - Learning curves (loss/metric vs epoch)
   - Method comparison (bar chart with error bars)
   - Ablation results
   - Any domain-specific visualizations (samples, attention maps, etc.)
3. Write `auto_plot_aggregator.py` — a self-contained plotting script
4. Run it, check the output, iterate up to 5 times

**Plotting standards:**
- 300 DPI, font size 14+
- Remove top/right spines
- No underscores in labels
- Error bars/shading for multiple seeds
- Informative legends and captions
- Wrap each plot in try/except for fault isolation

**Output:**
```
experiments/{idea_name}/
├── figures/
│   ├── learning_curves.png
│   ├── method_comparison.png
│   ├── ablation_results.png
│   └── ... (4-8 figures)
└── auto_plot_aggregator.py
```

**Move to Stage 4 when:** Figures look publication-ready.

---

## Stage 4: Gather Citations

**Skill:** `gather-citations`

**Input:** `idea.md`, experiment summaries, and optionally `SLR/` materials

**What to do:**
1. **If `SLR/` directory exists:** Scan all `.md` files in `SLR/` for paper references (titles, authors, years, venues, arXiv IDs). Extract these as seed citations and add them to `references.bib` first. This avoids redundant API searches for papers already reviewed.
2. Make a checklist of citation categories needed (core problem, methods, datasets, baselines, related work, etc.). Mark categories already covered by SLR-extracted citations.
3. For remaining gaps, search for relevant papers using:
   - Semantic Scholar API: `curl "https://api.semanticscholar.org/graph/v1/paper/search?query=...&fields=title,authors,year,venue"`
   - Web search as fallback
3. Build BibTeX entries for each paper
4. Deduplicate by title
5. Save to `references.bib`
6. Optionally create `citation_notes.json` mapping each citation to where it belongs in the paper

**Target:** 15-30 references for a typical paper.

**Output:**
```
experiments/{idea_name}/
├── references.bib
└── citation_notes.json (optional)
```

**Move to Stage 5 when:** Bibliography covers all major categories.

---

## Stage 5: Write Paper

**Skill:** `write-paper`

**Input:** Everything from Stages 1-4, especially:
- `idea.md` (final evolved version — check `idea.json` version number)
- `idea_evolution.md` (use to craft the paper's narrative arc)
- `research_log.jsonl` (use to ensure all findings are reflected)

**What to do:**

0. **Read memory first:** Read `idea_evolution.md` to understand how the idea evolved. If the idea changed significantly, the paper should tell the story of discovery — frame unexpected findings as contributions, not as failures of the original hypothesis.
1. Create `latex/template.tex` using a standard ML paper structure
2. Write each section:
   - **Abstract** (150-250 words, write LAST)
   - **Introduction** (problem → gap → approach → contributions)
   - **Related Work** (grouped by theme, cite from `references.bib`)
   - **Method** (detailed approach with math)
   - **Experiments** (setup, main results table, figures, ablations)
   - **Conclusion** (summary, limitations, future work)
3. Include all figures from `figures/` with self-contained captions
4. Create results tables with mean +/- std
5. Compile: `pdflatex → bibtex → pdflatex → pdflatex`
6. Self-review and revise (up to 3 rounds):
   - Round 1: Structure and completeness
   - Round 2: Content accuracy and support for claims
   - Round 3: Polish, grammar, page limit

**Page limit:** 4 pages for workshop, 9 pages for conference (excluding references and appendix).

**Output:**
```
experiments/{idea_name}/
├── latex/
│   ├── template.tex
│   └── template.pdf (if pdflatex available)
```

**Move to Stage 6 when:** Paper compiles and looks complete.

---

## Stage 6: Perform Review

**Skill:** `perform-review`

**Input:** The compiled PDF or LaTeX source

**What to do:**
1. Read the entire paper carefully
2. **Cross-check with memory:** Read `research_log.jsonl` and `idea_evolution.md`. Verify that:
   - The paper's claims match the final idea version (not an outdated version)
   - All significant experimental findings are reported (no omitted contradictory results)
   - The narrative is consistent with how the idea actually evolved
3. Evaluate against NeurIPS review criteria
4. Produce a structured JSON review with:
   - Summary, Strengths, Weaknesses
   - Scores: Originality (1-4), Quality (1-4), Clarity (1-4), Significance (1-4)
   - Soundness (1-4), Presentation (1-4), Contribution (1-4)
   - Overall (1-10), Confidence (1-5)
   - Decision: Accept or Reject
   - Questions and Limitations
4. Optionally review each figure for accuracy
5. Reflect on the review for fairness and adjust if needed

**Output:**
```
experiments/{idea_name}/
└── review_text.json
```

