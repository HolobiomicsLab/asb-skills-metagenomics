---
name: hifi-long-read-metagenome-assembly
description: Use when HiFi long reads (FASTQ) from a metagenomic sample must be de
  novo assembled with multiple assemblers to compare assembly strategies or feed downstream
  binning/quality evaluation.
when_to_use_negative:
- The input is short-read or hybrid data for which an assembler like OPERA-MS (hybrid
  metagenomic assembler) is intended rather than the HiFi-only assemblers configured
  here
- Only a precomputed assembly needs quality evaluation with no assembly step — use
  the evaluation-only config (config_test_evaluation_only.yaml) or submit the assembly
  as a custom assembler instead of re-assembling
- The sample is a simple isolate/low-complexity genome where metagenome-specific assemblers
  and bin-quality diversity metrics add no value over a standard isolate assembly
  workflow
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: Mapler
  role: Snakemake-based pipeline that orchestrates the HiFi metagenome assembly with
    the configured assemblers and handles conda dependencies
  repo: https://github.com/Nimauric/Mapler
- name: metaMDBG
  role: Fast, memory-efficient HiFi metagenome assembler; one of the three built-in
    assembler choices in the Mapler configfile
  repo: https://github.com/GaetanBenoitDev/metaMDBG
- name: metaflye
  role: Long-read de novo assembler (Flye) used as a built-in assembler choice for
    HiFi metagenome assembly
  repo: https://github.com/mikolmogorov/Flye
- name: hifiasm-meta
  role: hifiasm fork for metagenome assembly from HiFi reads; built-in assembler option
    named hifiasm_meta in the configfile
  repo: https://github.com/xfengnefx/hifiasm-meta
- name: Snakemake
  role: Workflow engine required by Mapler (>=8.28) executing the assembly rules locally
    or via the SLURM executor plugin
- name: Conda
  role: Dependency manager for the mapler environment and per-rule pipeline environments
    stored in .snakemake/conda
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf334
    title: 'Mapler: a pipeline for assessing assembly quality in taxonomically rich
      metagenomes sequenced with HiFi reads'
  source_content_hash: sha256:f29bbca74365578797080e15c9d4f6476c248b856bc54cf9b92b7e16dfdb52cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Nimauric/Mapler
  tools:
  - Conda
  - GTDB-Tk
  - Mapler
  - MetaBAT2
  - MetaQUAST
  - Snakemake
  - hifiasm-meta
  - metaMDBG
  - metaflye
  - minimap2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# hifi-long-read-metagenome-assembly

## Summary

Assemble PacBio HiFi metagenomic reads with state-of-the-art long-read assemblers (metaMDBG, metaflye, hifiasm-meta) orchestrated by the Mapler Snakemake pipeline, producing per-assembler contig sets (assembly.fasta) that can then be quality-evaluated and compared on taxonomically rich samples.

## When to use

Use when HiFi long reads (FASTQ) from a metagenomic sample must be de novo assembled with multiple state-of-the-art assemblers to compare assembly strategies or feed downstream binning/quality evaluation.

## When NOT to use

- The input is short-read or hybrid data for which an assembler like OPERA-MS (hybrid metagenomic assembler) is intended rather than the HiFi-only assemblers configured here
- Only a precomputed assembly needs quality evaluation with no assembly step — use the evaluation-only config (config_test_evaluation_only.yaml) or submit the assembly as a custom assembler instead of re-assembling
- The sample is a simple isolate/low-complexity genome where metagenome-specific assemblers and bin-quality diversity metrics add no value over a standard isolate assembly workflow

## Inputs

- HiFi long-read FASTQ file(s) per sample (read_path in the Mapler configfile)
- Mapler YAML configfile derived from config/config_template.yaml (samples, assemblers list, controls, resource parameters)
- Optional user-provided assemblies at outputs/<sample_name>/<custom_assembly_process>/assembly.fasta

## Outputs

- outputs/<sample_name>/<assembler>/assembly.fasta per assembler (metaMDBG, metaflye, hifiasm_meta)
- Pipeline execution logs in logs/<analysis_name>/<date_hour> for wall-clock timing
- Downstream evaluation artifacts (metabat2_bins_reads_alignement, reads_on_contigs_mapping_evaluation/report.txt) when binning and read_mapping_evaluation are enabled

## How to apply

