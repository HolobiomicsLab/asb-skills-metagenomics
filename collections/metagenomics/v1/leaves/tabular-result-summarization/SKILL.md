---
name: tabular-result-summarization
description: Use when a paper's claims refer to numbers in a results table and you
  must verify them by loading the table, computing derived metrics (e.g. |OTUs − true
  strains|), and ranking pipelines per dataset.
when_to_use_negative:
- Input is already a feature/OTU table with precomputed quality metrics — derive new
  comparisons rather than re-summarizing
- The claim under test is qualitative or time-based (e.g. runtime benchmarking in
  seconds) and cannot be checked from the OTU-count table alone
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0699
- http://edamontology.org/topic_3174
tools:
- name: CDSnake
  role: Pipeline whose Table 1 OTU counts are summarized and compared against true
    mock strain counts
  repo: github.com/ydkondratenko/cdsnake
- name: DADA2
  role: Baseline QIIME2 pipeline whose OTU counts (e.g. 91 on Balanced) enter the
    deviation computation
- name: deblur
  role: Baseline QIIME2 pipeline whose OTU counts (e.g. 34 on Balanced) enter the
    deviation computation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-03591-6
    title: 'CDSnake: Snakemake pipeline for retrieval of annotated OTUs from paired-end
      reads using CD-HIT utilities'
  source_content_hash: sha256:425140305d5c421af4c1dfeb38b2bdd5a85c3a0434cb33bb890a0688ffa2537d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT-OTU-Miseq
  - CDSnake
  - DADA2
  - deblur
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0699
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# tabular-result-summarization

## Summary

Reproduces and summarizes a paper's result tables (here, CDSnake Table 1 OTU counts on Balanced, HMP, and Extreme mock communities) as derived metrics that can be checked against the paper's textual claims. It turns raw tabular counts into quantitative verdicts, e.g. absolute deviations |OTUs − true strains|.

## When to use

Use when a paper's claims refer to numbers in a results table and you must verify them by loading the table, computing derived metrics (e.g. |OTUs − true strains|), and ranking pipelines per dataset.

## When NOT to use

- Input is already a feature/OTU table with precomputed quality metrics — derive new comparisons rather than re-summarizing
- The claim under test is qualitative or time-based (e.g. runtime benchmarking in seconds) and cannot be checked from the OTU-count table alone

## Inputs

- Reproduced Table 1 OTU counts per pipeline (CDSnake, DADA2, deblur) per dataset (Balanced, HMP, Extreme), e.g. DADA2=91 and deblur=34 on Balanced
- True mock-community strain counts per dataset (59 for Balanced, 21 for HMP, 27 for Extreme)
- Paper's claim statements about which pipeline is closest and which outputs fewest OTUs

## Outputs

- otu_count_deviations.csv with |OTUs − true strains| per pipeline–dataset pair
- Per-claim verdict columns: closer_on_balanced_hmp, closer_on_extreme, fewer_otus_overall

## How to apply

Load the reproduced Table 1 counts (CDSnake, DADA2, and deblur OTU numbers on the Balanced, HMP, and Extreme mock datasets) plus the true mock-community strain counts (59, 21, and 27) into a pandas DataFrame. For each pipeline–dataset pair compute the absolute deviation |OTUs outputted − true strain count|, e.g. on Balanced with 59 true strains: |91 − 59| = 32 for DADA2 and |34 − 59| = 25 for deblur. Rank the three pipelines by deviation per dataset and check whether CDSnake has the smallest deviation on Balanced and HMP while DADA2/deblur are closer on Extreme; also sum total OTUs per pipeline to test the 'fewer OTUs overall' claim. Write the deviation table with per-claim verdict columns (closer_on_balanced_hmp, closer_on_extreme, fewer_otus_overall) to a CSV such as otu_count_deviations.csv, so each claim maps to a computed verdict rather than a restated assertion.

## Related tools

- **CDSnake** (Pipeline whose Table 1 OTU counts are summarized and compared against true mock strain counts) — github.com/ydkondratenko/cdsnake
- **DADA2** (Baseline QIIME2 pipeline whose OTU counts (e.g. 91 on Balanced) enter the deviation computation)
- **deblur** (Baseline QIIME2 pipeline whose OTU counts (e.g. 34 on Balanced) enter the deviation computation)

## Examples

```
import pandas as pd; df = pd.DataFrame({'otus':[91,34,55],'true':[59]*3}, index=['DADA2','deblur','CDSnake']); df['deviation']=(df.otus-df['true']).abs(); df.to_csv('otu_count_deviations.csv')
```

## Evaluation signals

- CDSnake ranks first (smallest |OTUs − true|) on Balanced (4 vs DADA2=32, deblur=25) and HMP (38 vs DADA2=47, deblur=32)
- On Extreme, DADA2 has the smallest deviation (3 vs CDSnake=15, deblur=9), not CDSnake
- The 'fewer OTUs overall' claim must fail versus deblur: CDSnake exceeds deblur on Balanced (55 vs 34), HMP (59 vs 53), and All (123 vs 39)
- Deviations are reproducible: each equals |table OTU count − true strain count| with true counts 59/21/27
- Output CSV contains all 9 pipeline–dataset rows and exactly the three verdict columns closer_on_balanced_hmp, closer_on_extreme, fewer_otus_overall

## Limitations

- Deviations are computed from reproduced table values; errors in transcription of Table 1 counts propagate directly into verdicts
- The paper's claim that 'CDSnake outputted less OTUs than DADA2 and deblur' is only partially supported — it holds versus DADA2 but not versus deblur on any dataset, so a naive summary would endorse it
- Results are specific to the three mock datasets (Balanced, HMP, Extreme) with highly overlapping 2 × 250 bp reads and do not transfer to other read layouts without recomputation

## Evidence

- [abstract] CDSnake outputted fewer OTUs than DADA2 and deblur: "CDSnake outputted less OTUs than DADA2 and deblur."
- [results] Balanced mock community composition: "The Balanced community contained 57 bacteria and archaea at nominally equal frequencies"
- [results] Extreme mock community composition: "the Extreme community contained 27 bacterial strains at frequencies spanning five orders of magnitude"
- [results] deblur 34 OTUs on Balanced: [evidence span withheld — anchor status: not_found]
- [results] DADA2 91 OTUs on Balanced: [evidence span withheld — anchor status: not_found]
