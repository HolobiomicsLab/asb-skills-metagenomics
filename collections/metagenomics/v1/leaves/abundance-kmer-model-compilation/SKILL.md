---
name: abundance-kmer-model-compilation
description: Use when you have genome FASTAs for a target species plus closely related
  background species and need a model to classify raw MinION fast5 reads (pure samples)
  by k-mer abundance without basecalling.
when_to_use_negative:
- Sample is amplified/metagenomic (e.g. 16S amplicon or microbial mixture) — use read_detection
  mode instead, which retains reads with >66% of target k-mers
- No reference genome FASTA is available for the target or background species
- Input is already basecalled sequence data where standard alignment/classification
  pipelines are appropriate
edam_operation: http://edamontology.org/operation_3840
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0102
tools:
- name: baseLess
  role: Provides 'compile_model --model-type abundance' to build the composite k-mer-detecting
    neural network from target/background FASTAs
  repo: https://github.com/cvdelannoy/baseLess
- name: mamba
  role: Fast conda drop-in replacement used to install baseLess from bioconda
- name: conda
  role: Alternative package manager for installing baseLess
- name: tombo
  role: Resquiggle-corrects training reads prior to --train-on-the-fly model compilation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad017
    title: 'baseLess: lightweight detection of sequences in raw MinION data'
  source_content_hash: sha256:b06bfbc9ded1a7c9eb62d60d0c9a72d711f136ebd25fd07290a3a64ce3aef708
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cvdelannoy/baseLess
  tools:
  - baseLess
  - conda
  - mamba
  - tombo
  edam_operation: http://edamontology.org/operation_3840
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# abundance-kmer-model-compilation

## Summary

Compile a baseLess abundance-mode .h5 model that discriminates one target genome from specified background genomes by comparing k-mer abundances found in MinION electrical signal to genome k-mer abundances, enabling species-level detection without basecalling.

## When to use

Use when you have genome FASTAs for a target species plus closely related background species and need a model to classify raw MinION fast5 reads (pure samples) by k-mer abundance without basecalling.

## When NOT to use

- Sample is amplified/metagenomic (e.g. 16S amplicon or microbial mixture) — use read_detection mode instead, which retains reads with >66% of target k-mers
- No reference genome FASTA is available for the target or background species
- Input is already basecalled sequence data where standard alignment/classification pipelines are appropriate

## Inputs

- Target genome FASTA (e.g. Illumina assembly of P. gracilis, P. januarius or P. turneri)
- One or more background genome FASTAs of related species (e.g. the other two Poeciliidae genomes)
- Optional: directory of corrected training reads (tombo resquiggle-corrected single fast5)
- Directory of pre-generated k-mer detection networks (--nn-directory), optionally fetched via baseLess update_16s_db

## Outputs

- Compiled baseLess abundance model (.h5) usable as input to 'baseLess run_inference'
- Optionally generated on-the-fly k-mer detection networks

## How to apply

Install baseLess from bioconda via mamba (channels bioconda, conda-forge, cvdelannoy), optionally downloading pre-generated 16S k-mer networks with 'baseLess update_16s_db'. For each target species, run 'baseLess compile_model --model-type abundance' supplying the target genome FASTA via --target-fasta and the other species' FASTAs via --background-fastas, saving the composite neural-network model as an .h5 file. If reads from species close to the target are available, add --training-reads with --train-on-the-fly (reads must be corrected, e.g. tombo resquiggle, before training) to increase the probability of finding salient k-mers; k-mer models are generated on the fly, typically taking <1 hour. Choose abundance mode (rather than read_detection mode) for pure samples and larger genomes, e.g. differentiating visually similar fish species or strains.

## Related tools

- **baseLess** (Provides 'compile_model --model-type abundance' to build the composite k-mer-detecting neural network from target/background FASTAs) — https://github.com/cvdelannoy/baseLess
- **mamba** (Fast conda drop-in replacement used to install baseLess from bioconda)
- **conda** (Alternative package manager for installing baseLess)
- **tombo** (Resquiggle-corrects training reads prior to --train-on-the-fly model compilation)

## Examples

```
baseLess compile_model --model-type abundance --target-fasta target.fasta --background-fastas background1.fasta background2.fasta --nn-directory directory/of/nns --out-model your_model_name.h5
```

## Evaluation signals

- Compiled model is a valid .h5 file loadable by 'baseLess run_inference --model'
- On a held-out readset (e.g. 2000 MinION reads), the model consistently assigns reads to the correct target species against all background species (as shown for the three Poeciliidae guppies)
- The model's stability metric (MSRD) plateaus within a modest read count (52, 352 and 84 reads for P. gracilis, P. januarius and P. turneri in the validation)
- With --train-on-the-fly, k-mer models are generated without errors and compilation finishes typically within <1 hour

## Limitations

- Abundance mode retains no reads (unlike read_detection mode); classification is sample/genome-level, not per-read
- Reads used for --train-on-the-fly must be corrected (tombo resquiggle) prior to usage
- Abundance mode is intended for pure samples; mixtures are better handled by read_detection mode (attuned to 16S)

## Evidence

- [readme] BaseLess determines whether the reads originate from a given target genome or not, by comparing the abundance of k-mers it finds to the abundance of k-mers in the genomes.: "BaseLess determines whether the reads originate from a given target genome or not, by comparing the abundance of k-mers it finds to the abundance of k-mers in the genomes."
- [methods] compile abundance model: "baseLess compile_model --model-type abundance --target-fasta target.fasta"
- [readme] training reads increase salient k-mer discovery: "If you have reads of species close to your target species, you can use these as training reads and increase the probability of finding salient k-mers in your training data significantly."
- [methods] install baseLess via mamba: "mamba install -c bioconda -c conda-forge -c cvdelannoy baseless"
- [results] BaseLess consistently called the correct species for each analyzed 2000-read set (Fig. 2A-C): [evidence span withheld — anchor status: not_found]
