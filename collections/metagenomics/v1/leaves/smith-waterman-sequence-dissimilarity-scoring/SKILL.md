---
name: smith-waterman-sequence-dissimilarity-scoring
description: Use when building a custom baseLess k-mer detection network set (e.g.
  a 1500 k-mer library for 16S detection) and you must ensure selected k-mers are
  dissimilar in sequence and produce distinguishable electrical squiggles.
when_to_use_negative:
- A pre-generated 16S k-mer network set is sufficient — obtain it with 'baseLess update_16s_db'
  instead of re-running evolutionary selection
- No large read set from a known species plus a good reference genome is available,
  since guppy basecalling and tombo resquiggle correction are prerequisites
- The goal is an abundance-mode model (distinguishing visually similar species/strains
  by k-mer abundance against background genomes) rather than a read-detection k-mer
  library
edam_operation: http://edamontology.org/operation_0258
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: baseLess
  role: run_production_pipeline performs the evolutionary selection that scores candidate
    k-mers by Smith–Waterman sequence dissimilarity and squiggle dissimilarity (dynamic
    time warping)
  repo: https://github.com/cvdelannoy/baseLess
- name: guppy
  role: basecalls the raw reads prior to correction and k-mer selection
- name: ont-fast5-api
  role: splits multi-fast5 output into single-read fast5 files required downstream
  repo: https://pypi.org/project/ont-fast5-api/
- name: tombo
  role: resquiggle corrects reads against a reference genome, producing signal-aligned
    reads used to evaluate squiggle dissimilarity
  repo: https://anaconda.org/bioconda/ont-tombo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad017
    title: 'baseLess: lightweight detection of sequences in raw MinION data'
  source_content_hash: sha256:2875c2b64a320b7d938e38643f72e3092e73d110d64a0e539382edd178c87576
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cvdelannoy/baseLess
  tools:
  - baseLess
  - guppy
  - ont-fast5-api
  - tombo
  edam_operation: http://edamontology.org/operation_0258
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# smith-waterman-sequence-dissimilarity-scoring

## Summary

Scores candidate k-mers by pairwise Smith–Waterman sequence dissimilarity so that an evolutionary selection in baseLess keeps 1500 k-mers that are mutually non-homologous (maximum Smith–Waterman score of 6 to any other selected k-mer) and yield easily distinguishable squiggles for 16S sequence detection.

## When to use

Use when building a custom baseLess k-mer detection network set (e.g. a 1500 k-mer library for 16S detection) and you must ensure selected k-mers are dissimilar in sequence and produce distinguishable electrical squiggles.

## When NOT to use

- A pre-generated 16S k-mer network set is sufficient — obtain it with 'baseLess update_16s_db' instead of re-running evolutionary selection
- No large read set from a known species plus a good reference genome is available, since guppy basecalling and tombo resquiggle correction are prerequisites
- The goal is an abundance-mode model (distinguishing visually similar species/strains by k-mer abundance against background genomes) rather than a read-detection k-mer library

## Inputs

- Raw nanopore reads (fast5, possibly multi-fast5) from a species with a known reference genome
- Reference genome FASTA for tombo resquiggle correction
- Training and test read directories (corrected reads)

## Outputs

- k-mer list txt file (one k-mer per line, 1500 entries) satisfying the max Smith–Waterman score 6 constraint
- k-mer detection networks directory usable by 'baseLess run_inference'

## How to apply

Basecall raw reads with guppy and split multi-fast5 output into single fast5 files using ont-fast5-api's single_to_multi_fast5, then correct the reads against a reference genome with tombo resquiggle. Run 'baseLess run_production_pipeline' on the corrected training and test reads, which invokes an evolutionary selection in which each candidate k-mer is scored by sequence dissimilarity (Smith–Waterman alignment against already-selected k-mers, discarding candidates whose best alignment score exceeds 6) and by squiggle dissimilarity (dynamic time warping on simulated/resquiggled signal). Candidates are also discarded by 16S abundance filtering and homopolymer rejection before surviving k-mers are written out. The rationale is that baseLess' neural network is a composite of many small k-mer-detecting networks, so k-mers with low sequence similarity produce distinct signal patterns that the small networks can detect unambiguously.

## Related tools

- **baseLess** (run_production_pipeline performs the evolutionary selection that scores candidate k-mers by Smith–Waterman sequence dissimilarity and squiggle dissimilarity (dynamic time warping)) — https://github.com/cvdelannoy/baseLess
- **guppy** (basecalls the raw reads prior to correction and k-mer selection)
- **ont-fast5-api** (splits multi-fast5 output into single-read fast5 files required downstream) — https://pypi.org/project/ont-fast5-api/
- **tombo** (resquiggle corrects reads against a reference genome, producing signal-aligned reads used to evaluate squiggle dissimilarity) — https://anaconda.org/bioconda/ont-tombo

## Examples

```
baseLess run_production_pipeline --training-reads path/to/training/reads --test-reads path/to/test/reads --out-dir path/to/networks/directory --cores 8 --kmer-list
```

## Evaluation signals

- The --kmer-list output file exists and contains exactly 1500 lines (one k-mer per line)
- Every selected k-mer has a maximum Smith–Waterman alignment score of 6 against all other selected k-mers
- Dynamic time warping scores computed on guppy-simulated squiggles confirm the selected k-mers produce easily distinguishable squiggles
- No surviving k-mer violates the 16S abundance filtering or homopolymer rejection constraints imposed during evolution

## Limitations

- The pairwise dissimilarity constraint (max Smith–Waterman score of 6) was attuned to and tested for 16S sequence detection; read detection mode is 'not necessarily restricted to any sequence' but currently attuned to 16S
- K-mer selection requires reads corrected with tombo resquiggle and substantial compute ('more CPUs == faster'), so it is not a quick reconfiguration step
- The current pre-generated k-mer network set is small (<1MB) and oriented to 16S; generating new sets requires a large read set of a known species and a good reference genome

## Evidence

- [methods] An evolutionary algorithm was used to select k-mers that are dissimilar in sequence (maximum Smith–Waterman score of 6 to other selected k-mers): [evidence span withheld — anchor status: not_found]
- [abstract] BaseLess makes use of an array of small neural networks, each of which efficiently detects a fixed-size subsequence of the target sequence directly from the electrical signal.: "BaseLess makes use of an array of small neural networks, each of which efficiently detects a fixed-size subsequence of the target sequence directly from the electrical signal."
- [readme] Correct your reads with [tombo](https://anaconda.org/bioconda/ont-tombo):: "Correct your reads with [tombo](https://anaconda.org/bioconda/ont-tombo):"
- [readme] baseLess comes with a pre-generated set of k-mer detection networks, for k-mers that should be particularly salient in the context of 16S detection.: "baseLess comes with a pre-generated set of k-mer detection networks, for k-mers that should be particularly salient in the context of 16S detection."
