---
name: influenza-segment-reconstruction-from-metagenomic-reads
description: Use when paired-end metagenomic FASTQ from influenza-positive respiratory
  samples must be assembled into contiguous genome segments and per-sample segment
  counts compared against the VirMAP reference results.
when_to_use_negative:
- The input is not raw paired-end metagenomic reads (e.g., already-assembled contigs
  or a taxonomic abundance table) — VirMAP's hybrid assembly pipeline expects FASTQ
  input.
- The host organism/pathogen is not viral or lacks reference representation in the
  Genbank viral (gbvrl) and phage (gbphg) divisions used to build the database.
- Available RAM is below 64 GB per instance, since the prebuilt Genbank-derived VirMAP
  database cannot be loaded and minimum requirements expand as Genbank grows.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: VirMAP
  role: 'Core pipeline: mapping assembly, improvement, and taxonomic classification
    via bits-per-base scoring to reconstruct influenza segments from metagenomic reads'
  repo: https://github.com/cmmr/VirMAP
- name: MEGAHIT
  role: Multi-kmer de-novo assembly engine enabled with --useMegahit to generate contigs
    merged with the mapping assembly
  repo: https://github.com/voutcn/megahit
- name: BBMap
  role: Read dereplication, digital normalization (--useBbnorm), and nucleotide alignment
    support within Virmap.pl
- name: DIAMOND
  role: Translated (protein) alignment of reads/contigs to the viral protein database
    (virDmnd.dmnd / gbBlastx.dmnd) for read assignment
  repo: https://github.com/bbuchfink/diamond
- name: Tadpole
  role: Alternative de-novo assembler that VirMAP can invoke instead of or alongside
    MEGAHIT
- name: Perl
  role: Implementation language in which Virmap.pl is run from the command line
  repo: https://github.com/cmmr/VirMAP
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:01f0ee72ff179d656bd58ed2642144e7e3d378200149101d9763f5368a1114c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cmmr/VirMAP
  tools:
  - BBMap
  - DIAMOND
  - MEGAHIT
  - Perl
  - Tadpole
  - VirMAP
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# influenza-segment-reconstruction-from-metagenomic-reads

## Summary

Reconstruct influenza genome segments from clinical respiratory metagenomes using VirMAP's hybrid mapping/de-novo assembly, reproducing the reported 9.05 average segments per influenza-positive sample (BioProject PRJEB7888) with one contig per reconstructed segment.

## When to use

Use when paired-end metagenomic FASTQ from influenza-positive respiratory samples must be assembled into contiguous genome segments and per-sample segment counts compared against the VirMAP reference results.

## When NOT to use

- The input is not raw paired-end metagenomic reads (e.g., already-assembled contigs or a taxonomic abundance table) — VirMAP's hybrid assembly pipeline expects FASTQ input.
- The host organism/pathogen is not viral or lacks reference representation in the Genbank viral (gbvrl) and phage (gbphg) divisions used to build the database.
- Available RAM is below 64 GB per instance, since the prebuilt Genbank-derived VirMAP database cannot be loaded and minimum requirements expand as Genbank grows.

## Inputs

- 25 paired-end influenza metagenome FASTQ sample pairs from BioProject PRJEB7888 (Sequence Read Archive)
- Prebuilt VirMAP database files: Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd
- Compute resources: >= 64 GB RAM per simultaneous Virmap.pl instance, multi-core CPU

## Outputs

- Per-sample table of influenza reads assigned by nucleotide and translated alignment and reconstructed segment counts (influenza_virmap_results.csv)
- VirMAP super-scaffold output with exactly one contig per reconstructed influenza segment per positive sample
- Cohort-level reproduction metrics: 9.05 average influenza segments per positive sample and 509,039 total recruited influenza reads
- Subsampled-depth segment-length comparison of VirMAP vs standard approach (Fig. 4 reproduction)

## How to apply

