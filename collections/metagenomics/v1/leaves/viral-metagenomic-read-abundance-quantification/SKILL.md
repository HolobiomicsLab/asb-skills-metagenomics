---
name: viral-metagenomic-read-abundance-quantification
description: Use when you have FASTQ reads and VirMAP-reconstructed viral contigs/super-scaffolds
  and need per-genome read abundance, read sets, and taxonomic classifications from
  metagenomic virome data.
when_to_use_negative:
- You only have a precomputed feature/abundance table and no raw reads to align to
  reconstructed viral genomes
- The goal is de-novo viral discovery or assembly itself rather than quantifying abundance
  of already-reconstructed viral genomes
- Sequences come from an isolate genome with no mixed-community background, so read
  sets will not require dereplication, normalization, or Jaccard-distance filtering
edam_topics:
- http://edamontology.org/topic_3621
tools:
- name: VirMAP
  role: Perl pipeline orchestrating read alignment, per-genome read assignment, LCA-based
    taxonomy, and final abundance computation (run via virmap.pl)
  repo: https://github.com/cmmr/VirMAP
- name: BBMap
  role: Nucleotide read mapping used to assign reads to viral genome sequences and
    score contig alignments in bits-per-base space
- name: DIAMOND
  role: Translated DNA alignment used to assign reads to viral protein pseudo-scaffolds
    alongside BBMap nucleotide mapping
  repo: https://github.com/bbuchfink/diamond
- name: khmer
  role: Provides normalize-by-median.py for read dereplication and digital normalization
    prior to read assignment
  repo: https://github.com/dib-lab/khmer
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:f7c161eb61ba11479d63fc08040d2f9c3ea3a9bfb63d94dc7939dd2f76ada3d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cmmr/VirMAP
  tools:
  - BBMap
  - DIAMOND
  - MEGAHIT
  - Tadpole
  - VirMAP
  - khmer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Viral metagenomic read abundance quantification

## Summary

Assign metagenomic reads to reconstructed viral genomes via nucleotide (BBMap) and translated (DIAMOND) alignments against VirMAP's custom viral databases, then compute per-genome read abundance from the converged read assignments of the final dual assembly.

## When to use

Use when you have FASTQ reads and VirMAP-reconstructed viral contigs/super-scaffolds and need per-genome read abundance, read sets, and taxonomic classifications from metagenomic virome data.

## When NOT to use

- You only have a precomputed feature/abundance table and no raw reads to align to reconstructed viral genomes
- The goal is de-novo viral discovery or assembly itself rather than quantifying abundance of already-reconstructed viral genomes
- Sequences come from an isolate genome with no mixed-community background, so read sets will not require dereplication, normalization, or Jaccard-distance filtering

## Inputs

- Paired-end FASTQ reads (e.g. SRR9875293_1.fastq / _2.fastq from the Sequence Read Archive)
- Converged viral contigs/super-scaffolds and protein pseudo-scaffolds from the VirMAP mapping-assembly and improvement stages
- VirMAP custom viral nucleotide and protein databases built from the Genbank viral (gbvrl) and phage (gbphg) divisions

## Outputs

- Per-genome read abundance estimates for the final viral genomes
- Per-genome read assignment sets (after Jaccard-distance filtering)
- Taxonomic classifications for the reconstructed viral genomes
- VirmapRun output directory contents with run diagnostics captured in VirmapRun.err

## How to apply

First dereplicate and digitally normalize the input reads with normalize-by-median.py (khmer) or bbnorm, then align processed reads to VirMAP's comprehensive viral nucleotide and viral protein databases using BBMap for nucleotide mapping and DIAMOND for translated alignment. Assign reads to viral genomes from both nucleotide and translated alignments, producing a set of reads per genome; filter genomic read sets by Jaccard distance when high numbers of non-unique read membership are observed relative to larger sets. After the iterative merge-and-rebuild improvement cycle converges, compute read abundance for the final viral genomes from the converged read assignments. VirMAP uses a bits-per-base scoring system with a rank-wise lowest common ancestor (LCA) algorithm over pass-one taxa within a dynamically calculated radius per contig to place genomes taxonomically.

## Related tools

- **VirMAP** (Perl pipeline orchestrating read alignment, per-genome read assignment, LCA-based taxonomy, and final abundance computation (run via virmap.pl)) — https://github.com/cmmr/VirMAP
- **BBMap** (Nucleotide read mapping used to assign reads to viral genome sequences and score contig alignments in bits-per-base space)
- **DIAMOND** (Translated DNA alignment used to assign reads to viral protein pseudo-scaffolds alongside BBMap nucleotide mapping) — https://github.com/bbuchfink/diamond
- **khmer** (Provides normalize-by-median.py for read dereplication and digital normalization prior to read assignment) — https://github.com/dib-lab/khmer

## Examples

```
Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq --useMegahit --useBbnorm --sampleName SRR9875293
```

## Evaluation signals

- The run completes and populates the VirmapRun output directory with taxonomic classifications and abundance results, with run diagnostics captured in VirmapRun.err
- Every final viral genome has a converged read-assignment set from both nucleotide (BBMap) and translated (DIAMOND) alignments
- Genomic read sets with high non-unique read membership relative to larger sets have been filtered by Jaccard distance before abundance computation
- Input reads were dereplicated and digitally normalized (normalize-by-median.py or bbnorm) before alignment, so abundance is not inflated by duplicates

## Limitations

- Taxonomic classification accuracy depends on Genbank viral/phage database coverage (gbvrl/gbphg); divergent viruses absent from the database cannot be resolved
- Reads mapping equally well to different genomes create ambiguous membership that must be handled by Jaccard-distance filtering of genomic read sets
- Supplementary Data 14 offers tunable sensitivity/specificity parameters, and supplementary notes (e.g. Supplementary Note 4 on the 16-division Genbank database) are not included in the main text
- No software versions are stated for VirMAP, BBMap, DIAMOND, MEGAHIT, or Tadpole, so reproducibility requires pinning tool versions independently

## Evidence

- [discussion] Reads are assigned to viral genomes by nucleotide and translated alignments resulting in a set of reads per genome.: "Reads are assigned to viral genomes by nucleotide and translated alignments resulting in a set of reads per genome."
- [discussion] Genomic read sets filtered by Jaccard distance: "Genomic read sets are then filtered by Jaccard distance if high numbers of non-unique read membership are observed relative to larger sets."
- [discussion] Read dereplication and digital normalization: "input reads are dereplicated and digitally normalized18 using default parameters"
- [intro] Bits-per-base taxonomic scoring: "a taxonomic classification algorithm that is centered around a bits-per-base scoring system"
- [methods] Run completion populates VirmapRun output: "Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq"
