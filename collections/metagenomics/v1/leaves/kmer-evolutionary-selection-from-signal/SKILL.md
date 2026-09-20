---
name: kmer-evolutionary-selection-from-signal
description: Use when generating a custom set of salient k-mer detection networks
  for baseLess from a large read set of a known species, e.g. a 1500 k-mer 16S library,
  instead of using the pre-generated 16S k-mer models.
when_to_use_negative:
- When the pre-generated 16S k-mer models suffice — run `baseLess update_16s_db` to
  download them instead of regenerating networks
- When you lack a large read set of a known species plus a good reference genome —
  training reads must be corrected against a reference before use
- When the goal is abundance-mode classification against a target genome rather than
  k-mer network generation for read detection
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: baseLess
  role: Runs run_production_pipeline with evolutionary k-mer selection, scoring candidates
    by Smith–Waterman sequence dissimilarity and dynamic time warping squiggle dissimilarity,
    and emits the k-mer list with --kmer-list
  repo: https://github.com/cvdelannoy/baseLess
- name: guppy
  role: Basecalls the raw MinION reads prior to read correction
- name: ont-fast5-api
  role: Splits multi-fast5 output into single-read fast5 files via single_to_multi_fast5
  repo: https://pypi.org/project/ont-fast5-api/
- name: tombo
  role: Corrects (resquiggles) the reads against a reference genome so signal can
    be aligned to bases for training and scoring
  repo: https://anaconda.org/bioconda/ont-tombo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad017
    title: 'baseLess: lightweight detection of sequences in raw MinION data'
  source_content_hash: sha256:042a51d84df9973a7bb5b04477e18f4e860e3a934b4d78734f5148294eaa5cc7
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# kmer-evolutionary-selection-from-signal

## Summary

An evolutionary selection procedure in baseLess's run_production_pipeline that picks k-mer detection networks from raw MinION signal, keeping k-mers that are dissimilar in sequence (Smith–Waterman) and produce easily distinguishable squiggles (dynamic time warping on resquiggled reads).

## When to use

Use when generating a custom set of salient k-mer detection networks for baseLess from a large read set of a known species, e.g. a 1500 k-mer 16S library, instead of using the pre-generated 16S k-mer models.

## When NOT to use

- When the pre-generated 16S k-mer models suffice — run `baseLess update_16s_db` to download them instead of regenerating networks
- When you lack a large read set of a known species plus a good reference genome — training reads must be corrected against a reference before use
- When the goal is abundance-mode classification against a target genome rather than k-mer network generation for read detection

## Inputs

- Raw Oxford Nanopore MinION reads (multi-fast5 or single fast5)
- Reference genome FASTA for tombo resquiggle
- Corrected training reads directory (post-resquiggle)
- Corrected test reads directory (post-resquiggle)
- 16S abundance filtering criteria and homopolymer rejection rules baked into the pipeline

## Outputs

- Directory of trained k-mer detection neural networks (.h5-style networks for the composite model)
- K-mer list txt file with one k-mer per line (e.g. 1500 lines for the 16S library)

## How to apply

Basecall raw reads with guppy and split multi-fast5 output into single-read fast5 files with ont-fast5-api's single_to_multi_fast5. Correct the reads against a reference genome using tombo resquiggle so the electrical signal is aligned to bases. Supply the corrected training and test reads to `baseLess run_production_pipeline`, which invokes an evolutionary algorithm that scores candidate k-mers by sequence dissimilarity (maximum Smith–Waterman alignment score of 6 to other selected k-mers) and by squiggle dissimilarity (dynamic time warping on squiggles), while discarding candidates rejected by 16S abundance filtering or homopolymer rejection. Write the surviving k-mers to a txt k-mer list with `--kmer-list`, one k-mer per line, alongside the trained neural networks. Validate that the k-mer list contains the expected number of entries (e.g. 1500 lines for the 16S library) and that selected k-mers satisfy the sequence- and squiggle-dissimilarity constraints.

## Related tools

- **baseLess** (Runs run_production_pipeline with evolutionary k-mer selection, scoring candidates by Smith–Waterman sequence dissimilarity and dynamic time warping squiggle dissimilarity, and emits the k-mer list with --kmer-list) — https://github.com/cvdelannoy/baseLess
- **guppy** (Basecalls the raw MinION reads prior to read correction)
- **ont-fast5-api** (Splits multi-fast5 output into single-read fast5 files via single_to_multi_fast5) — https://pypi.org/project/ont-fast5-api/
- **tombo** (Corrects (resquiggles) the reads against a reference genome so signal can be aligned to bases for training and scoring) — https://anaconda.org/bioconda/ont-tombo

## Examples

```
baseLess run_production_pipeline --training-reads path/to/training/reads --test-reads path/to/test/reads --out-dir path/to/networks/directory --cores 8 --kmer-list
```

## Evaluation signals

- The k-mer list output file exists and contains exactly the expected number of entries (e.g. 1500 lines for the 16S library), one k-mer per line
- Every selected k-mer has a maximum Smith–Waterman alignment score of 6 or lower against all other selected k-mers (sequence dissimilarity constraint)
- Selected k-mers produce easily distinguishable squiggles, verified by dynamic time warping scores computed on guppy-simulated squiggles
- No selected k-mer was rejected by 16S abundance filtering or homopolymer rejection during evolution
- Compiled detection networks load and run in `baseLess run_inference` on the held-out test reads

## Limitations

- Requires a large read set of a known species and a good reference genome for read correction; reads must be corrected with tombo resquiggle before training
- The pipeline takes longer to run because k-mer models are generated as needed (the README notes typically <1 hour for on-the-fly training, more CPUs speed it up)
- Pre-generated networks are currently limited to k-mers salient for 16S detection; other targets require generating your own networks
- baseLess is tuned to 16S detection; read_detection mode (>66% k-mer fraction threshold) has been attuned to 16S sequence specifically

## Evidence

- [other] An evolutionary algorithm was used to select k-mers that are dissimilar in sequence (maximum Smith–Waterman score of 6 to other selected k-mers): [evidence span withheld — anchor status: not_found]
- [readme] Correct your reads with [tombo](https://anaconda.org/bioconda/ont-tombo):: "Correct your reads with [tombo](https://anaconda.org/bioconda/ont-tombo):
``ntombo resquiggle single/fast5/output/folder reference_genome.fasta
"
- [readme] baseLess comes with a pre-generated set of k-mer detection networks, for k-mers that should be particularly salient in the context of 16S detection: "baseLess comes with a pre-generated set of k-mer detection networks, for k-mers that should be particularly salient in the context of 16S detection."
- [other] Run the baseLess run_production_pipeline on the corrected training and test reads, invoking the evolutionary selection: [evidence span withheld — anchor status: not_found]
- [intro] The neural network it uses is a composite of many smaller networks, each detecting a different k-mer: "The neural network it uses is a composite of many smaller networks, each detecting a different k-mer, thus reconfiguration of baseLess to detect a different sequence amounts to combining different"
