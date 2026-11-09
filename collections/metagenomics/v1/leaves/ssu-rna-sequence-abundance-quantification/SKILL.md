---
name: ssu-rna-sequence-abundance-quantification
description: Use when you have reconstructed SSU rRNA sequences (e.g. metagenome_SSU_sequences.fasta)
  and quality-controlled metagenomic reads, and need per-sequence relative abundance
  estimates before clustering or classification.
when_to_use_negative:
- Input is a feature/abundance table already derived from amplicon ASVs/OTUs rather
  than reconstructed SSU sequences needing read-mapping quantification
- You want to estimate relative abundance of raw short reads directly with a classifier
  (Kraken2, Centrifuge) instead of validating reconstructed SSU sequences
- The reference SSU sequences have not yet been reconstructed (BBMap mapping requires
  the metagenome_SSU_sequences.fasta reference to exist)
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: BBMap
  role: Maps unfiltered high-quality reads onto the reconstructed SSU rRNA sequences
    to tally per-sequence assigned read counts; part of BBTOOLS, run with RAM limited
    to ~80% of available memory
  repo: https://github.com/oschakoory/RiboTaxa
- name: BBTools
  role: Tool suite providing BBMap for abundance estimation and upstream quality-controlled
    trimmed reads used as mapping input
  repo: https://github.com/oschakoory/RiboTaxa
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:0dbefa8c0e601db3c9b021911f20e3859cb1072eef36f95ac5d62c7abb2386ef
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/oschakoory/RiboTaxa
  repo_url: https://github.com/oschakoory/RiboTaxa
  tools:
  - BBMap
  - BBTools
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# ssu-rna-sequence-abundance-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the relative abundance of reconstructed full-length SSU rRNA sequences in RiboTaxa by mapping unfiltered high-quality reads back onto them with BBMap and normalizing per-sequence read counts to percentages, prior to 97% clustering and taxonomic affiliation.

## When to use

Use when you have reconstructed SSU rRNA sequences (e.g. metagenome_SSU_sequences.fasta) and quality-controlled metagenomic reads, and need per-sequence relative abundance estimates before clustering or classification.

## When NOT to use

- Input is a feature/abundance table already derived from amplicon ASVs/OTUs rather than reconstructed SSU sequences needing read-mapping quantification
- You want to estimate relative abundance of raw short reads directly with a classifier (Kraken2, Centrifuge) instead of validating reconstructed SSU sequences
- The reference SSU sequences have not yet been reconstructed (BBMap mapping requires the metagenome_SSU_sequences.fasta reference to exist)

## Inputs

- Quality-controlled trimmed reads from BBTOOLS (paired-end or single-end, .fastq/.fastq.gz)
- Reconstructed full-length SSU rRNA sequences file metagenome_SSU_sequences.fasta

## Outputs

- metagenome_SSU_taxonomy_abundance.tsv with columns Sequence_ID, Assigned reads, Relative_Abundance(%)

## How to apply

Load the BBTOOLS-trimmed, quality-controlled reads together with the reconstructed full-length SSU sequences file metagenome_SSU_sequences.fasta, and map the short reads onto the SSU sequences with BBMap v38.87 from BBTOOLS, limiting RAM to about 80% of available memory (e.g. -Xmx12g on a 16 GB machine). Tally the number of reads assigned to each reconstructed SSU sequence from the mapping. Normalize by dividing each sequence's assigned read count by the sum of all assigned reads and multiplying by 100 to express relative abundance in %. Write per-sequence abundances to metagenome_SSU_taxonomy_abundance.tsv with columns Sequence_ID, Assigned reads and Relative_Abundance(%). Rationale: reconstructed sequences are correctly assigned while direct classification of short Illumina reads yields high false-positive rates, so abundance is estimated against the reconstruction before 97% VSEARCH clustering.

## Related tools

- **BBMap** (Maps unfiltered high-quality reads onto the reconstructed SSU rRNA sequences to tally per-sequence assigned read counts; part of BBTOOLS, run with RAM limited to ~80% of available memory) — https://github.com/oschakoory/RiboTaxa
- **BBTools** (Tool suite providing BBMap for abundance estimation and upstream quality-controlled trimmed reads used as mapping input) — https://github.com/oschakoory/RiboTaxa

## Examples

```
bbmap.sh ref=metagenome_SSU_sequences.fasta in=trimmed_reads.fastq.gz out=bbmap_out.sam -Xmx12g
```

## Evaluation signals

- Output file metagenome_SSU_taxonomy_abundance.tsv exists with the required headers: Sequence_ID, Assigned reads, Relative_Abundance(%)
- Relative abundances in the output sum to approximately 100%
- Every Sequence_ID in the abundance table corresponds to a sequence present in metagenome_SSU_sequences.fasta
- BBMap completed within the expected memory envelope (e.g. -Xmx12g on a 16 GB machine) without out-of-memory failures
- Assigned-read counts are non-negative integers and sequences with zero mapped reads are handled consistently in the normalization

## Limitations

- RiboTaxa does not apply 16S rRNA gene copy number normalization, so abundances reflect reads mapped per reconstructed sequence rather than per organism
- Abundances can be affected by in-silico artifacts of full-length sequence reconstruction; 'we cannot exclude that a part of this diversity originates from artificial diversity created in silico during full-length sequence reconstruction'
- Reconstructed sequences may contain occasional small indel errors, which in practice have little effect on classification but could subtly affect mapping
- No chimera detection was included, and chimeric database sequences could carry over into SSU reconstructions if reads map across the full length of the chimera

## Evidence

- [methods] unfiltered high-quality reads are mapped onto them with BBmap v38.87, and relative abundance is calculated from this mapping before the reconstructed sequences are clustered at 97% with VSEARCH: [evidence span withheld — anchor status: not_found]