Download the 25 paired-end influenza metagenome samples (BioProject PRJEB7888) from SRA with fasterq-dump, then run Virmap.pl per sample against the prebuilt database (Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd) with --useMegahit --useBbnorm and --threads set to all available cores, provisioning at least 64 GB RAM per simultaneous instance. VirMAP dereplicates and digitally normalizes reads (BBMap), performs de-novo and mapping assembly (MEGAHIT), assigns reads to viral genomes by nucleotide and translated alignments, filters genomic read sets by Jaccard distance, and reconstructs each genome via naive pileup assembly. For each of the 20 influenza-positive samples, tally influenza reads assigned by nucleotide and translated alignment and count reconstructed segments (one contig per segment) from the super-scaffold output. Compute cohort metrics (9.05 average influenza segments per positive sample; 509,039 total recruited influenza reads, 193% more than the standard approach) and write the per-sample table to influenza_virmap_results.csv. Optionally subsample the 1.75 billion reads to lower depths and rerun to reproduce the Fig. 4 VirMAP-vs-standard segment-length comparison.

## Related tools

- **VirMAP** (Core pipeline: mapping assembly, improvement, and taxonomic classification via bits-per-base scoring to reconstruct influenza segments from metagenomic reads) — https://github.com/cmmr/VirMAP
- **MEGAHIT** (Multi-kmer de-novo assembly engine enabled with --useMegahit to generate contigs merged with the mapping assembly) — https://github.com/voutcn/megahit
- **BBMap** (Read dereplication, digital normalization (--useBbnorm), and nucleotide alignment support within Virmap.pl)
- **DIAMOND** (Translated (protein) alignment of reads/contigs to the viral protein database (virDmnd.dmnd / gbBlastx.dmnd) for read assignment) — https://github.com/bbuchfink/diamond
- **Tadpole** (Alternative de-novo assembler that VirMAP can invoke instead of or alongside MEGAHIT)
- **Perl** (Implementation language in which Virmap.pl is run from the command line) — https://github.com/cmmr/VirMAP

## Examples

```
Virmap.pl --threads $(nproc) --readF SRR9875293_1.fastq --useMegahit --useBbnorm --sampleName SRR9875293
```

## Evaluation signals

- Average reconstructed influenza segments across the 20 influenza-positive samples equals 9.05.
- Total recruited influenza reads across the cohort equals 509,039 (193% more than the standard approach).
- Every reconstructed influenza segment consists of exactly one contig in the super-scaffold output (contiguous reconstruction).
- Per-sample table influenza_virmap_results.csv contains counts for all 20 positive samples and tallies both nucleotide and translated-alignment read assignments.
- Subsampling the 1.75 billion reads to lower depths and rerunning reproduces the VirMAP-vs-standard segment-length comparison of Fig. 4.

## Limitations

- Software versions for VirMAP, BBMap, DIAMOND, MEGAHIT, and Tadpole are not stated, and no Genbank database snapshot or download date is specified, complicating exact reproduction.
- Minimum SSD and RAM requirements expand as Genbank grows; at least 64 GB RAM per simultaneous instance is required.
- Supplementary Data 14 (sensitivity/specificity tuning parameters) and Supplementary Note 4 (16-division custom Genbank database) are referenced but not provided in the main text.
- Dynamically calculated LCA radius and majority-threshold formulas are not fully specified in the main text.

## Evidence

- [abstract] VirMAP merges nucleotide and protein information for taxonomic classification: "merge nucleotide and protein information to taxonomically classify viral reconstructions independent of genome coverage or read overlap"
- [discussion] Hybrid de-novo and mapping assembly: "VirMAP employs both de-novo and mapping assembly strategies to create a working set of contigs."
- [discussion] Jaccard-distance filtering of genomic read sets: "Genomic read sets are then filtered by Jaccard distance"
- [methods] Virmap.pl command-line invocation with normalization and MEGAHIT options: "--useMegahit --useBbnorm --sampleName SRR9875293"
- [intro] Genbank viral and phage database source: "clustered nucleotide and translated alignments to the Genbank viral and phage divisions (gbvrl and gbphage)"
