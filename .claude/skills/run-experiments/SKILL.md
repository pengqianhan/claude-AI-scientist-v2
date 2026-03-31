---
name: run-experiments
description: Write and iteratively run ML experiment code through 4 stages (implementation, baselines, full experiments, ablations). Use this skill when the user wants to run experiments for a research paper, implement and test a research idea, iterate on experiment code, debug ML training scripts, or generate experiment results. Also use when the user asks to "run the experiments," "implement the method," or "get the results."
---

# Run Experiments

You are an ML researcher implementing and running experiments for a scientific paper. Your job is to write Python code, execute it, analyze the results, fix bugs, and iterate — progressing through 4 experiment stages until you have comprehensive results.

## Prerequisites

- A workspace set up by the `idea-setup` skill (or equivalent) containing `idea.md`, `idea.json`, and `config.yaml`
- Initialized long-term memory: `research_log.jsonl` and `idea_evolution.md`
- Python environment with relevant ML libraries installed (numpy, torch/tensorflow, sklearn, matplotlib, etc.)

## Core Principle: Idea-Experiment Feedback Loop

The research idea is **not static**. After each experiment phase, evaluate whether results support the current hypothesis. If they don't, update the idea — shift claims, change metrics, or pivot the approach. This mirrors real research: hypotheses evolve as evidence accumulates.

**After every phase, run this feedback protocol:**

1. **Log** the experiment results to `research_log.jsonl`
2. **Evaluate** results against the current hypothesis in `idea.md`
3. **Classify** the outcome:
   - **Supports** → log confirmation, proceed
   - **Partially supports** → update hypothesis to emphasize what works
   - **Contradicts** → pivot the idea, redefine success criteria
   - **Unexpected finding** → expand the idea to include new contribution
4. **If idea needs updating:**
   - Read current `idea.json` to get the version number
   - Append a new version entry to `idea_evolution.md` (what changed, why, impact)
   - Rewrite `idea.md` with the updated hypothesis, method, or metrics
   - Update `idea.json` (increment version, update `last_updated` and `update_reason`)
   - Append `idea_updated` event to `research_log.jsonl`

**Never ignore contradictory evidence.** If the experiment doesn't support the claim, update the claim — don't cherry-pick results.

## The 4 Experiment Stages

Work through these stages sequentially. Each stage builds on the previous one's results.

### Stage 1: Initial Implementation (get it working)

**Goal:** Produce a correct, running implementation of the proposed method.

1. Read `idea.md` (check version in `idea.json`) to understand what needs to be implemented
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
6. **→ Log & Evaluate:** Append `experiment_completed` event to `research_log.jsonl`. Assess if initial results are consistent with the hypothesis.

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

0. **Read memory:** Check `research_log.jsonl` for Phase 1 findings. Re-read `idea.md` in case it was updated.
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
6. **→ Log & Evaluate:** This is the most critical feedback point. If baselines outperform the proposed method, **update the idea**:
   - Shift the claim (e.g., from "higher accuracy" to "better efficiency-accuracy tradeoff")
   - Add new metrics that capture the actual advantage
   - Or pivot the approach entirely
   - Document the change in `idea_evolution.md` and `research_log.jsonl`

### Stage 3: Full Experiments (comprehensive results)

**Goal:** Run full-scale experiments with thorough evaluation.

0. **Read memory:** Re-read `idea.md` — it may have been updated after Phase 2. Check `idea_evolution.md` to understand what changed and why.
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
5. **→ Log & Evaluate:** Full-scale results may reveal scaling behaviors not visible in Phase 2. If findings change the story, update the idea.

### Stage 4: Ablation Studies (validate components)

**Goal:** Systematically validate which components of the proposed method matter.

0. **Read memory:** Re-read `idea.md` — ablate the *current* version of the method, which may have evolved since Stage 1.
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
6. **→ Log & Evaluate:** If ablations show a component is unnecessary, consider simplifying the method and updating `idea.md`.

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
├── idea.md                       # Current idea (final version, may differ from v1)
├── idea.json                     # Metadata with version number
├── idea_evolution.md             # Complete history of idea changes
├── research_log.jsonl            # Full event log (all experiments + idea updates)
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
