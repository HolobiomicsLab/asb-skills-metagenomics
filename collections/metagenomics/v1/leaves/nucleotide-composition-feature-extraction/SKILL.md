---
name: nucleotide-composition-feature-extraction
description: Use when you need to compute nucleotide composition features (tetranucleotide/k-mer
  frequencies, GC, CpG, GC skew, codon usage) for viral scaffolds and build a length-7
  pairwise scaffold-by-scaffold distance table for downstream binning or machine learning.
when_to_use_negative:
- Input is an entire (unfiltered) metagenome rather than predicted viral scaffolds
  — vRhyme is built to run on viral sequences and 'is not meant to bin microbes'
- You need sequence-similarity-based distances (e.g. Mash/Nucmer dereplication) rather
  than composition-feature distances
- Scaffolds are shorter than the 2000bp minimum length cutoff and would be filtered
  out anyway
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3174
tools:
- name: vRhyme
  role: Host pipeline whose nucleotide processing step computes the seven composition
    features and pairwise distance table
  repo: https://github.com/AnantharamanLab/vRhyme
- name: Prodigal
  role: Predicts open reading frames ('-m -p meta') providing genes/proteins for the
    codon usage feature
  repo: https://github.com/hyattpd/Prodigal
- name: Numpy
  role: Vectorized computation of frequency vectors and cosine distances between scaffold
    pairs
- name: Pandas
  role: Assembles the length-7 pairwise feature vectors into the scaffold-by-scaffold
    distance table written to vRhyme_machine_distances.tsv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:74563454e8456ed841a8b83947314a819eadb68135f2f7680e4d5a48bb5546f8
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
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# nucleotide-composition-feature-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs vRhyme's per-scaffold nucleotide feature extraction and pairwise distance module, computing seven composition features (k-mer/tetranucleotide frequencies, GC content, CpG content, GC skew, codon usage from Prodigal ORFs) and comparing scaffold pairs via cosine distances for frequency vectors and absolute differences for scalars, written to vRhyme_machine_distances.tsv as input to the supervised machine learning models.

## When to use

Use when you need to compute nucleotide composition features (tetranucleotide/k-mer frequencies, GC, CpG, GC skew, codon usage) for viral scaffolds and build a length-7 pairwise scaffold-by-scaffold distance table for downstream binning or machine learning.

## When NOT to use

- Input is an entire (unfiltered) metagenome rather than predicted viral scaffolds — vRhyme is built to run on viral sequences and 'is not meant to bin microbes'
- You need sequence-similarity-based distances (e.g. Mash/Nucmer dereplication) rather than composition-feature distances
- Scaffolds are shorter than the 2000bp minimum length cutoff and would be filtered out anyway

## Inputs

- Viral scaffold FASTA file (predicted viral sequences, e.g. from VIBRANT or VirSorter)
- Prodigal-predicted ORFs/proteins in '-p meta' mode for codon usage computation

## Outputs

- vRhyme_machine_distances.tsv — scaffold-by-scaffold table of length-7 pairwise feature distance vectors M_i,j
- Per-scaffold nucleotide composition feature vectors (k-mer frequencies, GC content, CpG content, GC skew, codon usage)

## How to apply

Load the viral scaffold FASTA and filter out scaffolds shorter than the 2000bp minimum length for binning. Predict open reading frames with Prodigal using '-m -p meta' to obtain genes/proteins for the codon usage feature. Compute per-scaffold nucleotide composition features: tetranucleotide (k-mer) frequencies, GC content, CpG content, GC skew, and codon usage from predicted ORFs. For each scaffold pair, calculate the seven feature distances — cosine distance for frequency vectors (codon usage CU, relative tetranucleotide frequency RTF, tetranucleotide usage deviation TUD, zero'th-order Markov ZOM) and absolute difference for scalar compositions such as GC — applying the pre-filters of maximum GC distance 0.20 and minimum k-mer distance 0.60 to skip dissimilar pairs. Assemble the length-7 pairwise feature vectors M_i,j into a scaffold-by-scaffold table and write it to vRhyme_machine_distances.tsv for the supervised machine learning classification step.

## Related tools

- **vRhyme** (Host pipeline whose nucleotide processing step computes the seven composition features and pairwise distance table) — https://github.com/AnantharamanLab/vRhyme
- **Prodigal** (Predicts open reading frames ('-m -p meta') providing genes/proteins for the codon usage feature) — https://github.com/hyattpd/Prodigal
- **Numpy** (Vectorized computation of frequency vectors and cosine distances between scaffold pairs)
- **Pandas** (Assembles the length-7 pairwise feature vectors into the scaffold-by-scaffold distance table written to vRhyme_machine_distances.tsv)

## Examples

```
prodigal -i scaffolds.fna -o genes.gff -a proteins.faa -p meta
```

## Evaluation signals

- vRhyme_machine_distances.tsv exists and contains raw distance values for every scaffold pair
- Each pairwise entry is a length-7 feature vector M_i,j covering all seven composition distances
- Scalar features (e.g. GC) use absolute difference while the CU, RTF, TUD and ZOM frequency vectors use cosine distance
- Pre-filters are respected: pairs with GC distance > 0.20 or k-mer distance < 0.60 are skipped as dissimilar
- Scaffolds shorter than the 2000bp minimum length are excluded from the distance table

## Limitations

- Pre-filters (max GC distance 0.20, min k-mer distance 0.60) deliberately skip dissimilar pairs, so missing entries in the table reflect the filters, not computation failure
- Codon usage depends on Prodigal ORF prediction quality; Prodigal must be installed if vRhyme's '-p' or '-g' flags are used
- The skill is virus-centric: composition features are meant to identify fragments of the same viral genome and are not evaluated for microbial binning

## Evidence

- [methods] Frequency-vector feature distances use cosine metric: [evidence span withheld — anchor status: not_found]
- [methods] Prodigal ORF prediction feeds codon usage features: [evidence span withheld — anchor status: not_found]
