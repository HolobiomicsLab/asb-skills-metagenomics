---
name: pairwise-scaffold-distance-matrix-construction
description: Use when you have a FASTA of predicted viral scaffolds and need a scaffold-by-scaffold
  nucleotide feature distance table (7 features per pair) to feed ML-based viral binning
  like vRhyme's.
when_to_use_negative:
- Input is an entire unfiltered metagenome rather than predicted viral scaffolds —
  vRhyme's performance on whole metagenomes has not been fully evaluated and it is
  not meant to bin microbes
- Scaffolds shorter than 2000bp or circular (complete viral genome) scaffolds removed
  pre-binning — these are filtered before distance construction
- Binning should rely on coverage information rather than sequence composition — this
  skill produces only nucleotide-feature distances, not coverage effect-size comparisons
tools:
- name: vRhyme
  role: Host implementation whose nucleotide processing step performs the seven-feature
    pairwise distance calculation and writes vRhyme_machine_distances.tsv
  repo: https://github.com/AnantharamanLab/vRhyme
- name: Prodigal
  role: Predicts open reading frames with '-m -p meta' to supply codon usage features
    for the distance vectors
  repo: https://github.com/hyattpd/Prodigal
- name: Numpy
  role: Vectorized computation of cosine distances between scaffold frequency vectors
- name: Pandas
  role: Assembles the length-7 pairwise vectors into the scaffold-by-scaffold distance
    table and writes the TSV output
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:1f6f4e45bc0bd7af9d86cf8fc44331bd7ce61e38b90aec6135c0431c936dc5a8
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# pairwise-scaffold-distance-matrix-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs vRhyme's nucleotide feature distance module: compute per-scaffold composition features (tetranucleotide frequencies, GC, CpG, GC skew, codon usage) and convert them into length-7 pairwise distance vectors M_i,j for every scaffold pair, written to vRhyme_machine_distances.tsv as input to vRhyme's supervised ML binning models.

## When to use

Use when you have a FASTA of predicted viral scaffolds and need a scaffold-by-scaffold nucleotide feature distance table (7 features per pair) to feed ML-based viral binning like vRhyme's.

## When NOT to use

- Input is an entire unfiltered metagenome rather than predicted viral scaffolds — vRhyme's performance on whole metagenomes has not been fully evaluated and it is not meant to bin microbes
- Scaffolds shorter than 2000bp or circular (complete viral genome) scaffolds removed pre-binning — these are filtered before distance construction
- Binning should rely on coverage information rather than sequence composition — this skill produces only nucleotide-feature distances, not coverage effect-size comparisons

## Inputs

- Viral scaffold FASTA file (scaffolds >= 2000bp for binning)
- Prodigal gene/protein predictions (prodigal -m -p meta) for codon usage features

## Outputs

- vRhyme_machine_distances.tsv: scaffold-by-scaffold table of length-7 pairwise feature distance vectors (raw distance values per scaffold pair)
- Per-scaffold nucleotide composition features (tetranucleotide frequencies, GC content, CpG content, GC skew, codon usage)

## How to apply

Load the viral scaffold FASTA and drop scaffolds shorter than the 2000bp minimum binning length. Predict ORFs with Prodigal using '-m -p meta' to supply codon usage (CU) features, then compute per-scaffold nucleotide composition features: tetranucleotide (k-mer) frequencies, GC content, CpG content, GC skew, and codon usage from the predicted ORFs. For each scaffold pair, build a length-7 feature distance vector M_i,j using cosine distance for frequency-vector metrics (CU, relative tetranucleotide frequency RTF, tetranucleotide usage deviation TUD, zero'th-order Markov ZOM) and absolute difference for scalar compositions such as GC. Before computing, apply the pre-filters — skip pairs whose GC distance exceeds 0.20 or whose k-mer distance is below 0.60 — so dissimilar pairs never enter the distance table. Assemble all pairwise vectors into a scaffold-by-scaffold distance table and write it to vRhyme_machine_distances.tsv containing raw distance values for every qualifying pair, suitable as direct input to the neural network / decision tree ML models.

## Related tools

- **vRhyme** (Host implementation whose nucleotide processing step performs the seven-feature pairwise distance calculation and writes vRhyme_machine_distances.tsv) — https://github.com/AnantharamanLab/vRhyme
- **Prodigal** (Predicts open reading frames with '-m -p meta' to supply codon usage features for the distance vectors) — https://github.com/hyattpd/Prodigal
- **Numpy** (Vectorized computation of cosine distances between scaffold frequency vectors)
- **Pandas** (Assembles the length-7 pairwise vectors into the scaffold-by-scaffold distance table and writes the TSV output)

## Examples

```
prodigal -i scaffolds.fna -o genes -a proteins.faa -m -p meta
```

## Evaluation signals

- vRhyme_machine_distances.tsv exists and contains raw distance values for every scaffold pair, suitable as direct input to the machine learning models
- Each pairwise entry is a length-7 feature vector M_i,j (GC, CpG, GC skew, CU, RTF, TUD, ZOM), with cosine distances for the four frequency-vector metrics and absolute differences for scalar compositions
- Pre-filters were applied: pairs with GC distance > 0.20 or k-mer distance < 0.60 are skipped as dissimilar
- Distance table is symmetric with zero diagonal for self-pairs
- All scaffolds in the table satisfy the 2000bp minimum binning length filter

## Limitations

- vRhyme is built for viral sequences/scaffolds; performance on a whole metagenome input has not been fully evaluated and it is not meant to bin microbes
- vRhyme had difficulties separating multiple inactive (non-replicating) prophages from the same host genome
- Pre-filtering (max GC distance 0.20, min k-mer distance 0.60) means the table omits dissimilar pairs rather than containing all N(N-1)/2 distances

## Evidence

- [methods] cosine distance for frequency-vector metrics: [evidence span withheld — anchor status: not_found]
- [readme] Prodigal metagenome mode invocation: [evidence span withheld — anchor status: external_doc]
