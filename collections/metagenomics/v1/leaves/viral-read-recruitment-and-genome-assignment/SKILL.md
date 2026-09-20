---
name: viral-read-recruitment-and-genome-assignment
description: Use when paired-end viral metagenomic FASTQ data (e.g., SRA clinical
  respiratory samples) must have reads assigned to specific viral genomes and genomes
  reconstructed as super-scaffolds, especially with low coverage or divergent viruses.
when_to_use_negative:
- Input is already a viral taxonomic abundance table or contig catalog rather than
  raw metagenomic reads — read recruitment and assembly are unnecessary and the 64
  GB RAM pipeline cost is not justified.
- The goal is only rough abundance profiling of known viruses per read, without genome
  reconstruction; individual-read classifiers are far cheaper than VirMAP's assembly/improvement
  pipeline.
- Samples contain almost no viral signal and the study target is host or bacterial
  genomics — contamination filtering thresholds (>50% aligning bases resolving to
  virus) will filter nearly everything.
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3307
tools:
- name: VirMAP
  role: 'Primary pipeline: assigns reads to viral genomes via tiered nucleotide (BBMap)
    and translated (DIAMOND) alignments, filters by Jaccard distance, and builds super-scaffolds;
    implemented in Perl (Virmap.pl).'
  repo: https://github.com/cmmr/VirMAP
- name: BBMap
  role: Nucleotide alignment engine used by VirMAP to recruit reads to the viral nucleotide
    database.
- name: DIAMOND
  role: Translated/protein alignment engine (gbBlastx.dmnd, virDmnd.dmnd) used by
    VirMAP to assign reads via protein information.
  repo: https://github.com/bbuchfink/diamond
- name: MEGAHIT
  role: Multi-kmer de-novo assembler invoked via --useMegahit to provide contigs merged
    with the mapping assembly.
  repo: https://github.com/voutcn/megahit
- name: Tadpole
  role: Alternative de-novo assembler within VirMAP's hybrid assembly strategy.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:d1dd470aad18261de98fb6b6908736671b69178fa6414cb397ce2bc5389f19c5
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
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# viral-read-recruitment-and-genome-assignment

## Summary

Assign metagenomic reads to viral genomes by tiered nucleotide and translated (protein) alignments to the Genbank viral/phage divisions (gbvrl/gbphg), then filter and pileup-assemble per-genome read sets into virus-like super-scaffolds using VirMAP. The skill reproduces VirMAP's influenza reconstruction on the UMERS respiratory dataset (PRJEB7888), where it recruited 509,039 influenza reads and averaged 9.05 contiguous segments per positive sample.

## When to use

Use when paired-end viral metagenomic FASTQ data (e.g., SRA clinical respiratory samples) must have reads assigned to specific viral genomes and genomes reconstructed as super-scaffolds, especially with low coverage or divergent viruses.

## When NOT to use

- Input is already a viral taxonomic abundance table or contig catalog rather than raw metagenomic reads — read recruitment and assembly are unnecessary and the 64 GB RAM pipeline cost is not justified.
- The goal is only rough abundance profiling of known viruses per read, without genome reconstruction; individual-read classifiers are far cheaper than VirMAP's assembly/improvement pipeline.
- Samples contain almost no viral signal and the study target is host or bacterial genomics — contamination filtering thresholds (>50% aligning bases resolving to virus) will filter nearly everything.

## Inputs

- Paired-end viral metagenomic FASTQ files (e.g., SRA runs from BioProject PRJEB7888, downloaded via fasterq-dump)
- Prebuilt VirMAP database files (Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd) from Genbank viral (gbvrl) and phage (gbphg) divisions

## Outputs

- Per-sample table of influenza-assigned reads and reconstructed genome segments (influenza_virmap_results.csv)
- Virus-like super-scaffold reconstructions (one contig per influenza genome segment)
- Taxonomic assignments with bits-per-base scores for viral reads/contigs

## How to apply

