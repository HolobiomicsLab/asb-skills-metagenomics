---
name: cosine-distance-similarity-computation
description: Use when viral scaffold FASTA sequences must be compared pairwise by
  nucleotide composition (tetranucleotide frequencies, codon usage, GC content, CpG,
  GC skew) to build distance features for vMAG binning.
when_to_use_negative:
- Input is a whole metagenome rather than predicted viral scaffolds — vRhyme's performance
  on whole metagenomes 'has not been fully evaluated'
- The goal is binning microbial (bacterial/archaeal) genomes — 'vRhyme is not meant
  to bin microbes'
- Scaffolds already have a precomputed feature/distance table, so recomputing composition
  metrics is redundant
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: vRhyme
  role: Implements the nucleotide processing step that computes the pairwise composition
    distances and writes vRhyme_machine_distances.tsv
  repo: https://github.com/AnantharamanLab/vRhyme
- name: Prodigal
  role: Predicts open reading frames ('-m -p meta') supplying genes/proteins for the
    codon usage feature
  repo: https://github.com/hyattpd/Prodigal
- name: Numpy
  role: Python dependency used for vectorized frequency-vector and distance computations
- name: Pandas
  role: Python dependency used to assemble and write the scaffold-by-scaffold distance
    table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:b9c22f98d860649893909b0bb44fb7abb9b18c3bd9bf13b64fe1e502505c50bb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/AnantharamanLab/vRhyme
  repo_url: https://github.com/AnantharamanLab/vRhyme
  tools:
  - Numpy
  - Pandas
  - Prodigal
  - Python
  - vRhyme
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# cosine-distance-similarity-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes pairwise nucleotide-composition similarity between viral scaffolds in vRhyme's nucleotide processing step, using cosine distance for frequency-vector features (codon usage, relative tetranucleotide frequency, tetranucleotide usage deviation, zero'th-order Markov) and absolute difference for scalar features, yielding the length-7 feature vectors fed to vRhyme's supervised machine learning models.

## When to use

Use when viral scaffold FASTA sequences must be compared pairwise by nucleotide composition (tetranucleotide frequencies, codon usage, GC content, CpG, GC skew) to build distance features for vMAG binning.

## When NOT to use

- Input is a whole metagenome rather than predicted viral scaffolds — vRhyme's performance on whole metagenomes 'has not been fully evaluated'
- The goal is binning microbial (bacterial/archaeal) genomes — 'vRhyme is not meant to bin microbes'
- Scaffolds already have a precomputed feature/distance table, so recomputing composition metrics is redundant

## Inputs

- Viral scaffold FASTA file (predicted viruses, e.g. from VIBRANT or VirSorter), scaffolds >= 2000 bp
- Prodigal-predicted ORFs/proteins (from '-m -p meta') for codon usage features

## Outputs

- vRhyme_machine_distances.tsv — scaffold-by-scaffold table of length-7 pairwise feature vectors M_i,j (raw distances for CU, RTF, TUD, ZOM, GC, CpG, GC skew)
- Per-scaffold nucleotide composition features (tetranucleotide/k-mer frequencies, GC content, CpG content, GC skew, codon usage)

## How to apply

Load the viral scaffold FASTA, filtering out scaffolds shorter than the 2000 bp minimum length for binning. Predict open reading frames with Prodigal using '-m -p meta' to obtain genes/proteins for codon usage. Compute per-scaffold features: tetranucleotide (k-mer) frequencies, GC content, CpG content, GC skew, and codon usage from predicted ORFs. For each scaffold pair, compare the four frequency-vector metrics (CU, RTF, TUD, ZOM) as cosine distances and scalar compositions by absolute difference, applying the pre-filters of maximum GC distance 0.20 and minimum k-mer distance 0.60 to skip dissimilar pairs before full computation. Assemble the length-7 pairwise feature vectors M_i,j into a scaffold-by-scaffold distance table and write it to vRhyme_machine_distances.tsv for direct input to the neural network/decision tree machine learning models.

## Related tools

- **vRhyme** (Implements the nucleotide processing step that computes the pairwise composition distances and writes vRhyme_machine_distances.tsv) — https://github.com/AnantharamanLab/vRhyme
- **Prodigal** (Predicts open reading frames ('-m -p meta') supplying genes/proteins for the codon usage feature) — https://github.com/hyattpd/Prodigal
- **Numpy** (Python dependency used for vectorized frequency-vector and distance computations)
- **Pandas** (Python dependency used to assemble and write the scaffold-by-scaffold distance table)

## Examples

```
prodigal -i viral_scaffolds.fna -o genes.gff -a proteins.faa -p meta -m
```

## Evaluation signals

- Output file vRhyme_machine_distances.tsv exists and contains raw distance values for every scaffold pair
- Each pairwise feature vector M_i,j has length 7 (four vector metrics + GC, CpG, GC skew)
- Cosine distances for non-negative frequency vectors (CU, RTF, TUD, ZOM) fall in [0, 1]
- Pre-filters were respected: pairs with GC distance > 0.20 or k-mer distance < 0.60 are skipped as dissimilar
- Scaffolds shorter than the 2000 bp minimum length are excluded from the distance table

## Limitations

- Pre-filters (max GC distance 0.20, min k-mer distance 0.60) deliberately skip dissimilar pairs, so distances are only computed for candidate-similar scaffold pairs
- vRhyme is built for viral sequences; performance for a whole metagenome input 'has not been fully evaluated'
- vRhyme had difficulties separating multiple inactive (non-replicating) prophages from the same host genome, which composition-based distances alone cannot resolve

## Evidence

- [abstract] vRhyme utilizes single- or multi-sample coverage effect size comparisons between scaffolds: "vRhyme utilizes single- or multi-sample coverage effect size comparisons between scaffolds"
- [methods] ORF prediction (Prodigal): "ORF prediction (Prodigal)"
- [readme] README on feature basis: "The similarity in sequence features between two scaffolds can be used to identify fragments of the same genome, such as tetranucleotide frequencies, codon usage or GC content."
- [readme] README on input scope: "vRhyme is built to run on viral sequences/scaffolds."
