---
name: insertion-length-summary-statistics
description: Use when you have an rRNA insertion catalog keyed to 16S rRNA genes and
  need to quantify what fraction of genes carry large (≥10 bp) insertions and the
  max/mean/s.d. of insertion lengths.
when_to_use_negative:
- The insertion catalog includes only insertions shorter than 10 bp, so the ≥10 bp
  flagging rule would flag no genes and statistics would be vacuous
- The goal is intron identification or splicing validation (e.g. metatranscriptomic
  mapping to show insertions are not retained in transcribed RNAs) rather than length/frequency
  summary statistics
- The gene set is not the curated 1,543-gene ≥800 bp set (e.g. raw uncurated assemblies),
  so the 31% fraction denominator would not match the paper's value
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0199
tools:
- name: pandas/numpy
  role: Lightweight Python computation used to apply the ≥10 bp cutoff and compute
    the fraction, maximum, mean, and sample standard deviation of insertion lengths
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:d8b5114a1588bda34debcdd606ebc9554992e12b41f2c6638c5671bb812f4c21
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - pandas/numpy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# insertion-length-summary-statistics

## Summary

Recomputes the paper's summary statistics for large insertions in bacterial 16S rRNA genes: the fraction of the 1,543 curated genes (≥800 bp) carrying a ≥10 bp insertion, plus the maximum (2,004 bp), mean (519 bp), and standard deviation (372 bp) of insertion lengths.

## When to use

Use when you have an rRNA insertion catalog keyed to 16S rRNA genes and need to quantify what fraction of genes carry large (≥10 bp) insertions and the max/mean/s.d. of insertion lengths.

## When NOT to use

- The insertion catalog includes only insertions shorter than 10 bp, so the ≥10 bp flagging rule would flag no genes and statistics would be vacuous
- The goal is intron identification or splicing validation (e.g. metatranscriptomic mapping to show insertions are not retained in transcribed RNAs) rather than length/frequency summary statistics
- The gene set is not the curated 1,543-gene ≥800 bp set (e.g. raw uncurated assemblies), so the 31% fraction denominator would not match the paper's value

## Inputs

- Curated 16S rRNA gene set: 1,543 bacterial genes ≥800 bp (Supplementary Data 2)
- rRNA insertion catalog with per-gene insertion boundary/length records (Supplementary Table 5), derived from gaps in the HMM/covariance-model alignment against the rRNA model

## Outputs

- Single JSON record with four metrics: fraction of genes with ≥10 bp insertions (expected 31%), maximum insertion length (expected 2,004 bp), mean insertion length (expected 519 bp), standard deviation of insertion lengths (expected 372 bp)

## How to apply

Load the curated 16S rRNA gene set (1,543 bacterial genes ≥800 bp, Supplementary Data 2) and the insertion catalog (Supplementary Table 5), where insertion boundaries were defined by large gaps in the HMM/covariance-model alignment against the rRNA model. Apply the ≥10 bp length cutoff — mirroring the paper's rule that 'all insertions ≥10 bp were removed before multiple sequence alignment and phylogenetic analysis' — to flag which genes carry a large insertion. Compute the fraction over the 1,543-gene denominator (count genes carrying at least one qualifying insertion, not the number of insertions), and report the maximum, mean, and sample standard deviation (ddof=1) of insertion lengths. Write the four metrics as a single structured JSON record.

## Related tools

- **pandas/numpy** (Lightweight Python computation used to apply the ≥10 bp cutoff and compute the fraction, maximum, mean, and sample standard deviation of insertion lengths)

## Examples

```
python -c "import pandas as pd; L=pd.read_csv('insertions.tsv',sep='\t').query('len>=10')['len']; g=pd.read_csv('genes.csv'); print(L.max(), L.mean(), L.std(ddof=1), len(g[g.gene.isin(L.index)])/len(g))"
```

## Evaluation signals

- Recomputed fraction of genes with ≥10 bp insertions reproduces 31% within rounding tolerance when the denominator is the 1,543 curated genes ≥800 bp
- Recomputed maximum, mean, and standard deviation of insertion lengths reproduce 2,004 bp, 519 bp, and 372 bp respectively within rounding tolerance
- Gene-level flagging is used (a gene with multiple ≥10 bp insertions counts once toward the 31%), not insertion-level counting
- Standard deviation is the sample s.d. (ddof=1), matching the reported 372 bp
- Insertions below the 10 bp threshold are excluded before all statistics are computed

## Limitations

- Insertion boundaries depend on gaps detected in the HMM/covariance-model alignment against the rRNA model; mis-modeled divergent 16S sequences could be mistaken for insertions
- The ≥10 bp cutoff is a curation convention: all such insertions were removed before multiple sequence alignment and phylogenetic analysis, so the statistics describe genes as curated, not raw sequences
- Insertions are probably self-splicing introns not retained in transcribed RNAs, so these lengths do not reflect mature rRNA content

## Evidence
