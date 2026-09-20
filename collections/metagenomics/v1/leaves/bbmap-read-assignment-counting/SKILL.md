---
name: bbmap-read-assignment-counting
description: Use when you have reconstructed full-length SSU sequences (e.g. metagenome_SSU_sequences.fasta)
  from a metagenomic pipeline and need per-sequence relative abundance from the original
  high-quality reads before 97% clustering and taxonomic classification.
when_to_use_negative:
- Input is already a taxonomy or abundance table — this skill computes abundance from
  raw read-to-reconstruction mapping, not from existing profiles
- You need 16S rRNA gene copy-number corrected abundances — RiboTaxa explicitly chose
  not to use gene copy number correction, so this skill provides raw mapping-based
  relative abundance only
- You only have the SortMeRNA-filtered 16S/18S read subset — this skill requires mapping
  the unfiltered high-quality reads to estimate abundance
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: BBMap
  role: Maps unfiltered high-quality short reads onto the reconstructed SSU sequences
    to tally per-sequence assigned reads; run with RAM limited to ~80% of memory (e.g.
    RAM = 12 on 16 GB)
  repo: https://jgi.doe.gov/data-and-tools/bbtools/
- name: BBTools
  role: Tool suite providing BBMap and the upstream read trimming that produces the
    quality-controlled reads used as mapping input
  repo: https://jgi.doe.gov/data-and-tools/bbtools/
- name: RiboTaxa
  role: Pipeline that orchestrates the mapping, abundance normalization, and downstream
    97% VSEARCH clustering and QIIME2 sklearn classification
  repo: https://github.com/oschakoory/RiboTaxa
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:19048361d94056f4d48d2f93dfbb734ddaab7dfbba05c588cea30c7318125545
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/oschakoory/RiboTaxa
  tools:
  - BBMap
  - BBTools
  - RiboTaxa
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# bbmap-read-assignment-counting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

In the RiboTaxa pipeline, this skill estimates relative abundance of reconstructed full-length SSU rRNA sequences by mapping unfiltered high-quality metagenomic reads onto them with BBMap (BBTools) and normalizing per-sequence assigned read counts to percent, prior to 97% clustering (VSEARCH) and taxonomic affiliation.

## When to use

Use when you have reconstructed full-length SSU sequences (e.g. metagenome_SSU_sequences.fasta) from a metagenomic pipeline and need per-sequence relative abundance from the original high-quality reads before 97% clustering and taxonomic classification.

## When NOT to use

- Input is already a taxonomy or abundance table — this skill computes abundance from raw read-to-reconstruction mapping, not from existing profiles
- You need 16S rRNA gene copy-number corrected abundances — RiboTaxa explicitly chose not to use gene copy number correction, so this skill provides raw mapping-based relative abundance only
- You only have the SortMeRNA-filtered 16S/18S read subset — this skill requires mapping the unfiltered high-quality reads to estimate abundance

## Inputs

- Quality-controlled trimmed metagenomic reads (unfiltered high-quality FASTQ/FASTQ.GZ, single-end or paired-end, from BBTOOLS trimming)
- Reconstructed full-length SSU rRNA sequences reference FASTA (metagenome_SSU_sequences.fasta)
- Available machine memory setting (e.g. RAM = 12 for a 16 GB machine)

## Outputs

- Per-sequence abundance table metagenome_SSU_taxonomy_abundance.tsv with columns Sequence_ID, Assigned reads, Relative_Abundance(%)
- BBMap mapping results (read assignments of short reads to each reconstructed SSU sequence)

## How to apply

Load the quality-controlled trimmed reads (produced by BBTOOLS trimming) and the reconstructed SSU reference file metagenome_SSU_sequences.fasta. Map the short reads onto the SSU sequences using BBMap from BBTOOLS, capping RAM at about 80% of available memory (e.g. RAM = 12 on a 16 GB machine). Critically, the reads mapped are the unfiltered high-quality reads, not only the SortMeRNA-extracted 16S/18S subset, so abundance reflects the whole metagenome. Tally reads assigned to each reconstructed SSU sequence, then divide each count by the sum of all assigned reads and multiply by 100 to express Relative_Abundance(%). Write per-sequence abundances to metagenome_SSU_taxonomy_abundance.tsv with columns Sequence_ID, Assigned reads, and Relative_Abundance(%), and validate the file before downstream 97% VSEARCH clustering and sklearn-based affiliation.

## Related tools

- **BBMap** (Maps unfiltered high-quality short reads onto the reconstructed SSU sequences to tally per-sequence assigned reads; run with RAM limited to ~80% of memory (e.g. RAM = 12 on 16 GB)) — https://jgi.doe.gov/data-and-tools/bbtools/
- **BBTools** (Tool suite providing BBMap and the upstream read trimming that produces the quality-controlled reads used as mapping input) — https://jgi.doe.gov/data-and-tools/bbtools/
- **RiboTaxa** (Pipeline that orchestrates the mapping, abundance normalization, and downstream 97% VSEARCH clustering and QIIME2 sklearn classification) — https://github.com/oschakoory/RiboTaxa

## Examples

```
bbmap.sh ref=metagenome_SSU_sequences.fasta in=trimmed_reads.fastq ram=12
```

## Evaluation signals

- Output file metagenome_SSU_taxonomy_abundance.tsv exists and contains the exact column headers Sequence_ID, Assigned reads, Relative_Abundance(%)
- The Relative_Abundance(%) column sums to approximately 100% across all reconstructed SSU sequences
- Assigned reads values are non-negative integers, and each count is consistent with the BBMap mapping tally against metagenome_SSU_sequences.fasta
- The mapping used the unfiltered high-quality trimmed reads, not only the SortMeRNA-extracted 16S/18S subset
- RAM usage of the BBMap run was capped at ~80% of available memory (e.g. RAM = 12 on a 16 GB machine)

## Limitations

- RiboTaxa deliberately does not apply 16S rRNA gene copy number correction, so relative abundances are not adjusted for copy-number bias
- A portion of reconstructed sequence diversity may originate from artificial diversity created in silico during full-length sequence reconstruction
- Occasional small indel errors can occur in reconstructed sequences, which may slightly affect read assignment
- Chimera detection was not included in the 16S rRNA gene reconstruction workflow, and chimeric database sequences could carry over into reconstructions if reads map across the full length of the chimera

## Evidence

- [methods] BBMap mapping with RAM cap: [evidence span withheld — anchor status: not_found]
- [methods] Relative abundance normalization: [evidence span withheld — anchor status: not_found]
- [abstract] Pipeline tool compilation including BBMap: "We then compiled the best tools (BBTools, FastQC, SortMeRNA, MetaRib, EMIRGE, VSEARCH, BBMap and QIIME 2’s Sklearn classifier) to build a pipeline called RiboTaxa"
- [discussion] RiboTaxa scope including taxonomic profiling: "RiboTaxa is a user-friendly metagenomic classifier which supports database preparation, read trimming and SSU read extraction, SSU rRNA gene targeted assembly and taxonomic profiling."
