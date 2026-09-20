---
name: cross-species-viral-genome-comparison
description: 'Use when choosing or defending an overlap/identity cutoff for viral
  read recruitment or strain separation: you need to know how much sequence two viral
  genomes or a virus and another microbe can share by chance before setting the threshold.'
when_to_use_negative:
- The virus of interest shares no conservation with any reference genome — TAR-VIR's
  recruitment stage cannot detect a completely new virus, so LCS-based cutoff calibration
  is moot
- You already have a validated overlap cutoff for your read length and platform and
  only need to run recruitment (see TAR-VIR's -c parameter), not re-derive it
- Your genomes are so divergent that pairwise alignment statistics (not exact-substring
  measures) are needed to capture homology
edam_operation: http://edamontology.org/operation_0491
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3174
tools:
- name: TAR-VIR
  role: 'Consumer of the LCS-derived overlap cutoff: the Overlap_extension module
    recruits reads sharing overlaps longer than the cutoff (-c) to isolate and enrich
    target-virus reads'
  repo: https://github.com/chjiao/TAR-VIR
- name: Python
  role: Implementation language for computing pairwise LCS sizes (generalized suffix
    array / dynamic programming) and building the ROC/AUC analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-019-2878-2
    title: 'TAR-VIR: a pipeline for TARgeted VIRal strain reconstruction from metagenomic
      data'
  source_content_hash: sha256:0e39f6331f58bb4d08c91f6a2a7bd110504f1b819a72c7127a211346dec2f513
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chjiao/TAR-VIR
  tools:
  - Python
  - TAR-VIR
  edam_operation: http://edamontology.org/operation_0491
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cross-species-viral-genome-comparison

## Summary

Quantifies sequence conservation between viral (and virus-vs-other-microbe) genome pairs by computing longest common substring (LCS) sizes, then uses an ROC analysis over candidate LCS cutoffs to justify overlap thresholds for read recruitment in TAR-VIR. This grounds a cutoff such as 100 bp in the empirical separation between within-quasispecies and cross-species shared strings.

## When to use

Use when choosing or defending an overlap/identity cutoff for viral read recruitment or strain separation: you need to know how much sequence two viral genomes or a virus and another microbe can share by chance before setting the threshold.

## When NOT to use

- The virus of interest shares no conservation with any reference genome — TAR-VIR's recruitment stage cannot detect a completely new virus, so LCS-based cutoff calibration is moot
- You already have a validated overlap cutoff for your read length and platform and only need to run recruitment (see TAR-VIR's -c parameter), not re-derive it
- Your genomes are so divergent that pairwise alignment statistics (not exact-substring measures) are needed to capture homology

## Inputs

- Viral RefSeq genome records from NCBI (e.g. HBV NC_003977.2, HCV genotypes NC_009827.1–NC_009824.1, HPgV NC_001710.1)
- Non-viral microbial RefSeq genomes for virus-vs-other-microbe comparisons (~142 million pairs)
- Read length and insert-size parameters of the target sequencing setup (e.g. 250 bp MiSeq reads)

## Outputs

- LCS size distribution for all within-virus and virus-vs-other genome pairs
- Extreme cross-species LCS values (Vaccinia-Variola 469 bp; HCV g7-g5 154 bp)
- ROC curve and AUC quantifying separation of within-quasispecies vs cross-species LCS at each candidate cutoff (expected AUC close to 1)
- A justified overlap cutoff (e.g. 100 bp) written to a results table such as lcs_roc_analysis.csv

## How to apply

Download the relevant RefSeq genomes (e.g. HBV NC_003977.2, eight HCV genotypes NC_009827.1/NC_009823.1/NC_009825.1/NC_030791.1/NC_004102.1/NC_009826.1/NC_009824.1, HPgV NC_001710.1, plus non-viral microbial genomes) from NCBI. Compute the LCS size for every genome pair using an efficient string algorithm such as a generalized suffix array or dynamic programming, as described in Additional file 1 Section 1. Build an ROC curve where, for candidate cutoff l, the true positive rate is the probability that two strains within a quasispecies share an LCS of at least l and the false positive rate is the probability that two different microbial species share an LCS of at least l; an AUC close to 1 validates the cutoff. Verify extreme cross-species values (Vaccinia vs Variola 469 bp; HCV genotype 7 vs genotype 5 154 bp) and confirm only a small fraction of the ~142 million virus-vs-other pairs (e.g. 86 pairs) exceed 100 bp. In TAR-VIR the overlap threshold should also be set longer than half the read size to prevent recruiting chimeric reads, so combine the LCS statistics with the read length (e.g. 250 bp MiSeq reads) when selecting the final cutoff.

## Related tools

- **TAR-VIR** (Consumer of the LCS-derived overlap cutoff: the Overlap_extension module recruits reads sharing overlaps longer than the cutoff (-c) to isolate and enrich target-virus reads) — https://github.com/chjiao/TAR-VIR
- **Python** (Implementation language for computing pairwise LCS sizes (generalized suffix array / dynamic programming) and building the ROC/AUC analysis)

## Examples

```
python lcs_roc_analysis.py --genomes hbv.fa hcv_g1-8.fa hpgv.fa microbes.fa --pairs virus_vs_other --roc --cutoffs 100,150 --out lcs_roc_analysis.csv
```

## Evaluation signals

- Recomputed Vaccinia-vs-Variola LCS equals the reported 469 bp
- Recomputed HCV genotype 7 vs genotype 5 LCS equals the reported 154 bp
- ROC curve over candidate LCS cutoffs achieves an AUC close to 1
- Only a small number of the ~142 million virus-vs-other pairs (e.g. 86 pairs) exceed a 100 bp LCS, i.e. near-zero false positive rate at cutoffs above 100 bp
- Chosen cutoff also satisfies the rule that the overlap threshold be longer than half the read size (e.g. >125 bp for 250 bp reads) to prevent chimeric read recruitment

## Limitations

- Only a small number of cross-species pairs exceed the 100 bp cutoff, but they exist (e.g. 86 of ~142 million), so no cutoff perfectly eliminates false positives
- The analysis justifies cutoffs empirically for the surveyed genomes; it does not guarantee specificity for unassayed genome pairs
- The overlap cutoff must be reconciled with read length: thresholds longer than half the read size are needed to avoid chimeras, which may conflict with the LCS-derived minimum
- A completely new virus sharing no conservation with any reference genome will not be detected by this LCS-based reasoning or by TAR-VIR recruitment

## Evidence

- [supplementary] the largest between-virus LCS: [evidence span withheld — anchor status: not_found]
- [supplementary] ROC curve definition: [evidence span withheld — anchor status: not_found]
- [methods] chimera-prevention rule: "we set the overlap threshold longer than half of the read size to prevent recruiting these chimeric reads"
- [discussion] cutoff choice guidance: "We have provided guidance on choosing appropriate overlap cutoffs based on the analysis of the common string sizes between different species."
- [intro] new-virus limitation: [evidence span withheld — anchor status: not_found]
- [readme] overlap module usage: "./overlap -S align.sam -x prefix -f reads.fa -c overlap_cutoff -o recruited_reads.fa"
