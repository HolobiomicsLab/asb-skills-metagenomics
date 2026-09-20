---
name: homopolymer-kmer-rejection
description: Use when generating custom k-mer detection networks with baseLess run_production_pipeline
  and you need to purge homopolymeric candidate k-mers from the evolutionary pool
  so only signal-distinguishable 16S k-mers survive.
when_to_use_negative:
- You only need off-the-shelf 16S detection — baseLess ships a pre-generated set of
  k-mer networks for k-mers salient in 16S detection ('baseLess update_16s_db'), so
  no custom selection or homopolymer filtering is needed
- Your input is already-basecalled FASTQ without raw signal — the pipeline requires
  resquiggled fast5 signal reads for squiggle-based selection
- You are compiling an abundance-mode model with --train-on-the-fly against large
  non-16S genomes where the 16S-specific abundance filtering and homopolymer rejection
  tuning does not apply
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: baseLess
  role: Runs the production pipeline whose evolutionary k-mer selection enforces homopolymer
    rejection and 16S abundance filtering and emits the k-mer list
  repo: https://github.com/cvdelannoy/baseLess
- name: guppy
  role: Basecalls raw reads as a prerequisite step before fast5 splitting and resquiggling
- name: ont-fast5-api
  role: Splits multi-fast5 output into single-read fast5 files via single_to_multi_fast5
    so tombo resquiggle can process them
  repo: https://pypi.org/project/ont-fast5-api/
- name: tombo
  role: Resquiggles (signal-to-reference corrects) the single fast5 reads so squiggle
    dissimilarity can be scored during k-mer evolution
  repo: https://anaconda.org/bioconda/ont-tombo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad017
    title: 'baseLess: lightweight detection of sequences in raw MinION data'
  source_content_hash: sha256:26a1bd9b6325c0184e0d435f8bfb57e1b4c3d205044ef4de811eae0fdab28821
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cvdelannoy/baseLess
  tools:
  - baseLess
  - guppy
  - ont-fast5-api
  - tombo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# homopolymer-kmer-rejection

## Summary

A filtering rule inside baseLess's evolutionary k-mer selection that discards candidate k-mers containing homopolymer runs (repeated identical bases) before the surviving 1500 16S k-mers are written to the k-mer list, because homopolymeric k-mers produce ambiguous squiggles that small k-mer-detecting neural networks cannot reliably discriminate.

## When to use

Use when generating custom k-mer detection networks with baseLess run_production_pipeline and you need to purge homopolymeric candidate k-mers from the evolutionary pool so only signal-distinguishable 16S k-mers survive.

## When NOT to use

- You only need off-the-shelf 16S detection — baseLess ships a pre-generated set of k-mer networks for k-mers salient in 16S detection ('baseLess update_16s_db'), so no custom selection or homopolymer filtering is needed
- Your input is already-basecalled FASTQ without raw signal — the pipeline requires resquiggled fast5 signal reads for squiggle-based selection
- You are compiling an abundance-mode model with --train-on-the-fly against large non-16S genomes where the 16S-specific abundance filtering and homopolymer rejection tuning does not apply

## Inputs

- Raw MinION fast5 reads (multi-fast5 or single fast5)
- Reference genome or target 16S FASTA for tombo resquiggle correction
- Training and test read directories (resquiggled fast5) for baseLess run_production_pipeline

## Outputs

- Filtered k-mer list txt file (--kmer-list), one k-mer per line (1500 k-mers in the 16S library)
- Trained k-mer detection neural networks (.h5) in the output networks directory

## How to apply

First prepare resquiggled signal reads: basecall raw reads with guppy, split multi-fast5 output into single fast5 files using ont-fast5-api's single_to_multi_fast5, then correct them with 'tombo resquiggle' against the target reference genome. Run 'baseLess run_production_pipeline' on the corrected training and test reads; this invokes the evolutionary selection that, alongside 16S abundance filtering, discards candidate k-mers rejected by homopolymer rejection, so that surviving k-mers are dissimilar in sequence (maximum Smith–Waterman score of 6 to other selected k-mers) and yield easily distinguishable squiggles (dynamic time warping on simulated squiggles). Surviving k-mers are written via --kmer-list as a txt file with one k-mer per line; validate that the list contains exactly 1500 entries and that no selected k-mer carries a homopolymeric run that survived selection.

## Related tools

- **baseLess** (Runs the production pipeline whose evolutionary k-mer selection enforces homopolymer rejection and 16S abundance filtering and emits the k-mer list) — https://github.com/cvdelannoy/baseLess
- **guppy** (Basecalls raw reads as a prerequisite step before fast5 splitting and resquiggling)
- **ont-fast5-api** (Splits multi-fast5 output into single-read fast5 files via single_to_multi_fast5 so tombo resquiggle can process them) — https://pypi.org/project/ont-fast5-api/
- **tombo** (Resquiggles (signal-to-reference corrects) the single fast5 reads so squiggle dissimilarity can be scored during k-mer evolution) — https://anaconda.org/bioconda/ont-tombo

## Examples

```
baseLess run_production_pipeline --training-reads path/to/training/reads --test-reads path/to/test/reads --out-dir path/to/networks/directory --cores 8 --kmer-list
```

## Evaluation signals

- The output k-mer list txt file exists and contains exactly the expected number of entries (1500 lines for the 16S library)
- Every selected k-mer satisfies the sequence-dissimilarity constraint: maximum Smith–Waterman alignment score of 6 against any other selected k-mer
- Selected k-mers produce easily distinguishable squiggles, confirmed by dynamic time warping scores on simulated squiggles
- No k-mer rejected by homopolymer rejection or 16S abundance filtering appears in the final k-mer list
- tombo resquiggle completed on the single fast5 directory before run_production_pipeline was invoked (corrected reads are a required input)

## Limitations

- The homopolymer and abundance filters are attuned to 16S detection; baseLess read detection has 'been tested for and attuned to 16S sequence at the moment'
- Generating your own k-mer networks requires a large read set of a known species and a good reference genome; k-mer models are generated as needed and the training step 'will take longer to run (typically <1 hour)'
- On low-powered hardware (Nvidia Jetson Nano 2GB) memory must be constrained (baseLess at 256MB, docker at 512MB), which can slow pipeline runs

## Evidence

- [other] discarding candidates rejected by 16S abundance filtering or homopolymer rejection: [evidence span withheld — anchor status: not_found]
- [other] An evolutionary algorithm was used to select k-mers: [evidence span withheld — anchor status: not_found]
- [readme] Correct your reads with [tombo](https://anaconda.org/bioconda/ont-tombo):
tombo resquiggle single/fast5/output/folder reference_genome.fasta: "Correct your reads with [tombo](https://anaconda.org/bioconda/ont-tombo): tombo resquiggle single/fast5/output/folder reference_genome.fasta"
- [readme] --kmer-list                                # txt list, 1 line per k-mer: "--kmer-list                                # txt list, 1 line per k-mer"
- [readme] BaseLess comes with a pre-generated set of k-mer detection networks, for k-mers that should be particularly salient in the context of 16S detection.: "BaseLess comes with a pre-generated set of k-mer detection networks, for k-mers that should be particularly salient in the context of 16S detection."
