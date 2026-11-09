---
name: metagenome-assembly-and-binning
description: Use when starting from raw paired-end FASTQ reads of a multi-sample shotgun
  metagenomics study (e.g. reads.csv sample index) and you need de novo MAGs plus
  VAMB clusters as upstream input for MAGinator profiling.
when_to_use_negative:
- Input is already a feature/abundance table (e.g. MetaPhlAn2 profiles) rather than
  raw reads — assembly and binning would be unnecessary.
- You have only single-sample data with low depth where multi-sample VAMB binning
  performs poorly (VAMB performs excellently with multiple samples, 'pretty good on
  single-sample data').
- You only need reference-based taxonomic profiling (MetaPhlAn/mOTUs marker-gene mapping)
  rather than de novo MAG reconstruction.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: VAMB
  role: Variational-autoencoder metagenomic binner whose workflow rules are embedded
    in reads_to_bins.Snakefile to cluster contigs into bins and emit clusters.tsv
  repo: https://github.com/RasmussenLab/vamb
- name: Snakemake
  role: Workflow engine running the recommended preprocessing/assembly/binning pipeline
    (reads_to_bins.Snakefile) with --use-conda
- name: dRep
  role: Dereplicates bins into species-level MAG clusters; clustering parameters can
    be modified to create clusters at different taxonomic levels
  repo: https://github.com/MrOlm/drep
- name: MAGinator_setup.R
  role: R script in the recommended workflow that converts dRep output into the input
    files adapted for MAGinator
  repo: https://github.com/Russel88/MAGinator
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:74dd368344190ae7ab971f3ae5c9be1f916c0bcd5069740b69b630562ed4b147
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RasmussenLab/vamb
  tools:
  - MAGinator
  - MAGinator_setup.R
  - MetaPhlAn
  - Snakemake
  - VAMB
  - conda
  - dRep
  - mamba
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# metagenome-assembly-and-binning

## Summary

Preprocess shotgun metagenomics reads, assemble them per-sample, co-assemble contigs, and bin contigs into metagenome-assembled genomes (MAGs) with VAMB, producing the clusters.tsv, all_assemblies.fasta and dRep species-level cluster files that MAGinator requires as input.

## When to use

Use when starting from raw paired-end FASTQ reads of a multi-sample shotgun metagenomics study (e.g. reads.csv sample index) and you need de novo MAGs plus VAMB clusters as upstream input for MAGinator profiling.

## When NOT to use

- Input is already a feature/abundance table (e.g. MetaPhlAn2 profiles) rather than raw reads — assembly and binning would be unnecessary.
- You have only single-sample data with low depth where multi-sample VAMB binning performs poorly (VAMB performs excellently with multiple samples, 'pretty good on single-sample data').
- You only need reference-based taxonomic profiling (MetaPhlAn/mOTUs marker-gene mapping) rather than de novo MAG reconstruction.

## Inputs

- reads.csv sample index (SampleName,PathToForwardReads,PathToReverseReads) pointing to paired-end FASTQ files, e.g. ENA downloads of the Franzosa et al. IBD study
- Raw shotgun metagenomics reads for multiple samples

## Outputs

- clusters.tsv from VAMB (contig-to-bin cluster assignments)
- all_assemblies.fasta containing all contigs with unique names (leading '@' replaced by '_')
- dRep-derived species-level MAG-cluster input files (prepared via MAGinator_setup.R)
- Size-filtered bins/MAGs suitable for downstream GTDB-tk annotation

## How to apply

Format the raw paired-end reads (e.g. the 216 Franzosa et al. IBD samples from ENA) into a reads.csv index with columns SampleName,PathToForwardReads,PathToReverseReads. Run the recommended Snakemake preprocessing/assembly/binning workflow via `snakemake --use-conda -s reads_to_bins.Snakefile --resources mem_gb=180 --config reads=reads.csv --cores 10 --printshellcmds`; the binning rules are copied from the VAMB workflow, whose variational-autoencoder binner clusters the contig embedding into bins (filter bins by size with --binsize). Assemble outputs must be merged into a single contigs fasta (all_assemblies.fasta) and VAMB clusters.tsv with '@' characters replaced by '_' via sed. Dereplicate bins into species-level (or finer, since dRep clustering parameters can be tuned to other taxonomic levels) MAG clusters with dRep, then use the recommended_workflow/MAGinator_setup.R script to build the MAGinator input files from the dRep output. Correctness is judged by producing valid clusters.tsv, all_assemblies.fasta with unique contig names, and dRep-derived MAG-cluster input files that MAGinator accepts.

## Related tools

- **VAMB** (Variational-autoencoder metagenomic binner whose workflow rules are embedded in reads_to_bins.Snakefile to cluster contigs into bins and emit clusters.tsv) — https://github.com/RasmussenLab/vamb
- **Snakemake** (Workflow engine running the recommended preprocessing/assembly/binning pipeline (reads_to_bins.Snakefile) with --use-conda)
- **dRep** (Dereplicates bins into species-level MAG clusters; clustering parameters can be modified to create clusters at different taxonomic levels) — https://github.com/MrOlm/drep
- **MAGinator_setup.R** (R script in the recommended workflow that converts dRep output into the input files adapted for MAGinator) — https://github.com/Russel88/MAGinator

## Examples

```
snakemake --use-conda -s reads_to_bins.Snakefile --resources mem_gb=180 --config reads=reads.csv --cores 10 --printshellcmds
```

## Evaluation signals

- clusters.tsv and all_assemblies.fasta are produced with '@' characters replaced by '_' (sed 's/@/_/g'), since MAGinator requires uniquely named contigs.
- The number of samples surviving assembly matches expectations (216 samples for the Franzosa et al. IBD reanalysis) and reads.csv paths resolve to valid FASTQ files.
- dRep output successfully converts via MAGinator_setup.R into MAGinator-acceptable input files (MAG clusters at the chosen taxonomic level).
- Downstream MAGinator run completes and yields abundance_phyloseq.RData, relative_abundance_phyloseq.RData and gene_count_matrix.tsv, confirming bins were valid.
- Bins pass the --binsize size filter and each VAMB cluster maps to a taxonomic consensus in the MAGinator step.

## Limitations

- VAMB binning is 'pretty good on single-sample data' but performs best with multiple samples, so low-sample projects may bin poorly.
- dRep dereplication cost grows with genome count; v4 uses skani to keep this tractable, but very large bin sets still need care.
- Contig names containing '@' must be sanitized, otherwise MAGinator input is invalid.

## Evidence

- [readme] The input to the workflow is the reads.csv file. The workflow can be run using snakemake: "The input to the workflow is the reads.csv file. The workflow can be run using snakemake:"
- [readme] the rules for binning have been copied from VAMB: "we have created a recommended workflow, with preprocessing, assembly and binning* of your metagenomics reads (the rules for binning have been copied from VAMB"
- [readme] we recommend using a tool like dRep to create the species-level clusters: "Once the binning is done, we recommend using a tool like dRep (https://github.com/MrOlm/drep) to create the species-level clusters."
- [readme] dRep clustering parameters can be modified to create clusters that belong to different taxonomic levels: "The advantage of dRep is that the clustering parameters can be modified to create clusters that belong to different taxonomic levels."
- [readme] Vamb feeds kmer composition and abundance into a variational autoencoder and clusters the embedding to form bins: [evidence span withheld — anchor status: not_found]
- [readme] Its binners perform excellently with multiple samples, and pretty good on single-sample data: [evidence span withheld — anchor status: not_found]
