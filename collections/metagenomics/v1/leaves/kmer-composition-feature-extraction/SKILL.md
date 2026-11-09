---
name: kmer-composition-feature-extraction
description: 'Use when preparing composition features for contig binning: you have
  assembled contigs (FASTA, typically >=1000 bp after filtering) and need k-mer frequency
  vectors, optionally combined across multiple k values, to feed a clustering or binning
  algorithm.'
when_to_use_negative:
- Input is not assembled contigs (e.g. raw short reads or long reads) — this skill
  operates on the contig FASTA produced by a metagenomic assembler such as metaSPAdes
- The downstream binning tool expects only low-dimensional composition features (k<=4
  single setting) and cannot handle 500+ or 2000+ dimensional vectors without a high-dimensional
  clustering method such as convex hull distance
- Contigs shorter than the 1000 bp ContigLengthFilterBp threshold dominate the assembly
  — they should be filtered out before vectorization
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: seq2vec
  role: Converts contig FASTA/FASTQ sequences into k-mer frequency vectors used as
    the composition feature vectors
  repo: https://github.com/anuradhawick/seq2vec
- name: CH-Bin
  role: Consumes the composition vectors (KmerK parameter selects the k setting) together
    with coverage for convex hull distance based binning
  repo: https://github.com/kdsuneraavinash/CH-Bin
- name: CoverM
  role: Computes sample-wise read coverage for the assembled contigs, concatenated
    with the composition features
  repo: https://github.com/wwood/CoverM
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1016/j.compbiolchem.2022.107734
    title: 'CH-Bin: A convex hull based approach for binning metagenomic contigs'
  source_content_hash: sha256:394ab505e86f3810060433fae03bfc93adce86f1ac2d320c7c4e294747ba76b9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kdsuneraavinash/CH-Bin
  repo_url: https://github.com/anuradhawick/seq2vec
  tools:
  - CH-Bin
  - CoverM
  - FragGeneScan
  - HMMER3
  - Minimap2
  - SAMTools
  - metaSPAdes
  - seq2vec
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# kmer-composition-feature-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts high-dimensional oligonucleotide composition feature vectors for metagenomic contigs by computing k-mer frequencies (with reverse complements combined) for k=4, 5, and/or 6, and concatenates them with sample-wise read coverage. This is Step 1 of the CH-Bin pipeline and preserves more compositional information than conventional single-k (k<=4) approaches.

## When to use

Use when preparing composition features for contig binning: you have assembled contigs (FASTA, typically >=1000 bp after filtering) and need k-mer frequency vectors, optionally combined across multiple k values, to feed a clustering or binning algorithm.

## When NOT to use

- Input is not assembled contigs (e.g. raw short reads or long reads) — this skill operates on the contig FASTA produced by a metagenomic assembler such as metaSPAdes
- The downstream binning tool expects only low-dimensional composition features (k<=4 single setting) and cannot handle 500+ or 2000+ dimensional vectors without a high-dimensional clustering method such as convex hull distance
- Contigs shorter than the 1000 bp ContigLengthFilterBp threshold dominate the assembly — they should be filtered out before vectorization

## Inputs

- Assembled metagenomic contigs (FASTA), after the 1000 bp contig length filter
- Short reads or BAM files for computing sample-wise coverage with CoverM
- k-value setting (e.g. KmerK for k=4, 5, 6, or a combination such as k=5,6)

## Outputs

- O-dimensional k-mer composition feature vector per contig (e.g. 136/512/2080 dims for k=4/5/6; 648/2216/2728/2592 for combined settings)
- Coverage values per contig per sample (from CoverM), concatenated with the composition vectors for binning

## How to apply

Install seq2vec and run it on the contig FASTA to obtain k-mer frequency vectors; each k-mer and its reverse complement are combined, giving 136, 512, and 2080 dimensions for k=4, k=5, and k=6 respectively. Select the k-value setting via CH-Bin's KmerK configuration parameter (seven options: k=4; k=4,5; k=4,6; k=4,5,6; k=5; k=5,6; k=6), with concatenated dimensionalities of 648, 2216, 2728, and 2592 for the combined settings. Apply the default contig length filter (ContigLengthFilterBp=1000) so short contigs are discarded before vectorization. Compute sample-wise read coverage for the same contigs with CoverM and concatenate the coverage values with the composition vectors, since CH-Bin uses both coverage and composition information. Rationale: traditional Euclidean or Manhattan distance metrics become unreliable in high-dimensional space, which is why the downstream CH-Bin step pairs these richer feature vectors with convex hull distance based clustering.

## Related tools

- **seq2vec** (Converts contig FASTA/FASTQ sequences into k-mer frequency vectors used as the composition feature vectors) — https://github.com/anuradhawick/seq2vec
- **CH-Bin** (Consumes the composition vectors (KmerK parameter selects the k setting) together with coverage for convex hull distance based binning) — https://github.com/kdsuneraavinash/CH-Bin
- **CoverM** (Computes sample-wise read coverage for the assembled contigs, concatenated with the composition features) — https://github.com/wwood/CoverM

## Examples

```
./tools/seq2vec/seq2vec --help
```

## Evaluation signals

- Feature vector dimensionality matches the k setting: 136, 512, or 2080 for k=4, k=5, k=6; concatenated sums of 648 (k=4,5), 2216 (k=4,5,6), 2728 (k=4,6), or 2592 (k=5,6)
- Each vector entry counts a k-mer with its reverse complement combined, so reverse-complementary k-mers contribute to a single dimension
- Contigs shorter than 1000 bp (ContigLengthFilterBp) are absent from the feature matrix
- On Sim50, the downstream CH-Bin binning with the k=5,6 setting reproduces purity 95.28%, completeness 98.60%, F1 96.91%, accuracy 93.10%, and 46 bins under AMBER evaluation
- On small simulated datasets (Sim5, Sim10, Sim20), no k-setting should show significant improvement over others — deviations suggest a feature-extraction or configuration error

## Limitations

- CH-Bin only supports Linux; seq2vec requires cmake, build-essential, python3-dev, and libboost-all-dev to build
- Feature dimensionality grows rapidly with k (2080 dims for k=6), so downstream clustering must handle high-dimensional data; Euclidean/Manhattan distances become unreliable in this space
- The authors report no significant improvement among the different k-value settings on small simulated datasets (Sim5, Sim10, Sim20), so multi-k combinations do not guarantee gains
- The CH-Bin repository provides no examples, and no Zenodo/archival DOI is attached to the code release beyond the GitHub source link

## Evidence

- [intro] We explore the potential of utilizing k-mers of sizes larger than 4 (e.g., k = 5 and k = 6), and combinations of multiple k values for generating composition feature vectors: "We explore the potential of utilizing k-mers of sizes larger than 4 (e.g., k = 5 and k = 6), and combinations of multiple k values for generating composition feature vectors"
- [methods] In order to perform binning, we utilize coverage and composition information of the contigs. Similar to previous work... we represent each contig using an O dimensional composition vector: [evidence span withheld — anchor status: not_found]
- [methods] we use CoverM (available at https://github.com/wwood/CoverM) to compute sample-wise read coverage: "we use CoverM (available at https://github.com/wwood/CoverM) to compute sample-wise read coverage"
- [readme] You can use Seq2Vec to convert FASTA or FASTQ data sets into k-mer frequency vectors.: [evidence span withheld — anchor status: external_doc]
- [abstract] However, these tools use traditional Euclidean distance or Manhattan distance metrics which become unreliable in the high dimensional space.: "However, these tools use traditional Euclidean distance or Manhattan distance metrics which become unreliable in the high dimensional space."
