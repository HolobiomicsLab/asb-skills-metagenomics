---
name: 16s-abundance-kmer-filtering
description: Use when building or validating a k-mer detection library for baseLess
  16S sequence detection on MinION raw signal, where selected k-mers must maximize
  Smith–Waterman sequence and dynamic-time-warping squiggle dissimilarity.
when_to_use_negative:
- You only need species-level abundance classification with a pre-built model — use
  baseLess compile_model --model-type abundance or the pre-generated update_16s_db
  k-mer models instead.
- You are detecting a non-16S target for which no large read set of a known species
  and good reference genome is available — the README notes the pre-generated set
  is attuned to 16S.
- Input is already a trained baseLess .h5 model or a k-mer list; this skill generates
  and filters candidate k-mers, it does not run inference.
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
tools:
- name: baseLess
  role: Runs run_production_pipeline with the evolutionary k-mer selection scoring
    Smith–Waterman sequence dissimilarity and dynamic-time-warping squiggle dissimilarity,
    applying 16S abundance and homopolymer rejection filters, and writes the k-mer
    list
  repo: https://github.com/cvdelannoy/baseLess
- name: guppy
  role: Basecalls the raw MinION reads prior to read correction
- name: ont-fast5-api
  role: Splits multi-fast5 guppy output into single-read fast5 files via single_to_multi_fast5
- name: tombo
  role: Corrects reads against a reference genome (tombo resquiggle) before they are
    used as training/test input for k-mer network generation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad017
    title: 'baseLess: lightweight detection of sequences in raw MinION data'
  source_content_hash: sha256:277dd618066f7837b33b325404616a88311fd68f66a7446e59a51a8345da2a45
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cvdelannoy/baseLess
  tools:
  - baseLess
  - guppy
  - ont-fast5-api
  - tombo
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# 16s-abundance-kmer-filtering

## Summary

Selects a library of 1500 16S k-mers that are mutually dissimilar in sequence and produce easily distinguishable squiggles, using an evolutionary algorithm within baseLess's run_production_pipeline so that the k-mer detection neural networks give unambiguous signals.

## When to use

Use when building or validating a k-mer detection library for baseLess 16S sequence detection on MinION raw signal, where selected k-mers must maximize Smith–Waterman sequence and dynamic-time-warping squiggle dissimilarity.

## When NOT to use

- You only need species-level abundance classification with a pre-built model — use baseLess compile_model --model-type abundance or the pre-generated update_16s_db k-mer models instead.
- You are detecting a non-16S target for which no large read set of a known species and good reference genome is available — the README notes the pre-generated set is attuned to 16S.
- Input is already a trained baseLess .h5 model or a k-mer list; this skill generates and filters candidate k-mers, it does not run inference.

## Inputs

- Raw MinION fast5 reads (basecalled with guppy, split to single-read fast5 with ont-fast5-api)
- Tombo-resquiggled (signal-corrected) training and test reads
- Target 16S reference FASTA (e.g. reference_genome.fasta for resquiggle)

## Outputs

- k-mer list txt file with exactly 1500 lines, one k-mer per line
- K-mer detection networks (NN directory) for the selected salient 16S k-mers

## How to apply

Basecall the raw MinION reads with guppy and split multi-fast5 output into single-read fast5 files using ont-fast5-api's single_to_multi_fast5. Correct the reads against a reference genome with tombo resquiggle so signal is aligned to sequence. Run baseLess run_production_pipeline with --training-reads and --test-reads pointing at the corrected reads; this invokes an evolutionary selection that scores candidate k-mers by sequence dissimilarity (Smith–Waterman alignment, maximum score of 6 to any other selected k-mer) and squiggle dissimilarity (dynamic time warping on guppy-simulated squiggles), while discarding candidates rejected by 16S abundance filtering or homopolymer rejection. Write the surviving 1500 k-mers to a txt k-mer list (--kmer-list) with one k-mer per line. The rationale is that baseLess's composite network contains many small k-mer detectors, so poorly distinguishable k-mers would yield ambiguous detections on the electrical signal.

## Related tools

- **baseLess** (Runs run_production_pipeline with the evolutionary k-mer selection scoring Smith–Waterman sequence dissimilarity and dynamic-time-warping squiggle dissimilarity, applying 16S abundance and homopolymer rejection filters, and writes the k-mer list) — https://github.com/cvdelannoy/baseLess
- **guppy** (Basecalls the raw MinION reads prior to read correction)
- **ont-fast5-api** (Splits multi-fast5 guppy output into single-read fast5 files via single_to_multi_fast5)
- **tombo** (Corrects reads against a reference genome (tombo resquiggle) before they are used as training/test input for k-mer network generation)

## Examples

```
baseLess run_production_pipeline --training-reads path/to/training/reads --test-reads path/to/test/reads --out-dir path/to/networks/directory --cores 8 --kmer-list
```

## Evaluation signals

- The --kmer-list output txt file exists and contains exactly 1500 lines (one k-mer per line).
- Every selected k-mer has a maximum Smith–Waterman alignment score of 6 to any other selected k-mer (sequence-dissimilarity constraint).
- Dynamic time warping scores computed on guppy-simulated squiggles confirm the selected k-mers produce easily distinguishable squiggles.
- No selected k-mer is one that was rejected by 16S abundance filtering or homopolymer rejection during the evolutionary run.
- The generated k-mer detection networks load and the resulting baseLess model classifies reads containing >66% of total k-mers as target-origin.

## Limitations

- The pipeline requires a large read set of a known species, a good reference genome, and corrected reads; the pre-generated 16S k-mer set remains limited and the authors only 'plan to further expand the number of available k-mer networks'.
- K-mer model generation with --train-on-the-fly is slow (typically <1 hour) because k-mer models are generated as needed.
- K-mer generation is attuned to and tested for 16S sequence at the moment, not necessarily any target sequence.

## Evidence

- [methods] Evolutionary selection scores candidate k-mers by sequence dissimilarity (Smith-Waterman alignment): [evidence span withheld — anchor status: not_found]
- [methods] 1500 k-mer list validation: [evidence span withheld — anchor status: not_found]
- [readme] production pipeline command from README: "baseLess run_production_pipeline \
  --training-reads path/to/training/reads \
  --test-reads path/to/test/reads \
  --out-dir path/to/networks/directory"
- [intro] composite of many smaller networks: "The neural network it uses is a composite of many smaller networks, each detecting a different k-mer"
- [readme] pre-generated 16S k-mer networks scope: "baseLess comes with a pre-generated set of k-mer detection networks, for k-mers that should be particularly salient in the context of 16S detection."
