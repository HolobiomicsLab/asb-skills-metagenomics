---
name: paired-end-metagenomic-read-preprocessing
description: Use when reconstructing full-length SSU rRNA genes (16S/18S) from paired-end
  Illumina metagenomic FASTQ to enable species-level taxonomic profiling via RiboTaxa's
  dual-assembler strategy.
when_to_use_negative:
- When the goal is rapid taxonomic profiling of short reads without reconstruction
  — the article notes RiboTaxa 'is not as fast as short-read classifiers (i.e. Kraken2,
  Centrifuge)'
- When the input is already a set of full-length rRNA sequences or an ASV/OTU feature
  table rather than raw metagenomic reads
- When copy-number-corrected abundances are required — RiboTaxa explicitly does not
  apply 16S rRNA gene copy number normalization
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: SortMeRNA
  role: rRNA-specialized tool used to extract/filter 16S/18S SSU reads from the trimmed
    metagenomic reads before EMIRGE reconstruction
- name: EMIRGE (Emirge_amplicon.py)
  role: Reference-targeted assembler reconstructing SSU rRNA sequences from SortMeRNA-filtered
    reads against the Bowtie-indexed SILVA database; runs usearch internally
- name: MetaRib
  role: Reference-targeted assembler reconstructing SSU rRNA sequences from the unfiltered
    high-quality reads (minid=0.96, idfilter=0.98)
- name: BBTools
  role: Upstream quality control producing the adapter-trimmed high-quality reads
    that are split between the two assemblers
- name: USEARCH
  role: Internal dependency used by EMIRGE during full-length rRNA reconstruction;
    must be present in $PATH
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:2eb6d33fd780442dc517e355b69b129dab0094cfe007841f3e3255b0b1850ecf
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/oschakoory/RiboTaxa
  tools:
  - BBTools
  - EMIRGE (Emirge_amplicon.py)
  - MetaRib
  - SortMeRNA
  - USEARCH
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# paired-end-metagenomic-read-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

RiboTaxa's SSU rRNA gene reconstruction layer, which reconstructs full-length 16S/18S rRNA gene sequences from Illumina shotgun (meta)genomic FASTQ by running two reference-targeted assemblers in parallel — EMIRGE (Emirge_amplicon.py) on SortMeRNA-extracted SSU reads and MetaRib on the unfiltered high-quality reads — and merging both reconstructions into metagenome_SSU_sequences.fasta.

## When to use

Use when reconstructing full-length SSU rRNA genes (16S/18S) from paired-end Illumina metagenomic FASTQ to enable species-level taxonomic profiling via RiboTaxa's dual-assembler strategy.

## When NOT to use

- When the goal is rapid taxonomic profiling of short reads without reconstruction — the article notes RiboTaxa 'is not as fast as short-read classifiers (i.e. Kraken2, Centrifuge)'
- When the input is already a set of full-length rRNA sequences or an ASV/OTU feature table rather than raw metagenomic reads
- When copy-number-corrected abundances are required — RiboTaxa explicitly does not apply 16S rRNA gene copy number normalization

## Inputs

- Paired-end (or single-end) Illumina (meta)genomic reads as FASTQ or fastq.gz
- Adapter-trimmed high-quality reads from BBTOOLS QC (ktrim=r, kmer=21, minlength=60, trimq=20, qtrim=rl, maxns=1)
- Pre-indexed SILVA SSU 138.1 database for SortMeRNA (sortmerna_indexed_DB)
- Bowtie-indexed SILVA SSU database for EMIRGE

## Outputs

- *_16S18Sreads.fastq (SortMeRNA-filtered 16S/18S read files)
- SortMeRNA .log reporting the percentage of SSU reads filtered
- EMIRGE-reconstructed SSU rRNA sequences (from Emirge_amplicon.py, 120 iterations, IDENTITY=1)
- MetaRib-reconstructed SSU rRNA sequences (minid=0.96, idfilter=0.98)
- metagenome_SSU_sequences.fasta in the SSU_sequences output directory (combined EMIRGE + MetaRib full-length/nearly full-length 16S/18S sequences)

## How to apply

Start from adapter-trimmed high-quality reads produced by BBTOOLS quality control (ktrim=r, kmer=21, minlength=60, trimq=20, qtrim=rl, maxns=1). Split the read inputs between the two assemblers: first extract 16S/18S rRNA reads with SortMeRNA v2.1 against the pre-indexed SILVA SSU 138.1 database (sortmerna_indexed_DB), yielding *_16S18Sreads.fastq plus a .log with the percentage of SSU reads filtered, and feed those filtered reads to Emirge_amplicon.py (EMIRGE) against the Bowtie-indexed SILVA database with JOIN_TRESHOLD/IDENTITY = 1 for candidate-sequence merging over 120 iterations, running usearch internally. Concurrently feed the unfiltered high-quality reads to MetaRib with minid = 0.96 and idfilter = 0.98. The rationale, grounded in the article's benchmarking, is that SortMeRNA extracted reads that assemble better with EMIRGE-type tools, while running MetaRib on the unfiltered reads and combining the two reconstructions 'led a greater population characterisation'. Finally concatenate the EMIRGE and MetaRib outputs into metagenome_SSU_sequences.fasta in the SSU_sequences output directory.

## Related tools

- **SortMeRNA** (rRNA-specialized tool used to extract/filter 16S/18S SSU reads from the trimmed metagenomic reads before EMIRGE reconstruction)
- **EMIRGE (Emirge_amplicon.py)** (Reference-targeted assembler reconstructing SSU rRNA sequences from SortMeRNA-filtered reads against the Bowtie-indexed SILVA database; runs usearch internally)
- **MetaRib** (Reference-targeted assembler reconstructing SSU rRNA sequences from the unfiltered high-quality reads (minid=0.96, idfilter=0.98))
- **BBTools** (Upstream quality control producing the adapter-trimmed high-quality reads that are split between the two assemblers)
- **USEARCH** (Internal dependency used by EMIRGE during full-length rRNA reconstruction; must be present in $PATH)

## Examples

```
conda activate RiboTaxa_py27 && emirge_amplicon.py --help
```

## Evaluation signals

- metagenome_SSU_sequences.fasta exists in the SSU_sequences output directory and is valid FASTA
- The file contains full-length/nearly full-length 16S/18S sequences contributed by both EMIRGE and MetaRib reconstructions
- The SortMeRNA .log reports the percentage of SSU reads filtered from the input metagenomic FASTQ
- EMIRGE ran with JOIN_TRESHOLD/IDENTITY = 1 and 120 iterations; MetaRib ran with minid = 0.96 and idfilter = 0.98
- usearch resolves in $PATH (usearch --version succeeds) since EMIRGE calls it internally

## Limitations

- Chimera detection was not included in the 16S rRNA gene reconstruction workflow, and chimeric database sequences could carry over into SSU reconstructions if reads map across the full length of the chimera
- A portion of the reported diversity may originate from artificial diversity created in silico during full-length sequence reconstruction, and occasional small indel errors can occur in reconstructed sequences
- RiboTaxa is slower than short-read classifiers (Kraken2, Centrifuge), and some sequences cannot be assigned at species level (e.g. Enterobacteriaceae, Lachnospiraceae) due to high 16S rRNA conservation, stopping at genus level

## Evidence

- [readme] README tool roles: [evidence span withheld — anchor status: not_found]
