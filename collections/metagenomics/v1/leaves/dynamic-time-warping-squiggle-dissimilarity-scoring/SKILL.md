---
name: dynamic-time-warping-squiggle-dissimilarity-scoring
description: Use when selecting k-mers for a Nanopore signal-detection library (e.g.
  baseLess 16S detection) and you need to verify that candidate k-mers produce electrically
  distinguishable squiggles, not just sequence-dissimilar ones.
when_to_use_negative:
- Pre-generated 16S k-mer networks from baseLess update_16s_db already cover your
  target — do not rerun the evolutionary DTW selection
- You need basecalling-dependent analysis (full sequence reads) rather than target-detection-only
  analysis on raw signal
- You lack a good reference genome and large read set of a known species, which are
  required for resquiggle-based k-mer network generation
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: baseLess
  role: Runs run_production_pipeline with evolutionary k-mer selection that scores
    candidates by Smith–Waterman sequence dissimilarity and dynamic time warping squiggle
    dissimilarity; emits the k-mer list
  repo: https://github.com/cvdelannoy/baseLess
- name: guppy
  role: Basecalls raw reads prior to read correction; also used to simulate squiggles
    on which DTW scores are computed
- name: ont-fast5-api
  role: Splits multi-fast5 output into single fast5 files via single_to_multi_fast5
    so tombo can process them
- name: tombo
  role: Corrects (resquiggles) the raw reads against a reference genome to provide
    corrected signal for k-mer selection training
  repo: https://anaconda.org/bioconda/ont-tombo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad017
    title: 'baseLess: lightweight detection of sequences in raw MinION data'
  source_content_hash: sha256:281ece4d5bc8e326a70f80303d52634dfc52e67a4356b80589cbee98aacb41d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cvdelannoy/baseLess
  tools:
  - baseLess
  - guppy
  - ont-fast5-api
  - tombo
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# dynamic-time-warping-squiggle-dissimilarity-scoring

## Summary

Scores candidate k-mers by how distinguishable their Nanopore squiggles are, using dynamic time warping on guppy-simulated (resquiggled) electrical signals, so that the 1500 k-mer 16S library contains only k-mers that yield easily distinguishable squiggles.

## When to use

Use when selecting k-mers for a Nanopore signal-detection library (e.g. baseLess 16S detection) and you need to verify that candidate k-mers produce electrically distinguishable squiggles, not just sequence-dissimilar ones.

## When NOT to use

- Pre-generated 16S k-mer networks from baseLess update_16s_db already cover your target — do not rerun the evolutionary DTW selection
- You need basecalling-dependent analysis (full sequence reads) rather than target-detection-only analysis on raw signal
- You lack a good reference genome and large read set of a known species, which are required for resquiggle-based k-mer network generation

## Inputs

- tombo-resquiggled single-read fast5 files (training and test reads of a known species)
- Reference genome FASTA used for tombo resquiggle
- guppy-basecalled raw MinION reads (multi-fast5 input before splitting)

## Outputs

- K-mer list txt file with 1500 lines (one k-mer per line), selected for sequence and squiggle dissimilarity
- K-mer detection networks (NN directory) usable by baseLess compile_model / run_inference

## How to apply

Prepare raw MinION reads by basecalling with guppy, splitting multi-fast5 files into single fast5 with ont-fast5-api's single_to_multi_fast5, and correcting signals with tombo resquiggle against a reference genome. Feed the corrected training and test reads to baseLess run_production_pipeline, which invokes an evolutionary selection algorithm: candidate k-mers are scored both by sequence dissimilarity (Smith–Waterman alignment, rejecting candidates with alignment scores above a maximum of 6 to already-selected k-mers) and by squiggle dissimilarity computed with dynamic time warping on squiggles simulated with guppy. Candidates failing 16S abundance filtering or homopolymer rejection are discarded; the evolutionary loop retains k-mers that simultaneously maximize sequence and squiggle dissimilarity. Write the surviving 1500 k-mers to a txt k-mer list (one k-mer per line) via the --kmer-list flag.

## Related tools

- **baseLess** (Runs run_production_pipeline with evolutionary k-mer selection that scores candidates by Smith–Waterman sequence dissimilarity and dynamic time warping squiggle dissimilarity; emits the k-mer list) — https://github.com/cvdelannoy/baseLess
- **guppy** (Basecalls raw reads prior to read correction; also used to simulate squiggles on which DTW scores are computed)
- **ont-fast5-api** (Splits multi-fast5 output into single fast5 files via single_to_multi_fast5 so tombo can process them)
- **tombo** (Corrects (resquiggles) the raw reads against a reference genome to provide corrected signal for k-mer selection training) — https://anaconda.org/bioconda/ont-tombo

## Examples

```
baseLess run_production_pipeline --training-reads path/to/training/reads --test-reads path/to/test/reads --out-dir path/to/networks/directory --cores 8 --kmer-list
```

## Evaluation signals

- The k-mer list txt file exists and contains exactly 1500 lines (one k-mer per line)
- Every selected k-mer has a maximum Smith–Waterman alignment score of 6 to any other selected k-mer (sequence-dissimilarity constraint satisfied)
- Dynamic time warping scores computed on guppy-simulated squiggles confirm that selected k-mers produce easily distinguishable squiggles
- No selected k-mer was rejected by 16S abundance filtering or homopolymer rejection criteria
- Compiled read_detection model on the resulting networks classifies amplified/metagenomic 16S reads with reads retaining >66% of total k-mers assigned to the target

## Limitations

- Squiggle-dissimilarity scoring depends on guppy-simulated squiggles and tombo resquiggled reads, requiring a good reference genome and a large read set of a known species
- The pre-generated k-mer set is attuned to 16S detection; other targets require regenerating networks, and the authors plan to further expand the available k-mer networks
- Training with --train-on-the-fly is slower since k-mer models are generated as needed (typically <1 hour)

## Evidence

- [methods] evolutionary k-mer selection scores: [evidence span withheld — anchor status: not_found]
- [readme] production pipeline invocation: "baseLess run_production_pipeline --training-reads path/to/training/reads --test-reads path/to/test/reads --out-dir path/to/networks/directory --cores 8 --kmer-list"
- [readme] read correction requirement: "Basecall your reads with guppy if you haven't yet. If reads were output as multi-fast5, split them using from the ont-fast5-api package"
- [methods] tombo resquiggle step: "tombo resquiggle single/fast5/output/folder reference_genome.fasta"
- [readme] pre-generated network scope: "baseLess comes with a pre-generated set of k-mer detection networks, for k-mers that should be particularly salient in the context of 16S detection. We plan to further expand the number of available"
