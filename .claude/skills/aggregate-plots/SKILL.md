---
name: aggregate-plots
description: Create publication-quality figures from experiment results for a scientific paper. Use this skill when the user wants to generate paper figures, create publication plots, make professional charts from experiment data, aggregate multiple plots into final figures, or prepare visualizations for a paper submission. Also use when the user says "make the plots," "prepare figures," or mentions needing camera-ready figures.
---

# Aggregate Plots

You are creating the final publication-quality figures for a scientific paper. Your job is to read the experiment results (`.npy` files, JSON summaries, intermediate plots) and produce a comprehensive, professional set of figures saved to `figures/`.

## Prerequisites

- Completed experiment results in `experiment_results/` (`.npy` files, diagnostic plots)
- Summary JSONs (`baseline_summary.json`, `research_summary.json`, `ablation_summary.json`)
- `idea.md` for understanding the research context

## Step-by-Step Process

### 1. Survey Available Data

Read all summary JSONs and list all `.npy` files to understand what data is available:

```bash
ls experiment_results/*.npy
cat baseline_summary.json
cat research_summary.json
cat ablation_summary.json
```

### 2. Plan the Figures

A good ML paper typically needs these types of figures (pick what's relevant):

| Figure Type | When to Use | Example |
|-------------|-------------|---------|
| Learning curves | Almost always | Loss/accuracy vs. epoch for each method |
| Bar chart comparison | Comparing methods on metrics | Accuracy across methods with error bars |
| Ablation table/chart | Ablation studies | Performance impact of removing components |
| Hyperparameter sensitivity | Parameter sweeps | Metric vs. learning rate/hidden size |
| Sample outputs | Generative models, vision | Generated images, attention maps |
| Distribution plots | Statistical analysis | Histogram of scores, box plots |
| Scaling plots | Scaling experiments | Performance vs. dataset size/model size |
| t-SNE/UMAP | Representation learning | Learned embeddings visualization |

Aim for 4-8 figures total. Each figure should communicate one clear message.

### 3. Write the Plotting Script

Create `auto_plot_aggregator.py` in the experiment root. Follow these rules:

```python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib
import json
import os

matplotlib.rcParams.update({
    'font.size': 14,
    'axes.titlesize': 16,
    'axes.labelsize': 14,
    'xtick.labelsize': 12,
    'ytick.labelsize': 12,
    'legend.fontsize': 12,
    'figure.dpi': 300,
    'savefig.dpi': 300,
    'savefig.bbox_inches': 'tight',
})

os.makedirs("figures", exist_ok=True)

# ── Figure 1: Learning Curves ──────────────────────────────
try:
    fig, ax = plt.subplots(1, 1, figsize=(8, 5))

    proposed_loss = np.load("experiment_results/proposed_train_losses.npy")
    baseline_loss = np.load("experiment_results/baseline_train_losses.npy")

    ax.plot(proposed_loss, label="Proposed Method", linewidth=2)
    ax.plot(baseline_loss, label="Baseline", linewidth=2, linestyle="--")
    ax.set_xlabel("Epoch")
    ax.set_ylabel("Training Loss")
    ax.set_title("Training Convergence Comparison")
    ax.legend()
    ax.spines["top"].set_visible(False)
    ax.spines["right"].set_visible(False)

    plt.savefig("figures/learning_curves.png")
    plt.close()
    print("Created: figures/learning_curves.png")
except Exception as e:
    print(f"Failed to create learning curves: {e}")

# ── Figure 2: Method Comparison ────────────────────────────
try:
    # ... next figure ...
    pass
except Exception as e:
    print(f"Failed to create comparison: {e}")
```

### 4. Professional Styling Rules

Follow these to make figures publication-ready:

- **Remove top and right spines** — cleaner look
- **DPI: 300** — required for most venues
- **Font size: 14+** — readable when printed
- **No underscores in labels** — use spaces ("Test Accuracy" not "test_accuracy")
- **Informative legends** — "Proposed Method (Ours)" not "method_1"
- **Error bars / shading** — show std when you have multiple seeds
- **Consistent colors** — use the same color for the same method across all figures
- **Tight layout** — use `plt.tight_layout()` or `bbox_inches='tight'`
- **Meaningful titles** — describe the finding, not just the plot type
- **Group related plots** — use subplots (up to 3 per row) for related comparisons

### 5. Subplot Aggregation

When multiple plots are closely related, combine them:

```python
fig, axes = plt.subplots(1, 3, figsize=(18, 5))

# Left: Learning curves
axes[0].plot(...)
axes[0].set_title("(a) Training Loss")

# Middle: Validation accuracy
axes[1].plot(...)
axes[1].set_title("(b) Validation Accuracy")

# Right: Test performance
axes[2].bar(...)
axes[2].set_title("(c) Final Test Results")

plt.tight_layout()
plt.savefig("figures/main_results.png")
```

### 6. Execute and Iterate

Run the plotting script:

```bash
python auto_plot_aggregator.py
```

Check the output — look at each generated figure and ask:
- Is the message clear?
- Are labels readable?
- Are colors distinguishable?
- Is anything misleading?
- Can any plots be combined into subplots?

Fix issues and re-run. Repeat up to 5 times until the figures are publication-ready.

### 7. Common Patterns

**Error bars (multiple seeds):**
```python
mean = np.mean(results, axis=0)
std = np.std(results, axis=0)
ax.plot(epochs, mean, label="Method")
ax.fill_between(epochs, mean - std, mean + std, alpha=0.2)
```

**Bar chart with error bars:**
```python
methods = ["Baseline", "Variant A", "Proposed"]
means = [0.82, 0.85, 0.91]
stds = [0.03, 0.02, 0.01]
ax.bar(methods, means, yerr=stds, capsize=5, color=["#999", "#66b", "#c44"])
```

**Ablation heatmap:**
```python
import seaborn as sns
sns.heatmap(ablation_matrix, annot=True, fmt=".2f", xticklabels=components, yticklabels=metrics)
```

## Output

```
experiments/{idea_name}/
├── figures/
│   ├── learning_curves.png
│   ├── method_comparison.png
│   ├── ablation_results.png
│   └── ... (4-8 figures total)
└── auto_plot_aggregator.py
```

Every `.png` file in `figures/` will be available for inclusion in the paper during the write-paper stage.
