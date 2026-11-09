---
name: bioconda-environment-management
description: Use when a workflow requires multiple short-read genome assemblers (ABySS,
  MEGAHIT, SPAdes, Velvet) or QUAST, and tool installation via Bioconda into a conda
  environment is needed before running assembly or quality assessment.
when_to_use_negative:
- QUAST will be run locally and you prefer its zero-install mode, since 'QUAST automatically
  compiles all its sub-parts when needed (on the first use)' and cloning the GitHub
  repo suffices
- The target platform cannot use conda (e.g. only system apt/brew package managers
  are permitted) — the QUAST README documents apt-get/brew prerequisite installation
  paths instead
- Only a single assembler output already exists and no new tool installation is required
  for the analysis
tools:
- name: QUAST
  role: Quality assessment tool installed from Bioconda to evaluate the four generated
    E. coli assemblies
  repo: https://github.com/ablab/quast
- name: ABySS
  role: Short-read assembler installed from Bioconda; run with the GAGE-B recipe because
    its default assembly was of very poor quality
- name: MEGAHIT
  role: Short-read assembler installed from Bioconda; run with default parameters
- name: SPAdes
  role: Short-read assembler installed from Bioconda; run with default/recommended
    parameters
- name: Velvet
  role: Short-read assembler installed from Bioconda; run via velveth/velvetg with
    recommended parameters
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkad406
    title: 'WebQUAST: online evaluation of genome assemblies'
  source_content_hash: sha256:2000af1503665def7834fdcec8e2f719be86d5ff92a8698e9600b627dc1211fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/quast
  tools:
  - ABySS
  - MEGAHIT
  - QUAST
  - SPAdes
  - Velvet
  - WebQUAST
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# bioconda-environment-management

## Summary

Set up an isolated conda environment and install short-read genome assemblers (ABySS, MEGAHIT, SPAdes, Velvet) and QUAST from Bioconda so that reproducible assembly workflows can run without manual dependency resolution. This is how the WebQUAST paper generated its demonstration assemblies of the E. coli K-12 MG1655 dataset.

## When to use

Use when a workflow requires multiple short-read genome assemblers (ABySS, MEGAHIT, SPAdes, Velvet) or QUAST, and tool installation via Bioconda into a conda environment is needed before running assembly or quality assessment.

## When NOT to use

- QUAST will be run locally and you prefer its zero-install mode, since 'QUAST automatically compiles all its sub-parts when needed (on the first use)' and cloning the GitHub repo suffices
- The target platform cannot use conda (e.g. only system apt/brew package managers are permitted) — the QUAST README documents apt-get/brew prerequisite installation paths instead
- Only a single assembler output already exists and no new tool installation is required for the analysis

## Inputs

- Conda environment specification or list of Bioconda package names (abyss, megahit, spades, velvet, quast)
- Trimmed paired-end sequencing reads (FASTQ, e.g. SRA accession ERR008613 for the E. coli K-12 MG1655 dataset)
- Host OS prerequisites (Python3 >=3.3, Perl >=5.6.0, GCC >=4.7, zlib dev files)

## Outputs

- Conda environment containing four functional assemblers and QUAST
- Four contig FASTA files (one per assembler: ABySS, MEGAHIT, SPAdes, Velvet)
- QUAST/WebQUAST evaluation reports (report.tsv, report.html, icarus.html)

## How to apply

Create a dedicated conda environment and install the shortlisted assemblers (ABySS, MEGAHIT, SPAdes, Velvet) from Bioconda; QUAST is likewise distributed on Bioconda per its README badge. Run ABySS with the GAGE-B parameter recipe (evaluating multiple k-mer sizes and selecting the best assembly) because its default assembly is of very poor quality, while MEGAHIT, SPAdes, and Velvet are run with default or documentation-recommended parameters on the trimmed paired-end reads. QUAST itself needs Python3 (>=3.3), Perl (>=5.6.0), GCC (>=4.7), GNU make/ar, and zlib dev files, with optional dependencies such as Java 1.8+ and R for GRIDSS and the Time::HiRes Perl module for GeneMark-ES. If Bioconda is unsuitable, QUAST can alternatively be installed with ./setup.py install (or install_full.sh for SV-detection and metagenomic reference-detection components) since it compiles sub-parts on first use. The rationale is that all tools come from one curated channel, keeping the four-assembly WebQUAST demonstration reproducible.

## Related tools

- **QUAST** (Quality assessment tool installed from Bioconda to evaluate the four generated E. coli assemblies) — https://github.com/ablab/quast
- **ABySS** (Short-read assembler installed from Bioconda; run with the GAGE-B recipe because its default assembly was of very poor quality)
- **MEGAHIT** (Short-read assembler installed from Bioconda; run with default parameters)
- **SPAdes** (Short-read assembler installed from Bioconda; run with default/recommended parameters)
- **Velvet** (Short-read assembler installed from Bioconda; run via velveth/velvetg with recommended parameters)

## Examples

```
conda create -n assemblers -c bioconda abyss megahit spades velvet quast
```

## Evaluation signals

- Each of the four assemblers runs without dependency errors and emits a contig FASTA file
- Total assembly lengths fall in the ~4.6-4.8 Mb range expected for E. coli K-12
- Contig counts fall within the reported 90-176 range: ~90-95 contigs for Velvet, SPAdes and MEGAHIT; ABySS the most fragmented at 176 contigs
- ABySS was configured with the GAGE-B k-mer evaluation recipe rather than its default settings
- QUAST from Bioconda executes its usage example (quast.py on test contigs with -r reference) successfully, producing report.tsv and icarus.html

## Limitations

- ABySS's default assembly is of very poor quality on this dataset, so default-parameter installation alone is insufficient — the GAGE-B recipe must be applied
- QUAST's full installation is much larger (~540 MB for setup_full, which bundles SV-detection and metagenomic reference-detection tools) versus ~120 MB for basic installation
- Some optional QUAST submodules require extra runtime dependencies not bundled by default (Java 1.8+ and R for GRIDSS; Time::HiRes Perl module for GeneMark-ES)

## Evidence

- [methods] all installed via Bioconda: MEGAHIT, SPAdes, and Velvet ran with default or documentation-recommended parameters, while ABySS used the GAGE-B recipe because its default assembly was of very poor quality.: [evidence span withheld — anchor status: not_found]
- [results] ABySS produced the largest (4.8 Mb versus 4.6 Mb) but also the most fragmented assembly (176 contigs versus 90–95 for Velvet, SPAdes and MEGAHIT): "ABySS produced the largest (4.8 Mb versus 4.6 Mb) but also the most fragmented assembly (176 contigs versus 90–95 for Velvet, SPAdes and MEGAHIT)"
- [readme] Time::HiRes perl module for GeneMark-ES (needed when using `--gene-finding --eukaryote`): "Time::HiRes perl module for GeneMark-ES (needed when using `--gene-finding --eukaryote`)"
- [readme] QUAST automatically compiles all its sub-parts when needed (on the first use).: "QUAST automatically compiles all its sub-parts when needed (on the first use)."
