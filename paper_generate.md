# Paper Generation Pipeline

This document guides Claude Code through the complete process of generating a scientific paper from a research idea — without depending on any existing codebase. Just follow the 6 stages in order.

## How to Use

Copy the `skills/` directory and this file into any repo. Then tell Claude Code:

> "Read paper_generate.md and follow the pipeline to generate a paper about [your idea]."

If you have a Systematic Literature Review (SLR), place your markdown files in the `SLR/` directory and tell Claude Code:

> "Read paper_generate.md. Use the SLR materials in `SLR/` to generate a research idea and produce a paper."

Claude Code will use the skills to work through each stage autonomously.

---

## SLR (Systematic Literature Review) Input

If an `SLR/` directory exists in the project root, it contains literature review materials in markdown format (one or more `.md` files). These files may include:

- Summaries of existing papers and their findings
- Research gaps and open questions identified during the review
- Comparisons of methods, datasets, or results across papers
- Key themes, trends, or contradictions in the literature

**How SLR feeds into the pipeline:**

- **Stage 1 (Idea Setup):** Read all `.md` files in `SLR/` first. Use them to understand the research landscape, identify gaps, and refine (or generate) the research idea. The SLR materials provide the "why" behind the idea.
- **Stage 4 (Gather Citations):** Extract paper titles, authors, and venues mentioned in the SLR files. Use these as seed citations before searching for additional references. This avoids redundant searches for papers already reviewed.
- **Stage 5 (Write Paper):** Use the SLR content to write a stronger Related Work section, grounded in the literature already reviewed.

---

## Pipeline Overview

```
[SLR Materials (optional)] ─┐
                             ├→ Stage 1: Setup workspace (idea from SLR or user)
[Research Idea] ─────────────┘
                              → Stage 2: Write & run experiments (4 phases)
                              → Stage 3: Create publication figures
                              → Stage 4: Gather citations (seeded from SLR)
                              → Stage 5: Write LaTeX paper (Related Work from SLR)
                              → Stage 6: Self-review with scores
                              → [Finished Paper PDF + Review]
```

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
3. Write `idea.md` (structured experiment plan with 4 stages)
4. Write `idea.json` (machine-readable metadata)
5. Write `config.yaml` (experiment hyperparameters)
6. Optionally scaffold a starter `runfile.py`

**Output:**
```
experiments/{idea_name}/
├── idea.md
├── idea.json
├── config.yaml
├── slr_synthesis.md    # (if SLR/ exists) Summary of literature review findings
├── data/
├── logs/
├── experiment_results/
└── figures/
```

**Move to Stage 2 when:** The workspace is set up and the experiment plan is clear.

---

## Stage 2: Run Experiments

**Skill:** `run-experiments`

**Input:** The workspace from Stage 1

**What to do:** Work through 4 sequential phases:

### Phase 1: Initial Implementation
- Write Python code implementing the proposed method
- Run it, debug errors, verify it produces reasonable results
- Save metrics to stdout as `METRIC_NAME: VALUE`
- Save numerical data as `.npy` files in `experiment_results/`

### Phase 2: Baseline Comparison
- Implement baseline methods
- Run all methods (proposed + baselines) with same conditions
- Use at least 3 random seeds
- Generate comparison data

### Phase 3: Full Experiments
- Scale up to full dataset and thorough evaluation
- Run multiple seeds, compute mean +/- std
- Generate learning curves, performance tables
- Run statistical significance tests if applicable

### Phase 4: Ablation Studies
- Identify key components of the proposed method
- Systematically remove/modify each one
- Measure and record the impact

**For each experiment run:**
- Execute the code
- If it errors: read traceback, fix, re-run (max 3 attempts)
- If results look wrong (NaN, zero, exploding): diagnose and fix
- Save results as `.npy` files and write summary JSONs

**Output:**
```
experiments/{idea_name}/
├── baseline_summary.json
├── research_summary.json
├── ablation_summary.json
├── experiment_results/
│   ├── *.npy (all numerical results)
│   └── *.png (diagnostic plots)
```

**Move to Stage 3 when:** All 4 phases are complete and results look reasonable.

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

**Input:** Everything from Stages 1-4

**What to do:**
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

**Page limit:** 4 pages for workshop, 8 pages for conference (excluding references and appendix).

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
2. Evaluate against NeurIPS review criteria
3. Produce a structured JSON review with:
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

---

## Quick Start (Copy-Paste Prompt)

Give Claude Code this prompt to run the full pipeline:

```
Read paper_generate.md. I want to generate a research paper about:

[Describe your research idea here]

Follow all 6 stages. Set up the workspace, implement and run the experiments,
create publication figures, gather citations, write the LaTeX paper, and
self-review it. Save everything under experiments/.
```

**If you have SLR materials**, place `.md` files in `SLR/` and use this prompt instead:

```
Read paper_generate.md. I have systematic literature review materials in SLR/.
Read the SLR files first, then generate a research idea based on the gaps
and opportunities you find. Follow all 6 stages to produce a complete paper.
```

---

## What You Need Installed

**Required:**
- Python 3.8+ with numpy, matplotlib
- ML framework (torch/tensorflow/sklearn) as needed for the experiments

**For PDF compilation (optional — LaTeX source is always produced):**
- pdflatex + bibtex (TexLive, MiKTeX, or Overleaf)

**For citations:**
- Internet access (for Semantic Scholar API / web search)

---

## Tips

- **Start small:** Give a focused idea. "Compare 3 optimizers on CIFAR-10" is better than "solve computer vision."
- **Iterate:** If Stage 6 review finds issues, go back and fix them. The pipeline is designed for iteration.
- **Be specific about metrics:** Tell Claude Code exactly what metrics matter for your domain.
- **Provide data if needed:** If your experiments need specific datasets, put them in `data/` or provide download instructions.
- **Check the numbers:** Always verify that the numbers in the paper match the actual experiment output.
