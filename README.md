# Claude AI Scientist

Use **Claude Code** (CLI / Desktop / IDE extension) to autonomously generate scientific papers — from idea to PDF — without any API keys. Just a Claude subscription.

This project extracts and adapts the core pipeline from [SakanaAI/AI-Scientist-v2](https://github.com/SakanaAI/AI-Scientist-v2) into a set of Claude Code skills, so the entire workflow runs natively inside Claude Code.

## Why This Exists

The original AI-Scientist-v2 requires:
- OpenAI / Anthropic API keys
- A Python orchestration layer
- Manual environment configuration

This repo replaces all of that with **6 Claude Code skills** and a single pipeline document. You just talk to Claude Code, and it handles everything.

## How It Works

```
                              → Stage 0: SLR Pre-Check (auto)
                                         ↓ (guides user to run deep research if SLR/ is empty)
[SLR Materials (optional)] ─┐
                             ├→ Stage 1: Set up workspace + init memory     (skill: idea-setup)
[Your Research Idea] ────────┘
                              → Stage 2: Run experiments (4 phases)          (skill: run-experiments)
                              │           ↕ feedback loop: results ↔ idea
                              → Stage 3: Create publication-quality figures   (skill: aggregate-plots)
                              → Stage 4: Search & compile citations           (skill: gather-citations)
                              → Stage 5: Write LaTeX paper (evolved idea)     (skill: write-paper)
                              → Stage 6: Self-review in NeurIPS format        (skill: perform-review)
                              → [Finished Paper PDF + Review JSON]
```

## Quick Start

### 1. Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) with an active Claude subscription (Pro / Team / Enterprise)
- Python 3.8+ with `numpy`, `matplotlib`
- ML framework as needed (`torch`, `tensorflow`, `sklearn`, etc.)
- (Optional) `pdflatex` + `bibtex` for PDF compilation — LaTeX source is always produced
- Internet access for citations (Semantic Scholar API / web search)

### 2. Clone

```bash
git clone https://github.com/pengqianhan/claude-AI-scientist-v2.git
cd claude-AI-scientist-v2
```

### 3. Run the Pipeline

Open Claude Code in this repo directory and give it a prompt:

```
Read paper_generate.md. I want to generate a research paper about:

[Describe your research idea here]

Run Stage 0 first, then follow Stages 1-6. Set up the workspace, implement and run the experiments,
create publication figures, gather citations, write the LaTeX paper, and
self-review it. Save everything under experiments/.
```

**With SLR materials:** If you have a systematic literature review, place your `.md` files in the `SLR/` directory and use this prompt instead:

```
Read paper_generate.md. I have systematic literature review materials in SLR/.
Read the SLR files first, then generate a research idea based on the gaps
and opportunities you find. Run Stage 0 first, then follow Stages 1-6 to
produce a complete paper.
```

**Generate SLR from scratch:** The repo includes a deep research prompt template at `SLR/deepresearch_prompt.md`. Use it to generate comprehensive literature reviews with any AI deep research tool:

1. Open `SLR/deepresearch_prompt.md`
2. Replace `{topic}` with your research topic
3. Paste the prompt into **ChatGPT** (Deep Research), **Claude**, or **Gemini** (Deep Research)
4. Copy the output as markdown and save it in `SLR/` (for example, `SLR/your_topic_review.md`)
5. Run the pipeline with the SLR prompt above

The deep research prompt generates a structured output including: literature database, taxonomy, gap analysis, and traceable research ideas — all of which feed directly into the pipeline.

Claude Code will work through all 6 stages autonomously, using the skills defined in `.claude/skills/`.

## Project Structure

```
.
├── paper_generate.md              # Main pipeline document (the "brain")
├── SLR/                           # (optional) Systematic literature review markdown files
│   ├── deepresearch_prompt.md     # Template prompt for AI deep research tools
│   ├── review_topic_A.md          # Deep research output (from ChatGPT/Claude/Gemini)
│   └── review_topic_B.md
├── .claude/
│   └── skills/
│       ├── idea-setup/            # Stage 1: Workspace & experiment planning
│       ├── run-experiments/       # Stage 2: Code, run, debug, iterate
│       ├── aggregate-plots/       # Stage 3: Publication-quality figures
│       ├── gather-citations/      # Stage 4: BibTeX bibliography
│       ├── write-paper/           # Stage 5: Full LaTeX paper
│       └── perform-review/        # Stage 6: NeurIPS-style peer review
└── README.md
```

After running, your experiment output will look like:

```
experiments/{idea_name}/
├── idea.md / idea.json / config.yaml
├── idea_evolution.md     # How the idea changed during experiments
├── research_log.jsonl    # Append-only event log (all experiments + idea updates)
├── slr_synthesis.md      (if SLR/ was used)
├── data/
├── logs/
├── experiment_results/   (*.npy, *.png)
├── figures/              (publication-ready plots)
├── references.bib
├── latex/
│   ├── template.tex
│   └── template.pdf
└── review_text.json
```

## Long-Term Memory & Idea Evolution

Inspired by [Hyperagents](https://github.com/facebookresearch/Hyperagents), this pipeline treats the research idea as a **living hypothesis** that evolves based on experiment results — just like real research.

### Three-Layer Memory

| Layer | File | Purpose |
|-------|------|---------|
| Event Log | `research_log.jsonl` | Append-only record of all experiments and idea changes (machine-readable) |
| Idea History | `idea_evolution.md` | Human-readable changelog: what changed, why, and how it affects the paper |
| Current State | `idea.md` + `idea.json` | Always reflects the latest hypothesis (version-tracked) |

### How It Works

After each experiment phase, the pipeline evaluates results against the current hypothesis:

- **Results support the hypothesis** → log confirmation, proceed
- **Results partially support** → shift emphasis (e.g., "faster" instead of "more accurate")
- **Results contradict** → pivot the idea, redefine success criteria
- **Unexpected finding** → expand the idea with a new contribution

The paper (Stage 5) is written based on the **final evolved idea**, and the review (Stage 6) cross-checks that the paper accurately reflects all experimental findings.

See `paper_generate.md` for full details on the memory architecture and event schema.

## Tips

- **Start focused.** "Compare 3 optimizers on CIFAR-10" works better than "solve computer vision."
- **Iterate.** If the Stage 6 review finds issues, ask Claude Code to go back and fix them.
- **Be specific about metrics.** Tell Claude Code exactly what matters for your domain.
- **Provide data if needed.** Put datasets in `data/` or give download instructions.
- **Verify numbers.** Always check that the paper's tables match the actual experiment output.

## Comparison with AI-Scientist-v2

| | AI-Scientist-v2 | This Repo |
|---|---|---|
| Requires API keys | Yes (OpenAI / Anthropic) | No |
| Orchestration | Python scripts | Claude Code skills |
| Setup complexity | High (env, keys, configs) | Clone + run |
| Cost | API usage fees | Claude subscription only |
| Flexibility | Fixed pipeline | Conversational, interactive |
| Idea evolution | Static hypothesis | Dynamic: idea updates based on results |

## Credits

- The skills and pipeline design are derived from [SakanaAI/AI-Scientist-v2](https://github.com/SakanaAI/AI-Scientist-v2). This repo adapts their approach for native use within Claude Code.
- The long-term memory and idea evolution system is inspired by [facebookresearch/Hyperagents](https://github.com/facebookresearch/Hyperagents), which uses a multi-layered memory architecture (append-only event logs, evolutionary lineage tracking, and performance-based selection) to enable agents to learn and evolve across generations.

## License

This project follows the same license terms as [AI-Scientist-v2](https://github.com/SakanaAI/AI-Scientist-v2).
