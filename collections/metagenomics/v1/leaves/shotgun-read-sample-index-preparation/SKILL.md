---
name: shotgun-read-sample-index-preparation
description: 'Use when starting a MAGinator/VAMB metagenomic profiling run: raw shotgun
  paired-end reads have been downloaded from ENA/SRA and must be indexed as a reads.csv
  before preprocessing, assembly and binning.'
when_to_use_negative:
- Input is already a feature/abundance table (e.g. a MetaPhlAn2 or MAGinator abundance
  matrix) rather than raw reads — no read index is needed.
- Data are single-end reads or non-fastq contig assemblies only — the index format
  requires paired forward/reverse fastq paths.
- A quick MAGinator re-run on existing VAMB clusters is planned where a valid reads.csv
  already exists; rebuilding the index is redundant.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MAGinator
  role: Consumes the reads.csv index (via -r/--reads) together with VAMB clusters.tsv
    and contigs.fasta to profile shotgun data
  repo: https://github.com/Russel88/MAGinator
- name: Snakemake
  role: Runs the recommended reads_to_bins.Snakefile preprocessing/assembly/binning
    workflow that takes reads.csv as its --config reads input
  repo: https://github.com/Russel88/MAGinator
- name: VAMB
  role: Downstream binner whose clusters.tsv and contigs files must be sed-sanitized
    before the MAGinator run
  repo: https://github.com/RasmussenLab/vamb
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:ce58fe99a09ba7287412d751e7c12c5af4629e4b0ac65c4d4b633d21f56da27a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Russel88/MAGinator
  tools:
  - MAGinator
  - MetaPhlAn
  - Snakemake
  - VAMB
  - conda
  - mamba
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Shotgun read sample index preparation

## Summary

Builds the comma-separated sample index (SampleName,PathToForwardReads,PathToReverseReads) that drives MAGinator and its recommended reads_to_bins.Snakefile workflow, by downloading raw shotgun reads (e.g. the 216 surviving Franzosa et al. IBD samples from ENA) and pointing the index at the paired fastq files.

## When to use

Use when starting a MAGinator/VAMB metagenomic profiling run: raw shotgun paired-end reads have been downloaded from ENA/SRA and must be indexed as a reads.csv before preprocessing, assembly and binning.

## When NOT to use

- Input is already a feature/abundance table (e.g. a MetaPhlAn2 or MAGinator abundance matrix) rather than raw reads — no read index is needed.
- Data are single-end reads or non-fastq contig assemblies only — the index format requires paired forward/reverse fastq paths.
- A quick MAGinator re-run on existing VAMB clusters is planned where a valid reads.csv already exists; rebuilding the index is redundant.

## Inputs

- Raw paired-end shotgun metagenomics reads (fastq) downloaded from ENA/SRA
- Sample metadata (sample names / accession IDs, e.g. 216 IBD samples surviving assembly, or SRA IDs dfc99c_A, f9d84e_A, 221641_A)
- maginator/test_data contigs.fasta.gz when using the bundled test set

## Outputs

- reads.csv sample index formatted as SampleName,PathToForwardReads,PathToReverseReads
- Sanitized all_assemblies.fasta and clusters.tsv (with '@' replaced by '_' via sed) ready as MAGinator inputs

## How to apply

Download the raw paired-end shotgun reads for all samples (for the IBD reanalysis this is the 216 Franzosa et al. samples surviving assembly, fetched from ENA) and record for each sample its name plus the paths to forward and reverse fastq files in a comma-separated reads.csv with columns SampleName,PathToForwardReads,PathToReverseReads. This same reads.csv is the sole input to the recommended Snakemake workflow (snakemake --use-conda -s reads_to_bins.Snakefile --config reads=reads.csv), so paths must be absolute or correct relative to the working directory. For the small built-in test, download the 3 SRA samples dfc99c_A, f9d84e_A, 221641_A (bioproject 715601), change the read paths in reads.csv, and unzip contigs.fasta.gz from maginator/test_data. After the workflow, remember the additional index hygiene steps: strip '@' characters from all_assemblies.fasta and vamb/clusters.tsv with sed before feeding MAGinator.

## Related tools

- **MAGinator** (Consumes the reads.csv index (via -r/--reads) together with VAMB clusters.tsv and contigs.fasta to profile shotgun data) — https://github.com/Russel88/MAGinator
- **Snakemake** (Runs the recommended reads_to_bins.Snakefile preprocessing/assembly/binning workflow that takes reads.csv as its --config reads input) — https://github.com/Russel88/MAGinator
- **VAMB** (Downstream binner whose clusters.tsv and contigs files must be sed-sanitized before the MAGinator run) — https://github.com/RasmussenLab/vamb

## Examples

```
snakemake --use-conda -s reads_to_bins.Snakefile --resources mem_gb=180 --config reads=reads.csv --cores 10 --printshellcmds
```

## Evaluation signals

- reads.csv has exactly three comma-separated fields per row (SampleName,PathToForwardReads,PathToReverseReads) and every referenced fastq file exists at the stated path.
- Number of rows matches the expected cohort (216 samples surviving assembly for the Franzosa et al. IBD reanalysis; 3 for the test set dfc99c_A, f9d84e_A, 221641_A).
- snakemake --use-conda -s reads_to_bins.Snakefile --config reads=reads.csv completes and produces assembly/all_assemblies.fasta and vamb/clusters.tsv.
- Post-sed files no longer contain '@' in contig or cluster identifiers (sed 's/@/_/g' applied to all_assemblies.fasta and clusters.tsv).
- MAGinator run (maginator -v clusters.tsv -r reads.csv -c all_assemblies.fasta) accepts the index without missing-sample errors; expected test output is cross-checkable against the Zenodo archive doi:10.5281/zenodo.8279036.

## Limitations

- MAGinator's high completeness of profiling may introduce noise or misclassification that influences downstream beta diversity estimation, so index errors propagate into abundance estimates.
- Running the full workflow after indexing is compute-intensive: on the test cluster each job needed 180 GB RAM and ~72 min wall time, so small servers require --max_cores and --max_mem tuning.
- Paths in reads.csv must be edited manually per environment (e.g. 'Change the paths to the read-files in reads.csv'), which is error-prone across test and production datasets.

## Evidence

- [readme] reads.csv format: "A comma-separated file giving the position of the fastq files with your sequencing reads formatted as: SampleName,PathToForwardReads,PathToReverseReads"
- [readme] workflow invocation: "snakemake --use-conda -s reads_to_bins.Snakefile --resources mem_gb=180 --config reads=reads.csv --cores 10 --printshellcmds"
- [other] task card ENA download step: [evidence span withheld — anchor status: not_found]
- [readme] test sample setup: "Download the 3 samples used for the test at SRA: https://www.ncbi.nlm.nih.gov/sra?LinkName=bioproject_sra_all&from_uid=715601 with the ID's dfc99c_A, f9d84e_A and 221641_A"
- [readme] index hygiene sed steps: "sed 's/@/_/g' assembly/all_assemblies.fasta > all_assemblies.fasta"
- [readme] resource demands caveat: "On the compute cluster each job have had access to 180gb RAM, with the following time consumption: 
real	72m27.379s"
