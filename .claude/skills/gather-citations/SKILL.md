---
name: gather-citations
description: Search for and compile academic citations for a research paper into a BibTeX bibliography. Use this skill when the user wants to find references, build a references.bib file, collect citations for a paper, search for related work, or create a bibliography. Also use when the user mentions "find papers," "related work," "references," or "BibTeX." This skill uses web search and the Semantic Scholar API.
---

# Gather Citations

You are building the bibliography for a scientific paper. Your job is to systematically search for relevant academic papers and compile them into a `references.bib` file with proper BibTeX entries.

## Prerequisites

- `idea.md` — the research idea (to know what to cite)
- Experiment summaries (baseline, research, ablation JSONs) — to know what methods/datasets were used
- Web search capability (for finding papers)

## Step-by-Step Process

### 1. Identify Citation Needs

Read `idea.md` and the experiment summaries. Make a checklist of papers you need to find:

**Category checklist:**
- [ ] The core problem being addressed (seminal papers in the area)
- [ ] The specific method/technique proposed (prior work on similar approaches)
- [ ] Datasets used (original dataset papers)
- [ ] Models and architectures referenced (e.g., ResNet, Transformer, GPT)
- [ ] Optimizers and training techniques (Adam, learning rate schedules)
- [ ] Evaluation metrics (if using non-standard metrics, cite the source)
- [ ] Baseline methods (papers for each baseline compared against)
- [ ] Related/competing approaches (recent work on the same problem)
- [ ] Theoretical foundations (if the method builds on specific theory)

### 2. Search for Papers

For each citation need, search using the Semantic Scholar API or web search.

**Using Semantic Scholar API (preferred):**

```bash
curl -s "https://api.semanticscholar.org/graph/v1/paper/search?query=attention+is+all+you+need&limit=5&fields=title,authors,year,venue,externalIds,citationCount,abstract" | python -m json.tool
```

**Using web search:**

Search for `"{paper title}" site:arxiv.org` or `"{paper title}" site:semanticscholar.org`.

### 3. Build BibTeX Entries

For each paper found, create a proper BibTeX entry. Use this format:

```bibtex
@article{vaswani2017attention,
  title={Attention is All You Need},
  author={Vaswani, Ashish and Shazeer, Noam and Parmar, Niki and Uszkoreit, Jakob and Jones, Llion and Gomez, Aidan N and Kaiser, {\L}ukasz and Polosukhin, Illia},
  journal={Advances in Neural Information Processing Systems},
  volume={30},
  year={2017}
}
```

**BibTeX key convention:** `{first_author_lastname}{year}{first_word_of_title}` in lowercase. Example: `vaswani2017attention`.

**Entry types:**
- `@article` — journal papers, arXiv preprints
- `@inproceedings` — conference papers (NeurIPS, ICML, ICLR, etc.)
- `@book` — textbooks
- `@misc` — software packages, websites, datasets

### 4. Iterative Gathering

Work through citations in rounds. Each round:

1. Check the checklist — what's still missing?
2. Search for the most important missing citation
3. Verify the paper exists and is relevant (read the abstract)
4. Create the BibTeX entry
5. Append to `references.bib`
6. Update the checklist

**Deduplication:** Before adding a new entry, check if a paper with the same title (case-insensitive) already exists in the file.

**Stop when:**
- All checklist categories have at least one citation
- You have 15-30 references total (typical for a 4-8 page paper)
- No more important citations are missing

### 5. Quality Checks

After gathering, verify:

- [ ] Every BibTeX key is unique
- [ ] No duplicate papers (same paper under different keys)
- [ ] Author names are properly formatted (`{Last}, First`)
- [ ] Years are correct
- [ ] Venue names are consistent (e.g., always "NeurIPS" not sometimes "NIPS")
- [ ] All entries have required fields (title, author, year)
- [ ] ArXiv papers have the `eprint` field

### 6. Create Citation Annotations

Optionally, create a `citation_notes.json` that maps each citation to where it should be used in the paper:

```json
{
  "vaswani2017attention": {
    "description": "Introduced the Transformer architecture with self-attention",
    "use_in_paper": "Related work section, when discussing attention mechanisms"
  },
  "he2016deep": {
    "description": "ResNet - deep residual learning",
    "use_in_paper": "Method section, as the backbone architecture"
  }
}
```

This helps the paper-writing stage place citations correctly.

## Common Citation Sources

| Source | How to Search | Best For |
|--------|--------------|----------|
| Semantic Scholar API | `curl` as shown above | Structured search, getting BibTeX |
| Google Scholar | Web search with `site:scholar.google.com` | Broad coverage |
| arXiv | Web search with `site:arxiv.org` | Preprints, recent papers |
| DBLP | `https://dblp.org/search?q={query}` | Computer science venues |

## BibTeX Template Reference

```bibtex
% Conference paper
@inproceedings{author2024title,
  title={Paper Title},
  author={Last, First and Last2, First2},
  booktitle={Proceedings of the International Conference on Machine Learning},
  year={2024}
}

% Journal paper
@article{author2024title,
  title={Paper Title},
  author={Last, First and Last2, First2},
  journal={Journal Name},
  volume={42},
  number={1},
  pages={1--15},
  year={2024}
}

% ArXiv preprint
@article{author2024title,
  title={Paper Title},
  author={Last, First and Last2, First2},
  journal={arXiv preprint arXiv:2401.12345},
  year={2024}
}
```

## Output

```
experiments/{idea_name}/
├── references.bib         # Complete BibTeX bibliography
└── citation_notes.json    # Where each citation should be used (optional)
```
