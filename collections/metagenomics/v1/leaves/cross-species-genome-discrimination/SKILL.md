---
name: cross-species-genome-discrimination
description: Use when raw ONT MinION fast5 reads must be assigned to one of several
  closely related species (e.g., Poeciliidae guppies) using only genome FASTAs, on
  inexpensive hardware without basecalling.
when_to_use_negative:
- Samples are amplified/metagenomic mixtures rather than pure samples — read_detection
  mode tuned for 16S is the appropriate mode instead
- Input data is already basecalled (FASTQ/BAM) — standard sequence classification
  tools avoid re-training signal-level models
- Reference genomes for the competing species are unavailable, since abundance mode
  requires both target and background FASTAs
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: baseLess
  role: Compiles abundance-mode k-mer models and runs signal-level inference on fast5
    reads to call the target species
  repo: https://github.com/cvdelannoy/baseLess
- name: mamba
  role: Installs baseLess from the bioconda/conda-forge/cvdelannoy channels
- name: ont-fast5-api
  role: Splits multi-fast5 output into single-read fast5 files expected by run_inference
- name: MinION
  role: Produces the raw electrical-signal fast5 readsets that are classified without
    basecalling
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad017
    title: 'baseLess: lightweight detection of sequences in raw MinION data'
  source_content_hash: sha256:ee614df4b4a9c87ba13e28524a1085420f90a2935566912642812a341eb1a31e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cvdelannoy/baseLess
  tools:
  - MinION
  - baseLess
  - conda
  - mamba
  - ont-fast5-api
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# cross-species-genome-discrimination

## Summary

Configure baseLess in abundance mode to discriminate between closely related species from raw MinION fast5 signal, compiling one model per target species against the genomes of the related species as background, without basecalling.

## When to use

Use when raw ONT MinION fast5 reads must be assigned to one of several closely related species (e.g., Poeciliidae guppies) using only genome FASTAs, on inexpensive hardware without basecalling.

## When NOT to use

- Samples are amplified/metagenomic mixtures rather than pure samples — read_detection mode tuned for 16S is the appropriate mode instead
- Input data is already basecalled (FASTQ/BAM) — standard sequence classification tools avoid re-training signal-level models
- Reference genomes for the competing species are unavailable, since abundance mode requires both target and background FASTAs

## Inputs

- Target species genome FASTA (one per species in the comparison set)
- Background species genome FASTAs (the remaining species' genomes)
- Raw MinION fast5 read directory (single-read fast5; split multi-fast5 with ont-fast5-api if necessary)
- Pre-generated k-mer detection networks (baseLess update_16s_db or run_production_pipeline)

## Outputs

- Compiled abundance-model .h5 file per target species
- Per-species classification of the readset (correct species call)
- MSRD-vs-reads stability curve with a stabilization point (e.g., 52/352/84 reads for P. gracilis/P. januarius/P. turneri)

## How to apply

For each species, compile an abundance-mode model with 'baseLess compile_model --model-type abundance', passing that species' genome FASTA as --target-fasta and the other species' genome FASTAs as --background-fastas, saving each as an .h5 model. Abundance mode compares the abundance of detected k-mers in reads against k-mer abundances in the target versus background genomes, which suits pure samples and visually similar species/strains. Run 'baseLess run_inference --inference-mode once' with each compiled model on the corresponding single-read fast5 set (split multi-fast5 with ont-fast5-api's single_to_multi_fast5 if needed). Track the mean squared read difference (MSRD) stability metric as reads accumulate; in the guppy validation, MSRD stabilized after 52, 352 and 84 reads for P. gracilis, P. januarius and P. turneri respectively, far fewer than the available 2000 reads. Accept the classification only when each model calls the correct target species and the MSRD-vs-reads curve plateaus.

## Related tools

- **baseLess** (Compiles abundance-mode k-mer models and runs signal-level inference on fast5 reads to call the target species) — https://github.com/cvdelannoy/baseLess
- **mamba** (Installs baseLess from the bioconda/conda-forge/cvdelannoy channels)
- **ont-fast5-api** (Splits multi-fast5 output into single-read fast5 files expected by run_inference)
- **MinION** (Produces the raw electrical-signal fast5 readsets that are classified without basecalling)

## Examples

```
baseLess compile_model --model-type abundance --target-fasta p_gracilis.fasta --background-fastas p_januarius.fasta p_turneri.fasta --out-model gracilis.h5
```

## Evaluation signals

- Each abundance model assigns the 2000-read fast5 set to the correct target species (Fig. 2A-C reproduction)
- MSRD-vs-reads curves plateau, reaching stable values after ~52, 352 and 84 reads for P. gracilis, P. januarius and P. turneri
- Full 2000-read sets are not required — classification should be stable well before all reads are processed
- Compiled models exist as valid .h5 files for every species in the comparison set
- run_inference terminates after processing all reads when invoked with --inference-mode once

## Limitations

- Abundance mode retains no reads (unlike read_detection mode, which keeps target-containing fast5s); it only yields an aggregate call
- Requires genome FASTAs for all species being discriminated; salient k-mer discovery improves markedly when training reads of closely related species are supplied (with tombo resquiggle correction)
- baseLess runs indefinitely in streaming mode and must be stopped manually; only --inference-mode once limits it to a directory of reads

## Evidence

- [readme] abundance model for visually similar species: "Useful for differentiating visually similar species or strains"
- [methods] abundance model construction: "baseLess compile_model --model-type abundance --target-fasta target.fasta"
- [abstract] fish species discrimination finding: "baseLess can accurately determine the identity of reads between three closely related fish species"
- [intro] hardware rationale: "BaseLess reduces your MinION sequencing device to a simple species detector. As a trade-off, it runs on inexpensive computational hardware like single-board computers."
- [readme] inference invocation: "baseLess run_inference --fast5-in directory/of/fast5s --out-dir your/output/directory --model your_model_name.h5"
