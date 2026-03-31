---
name: write-paper
description: Write a complete scientific paper in LaTeX from experiment results, figures, and citations. Use this skill when the user wants to write a paper, generate a LaTeX document, create a PDF paper, draft a research manuscript, or compile a paper from results. Also use when discussing paper structure, LaTeX formatting, paper sections, or the user says "write it up" or "draft the paper." Supports both 4-page workshop and 8-page conference formats.
---

# Write Paper

You are writing a complete scientific paper in LaTeX. Your job is to synthesize the research idea, experiment results, figures, and citations into a well-structured, clearly written manuscript that compiles into a PDF.

## Prerequisites

- `idea.md` — the research plan
- Experiment summaries (`baseline_summary.json`, `research_summary.json`, `ablation_summary.json`)
- `figures/` directory with publication-quality plots
- `references.bib` — bibliography
- Optionally: `citation_notes.json` — where each citation belongs

## Step-by-Step Process

### 1. Choose Paper Format

| Format | Pages | Use Case |
|--------|-------|----------|
| Workshop (ICBINB-style) | 4 | Short focused paper |
| Conference (NeurIPS-style) | 8 | Full research paper |

### 2. Create the LaTeX Project

Create a `latex/` directory and write `template.tex`. Use a standard ML venue template or start from a clean article class.

**Minimal LaTeX skeleton:**

```latex
\documentclass{article}
\usepackage[utf8]{inputenc}
\usepackage{amsmath,amssymb}
\usepackage{graphicx}
\usepackage{booktabs}
\usepackage{hyperref}
\usepackage{natbib}
\usepackage[margin=1in]{geometry}

\title{Your Paper Title}
\author{Author Names}
\date{}

\begin{document}
\maketitle

\begin{abstract}
% 150-250 words summarizing the problem, method, results, and conclusion
\end{abstract}

\section{Introduction}
% What problem? Why important? What's your approach? What are the results?

\section{Related Work}
% Position your work relative to existing literature

\section{Method}
% Detailed description of your approach

\section{Experiments}
% Experimental setup, datasets, baselines, implementation details

\subsection{Results}
% Main results with tables and figures

\subsection{Ablation Studies}
% Component analysis

\section{Discussion}
% Interpret results, limitations, broader impact

\section{Conclusion}
% Summary of contributions and future work

\bibliographystyle{plainnat}
\bibliography{references}

\appendix
\section{Additional Results}
% Extra figures, tables, proofs

\end{document}
```

### 3. Write Each Section

#### Abstract (150-250 words)
Structure: Problem → Gap → Method → Key Result → Implication.
Write this LAST, after all other sections are complete.

#### Introduction (1-1.5 pages)
- Paragraph 1: Problem context and motivation
- Paragraph 2: Why existing approaches fall short
- Paragraph 3: Your approach and key insight
- Paragraph 4: Summary of contributions (use a bulleted list)
- Paragraph 5: Paper organization (optional)

#### Related Work (0.5-1 page)
- Group by theme, not chronologically
- For each group: summarize the approaches, then explain how yours differs
- Cite every paper from `references.bib` that's relevant
- Be fair — acknowledge prior work's strengths

#### Method (1-2 pages)
- Start with a high-level overview (architecture diagram reference if applicable)
- Then detail each component
- Include mathematical formulations where appropriate
- Reference figures that illustrate the method

#### Experiments (1.5-3 pages)
- **Setup**: datasets, metrics, baselines, implementation details (learning rate, epochs, etc.)
- **Main results**: table comparing all methods, reference the comparison figure
- **Ablation studies**: table or figure showing impact of each component
- Use `\ref{fig:...}` and `\ref{tab:...}` for cross-references

#### Discussion / Conclusion (0.5-1 page)
- Interpret key findings
- Acknowledge limitations honestly
- Suggest future work directions

### 4. Include Figures

Reference figures from `figures/` directory:

```latex
\begin{figure}[t]
\centering
\includegraphics[width=\linewidth]{figures/learning_curves.png}
\caption{Training convergence comparison. Our method (blue) converges
faster than the baseline (orange dashed) and achieves lower final loss.}
\label{fig:learning_curves}
\end{figure}
```

Rules for figures:
- Every figure in `figures/` should be referenced at least once
- Place figures near their first reference in the text
- Write captions that are self-contained (a reader should understand the figure from the caption alone)
- Use `[t]` or `[h]` placement, not `[H]` (unless necessary)

### 5. Create Results Tables

```latex
\begin{table}[t]
\centering
\caption{Comparison of methods on the test set. Best results in \textbf{bold}.
Mean $\pm$ std over 3 seeds.}
\label{tab:main_results}
\begin{tabular}{lcc}
\toprule
Method & Accuracy (\%) & F1 Score \\
\midrule
Random Baseline & $50.2 \pm 1.1$ & $0.48 \pm 0.02$ \\
Linear Model & $72.3 \pm 0.8$ & $0.70 \pm 0.01$ \\
Prior Work \citep{smith2023method} & $85.1 \pm 0.5$ & $0.83 \pm 0.01$ \\
\textbf{Ours} & $\mathbf{91.2 \pm 0.3}$ & $\mathbf{0.90 \pm 0.01}$ \\
\bottomrule
\end{tabular}
\end{table}
```

### 6. Compile the PDF

```bash
cd latex
pdflatex -interaction=nonstopmode template.tex
bibtex template
pdflatex -interaction=nonstopmode template.tex
pdflatex -interaction=nonstopmode template.tex
```

Run pdflatex 3 times to resolve all cross-references.

If `pdflatex` is not available, note the LaTeX source is complete and the user can compile it with their preferred TeX distribution (Overleaf, TexLive, MiKTeX).

### 7. Self-Review and Revise (3 rounds)

After the first draft compiles, do up to 3 revision rounds:

**Round 1 — Structural review:**
- Does the paper flow logically?
- Are all figures and tables referenced?
- Is the page limit respected? (check by counting pages in the PDF)
- Are there any LaTeX errors or warnings?

**Round 2 — Content review:**
- Are claims supported by evidence?
- Are results accurately described (match the actual numbers)?
- Is the writing clear and concise?
- Are all citations properly placed?

**Round 3 — Polish:**
- Fix grammar and awkward phrasing
- Ensure consistent terminology throughout
- Check that the abstract accurately summarizes the final paper
- Verify figure/table captions are self-contained and informative

### 8. Page Limit Management

If over the page limit:
- Combine related figures into subplots
- Move less critical results to the appendix
- Tighten prose (eliminate redundancy)
- Reduce whitespace in tables

If under the page limit:
- Add more analysis or discussion
- Include additional experimental details
- Expand related work coverage

## Writing Quality Guidelines

- **Be precise**: "improves accuracy by 7.3%" not "significantly improves accuracy"
- **Be honest**: acknowledge limitations and negative results
- **Be consistent**: same term for the same concept throughout
- **Avoid filler**: "It is worth noting that" → just state the fact
- **Active voice**: "We propose" not "A method is proposed"
- **No plagiarism**: rephrase ideas from cited work in your own words

## Output

```
experiments/{idea_name}/
├── latex/
│   ├── template.tex        # Complete LaTeX source
│   └── template.pdf        # Compiled PDF (if pdflatex available)
└── references.bib          # (already exists from gather-citations)
```
