---
name: ssu-rrna-read-filtering
description: Use when metagenomic Illumina (meta)genomic FASTQ reads need enrichment
  for SSU rRNA fragments before targeted full-length 16S/18S reconstruction, e.g.
  RiboTaxa-style extraction against SILVA SSU 138.1.
when_to_use_negative:
- The reads are long assembled contigs or genomes rather than short Illumina reads
  — use rRNA feature annotation (e.g. barrnap) instead
- The goal is taxonomic classification directly on reads; the article warns that 'inferring
  taxa directly to short-length Illumina reads resulted in a high rate of false positives
  while reconstructed sequences were correctly assigned'
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3941
- http://edamontology.org/topic_3168
tools:
- name: SortMeRNA
  role: rRNA-specialized read mapper used to extract/filter 16S/18S reads against
    the pre-indexed SILVA SSU 138.1 database; benchmarked as better than BBMap for
    extracting assemblable rRNA reads
- name: BBTools
  role: Upstream adapter removal and quality trimming (ktrim=r, kmer=21, minlength=60,
    trimq=20, qtrim=rl, maxns=1) producing the clean FASTQ input for SSU extraction
  repo: https://jgi.doe.gov/data-and-tools/bbtools/
- name: EMIRGE (Emirge_amplicon.py)
  role: 'Downstream consumer: reconstructs full-length SSU rRNA sequences from the
    SortMeRNA-filtered *_16S18Sreads.fastq reads'
  repo: https://github.com/oschakoory/RiboTaxa
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:b8fff4c71ba86e395784c961abee8f1ccc41883091f3a597d2a45f3efc878c20
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
  - SortMeRNA
  - USEARCH
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3941
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# ssu-rrna-read-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filters 16S/18S SSU rRNA reads out of adapter-trimmed metagenomic FASTQ reads using SortMeRNA against a pre-indexed SILVA SSU 138.1 database, producing rRNA-enriched FASTQ files that feed EMIRGE-based SSU gene reconstruction in the RiboTaxa pipeline.

## When to use

Use when metagenomic Illumina (meta)genomic FASTQ reads need enrichment for SSU rRNA fragments before targeted full-length 16S/18S reconstruction, e.g. RiboTaxa-style extraction against SILVA SSU 138.1.

## When NOT to use

- The reads are long assembled contigs or genomes rather than short Illumina reads — use rRNA feature annotation (e.g. barrnap) instead
- The goal is taxonomic classification directly on reads; the article warns that 'inferring taxa directly to short-length Illumina reads resulted in a high rate of false positives while reconstructed sequences were correctly assigned'

## Inputs

- Adapter-trimmed, quality-trimmed single-end or paired-end Illumina reads (.fastq or fastq.gz) from BBTOOLS QC
- Pre-indexed SILVA SSU 138.1 SortMeRNA database (sortmerna_indexed_DB)

## Outputs

- *_16S18Sreads.fastq files containing SSU-enriched reads per input sample
- SortMeRNA .log file reporting the percentage of SSU reads filtered
- SSU-enriched read set fed to Emirge_amplicon.py for full-length SSU reconstruction

## How to apply

Start from adapter-trimmed, quality-trimmed reads produced by BBTOOLS (ktrim=r, kmer=21, minlength=60, trimq=20, qtrim=rl, maxns=1). Run SortMeRNA (the rRNA-specialized extraction tool) against the pre-indexed SILVA SSU 138.1 database (sortmerna_indexed_DB) to identify SSU rRNA fragments in the large dataset, emitting *_16S18Sreads.fastq per sample plus a .log reporting the percentage of SSU reads filtered. The article's benchmarking showed SortMeRNA outperformed the general-purpose mapper BBMap for extracting reads that could subsequently be assembled with EMIRGE (Emirge_amplicon.py), MATAM and MetaRib, which justifies SortMeRNA as the extraction engine. These filtered reads are then handed to Emirge_amplicon.py against the Bowtie-indexed SILVA database, while MetaRib concurrently receives the unfiltered high-quality reads; the two reconstructions are merged into metagenome_SSU_sequences.fasta.

## Related tools

- **SortMeRNA** (rRNA-specialized read mapper used to extract/filter 16S/18S reads against the pre-indexed SILVA SSU 138.1 database; benchmarked as better than BBMap for extracting assemblable rRNA reads)
- **BBTools** (Upstream adapter removal and quality trimming (ktrim=r, kmer=21, minlength=60, trimq=20, qtrim=rl, maxns=1) producing the clean FASTQ input for SSU extraction) — https://jgi.doe.gov/data-and-tools/bbtools/
- **EMIRGE (Emirge_amplicon.py)** (Downstream consumer: reconstructs full-length SSU rRNA sequences from the SortMeRNA-filtered *_16S18Sreads.fastq reads) — https://github.com/oschakoory/RiboTaxa

## Evaluation signals

- A *_16S18Sreads.fastq file exists for each input sample and is valid FASTQ containing only SSU-enriched reads
- The SortMeRNA .log reports a plausible percentage of SSU reads filtered (shotgun metagenomes typically yield a small rRNA fraction)
- The extraction used the pre-indexed SILVA SSU 138.1 database (sortmerna_indexed_DB), consistent with the pipeline's database setup
- Downstream, Emirge_amplicon.py consumes the SortMeRNA-filtered reads while MetaRib receives the unfiltered high-quality reads, and both reconstructions appear in metagenome_SSU_sequences.fasta
- The input FASTQ passed BBTOOLS QC with the documented parameters (ktrim=r, kmer=21, minlength=60, trimq=20, qtrim=rl, maxns=1) before filtering

## Limitations

- SortMeRNA outperformed BBMap in the RiboTaxa benchmark, but read-extraction quality still conditions assemblability of downstream EMIRGE/MetaRib reconstructions
- Chimeric database sequences could carry over into SSU reconstructions if reads map across the full length of a chimera, and no chimera detection was included in the 16S rRNA gene reconstruction workflow
- Occasional small indel errors in reconstructed sequences may occur downstream of filtering

## Evidence

- [methods] This folder contains filtered 16S/18S sequences from your trimmed (meta)genomics sequence files: [evidence span withheld — anchor status: not_found]
