---
name: idea-setup
description: Set up a research experiment workspace from a research idea. Use this skill when the user wants to start a new research project, create an experiment from an idea, prepare a workspace for ML experiments, or structure a research idea into a runnable project. Also use when the user describes a research concept and wants to turn it into an organized experiment plan.
---

# Idea Setup

You are setting up a structured research experiment workspace from a research idea. The goal is to take a vague or detailed research concept and produce a well-organized project directory with a clear experiment plan that subsequent stages can build on.

## What You Produce

A workspace directory with everything needed to start experimenting, including initialized long-term memory:

```
experiments/{idea_name}/
├── idea.md              # Structured research plan (version 1)
├── idea.json            # Machine-readable metadata (version 1)
├── idea_evolution.md    # Idea changelog (initialized with version 1)
├── research_log.jsonl   # Append-only event log (initialized)
├── config.yaml          # Experiment configuration
├── slr_synthesis.md     # (if SLR/ exists) Literature review synthesis
├── data/                # For datasets (empty initially)
├── logs/                # For experiment logs
├── experiment_results/  # For results, .npy files, intermediate plots
└── figures/             # For final publication plots
```

## Step-by-Step Process

### 0. Read SLR Materials (if available)

If an `SLR/` directory exists in the project root, read all `.md` files in it before proceeding. These contain systematic literature review notes. Synthesize them to:

- Understand the research landscape and state of the art
- Identify research gaps and open questions
- Extract baselines, datasets, and methods mentioned in the reviewed papers
- Generate or refine the research idea based on opportunities found

Write an `slr_synthesis.md` file in the experiment workspace summarizing:
- Key findings from the literature
- Identified research gaps
- How the proposed idea addresses these gaps
- Relevant papers and their contributions (for later citation gathering)

### 1. Clarify the Research Idea

If the user gives a vague idea, ask clarifying questions to pin down:
- **What is the core hypothesis or question?**
- **What method/approach will be tested?**
- **What datasets or data generation approach will be used?**
- **What metrics define success?**
- **What baselines should be compared against?**

If the user provides a detailed idea (or a JSON file), extract these directly.
If SLR materials were read in Step 0, use the synthesis to inform and strengthen these answers.

### 2. Create the Workspace Directory

```bash
mkdir -p experiments/{idea_name}/{data,logs,experiment_results,figures}
```

Use a clean, lowercase, underscore-separated name derived from the idea title.

### 3. Write idea.md

This is the core document that all subsequent stages reference. Structure it as:

```markdown
## Title
{Full title of the research}

## Problem Statement
{What problem does this address? Why does it matter?}

## Hypothesis
{The specific claim being tested}

## Proposed Method
{Detailed description of the approach}

## Experiment Plan

### Stage 1: Initial Implementation
- Implement the basic version of the proposed method
- Get it running on a small-scale version of the data
- Verify correctness with sanity checks

### Stage 2: Baseline Comparison
- Implement or obtain baseline methods
- Run all methods on the same data with the same evaluation protocol
- Tune hyperparameters fairly for both baseline and proposed method

### Stage 3: Full Experiments
- Scale up to full dataset
- Run multiple seeds for statistical significance
- Generate all comparison plots and tables

### Stage 4: Ablation Studies
- Identify key components of the proposed method
- Systematically remove/modify each component
- Measure the impact on performance

## Evaluation Metrics
{List specific metrics: accuracy, F1, BLEU, loss, etc.}

## Expected Outcomes
{What results would support the hypothesis?}

## Potential Risks and Mitigations
{What could go wrong? How to handle it?}
```

Adapt the structure to the specific research area. For instance, a generative modeling project might have stages around sample quality, while a reinforcement learning project might focus on reward curves.

### 4. Write idea.json

Store machine-readable metadata with version tracking:

```json
{
  "Name": "idea_short_name",
  "Title": "Full Research Title",
  "Experiment": "Brief experiment description",
  "Metrics": ["accuracy", "f1_score"],
  "Baselines": ["random", "linear_model"],
  "Datasets": ["CIFAR-10"],
  "Interestingness": 8,
  "Feasibility": 7,
  "Novelty": 7,
  "version": 1,
  "last_updated": "2026-03-31T10:00:00",
  "update_reason": "Initial idea"
}
```

### 5. Write config.yaml

Basic experiment configuration:

```yaml
# Experiment settings
experiment_name: "{idea_name}"
seed: 42
num_seeds: 3

# Data
data_dir: "data"
dataset: "{dataset_name}"

# Training
epochs: 100
batch_size: 64
learning_rate: 0.001
optimizer: "adam"

# Evaluation
eval_every: 10
metrics: ["accuracy", "loss"]

# Resources
device: "cuda"  # or "cpu"
num_workers: 4

# Output
results_dir: "experiment_results"
figures_dir: "figures"
log_dir: "logs"
```

Adjust the config to match the specific experiment type (e.g., add `latent_dim` for VAEs, `gamma` for RL).

### 6. Initialize Long-Term Memory

Create the memory files that track idea evolution throughout the pipeline:

**`research_log.jsonl`** — Append-only event log. Initialize with:
```jsonl
{"timestamp": "<current_time>", "event": "idea_initialized", "version": 1, "summary": "<one-line hypothesis>", "source": "<user|SLR|user+SLR>"}
```

**`idea_evolution.md`** — Human-readable changelog. Initialize with:
```markdown
# Idea Evolution Log

This document tracks how the research idea evolves based on experiment results.

## Version 1 (Initial)
- **Hypothesis:** <the specific claim being tested>
- **Source:** <where the idea came from: user input, SLR analysis, or both>
- **Key assumptions:** <what must be true for the hypothesis to hold>
- **Success criteria:** <what experimental results would support the hypothesis>
```

These memory files will be updated by the `run-experiments` skill whenever experiment results require changes to the hypothesis.

### 7. Optionally Scaffold Starter Code

If the idea involves common ML patterns, create a minimal `runfile.py` skeleton:

```python
import numpy as np
import json
import os

# ============================================================
# TODO: Implement your experiment here
# ============================================================

def main():
    # Load config
    # Load/generate data
    # Train model
    # Evaluate
    # Save results as .npy files to experiment_results/
    # Print metrics to stdout in format: "METRIC_NAME: VALUE"
    pass

if __name__ == "__main__":
    main()
```

The key convention: experiments should print metrics to stdout as `METRIC_NAME: VALUE` and save numerical data as `.npy` files in `experiment_results/`.

## Output Conventions

These conventions ensure downstream stages (plotting, writeup, review) can find what they need:

- **Metrics** → printed to stdout as `metric_name: value`
- **Numerical data** → saved as `.npy` files in `experiment_results/`
- **Intermediate plots** → saved as `.png` in `experiment_results/`
- **Final plots** → saved as `.png` in `figures/`
- **Summaries** → saved as `.json` in the experiment root