Clone the Mapler repository, create the 'mapler' conda environment (bioconda::snakemake>=8.28, conda-forge::conda>=24.1.2, and snakemake-executor-plugin-slurm for clusters), and copy config/config_template.yaml as a per-dataset configfile listing samples with read_path to the HiFi FASTQ and uncommenting the desired assemblers (metaMDBG, metaflye, hifiasm_meta; note hifiasm-meta is named hifiasm_meta in the config). Optionally pre-build conda environments with 'snakemake --use-conda --conda-create-envs-only -c1' to speed up the first run, then launch with ./local_pipeline.sh <configfile> locally or 'sbatch pipeline.sh <configfile>' on SLURM, setting resource parameters (e.g. 48 threads, 200G memory) in the configfile. Each assembler writes outputs/<sample_name>/<assembler>/assembly.fasta; custom user assemblies can be evaluated by placing assembly.fasta under outputs/<sample_name>/<custom_assembly_process>/ and adding that name to the assemblers list, but the custom process cannot share a name with built-ins (metaMDBG, custom_assembly, metaflye, hifiasm_meta, operaMS). Verify completion by checking that assembly.fasta exists per assembler before running the evaluation modules (MetaBAT2 binning, CheckM classification, read-on-contig mapping ratios with minimap2).

## Related tools

- **Mapler** (Snakemake-based pipeline that orchestrates the HiFi metagenome assembly with the configured assemblers and handles conda dependencies) — https://github.com/Nimauric/Mapler
- **metaMDBG** (Fast, memory-efficient HiFi metagenome assembler; one of the three built-in assembler choices in the Mapler configfile) — https://github.com/GaetanBenoitDev/metaMDBG
- **metaflye** (Long-read de novo assembler (Flye) used as a built-in assembler choice for HiFi metagenome assembly) — https://github.com/mikolmogorov/Flye
- **hifiasm-meta** (hifiasm fork for metagenome assembly from HiFi reads; built-in assembler option named hifiasm_meta in the configfile) — https://github.com/xfengnefx/hifiasm-meta
- **Snakemake** (Workflow engine required by Mapler (>=8.28) executing the assembly rules locally or via the SLURM executor plugin)
- **Conda** (Dependency manager for the mapler environment and per-rule pipeline environments stored in .snakemake/conda)

## Examples

```
conda activate mapler && sbatch pipeline.sh config/config_deadwood.yaml
```

## Evaluation signals

- outputs/<sample_name>/<assembler>/assembly.fasta exists for every assembler listed in the configfile (metaMDBG, metaflye, hifiasm_meta)
- Pipeline completes without crash; if a crash occurs, the working directory is unlocked with 'snakemake --unlock --configfile <configfile> -np' before rerun
- When read_mapping_evaluation is enabled, reads_on_contigs_mapping_evaluation/report.txt is produced with aligned read ratio (aligned read count / total read count) and aligned base ratio (aligned read bases / total read length)
- On the Deadwood co-assembly, reproduced unaligned-base percentages match reported values: 52.7% (metaMDBG), 76.7% (metaFlye), 66.1% (hifiasm-meta)
- The test dataset config (config_test.yaml) with 16 CPUs and 15G RAM finishes in roughly 15 minutes excluding dependency installation

## Limitations

- Only built-in HiFi-oriented assemblers are supported; custom assembly processes cannot share names with built-ins (metaMDBG, custom_assembly, metaflye, hifiasm_meta, operaMS)
- First execution requires downloading conda dependencies unless pre-built with --conda-create-envs-only; the test run only covers part of the analysis
- Metagenome assembly remains challenging in complex environmental samples with hundreds to thousands of populations, even with HiFi long reads; low-abundance populations may stay uncaptured (low abundance reads are more likely abundant in the unmapped portions)
- External Kraken 2 and GTDB-Tk databases must be manually provisioned and their paths set in the configfile (kraken2db, gtdbtk_database) if taxonomic assignment is desired

## Evidence

- [abstract] Mapler can assemble HiFi reads itself: "Mapler is a metagenome assembly and evaluation pipeline with a focus on evaluating the quality of Hi-Fi long read metagenome assemblies."
- [methods] multiple assemblers supported: "evaluate HiFi long-reads metagenomic assemblies, and it can either perform the assembly itself using multiple state-of-the-art assemblers"
- [readme] conda environment setup: "conda create -n mapler -c bioconda -c conda-forge 'bioconda::snakemake>=8.28' 'conda-forge::conda>=24.1.2' bioconda::snakemake-executor-plugin-slurm"
- [readme] custom assembler naming constraint: "<custom_assembly_process> cannot have the same name as any of the built-in assembly processes (metaMDBG, custom_assembly, metaflye, hifiasm_meta, operaMS)"
- [methods] low-abundance bias of unmapped reads: "Typically, low abundance reads are more libely to be more abundant in the unmapped portions"
