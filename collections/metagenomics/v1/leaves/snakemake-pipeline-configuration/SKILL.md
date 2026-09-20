---
name: snakemake-pipeline-configuration
description: 'Use when reproducing Mapler assembly-evaluation runs on HiFi metagenomic
  datasets: you need to wire sample FASTQ paths, custom assemblies, and analysis flags
  into a configfile and execute the Snakemake workflow locally or on SLURM.'
when_to_use_negative:
- The analysis goal is a single isolated tool run (e.g. only metaMDBG assembly) rather
  than the integrated Mapler assembly-plus-evaluation workflow with per-bin-quality
  read mapping metrics.
- Input data are short reads or non-HiFi data outside Mapler's HiFi long-read metagenomic
  assembly scope.
- A working directory from a previous crashed run has not been unlocked first — 'snakemake
  --unlock --configfile <configfile> -np' is required before relaunching.
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Snakemake
  role: Workflow engine executing the Mapler pipeline; requires snakemake>=8.28 from
    bioconda, launched via local_pipeline.sh or pipeline.sh configfiles
  repo: https://github.com/Nimauric/Mapler
- name: Conda
  role: Environment manager provisioning Snakemake and the pipeline's per-rule dependencies
    (stored in .snakemake/conda); requires conda>=24.1.2
  repo: https://github.com/Nimauric/Mapler
- name: snakemake-executor-plugin-slurm
  role: SLURM executor plugin enabling cluster execution via 'sbatch pipeline.sh <configfile>'
  repo: https://github.com/Nimauric/Mapler
- name: metaMDBG
  role: Built-in assembler selectable in the configfile assemblers field; also the
    assembler used for the test dataset output tree
  repo: https://github.com/GaetanBenoitDev/metaMDBG
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf334
    title: 'Mapler: a pipeline for assessing assembly quality in taxonomically rich
      metagenomes sequenced with HiFi reads'
  source_content_hash: sha256:848e08929c8e814ed9b4eda9ee3f766b230fb42bf617370a75f679bcc588fc8a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Nimauric/Mapler
  tools:
  - Bash
  - Conda
  - Mapler
  - MetaBAT2
  - Python
  - Snakemake
  - hifiasm-meta
  - metaMDBG
  - metaflye
  - minimap2
  - snakemake-executor-plugin-slurm
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# snakemake-pipeline-configuration

## Summary

Configure and launch the Mapler Snakemake pipeline by copying config/config_template.yaml, declaring samples (read_path FASTQ), assemblers, binners, and analysis toggles (read_mapping_evaluation, binning, checkm), then running via ./local_pipeline.sh or sbatch pipeline.sh with the 'mapler' conda environment.

## When to use

Use when reproducing Mapler assembly-evaluation runs on HiFi metagenomic datasets: you need to wire sample FASTQ paths, custom assemblies, and analysis flags into a configfile and execute the Snakemake workflow locally or on SLURM.

## When NOT to use

- The analysis goal is a single isolated tool run (e.g. only metaMDBG assembly) rather than the integrated Mapler assembly-plus-evaluation workflow with per-bin-quality read mapping metrics.
- Input data are short reads or non-HiFi data outside Mapler's HiFi long-read metagenomic assembly scope.
- A working directory from a previous crashed run has not been unlocked first — 'snakemake --unlock --configfile <configfile> -np' is required before relaunching.

## Inputs

- HiFi long-read metagenomic FASTQ read files (declared per sample via read_path in the configfile)
- config/config_template.yaml template copied as a working configfile
- Optional user-provided assembly FASTA at outputs/<sample_name>/<custom_assembly_process>/assembly.fasta
- Optional precomputed bins at outputs/<sample_name>/<assembly>/<custom_binning_process>_bins_reads_alignement/bins/<bins.fa>
- Optional Kraken 2 and GTDB-Tk reference databases (paths set in configfile)
- Bundled test dataset test/test_dataset.tar.gz for installation validation

## Outputs

- outputs/<sample_name>/<assembler>/assembly.fasta
- outputs/<sample>/<assembler>/reads_on_contigs_mapping_evaluation/report.txt (aligned read and aligned base ratios)
- outputs/<sample>/<assembler>/metabat2_bins_reads_alignement/read_contig_mapping.txt and read_contig_mapping_plot.pdf (ratios split by bin quality)
- checkm/checkm_report.txt and checkm-plot.pdf (bin completeness/contamination counts by quality class)
- Optional fraction-level outputs under fastqc/, kat/, kraken2/, metaquast/ depending on config toggles

## How to apply

