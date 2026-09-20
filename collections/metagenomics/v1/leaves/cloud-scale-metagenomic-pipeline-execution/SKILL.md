---
name: cloud-scale-metagenomic-pipeline-execution
description: Use when a reproduction target requires executing a multi-component metagenomic
  pipeline (VirMAP/MEGAHIT/BBMap/DIAMOND) over many SRA samples, with resource provisioning
  and cohort-level metric computation such as average reconstructed genome segments
  per sample.
when_to_use_negative:
- Input is a single already-classified read set or feature table where no de-novo
  plus mapping assembly reconstruction is required
- The host environment cannot provide ~64 GB RAM per simultaneous Virmap.pl instance
  or all cores for --threads
- The target virus/genome is absent from the Genbank viral and phage divisions (gbvrl/gbphg)
  database snapshot the VirMAP database was built from
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3316
tools:
- name: VirMAP (Virmap.pl)
  role: 'Perl driver script executing the full pipeline: dereplication, digital normalization,
    BBMap/DIAMOND alignments, MEGAHIT/Tadpole assembly, and super-scaffold construction'
  repo: https://github.com/cmmr/VirMAP
- name: DIAMOND
  role: Translated (protein) alignment of reads against the viral protein database
    during read assignment
  repo: https://github.com/bbuchfink/diamond
- name: MEGAHIT
  role: Multi-kmer de-novo assembler invoked via --useMegahit to create the working
    contig set
  repo: https://github.com/voutcn/megahit
- name: BBMap/Tadpole
  role: Nucleotide alignment to the viral database and digital normalization (--useBbnorm);
    alternative de-novo assembler (Tadpole)
- name: khmer
  role: Provides normalize-by-median.py used for digital normalization of input reads
  repo: https://github.com/dib-lab/khmer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:4dd677dd1ea298ff6eb25526972ccdc982ca3077e33cc871075ac26fa3439c3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cmmr/VirMAP
  tools:
  - BBMap
  - BBMap/Tadpole
  - DIAMOND
  - MEGAHIT
  - Perl
  - Tadpole
  - VirMAP
  - VirMAP (Virmap.pl)
  - khmer
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# cloud-scale-metagenomic-pipeline-execution

## Summary

Runs the VirMAP Perl pipeline end-to-end on large clinical metagenomic datasets (e.g., the UMERS influenza respiratory dataset, BioProject PRJEB7888) by downloading SRA reads, executing Virmap.pl against the prebuilt viral database, and collecting per-sample super-scaffold reconstruction results at cohort scale.

## When to use

Use when a reproduction target requires executing a multi-component metagenomic pipeline (VirMAP/MEGAHIT/BBMap/DIAMOND) over many SRA samples, with resource provisioning and cohort-level metric computation such as average reconstructed genome segments per sample.

## When NOT to use

- Input is a single already-classified read set or feature table where no de-novo plus mapping assembly reconstruction is required
- The host environment cannot provide ~64 GB RAM per simultaneous Virmap.pl instance or all cores for --threads
- The target virus/genome is absent from the Genbank viral and phage divisions (gbvrl/gbphg) database snapshot the VirMAP database was built from

## Inputs

- Paired-end FASTQ files from SRA (BioProject PRJEB7888, e.g., accession SRR9875293), downloaded with fasterq-dump
- Prebuilt VirMAP database files: Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd (Genbank viral gbvrl and phage gbphg divisions)

## Outputs

- Per-sample super-scaffold output with reconstructed influenza genome segments (one contig per segment)
- influenza_virmap_results.csv: per-sample table of influenza reads recruited and segments reconstructed
- Cohort-level metrics: 9.05 average influenza segments per positive sample and 509,039 total recruited influenza reads

## How to apply

Download the paired-end influenza metagenome samples (BioProject PRJEB7888, 25 paired-end samples) from the Sequence Read Archive using fasterq-dump. For each sample, run Virmap.pl against the prebuilt VirMAP database (Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd) with --useMegahit --useBbnorm enabled and --threads set to all available cores; provision at least 64 GB RAM per simultaneous instance because the pipeline chains dereplication/digital normalization (BBMap), nucleotide and translated alignments (BBMap/DIAMOND), and multi-kmer de-novo assembly (MEGAHIT or Tadpole). For each influenza-positive sample, tally influenza reads assigned to viral genomes by nucleotide and translated alignment and count reconstructed segments (one contig per segment) from the VirMAP super-scaffold output. Compute cohort-level reproduction metrics — 9.05 average influenza segments per positive sample and 509,039 total recruited influenza reads — and write the per-sample table to influenza_virmap_results.csv. Optionally subsample the 1.75 billion reads to lower depths, rerun reconstruction, and compare VirMAP versus standard-approach segment lengths to reproduce the Fig. 4 comparison.

## Related tools

- **VirMAP (Virmap.pl)** (Perl driver script executing the full pipeline: dereplication, digital normalization, BBMap/DIAMOND alignments, MEGAHIT/Tadpole assembly, and super-scaffold construction) — https://github.com/cmmr/VirMAP
- **DIAMOND** (Translated (protein) alignment of reads against the viral protein database during read assignment) — https://github.com/bbuchfink/diamond
- **MEGAHIT** (Multi-kmer de-novo assembler invoked via --useMegahit to create the working contig set) — https://github.com/voutcn/megahit
- **BBMap/Tadpole** (Nucleotide alignment to the viral database and digital normalization (--useBbnorm); alternative de-novo assembler (Tadpole))
- **khmer** (Provides normalize-by-median.py used for digital normalization of input reads) — https://github.com/dib-lab/khmer

## Examples

```
fasterq-dump -t /dev/shm -e 4 -O /home/$USER/VirmapTest SRR9875293; Virmap.pl --threads $(nproc) --useMegahit --useBbnorm --readF /home/$USER/VirmapTest/SRR9875293_1.fastq --sampleName SRR9875293
```

## Evaluation signals

- Across the 20 influenza-positive UMERS samples, reconstructed segments average 9.05 influenza segments per sample
- Total recruited influenza reads across the cohort equal 509,039 (193% more than the standard approach)
- Every reconstructed segment consists of exactly one contig, confirming contiguous genome reconstruction
- The per-sample table influenza_virmap_results.csv is populated for all 20 positive samples with read and segment counts
- Each Virmap.pl instance completes without memory exhaustion under the ≥64 GB RAM provisioning rule

## Limitations

- Provisioning at least 64 GB RAM per simultaneous instance is required, and Genbank database growth expands minimum SSD and RAM requirements
- No software versions are stated for VirMAP, BBMap, DIAMOND, MEGAHIT, or Tadpole, and the Genbank database snapshot/download date is unspecified, complicating exact reproduction
- Dynamically calculated LCA radius and majority threshold formulas are not fully specified in the article text, and Supplementary Data 14 (sensitivity/specificity tuning parameters) is not provided

## Evidence

- [methods] Virmap.pl invocation and flags: "`Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq`"
- [methods] Pipeline assembler options: "--useMegahit --useBbnorm --sampleName SRR9875293"
- [methods] SRA download step: "`fasterq-dump -t /dev/shm -e 4 -O /home/$USER/VirmapTest SRR9875293`"
- [discussion] Software availability and implementation: "The VirMAP software is open-source and available online at https://github.com/cmmr/VirMAP. VirMAP is implemented in Perl."
- [discussion] Assembly component choice: "a multi-kmer de-novo assembly using either MEGAHIT21,22 or Tadpole19, or both"
