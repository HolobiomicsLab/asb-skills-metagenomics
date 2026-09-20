---
name: 16s-rrna-kmer-uniqueness-analysis
description: Use when validating that a salient 16S k-mer subset (baseLess read_detection
  k-mers) can support species detection against an arbitrary genomic background, by
  quantifying unique-k-mer identifiability across simulated mixture sizes from 1 up
  to 10,000 constituents.
when_to_use_negative:
- The goal is to run baseLess signal-level inference on real fast5 reads rather than
  evaluate k-mer uniqueness in silico (use baseLess run_inference instead).
- Targets are whole genomes or non-16S sequences; read_detection k-mers have been
  tested for and attuned to 16S sequence.
- Sample composition is already known (e.g. the 20-bacterial-species classification
  benchmark), so uniqueness against an arbitrary background need not be simulated.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0654
tools:
- name: baseLess
  role: Source of the salient 16S k-mer subset and of the --kmer-list one-k-mer-per-line
    text format the analysis must match; k-mer networks can be fetched via baseLess
    update_16s_db.
  repo: https://github.com/cvdelannoy/baseLess
- name: mamba
  role: Fast drop-in replacement for conda used to install baseLess before running
    the analysis.
- name: conda
  role: Alternative package manager for installing baseLess from the bioconda, conda-forge
    and cvdelannoy channels.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad017
    title: 'baseLess: lightweight detection of sequences in raw MinION data'
  source_content_hash: sha256:34ee18024d072a357587deccfa7135378341a2c7015c9859820ea0e6001b3ddc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cvdelannoy/baseLess
  tools:
  - baseLess
  - conda
  - mamba
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# 16s-rrna-kmer-uniqueness-analysis

## Summary

In silico benchmark of the salient 16S k-mer subset used by baseLess: for mixtures of 1 to 10,000 species drawn from the NCBI 16S pool (BioProject PRJNA33175), it computes the fraction of species possessing at least one k-mer that occurs uniquely within the mixture, demonstrating the subset identifies the majority of species even at 10,000 constituents (Supplementary Fig. S2).

## When to use

Use when validating that a salient 16S k-mer subset (baseLess read_detection k-mers) can support species detection against an arbitrary genomic background, by quantifying unique-k-mer identifiability across simulated mixture sizes from 1 up to 10,000 constituents.

## When NOT to use

- The goal is to run baseLess signal-level inference on real fast5 reads rather than evaluate k-mer uniqueness in silico (use baseLess run_inference instead).
- Targets are whole genomes or non-16S sequences; read_detection k-mers have been tested for and attuned to 16S sequence.
- Sample composition is already known (e.g. the 20-bacterial-species classification benchmark), so uniqueness against an arbitrary background need not be simulated.

## Inputs

- NCBI 16S reference sequences from BioProject PRJNA33175 (species pool)
- Selected salient 16S k-mer subset as a plain-text list, one k-mer per line (baseLess --kmer-list format)
- baseLess package installed via conda/mamba

## Outputs

- Per-mixture-size uniqueness fraction: fraction of species uniquely identified by at least one subset k-mer
- Matplotlib figure supplementary_figure_s2.png plotting uniqueness fraction versus number of constituents

## How to apply

Install baseLess from bioconda/conda-forge/cvdelannoy channels via mamba, then obtain the pre-generated 16S k-mer set (these k-mers are chosen to be particularly salient in the context of 16S detection). Download the NCBI 16S reference sequences from BioProject PRJNA33175 into memory as the species pool, and load the selected k-mer subset as a plain-text list with one k-mer per line, matching the --kmer-list format of baseLess. For each mixture size from 1 to 10,000, repeatedly sample in silico samples of species from the pool and, for each species, test whether at least one k-mer from the subset occurs uniquely in that species relative to all other constituents in the mixture. Compute the per-size fraction of species uniquely identified, plot it against the number of constituents with matplotlib, and save as supplementary_figure_s2.png. The validation criterion is that the uniqueness fraction stays above the majority-of-species level even at 10,000 constituents, reproducing the trend of Supplementary Fig. S2.

## Related tools

- **baseLess** (Source of the salient 16S k-mer subset and of the --kmer-list one-k-mer-per-line text format the analysis must match; k-mer networks can be fetched via baseLess update_16s_db.) — https://github.com/cvdelannoy/baseLess
- **mamba** (Fast drop-in replacement for conda used to install baseLess before running the analysis.)
- **conda** (Alternative package manager for installing baseLess from the bioconda, conda-forge and cvdelannoy channels.)

## Examples

```
plt.plot(mixture_sizes, uniqueness_fractions); plt.savefig('supplementary_figure_s2.png')
```

## Evaluation signals

- At 10,000 constituents the uniqueness fraction remains above 0.5, i.e. the majority of species are still uniquely identified, matching Supplementary Fig. S2.
- Uniqueness fraction decreases monotonically as the number of constituents grows from 1 to 10,000.
- The k-mer list is a plain-text file with exactly one k-mer per line, compatible with baseLess --kmer-list.
- Every simulated constituent is tested for at least one subset k-mer that occurs uniquely relative to all other species in that mixture.
- Figure is saved as supplementary_figure_s2.png and shows the reported downward trend in unique identification with mixture size.

## Limitations

- Uniqueness is evaluated against in silico mixtures sampled from the PRJNA33175 16S reference pool, not real metagenomic samples.
- The read_detection k-mer framework is attuned to 16S sequence; uniqueness results may not transfer to other targets.
- At the read level baseLess only requires >66% of total k-mers by default, whereas this analysis assesses per-species uniqueness, so the two criteria are not interchangeable.

## Evidence

- [supplementary] The selected 16S k-mer subset suffices, on average, to uniquely identify the majority of species in samples containing up to 10,000 different constituents (Supplementary Fig. S2): [evidence span withheld — anchor status: not_found]
- [readme] baseLess comes with a pre-generated set of k-mer detection networks, for k-mers that should be particularly salient in the context of 16S detection: "baseLess comes with a pre-generated set of k-mer detection networks, for k-mers that should be particularly salient in the context of 16S detection"
- [methods] mamba install -c bioconda -c conda-forge -c cvdelannoy baseless: "mamba install -c bioconda -c conda-forge -c cvdelannoy baseless"