Clone the Mapler repository (gitlab.inria.fr/mistic/mapler), create the 'mapler' conda environment with 'bioconda::snakemake>=8.28', 'conda-forge::conda>=24.1.2', and bioconda::snakemake-executor-plugin-slurm. Copy config/config_template.yaml as a working configfile and fill its three sections: Inputs (sample names and read_path entries pointing to HiFi FASTQ files, plus optional kraken2db/gtdbtk_database paths), Controls (binary toggles such as binning: true, checkm: true, read_mapping_evaluation: true, fastqc/kat/kraken2/metaquast), and Parameters (memory, threads, execution time). For user-provided assemblies, place FASTA at outputs/<sample_name>/<custom_assembly_process>/assembly.fasta and add <custom_assembly_process> to the assemblers list (names must not clash with built-ins metaMDBG, custom_assembly, metaflye, hifiasm_meta, operaMS); the same pattern applies for custom binner names vs. built-ins metabat2 and custom. Launch with ./local_pipeline.sh <configfile> > mylog.txt locally, sbatch pipeline.sh <configfile> on SLURM, or ./np_pipeline.sh to preview; unlock a crashed working directory with 'snakemake --unlock --configfile <configfile> -np'. Pre-create conda environments with 'snakemake --use-conda --conda-create-envs-only -c1 --configfile config/config_test.yaml' to speed up the first execution, and validate the installation using the bundled test dataset (~15 minutes with 16 CPUs and 15G RAM).

## Related tools

- **Snakemake** (Workflow engine executing the Mapler pipeline; requires snakemake>=8.28 from bioconda, launched via local_pipeline.sh or pipeline.sh configfiles) — https://github.com/Nimauric/Mapler
- **Conda** (Environment manager provisioning Snakemake and the pipeline's per-rule dependencies (stored in .snakemake/conda); requires conda>=24.1.2) — https://github.com/Nimauric/Mapler
- **snakemake-executor-plugin-slurm** (SLURM executor plugin enabling cluster execution via 'sbatch pipeline.sh <configfile>') — https://github.com/Nimauric/Mapler
- **metaMDBG** (Built-in assembler selectable in the configfile assemblers field; also the assembler used for the test dataset output tree) — https://github.com/GaetanBenoitDev/metaMDBG

## Examples

```
./local_pipeline.sh config/config_test.yaml > mylog.txt
```

## Evaluation signals

- The test dataset run (16 CPUs, 15G RAM) completes in ~15 minutes excluding dependency installation and reproduces the documented outputs/test_dataset/metaMDBG output tree including assembly.fasta, read_contig_mapping.txt, and reads_on_contigs_mapping_evaluation/report.txt.
- Conda environments are pre-created into .snakemake/conda by 'snakemake --use-conda --conda-create-envs-only -c1 --configfile config/config_test.yaml' without errors.
- The configfile parses with all three sections populated (Inputs, Controls, Parameters) and no custom assembler/binner names colliding with built-ins (metaMDBG, custom_assembly, metaflye, hifiasm_meta, operaMS; metabat2, custom).
- checkm_report.txt reports counts of near complete, high quality, medium quality and low quality bins followed by per-bin details, confirming checkm: true took effect.
- reads_on_contigs_mapping_evaluation/report.txt exists and reports aligned read and aligned base ratios consistent with the configured analysis (e.g. 96.9%/96.1% Mock, 49.5%/47.8% Gut, 12.6%/9.4% Deadwood at medium quality or higher in the reference study).

## Limitations

- The pipeline must be launched from within the pipeline directory.
- Only part of the analysis runs on the bundled test dataset, and config_test_evaluation_only.yaml restricts it to evaluation of a precomputed assembly and bins.
- Custom assembly/binning process names cannot duplicate built-in names (metaMDBG, custom_assembly, metaflye, hifiasm_meta, operaMS; metabat2, custom).
- Taxonomic assignment requires external Kraken 2 and GTDB-Tk databases whose paths must be set in the configfile and which consume disk space.

## Evidence

- [methods] The pipeline requires Snakemake and Conda. If running on a SLURM cluster, the Snakemake SLURM executor plugin is also needed.: "The pipeline requires Snakemake and Conda. If running on a SLURM cluster, the Snakemake SLURM executor plugin is also needed"
- [readme] It is recommended to copy config/config_template.yaml as a configfile, and to tweak it according to the analysis's needs.: "It is recommended to copy `config/config_template.yaml` as a configfile, and to tweak it according to the analysis's needs."
- [readme] The configfile is structured in three parts: Inputs, Controls, and Parameters.: "The configfile is structured in three parts:
- Inputs: path to samples and external programs
- Controls: allowing a choice among multiple non-exclusive options
- Parameters: functional parameters"
- [readme] Please note that <custom_assembly_process> cannot have the same name as any of the built-in assembly processes (metaMDBG, custom_assembly, metaflye, hifiasm_meta, operaMS).: "Please note that <custom_assembly_process> cannot have the same name as any of the built-in assembly processes (metaMDBG, custom_assembly, metaflye, hifiasm_meta, operaMS)."
- [readme] with 16 CPUs and 15G of RAM, it should require around 15 minutes, not including dependency installation, to produce the following results: "with 16 CPUs and 15G of RAM, it should require around 15 minutes, not including dependency installation, to produce the following results:"
