---
name: metagenomic-short-read-mapping
description: Use when you have reconstructed full-length SSU rRNA sequences (e.g.
  metagenome_SSU_sequences.fasta from EMIRGE/MetaRib assembly) and need per-sequence
  relative abundances before 97% VSEARCH clustering.
when_to_use_negative:
- You only need to extract/filter SSU reads from raw metagenomic data — that role
  belongs to SortMeRNA (or BBMap read extraction), not abundance mapping of reconstructed
  sequences
- Input is an already-clustered (97% VSEARCH) representative sequence set rather than
  the pre-clustering reconstructed SSU sequences
- You want to classify reconstructed sequences taxonomically — that is done with VSEARCH
  and QIIME 2's sklearn classifier, not read mapping
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: BBMap
  role: General-purpose short-read mapper used to map unfiltered high-quality reads
    onto the reconstructed SSU rRNA sequences for relative abundance estimation
  repo: https://github.com/oschakoory/RiboTaxa
- name: BBTools
  role: Tool suite providing BBMap; also used upstream for adapter removal and trimming
    (ktrim, minlength, trimq, qtrim, maxns) producing the quality-controlled reads
    that are mapped
  repo: https://github.com/oschakoory/RiboTaxa
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:eebbf9e39bee1c2cbade8063fc1493512e635cd2359e7353b9debd80693de1d7
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# metagenomic-short-read-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps unfiltered high-quality metagenomic reads back onto reconstructed full-length SSU rRNA sequences using BBMap (BBTools v38.87) to estimate the relative abundance of each reconstructed sequence in the RiboTaxa pipeline.

## When to use

Use when you have reconstructed full-length SSU rRNA sequences (e.g. metagenome_SSU_sequences.fasta from EMIRGE/MetaRib assembly) and need per-sequence relative abundances before 97% VSEARCH clustering.

## When NOT to use

- You only need to extract/filter SSU reads from raw metagenomic data — that role belongs to SortMeRNA (or BBMap read extraction), not abundance mapping of reconstructed sequences
- Input is an already-clustered (97% VSEARCH) representative sequence set rather than the pre-clustering reconstructed SSU sequences
- You want to classify reconstructed sequences taxonomically — that is done with VSEARCH and QIIME 2's sklearn classifier, not read mapping

## Inputs

- Quality-controlled trimmed short reads (BBTOOLS output, FASTQ/FASTQ.GZ, single- or paired-end Illumina)
- Reconstructed full-length SSU rRNA sequences file metagenome_SSU_sequences.fasta (reference)

## Outputs

- Per-sequence abundance table metagenome_SSU_taxonomy_abundance.tsv with columns Sequence_ID, Assigned reads, Relative_Abundance(%)

## How to apply

Load the quality-controlled trimmed reads produced by BBTOOLS together with the reconstructed full-length SSU sequences file metagenome_SSU_sequences.fasta. Map the short reads onto the SSU sequences with BBMap from BBTOOLS (v38.87), limiting RAM to approximately 80% of available memory (e.g. RAM = 12 on a 16 GB machine). Tally the reads assigned to each reconstructed SSU sequence, then normalize by dividing each count by the sum of all assigned reads and multiplying by 100 to express relative abundance in %. Write the per-sequence abundances to metagenome_SSU_taxonomy_abundance.tsv with the columns Sequence_ID, Assigned reads and Relative_Abundance(%). This mapping step precedes the 97% clustering (VSEARCH) and taxonomic affiliation steps so that abundances are estimated on unfiltered reads against the reconstructed reference.

## Related tools

- **BBMap** (General-purpose short-read mapper used to map unfiltered high-quality reads onto the reconstructed SSU rRNA sequences for relative abundance estimation) — https://github.com/oschakoory/RiboTaxa
- **BBTools** (Tool suite providing BBMap; also used upstream for adapter removal and trimming (ktrim, minlength, trimq, qtrim, maxns) producing the quality-controlled reads that are mapped) — https://github.com/oschakoory/RiboTaxa

## Examples

```
bbmap.sh ref=metagenome_SSU_sequences.fasta in=trimmed_reads.fastq ram=12
```

## Evaluation signals

- Output file metagenome_SSU_taxonomy_abundance.tsv exists and contains the required headers: Sequence_ID, Assigned reads, Relative_Abundance(%)
- The Relative_Abundance(%) column sums to approximately 100%
- Each assigned read count is a non-negative integer and every Sequence_ID corresponds to a header in metagenome_SSU_sequences.fasta
- Mapping was run with BBMap RAM capped near 80% of machine memory (e.g. ram=12 for 16 GB) so the job completed without memory errors
- Abundances were computed from unfiltered high-quality reads (not the SortMeRNA-filtered SSU subset), per the RiboTaxa design

## Limitations

- SortMeRNA performed better than BBMap for extracting assemblable rRNA reads, so BBMap-based mapping is used here for abundance estimation rather than read extraction
- No 16S rRNA gene copy number normalization is applied in RiboTaxa, so abundances are not copy-number corrected
- Artificial diversity created in silico during full-length sequence reconstruction cannot be excluded, and reconstructed sequences may carry small indel errors affecting mapping
- RiboTaxa is slower than short-read classifiers such as Kraken2 and Centrifuge

## Evidence

- [methods] In RiboTaxa, the reconstructed SSU sequences serve as the reference: unfiltered high-quality reads are mapped onto them with BBmap v38.87, and relative abundance is calculated from this mapping before the reconstructed sequences are clustered at 97% with VSEARCH.: [evidence span withheld — anchor status: not_found]
- [methods] Map the short reads onto the SSU sequences with BBMap from BBTOOLS, limiting RAM to approximately 80% of available memory (e.g. RAM = 12 for a 16 GB machine).: [evidence span withheld — anchor status: not_found]
