---
name: mapping-assembly-super-scaffold-construction
description: Use when reconstructing full-length viral genome segments (e.g. influenza's
  8 segments) from low-coverage metagenomic FASTQ data where de-novo assembly alone
  yields fragmented contigs.
when_to_use_negative:
- Input is already assembled contigs or a feature/taxonomy table rather than raw FASTQ
  metagenomic reads
- 'The target organism is non-viral: contigs mapping outside the viral superkingdom
  are filtered by design'
- Available memory is below the ~64 GB RAM required per simultaneous Virmap.pl instance
  when processing large datasets (e.g. 1.75 billion reads)
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3174
tools:
- name: VirMAP
  role: Implements the mapping assembly and super-scaffold construction pipeline in
    Perl (Virmap.pl)
  repo: https://github.com/cmmr/VirMAP
- name: MEGAHIT
  role: Multi-kmer de-novo assembly generating contigs that are merged with the mapping
    assembly (--useMegahit)
  repo: https://github.com/voutcn/megahit
- name: BBMap
  role: Mapping assembly of reads to viral nucleotide references; --useBbnorm performs
    digital normalization
- name: DIAMOND
  role: Translated alignment of reads to the viral protein database (gbBlastx.dmnd)
    for protein-level read assignment
  repo: https://github.com/bbuchfink/diamond
- name: Tadpole
  role: Alternative de-novo assembler that VirMAP can use instead of, or alongside,
    MEGAHIT
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:0d61d1692177f7fb8c079d1d0652b9437da981f9fafaae22f165478c14e2b9ee
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
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# mapping-assembly-super-scaffold-construction

## Summary

Constructs virus-like super-scaffolds by hybridizing a mapping assembly (read recruitment against Genbank viral/phage references) with a de-novo assembly, using VirMAP's tiered nucleotide and amino-acid alignment merging to reconstruct contiguous viral genomes from metagenomic reads.

## When to use

Use when reconstructing full-length viral genome segments (e.g. influenza's 8 segments) from low-coverage metagenomic FASTQ data where de-novo assembly alone yields fragmented contigs.

## When NOT to use

- Input is already assembled contigs or a feature/taxonomy table rather than raw FASTQ metagenomic reads
- The target organism is non-viral: contigs mapping outside the viral superkingdom are filtered by design
- Available memory is below the ~64 GB RAM required per simultaneous Virmap.pl instance when processing large datasets (e.g. 1.75 billion reads)

## Inputs

- Paired-end FASTQ metagenomic reads (e.g. SRA downloads from BioProject PRJEB7888)
- Prebuilt VirMAP database files: Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd

## Outputs

- Virus-like super-scaffolds (one contig per reconstructed genome segment)
- Per-genome read assignment counts from nucleotide and translated alignments
- Per-sample table of reconstructed segments (e.g. influenza_virmap_results.csv)

## How to apply

Run Virmap.pl on each paired-end sample against the prebuilt VirMAP database (Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd) with --useMegahit and --useBbnorm enabled and --threads set to all available cores; provision at least 64 GB RAM per simultaneous instance. The algorithm first dereplicates and digitally normalizes input reads, then employs both de-novo (MEGAHIT) and mapping assembly strategies to create a working set of contigs, aligning reads to comprehensive viral nucleotide and protein databases (Genbank gbvrl and gbphg divisions). Reads are assigned to viral genomes by nucleotide and translated alignments, filtered by Jaccard distance when non-unique read membership is high, and each non-filtered genome is individually reconstructed via naive pileup assembly with reconstructed protein coding sequences tiled onto the originating genome. An improvement algorithm iteratively merges and rebuilds contig information, and contigs whose top pass-one taxon is not viral or with >50% of aligning bases unresolved to the viral superkingdom are filtered, producing one contig per genome segment in the super-scaffold output.

## Related tools

- **VirMAP** (Implements the mapping assembly and super-scaffold construction pipeline in Perl (Virmap.pl)) — https://github.com/cmmr/VirMAP
- **MEGAHIT** (Multi-kmer de-novo assembly generating contigs that are merged with the mapping assembly (--useMegahit)) — https://github.com/voutcn/megahit
- **BBMap** (Mapping assembly of reads to viral nucleotide references; --useBbnorm performs digital normalization)
- **DIAMOND** (Translated alignment of reads to the viral protein database (gbBlastx.dmnd) for protein-level read assignment) — https://github.com/bbuchfink/diamond
- **Tadpole** (Alternative de-novo assembler that VirMAP can use instead of, or alongside, MEGAHIT)

## Examples

```
Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq --useMegahit --useBbnorm --sampleName SRR9875293
```

## Evaluation signals

- Each reconstructed segment in the super-scaffold output consists of exactly one contig (contiguous reconstruction)
- Average number of reconstructed influenza segments per positive sample matches the reported 9.05 across the 20 influenza-positive UMERS samples
- Total recruited influenza reads matches the reported 509,039 (193% more than the standard approach)
- Filtered contigs' top pass-one taxon resolves to the superkingdom virus with >50% of aligning bases resolving to virus
- Per-sample segment counts written to influenza_virmap_results.csv are consistent across nucleotide and translated alignment tallies

## Limitations

- Requires at least 64 GB SSD/RAM per simultaneous instance, and minimum requirements grow as Genbank expands
- No software versions stated for VirMAP, BBMap, DIAMOND, MEGAHIT, or Tadpole, complicating exact reproduction
- Dynamically calculated LCA radius and majority threshold formulas are not fully specified in the article text

## Evidence

- [intro] mapping assembly algorithm combining nucleotide and amino-acid alignment information: "a mapping assembly algorithm combining nucleotide and amino-acid alignment information in a tiered manner to build virus-like super-scaffolds"
- [intro] Genbank viral and phage divisions alignments: "A putative list of viral genomes and protein pseudo-scaffolds are constructed from clustered nucleotide and translated alignments to the Genbank viral and phage divisions (gbvrl and gbphage)."
- [discussion] Jaccard distance filtering of genomic read sets: "Genomic read sets are then filtered by Jaccard distance if high numbers of non-unique read membership are observed relative to larger sets."
- [discussion] Viral contig filtering rule: "A contig is filtered if its top scoring pass one taxon is not in the superkingdom virus, or if > 50% of its aligning bases do not resolve to the superkingdom virus."
- [methods] Virmap.pl methods invocation: "`Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq`"
- [discussion] MEGAHIT/Tadpole assembly options: "a multi-kmer de-novo assembly using either MEGAHIT21,22 or Tadpole19, or both"
