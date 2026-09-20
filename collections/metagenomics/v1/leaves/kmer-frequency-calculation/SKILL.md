---
name: kmer-frequency-calculation
description: Use when input is a set of viral scaffolds (FASTA, ≥2000 bp) destined
  for binning and you need nucleotide-composition feature vectors (k-mer frequencies,
  CU, RTF, TUD, ZOM, GC) to compare scaffolds pairwise.
when_to_use_negative:
- Input scaffolds are shorter than the 2000 bp minimum length for binning, or the
  input is a whole metagenome rather than predicted viral sequences (vRhyme performance
  on whole metagenomes has not been fully evaluated)
- The goal is binning microbial (non-viral) genomes — vRhyme is 'not meant to bin
  microbes'
- A scaffold feature/distance table already exists and only downstream clustering
  or machine learning is needed
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3697
tools:
- name: vRhyme
  role: Tool whose nucleotide processing step computes the composition features and
    pairwise distances; writes vRhyme_machine_distances.tsv as ML input
  repo: https://github.com/AnantharamanLab/vRhyme
- name: Prodigal
  role: Predicts open reading frames with '-m -p meta' to obtain genes/proteins used
    for codon usage features
  repo: https://github.com/hyattpd/Prodigal
- name: Numpy
  role: Python dependency used for vectorized k-mer/composition feature computation
    and cosine distance calculation
- name: Pandas
  role: Python dependency used to assemble the scaffold-by-scaffold distance table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:71d686e4039df1cd8bd1b4c1afc7c13f4ea4e985ad9d489d5552c6d1686362d2
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
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# kmer-frequency-calculation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-scaffold nucleotide composition features — tetranucleotide (k-mer) frequencies, GC content, CpG content, GC skew, and codon usage from predicted ORFs — as used by vRhyme to quantify sequence-feature similarity between viral scaffolds for machine-learning-based binning.

## When to use

Use when input is a set of viral scaffolds (FASTA, ≥2000 bp) destined for binning and you need nucleotide-composition feature vectors (k-mer frequencies, CU, RTF, TUD, ZOM, GC) to compare scaffolds pairwise.

## When NOT to use

- Input scaffolds are shorter than the 2000 bp minimum length for binning, or the input is a whole metagenome rather than predicted viral sequences (vRhyme performance on whole metagenomes has not been fully evaluated)
- The goal is binning microbial (non-viral) genomes — vRhyme is 'not meant to bin microbes'
- A scaffold feature/distance table already exists and only downstream clustering or machine learning is needed

## Inputs

- Viral scaffold FASTA file (scaffolds >= 2000 bp after filtering)
- Prodigal-predicted ORFs/proteins ('-m -p meta') for codon usage features

## Outputs

- Per-scaffold nucleotide composition feature vectors: tetranucleotide frequencies, CU, RTF, TUD, ZOM, GC content, CpG content, GC skew
- Scaffold-by-scaffold pairwise distance table written to vRhyme_machine_distances.tsv (length-7 feature vectors per scaffold pair)

## How to apply

Load the viral scaffold FASTA and filter out scaffolds shorter than the 2000 bp minimum length for binning. Predict open reading frames with Prodigal using '-m -p meta' to supply genes/proteins for codon usage features. For each scaffold compute per-scaffold nucleotide composition features: tetranucleotide (k-mer) frequencies, GC content, CpG content, GC skew, and codon usage from the predicted ORFs; the frequency-vector metrics comprise codon usage (CU), relative tetranucleotide frequency (RTF), tetranucleotide usage deviation (TUD), and zero'th-order Markov (ZOM). For each scaffold pair, reduce these vectors to the seven-feature distances — cosine distance for frequency vectors such as CU/RTF/TUD/ZOM, absolute difference for scalar compositions like GC — applying pre-filters of maximum GC distance 0.20 and minimum k-mer distance 0.60 to skip dissimilar pairs before assembling the length-7 pairwise vectors M_i,j into a scaffold-by-scaffold distance table. The rationale is that scaffolds sharing these composition features are likely fragments of the same viral genome, which is what vRhyme's supervised machine learning exploits for binning.

## Related tools

- **vRhyme** (Tool whose nucleotide processing step computes the composition features and pairwise distances; writes vRhyme_machine_distances.tsv as ML input) — https://github.com/AnantharamanLab/vRhyme
- **Prodigal** (Predicts open reading frames with '-m -p meta' to obtain genes/proteins used for codon usage features) — https://github.com/hyattpd/Prodigal
- **Numpy** (Python dependency used for vectorized k-mer/composition feature computation and cosine distance calculation)
- **Pandas** (Python dependency used to assemble the scaffold-by-scaffold distance table)

## Examples

```
prodigal -i my.metagenome.fna -o my.genes -a my.proteins.faa -p meta
```

## Evaluation signals

- vRhyme_machine_distances.tsv exists and contains raw distance values for every scaffold pair, suitable as direct input to the machine learning models
- Each scaffold pair yields a length-7 feature vector M_i,j covering CU, RTF, TUD, ZOM, GC and related composition distances
- Frequency-vector metrics (CU, RTF, TUD, ZOM) are compared via cosine distance, while scalar compositions (e.g., GC) use absolute difference
- Pairs with GC distance > 0.20 or k-mer distance < 0.60 are pre-filtered (skipped) as dissimilar
- No scaffolds shorter than 2000 bp appear in the computed features

## Limitations

- vRhyme is built to run on viral sequences/scaffolds; performance on whole metagenomes has not been fully evaluated and it is not meant to bin microbes
- The k-mer/composition features alone can be insufficient — coverage variance helps separate scaffolds that sound the same but are different genomes
- vRhyme had difficulties separating multiple inactive (non-replicating) prophages from the same host genome

## Evidence

- [methods] compute per-scaffold nucleotide composition features: [evidence span withheld — anchor status: not_found]
- [methods] cosine distances between frequency vectors: [evidence span withheld — anchor status: not_found]
