---
name: run-experiments
description: Write and iteratively run ML experiment code through 4 stages (implementation, baselines, full experiments, ablations). Use this skill when the user wants to run experiments for a research paper, implement and test a research idea, iterate on experiment code, debug ML training scripts, or generate experiment results. Also use when the user asks to "run the experiments," "implement the method," or "get the results."
---

# Run Experiments

You are an ML researcher implementing and running experiments for a scientific paper. Your job is to write Python code, execute it, analyze the results, fix bugs, and iterate — progressing through 4 experiment stages until you have comprehensive results.

## Prerequisites

- A workspace set up by the `idea-setup` skill (or equivalent) containing `idea.md` and `config.yaml`
- Python environment with relevant ML libraries installed (numpy, torch/tensorflow, sklearn, matplotlib, etc.)

## The 4 Experiment Stages

Work through these stages sequentially. Each stage builds on the previous one's results.

### Stage 1: Initial Implementation (get it working)

**Goal:** Produce a correct, running implementation of the proposed method.

1. Read `idea.md` to understand what needs to be implemented
2. Write `runfile.py` (or a more descriptively named script) that:
   - Loads or generates the required data
   - Implements the proposed method
   - Trains/runs the model
   - Evaluates on relevant metrics
   - Prints results to stdout as `METRIC_NAME: VALUE`
   - Saves numerical results as `.npy` files in `experiment_results/`
   - Generates basic diagnostic plots (loss curves, sample outputs, etc.)
3. Run the script and check the output
4. If there are errors, debug and re-run (up to 3 debug attempts per issue)
5. Verify the results make basic sense (sanity checks)

**Output format for metrics** (print to stdout):
```
train_loss: 0.342
test_accuracy: 0.891
val_f1: 0.756
```

**Output format for data** (save to experiment_results/):
```python
np.save("experiment_results/train_losses.npy", train_losses)
np.save("experiment_results/test_accuracies.npy", test_accuracies)
```

### Stage 2: Baseline Comparison (establish context)

**Goal:** Implement baselines and run fair comparisons.

1. Implement each baseline method described in `idea.md`
2. Run all methods (proposed + baselines) under identical conditions:
   - Same data splits
   - Same evaluation protocol
   - Same number of seeds (at least 3)
   - Fair hyperparameter tuning for all methods
3. Save per-method results:
   ```python
   np.save("experiment_results/baseline_random_accuracies.npy", results)
   np.save("experiment_results/proposed_method_accuracies.npy", results)
   ```
4. Generate comparison plots (bar charts, learning curves)
5. Write a summary:
   ```python
   summary = {
       "methods": {
           "proposed": {"accuracy": {"mean": 0.89, "std": 0.02}},
           "baseline_1": {"accuracy": {"mean": 0.82, "std": 0.03}},
       },
       "best_method": "proposed",
       "analysis": "Proposed method outperforms baseline by 7% on accuracy..."
   }
   with open("baseline_summary.json", "w") as f:
       json.dump(summary, f, indent=2)
   ```

### Stage 3: Full Experiments (comprehensive results)

**Goal:** Run full-scale experiments with thorough evaluation.

1. Scale up from Stage 2:
   - Full dataset (not subsampled)
   - More evaluation metrics
   - Multiple random seeds (3-5)
   - Statistical significance tests if applicable
2. Generate comprehensive results:
   - Learning curves (loss/metric vs. epoch for all methods)
   - Performance tables (mean +/- std across seeds)
   - Visualization of model outputs (if applicable: generated samples, attention maps, etc.)
   - Distribution plots for key metrics
3. Save everything:
   ```python
   np.save("experiment_results/full_learning_curves.npy", curves)
   np.save("experiment_results/full_comparison_results.npy", results)
   ```
4. Write research summary:
   ```python
   summary = {
       "key_findings": ["..."],
       "best_configuration": {...},
       "statistical_tests": {...},
       "analysis": "..."
   }
   with open("research_summary.json", "w") as f:
       json.dump(summary, f, indent=2)
   ```

### Stage 4: Ablation Studies (validate components)

**Goal:** Systematically validate which components of the proposed method matter.

1. Identify the key components to ablate (from `idea.md` and Stage 3 results)
2. For each ablation:
   - Remove or replace one component at a time
   - Run with the same setup as Stage 3
   - Record the impact on all metrics
3. Common ablation patterns:
   - Remove a module → measure degradation
   - Replace a component with a simpler alternative
   - Vary a key hyperparameter across a range
   - Test with different dataset sizes (scaling behavior)
4. Save ablation data:
   ```python
   np.save("experiment_results/ablation_no_attention.npy", results)
   np.save("experiment_results/ablation_lr_sweep.npy", results)
   ```
5. Write ablation summary:
   ```python
   summary = {
       "ablations": [
           {
               "name": "without_attention",
               "change": "Removed self-attention layer",
               "impact": {"accuracy": -0.05, "loss": +0.12},
               "conclusion": "Attention is important for ..."
           }
       ],
       "analysis": "The most critical component is ..."
   }
   with open("ablation_summary.json", "w") as f:
       json.dump(summary, f, indent=2)
   ```

## Iteration and Debugging

For each experiment run:

1. **Execute** the script via `python runfile.py` (or the relevant script)
2. **Check output**: Read stdout for metrics and stderr for errors
3. **If error occurs**:
   - Read the traceback carefully
   - Identify the root cause (shape mismatch, missing import, data issue, etc.)
   - Fix the code
   - Re-run (max 3 debug attempts per issue)
4. **If results look wrong** (NaN, zero accuracy, loss exploding):
   - Check for common issues: learning rate too high, data not normalized, loss function mismatch
   - Add debugging prints if needed
   - Fix and re-run
5. **If results look reasonable**: Save and proceed to next stage

## Key Principles

- **Reproducibility**: Always set random seeds. Save exact configurations alongside results.
- **Fair comparison**: Give baselines the same tuning budget as the proposed method.
- **Statistical rigor**: Use multiple seeds. Report mean +/- std. Note if differences are significant.
- **Save everything**: Any number that might appear in the paper should be saved as a `.npy` file.
- **Diagnostic plots**: Generate plots as you go — they help catch bugs early.

## Final Output Structure

After all 4 stages, the workspace should contain:

```
experiments/{idea_name}/
├── idea.md
├── config.yaml
├── runfile.py                    # Main experiment script(s)
├── baseline_summary.json         # Stage 2 summary
├── research_summary.json         # Stage 3 summary
├── ablation_summary.json         # Stage 4 summary
├── experiment_results/
│   ├── train_losses.npy
│   ├── test_accuracies.npy
│   ├── baseline_*.npy
│   ├── ablation_*.npy
│   ├── full_*.npy
│   └── *.png                     # Intermediate diagnostic plots
├── figures/                      # (populated by aggregate-plots skill)
└── logs/
    └── experiment_log.txt
```
