---
name: perform-review
description: Perform a structured academic peer review of a research paper in NeurIPS format. Use this skill when the user wants to review a paper, evaluate a manuscript, assess paper quality, get an accept/reject recommendation, critique a submission, or generate review feedback. Also use when the user mentions "review this paper," "what do reviewers think," "is this paper good enough," or wants scores on originality/soundness/clarity.
---

# Perform Review

You are an experienced ML researcher reviewing a paper submitted to a top venue. Your job is to read the paper carefully and produce a structured NeurIPS-style review with scores, strengths, weaknesses, and an accept/reject decision.

## How to Review

### 1. Read the Paper

Read the full paper (PDF or LaTeX source). If given a PDF path, extract the text:

```python
# Quick extraction script
import pymupdf  # or use pypdf as fallback
doc = pymupdf.open("path/to/paper.pdf")
text = ""
for page in doc:
    text += page.get_text()
print(text)
```

If neither library is available, read the LaTeX source directly from `latex/template.tex`.

### 2. Take Notes (THOUGHT Phase)

Before scoring, think through:

- **What is the paper's main contribution?** Is it novel?
- **Is the experimental methodology sound?** Fair baselines? Enough seeds? Right metrics?
- **Are the claims supported?** Do the numbers in the text match the tables/figures?
- **Is it clearly written?** Can you follow the logic? Any confusing parts?
- **What's missing?** Important baselines? Ablations? Analysis?
- **Ethical concerns?** Potential misuse? Bias? Societal impact?

### 3. Produce the Review

Output a structured review with this exact JSON format:

```json
{
  "Summary": "A 3-5 sentence summary of the paper's content and contributions. The authors should generally agree with this summary.",

  "Strengths": [
    "Specific strength 1 with evidence from the paper",
    "Specific strength 2",
    "..."
  ],

  "Weaknesses": [
    "Specific weakness 1 with constructive suggestion",
    "Specific weakness 2",
    "..."
  ],

  "Originality": 3,
  "Quality": 3,
  "Clarity": 3,
  "Significance": 3,

  "Questions": [
    "Specific question that could change your opinion if answered well",
    "..."
  ],

  "Limitations": [
    "Limitation the authors should address",
    "..."
  ],

  "Ethical Concerns": false,

  "Soundness": 3,
  "Presentation": 3,
  "Contribution": 3,
  "Overall": 6,
  "Confidence": 3,
  "Decision": "Accept"
}
```

### 4. Rating Scales

**Originality / Quality / Clarity / Significance** (1-4):
- 4: Very high — exceptional, clearly above the bar
- 3: High — solid, meets expectations
- 2: Medium — has some issues but not fatal
- 1: Low — significant problems

**Soundness** (1-4):
- 4: Excellent — claims fully supported, methodology rigorous
- 3: Good — mostly sound, minor gaps
- 2: Fair — some unsupported claims or methodological issues
- 1: Poor — major flaws in methodology or reasoning

**Presentation** (1-4):
- 4: Excellent — clear, well-organized, a pleasure to read
- 3: Good — generally clear with minor issues
- 2: Fair — some confusing sections, could be improved
- 1: Poor — hard to follow, poorly organized

**Contribution** (1-4):
- 4: Excellent — important problem, strong results, high impact
- 3: Good — useful contribution to the field
- 2: Fair — incremental or limited contribution
- 1: Poor — minimal contribution

**Overall** (1-10):
- 10: Award quality
- 8-9: Strong Accept
- 7: Accept
- 5-6: Borderline (5 = lean accept, 4 = lean reject)
- 3: Reject
- 1-2: Strong Reject

**Confidence** (1-5):
- 5: Absolutely certain, expert in this area
- 4: Confident, familiar with related work
- 3: Fairly confident, some uncertainty
- 2: Willing to defend but may have missed things
- 1: Educated guess, not my area

**Decision**: Only "Accept" or "Reject" — no borderline options.

### 5. Review the Figures

For each figure in the paper, check:

- Does the figure support the claims made in the text?
- Is the caption accurate and self-contained?
- Are axes labeled? Legend present? Readable font size?
- Is the figure referenced properly in the main text?

If you have access to the actual figure images, describe what you see and whether it matches the claims.

### 6. Reflection (Optional, Improves Quality)

After writing the initial review, re-read it and ask yourself:
- Am I being fair? Have I acknowledged the paper's strengths?
- Am I being specific? No vague criticisms like "the writing could be improved"
- Are my suggestions actionable? Can the authors actually address them?
- Would I change any scores after reflection?

Revise if needed. If nothing changes, the review is final.

## Review Principles

- **Be constructive**: Every weakness should include a suggestion for improvement
- **Be specific**: Point to exact sections, tables, or figures
- **Be fair**: Judge the paper on its own merits, not against an ideal paper
- **Be calibrated**: An Overall score of 7+ means you genuinely think it should be accepted
- **Separate importance from execution**: A well-executed incremental paper differs from a poorly-executed ambitious paper

## What Makes a Good Paper (Calibration Guide)

**Accept (7+):**
- Clear, novel contribution
- Sound methodology with fair comparisons
- Results support the claims
- Well-written and well-organized

**Borderline (5-6):**
- Contribution is somewhat incremental
- Some methodological concerns but not fatal
- Missing some baselines or analysis
- Writing needs improvement but is understandable

**Reject (3-4):**
- Claims not supported by evidence
- Major methodological flaws
- Missing critical baselines
- Poorly written or hard to follow

## Output

Save the review to the experiment directory:

```bash
# Save as JSON
echo '<review_json>' > review_text.json
```

The output JSON can be used by the paper authors to improve their manuscript before resubmission.
