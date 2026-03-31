# Claude AI Scientist v2

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
[Your Research Idea]
    → Stage 1: Set up experiment workspace        (skill: idea-setup)
    → Stage 2: Write & run experiments (4 phases)  (skill: run-experiments)
    → Stage 3: Create publication-quality figures   (skill: aggregate-plots)
    → Stage 4: Search & compile citations           (skill: gather-citations)
    → Stage 5: Write full LaTeX paper               (skill: write-paper)
    → Stage 6: Self-review in NeurIPS format        (skill: perform-review)
    → [Finished Paper PDF + Review JSON]
```

## Quick Start

### 1. Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) with an active Claude subscription (Pro / Team / Enterprise)
- Python 3.8+ with `numpy`, `matplotlib`
- ML framework as needed (`torch`, `tensorflow`, `sklearn`, etc.)
- (Optional) `pdflatex` + `bibtex` for PDF compilation — LaTeX source is always produced

### 2. Clone

```bash
git clone https://github.com/YOUR_USERNAME/claude-AI-scientist-v2.git
cd claude-AI-scientist-v2
```

### 3. Run the Pipeline

Open Claude Code in this repo directory and give it a prompt:

```
Read paper_generate.md. I want to generate a research paper about:

[Describe your research idea here]

Follow all 6 stages. Set up the workspace, implement and run the experiments,
create publication figures, gather citations, write the LaTeX paper, and
self-review it. Save everything under experiments/.
```

Claude Code will work through all 6 stages autonomously, using the skills defined in `.claude/skills/`.

## Project Structure

```
.
├── paper_generate.md              # Main pipeline document (the "brain")
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

## Credits

The skills and pipeline design are derived from [SakanaAI/AI-Scientist-v2](https://github.com/SakanaAI/AI-Scientist-v2). This repo adapts their approach for native use within Claude Code.

## License

This project follows the same license terms as [AI-Scientist-v2](https://github.com/SakanaAI/AI-Scientist-v2).