Download paired-end samples from the SRA with fasterq-dump, then run Virmap.pl against the prebuilt VirMAP database (Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd) with --useMegahit --useBbnorm and --threads set to all cores, provisioning at least 64 GB RAM per simultaneous instance. VirMAP dereplicates and digitally normalizes reads, aligns them to a comprehensive viral nucleotide and viral protein database using BBMap and DIAMOND, and assigns reads to genomes by both nucleotide and translated alignments. Genomic read sets are filtered by Jaccard distance when non-unique read membership is high, and contigs are dropped if the top pass-one taxon is not in the superkingdom virus or >50% of aligning bases do not resolve to virus. Each non-filtered genome's reads undergo naive pileup assembly; reconstructed CDS are tiled back onto the originating genome, merged with the de-novo (MEGAHIT/Tadpole) assembly, and improved iteratively to form super-scaffolds. For validation on PRJEB7888, tally influenza-assigned reads and count reconstructed segments (one contig per segment) from the super-scaffold output for the 20 influenza-positive samples.

## Related tools

- **VirMAP** (Primary pipeline: assigns reads to viral genomes via tiered nucleotide (BBMap) and translated (DIAMOND) alignments, filters by Jaccard distance, and builds super-scaffolds; implemented in Perl (Virmap.pl).) — https://github.com/cmmr/VirMAP
- **BBMap** (Nucleotide alignment engine used by VirMAP to recruit reads to the viral nucleotide database.)
- **DIAMOND** (Translated/protein alignment engine (gbBlastx.dmnd, virDmnd.dmnd) used by VirMAP to assign reads via protein information.) — https://github.com/bbuchfink/diamond
- **MEGAHIT** (Multi-kmer de-novo assembler invoked via --useMegahit to provide contigs merged with the mapping assembly.) — https://github.com/voutcn/megahit
- **Tadpole** (Alternative de-novo assembler within VirMAP's hybrid assembly strategy.)

## Examples

```
Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq --useMegahit --useBbnorm --sampleName SRR9875293
```

## Evaluation signals

- Across the 20 influenza-positive UMERS (PRJEB7888) samples, the average number of reconstructed influenza segments per sample matches the reported 9.05.
- Total recruited influenza reads across the cohort equals the reported 509,039 (193% more than the standard approach).
- Every reconstructed influenza segment consists of exactly one contig, confirming contiguous super-scaffold reconstruction.
- No output contig has a top pass-one taxon outside the superkingdom virus, and no retained contig has >50% of aligning bases unresolved to virus.
- Genomic read sets with high non-unique read membership relative to larger sets are filtered by Jaccard distance before pileup assembly.

## Limitations

- The VirMAP database build scales with Genbank: 'As Genbank expands, the minimum SSD and RAM requirements will expand as well'; at least 64 GB RAM is needed per simultaneous Virmap.pl instance.
- Supplementary tuning material (Supplementary Data 14 sensitivity/specificity parameters and Supplementary Note 4 describing the 16-division custom Genbank database) is not included in the article, limiting exact parameter reproduction.
- No software versions for VirMAP, BBMap, DIAMOND, MEGAHIT, or Tadpole are stated, and no Genbank database snapshot/download date is specified, so exact reimplementation may differ.

## Evidence

- [discussion] Reads are assigned to viral genomes by nucleotide and translated alignments: "Reads are assigned to viral genomes by nucleotide and translated alignments resulting in a set of reads per genome."
- [discussion] Jaccard distance filtering of genomic read sets: "Genomic read sets are then filtered by Jaccard distance"
- [discussion] Contig virus filter threshold: "A contig is filtered if its top scoring pass one taxon is not in the superkingdom virus, or if > 50% of its aligning bases do not resolve to"
- [methods] Virmap.pl command from methods: "`Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq`"
- [intro] Super-scaffold construction rationale: "a mapping assembly algorithm combining nucleotide and amino-acid alignment information in a tiered manner to build virus-like super-scaffolds"
