---
name: metarib-targeted-assembly
description: 'Use when reconstructing SSU rRNA genes from metagenomic reads as part
  of the RiboTaxa pipeline: specifically, when MetaRib must assemble rRNA targets
  from the unfiltered high-quality (BBTools-trimmed) reads, complementary to the SortMeRNA-filtered
  EMIRGE branch.'
when_to_use_negative:
- When only the SortMeRNA-filtered 16S/18S reads are available as the sole reconstruction
  input — that read set belongs to the EMIRGE (Emirge_amplicon.py) branch, while MetaRib
  is fed unfiltered high-quality reads.
- When reads are raw and have not passed BBTOOLS adapter trimming/quality filtering
  (minlength=60, trimq=20, maxns=1) or when data is long-read rather than Illumina
  metagenomic data.
- When only short-read taxonomic classification is needed (e.g. Kraken2, Centrifuge)
  — the article notes RiboTaxa is not as fast as these short-read classifiers.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0637
tools:
- name: MetaRib
  role: Reference-targeted assembler that reconstructs SSU rRNA gene sequences from
    the unfiltered high-quality reads (minid = 0.96, idfilter = 0.98)
- name: EMIRGE
  role: Complementary assembler (via Emirge_amplicon.py, with USEARCH internally,
    120 iterations, JOIN_TRESHOLD/IDENTITY = 1) run on SortMeRNA-extracted 16S/18S
    reads; outputs are combined with MetaRib reconstructions
- name: SortMeRNA
  role: rRNA-specialized tool that extracts 16S/18S reads against SILVA SSU 138.1
    for the EMIRGE branch, producing *_16S18Sreads.fastq and a .log with the percentage
    of SSU reads filtered
- name: BBTools
  role: Quality control/adapter trimming (ktrim=r, kmer=21, minlength=60, trimq=20,
    qtrim=rl, maxns=1) producing the high-quality reads that feed MetaRib
  repo: https://jgi.doe.gov/data-and-tools/bbtools/
- name: RiboTaxa
  role: Pipeline that orchestrates QC, SSU read extraction, targeted SSU rRNA reconstruction
    and taxonomic classification
  repo: https://github.com/oschakoory/RiboTaxa
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:84487da607b5c245e40a45ea0a2368b68854f76650b189fcd6e475693c1323c6
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
  - EMIRGE
  - MetaRib
  - RiboTaxa
  - SortMeRNA
  - USEARCH
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# metarib-targeted-assembly

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct full-length/nearly full-length SSU rRNA gene sequences from unfiltered, adapter-trimmed high-quality Illumina metagenomic reads using MetaRib with minid = 0.96 and idfilter = 0.98, as one of the two assembler branches in RiboTaxa's SSU_sequences reconstruction layer.

## When to use

Use when reconstructing SSU rRNA genes from metagenomic reads as part of the RiboTaxa pipeline: specifically, when MetaRib must assemble rRNA targets from the unfiltered high-quality (BBTools-trimmed) reads, complementary to the SortMeRNA-filtered EMIRGE branch.

## When NOT to use

- When only the SortMeRNA-filtered 16S/18S reads are available as the sole reconstruction input — that read set belongs to the EMIRGE (Emirge_amplicon.py) branch, while MetaRib is fed unfiltered high-quality reads.
- When reads are raw and have not passed BBTOOLS adapter trimming/quality filtering (minlength=60, trimq=20, maxns=1) or when data is long-read rather than Illumina metagenomic data.
- When only short-read taxonomic classification is needed (e.g. Kraken2, Centrifuge) — the article notes RiboTaxa is not as fast as these short-read classifiers.

## Inputs

- Adapter-trimmed high-quality Illumina metagenomic reads from BBTOOLS QC (fastq/fastq.gz, single- or paired-end, uncompressed or compressed)
- Indexed reference for rRNA read extraction (SILVA SSU 138.1, e.g. sortmerna_indexed_DB) used in the parallel EMIRGE branch
- Bowtie-indexed SILVA SSU 138.1 database used by EMIRGE on the SortMeRNA-filtered reads

## Outputs

- MetaRib-reconstructed SSU rRNA sequences (FASTA)
- metagenome_SSU_sequences.fasta in the SSU_sequences output directory, combining EMIRGE and MetaRib full-length/nearly full-length 16S/18S reconstructions

## How to apply

After BBTOOLS quality control (ktrim=r, kmer=21, minlength=60, trimq=20, qtrim=rl, maxns=1), take the adapter-trimmed high-quality reads WITHOUT SortMeRNA filtering and run MetaRib with minid = 0.96 and idfilter = 0.98 to reconstruct SSU rRNA sequences. Rationale: the article found that SortMeRNA outperformed BBMap at extracting assemblable rRNA reads (which feed the EMIRGE branch), but combining reference-targeted assembly on both the filtered (EMIRGE) and unfiltered (MetaRib) read sets gave greater population characterisation. In parallel, EMIRGE (Emirge_amplicon.py) runs on the SortMeRNA-extracted 16S/18S reads against the Bowtie-indexed SILVA SSU 138.1 database with JOIN_TRESHOLD/IDENTITY = 1 for candidate merging and 120 iterations. Finally, concatenate the MetaRib and EMIRGE outputs into metagenome_SSU_sequences.fasta in the SSU_sequences output directory; this combined FASTA is the input for downstream QIIME 2 sklearn taxonomic classification.

## Related tools

- **MetaRib** (Reference-targeted assembler that reconstructs SSU rRNA gene sequences from the unfiltered high-quality reads (minid = 0.96, idfilter = 0.98))
- **EMIRGE** (Complementary assembler (via Emirge_amplicon.py, with USEARCH internally, 120 iterations, JOIN_TRESHOLD/IDENTITY = 1) run on SortMeRNA-extracted 16S/18S reads; outputs are combined with MetaRib reconstructions)
- **SortMeRNA** (rRNA-specialized tool that extracts 16S/18S reads against SILVA SSU 138.1 for the EMIRGE branch, producing *_16S18Sreads.fastq and a .log with the percentage of SSU reads filtered)
- **BBTools** (Quality control/adapter trimming (ktrim=r, kmer=21, minlength=60, trimq=20, qtrim=rl, maxns=1) producing the high-quality reads that feed MetaRib) — https://jgi.doe.gov/data-and-tools/bbtools/
- **RiboTaxa** (Pipeline that orchestrates QC, SSU read extraction, targeted SSU rRNA reconstruction and taxonomic classification) — https://github.com/oschakoory/RiboTaxa

## Examples

```
metarib -r metagenome_R1_trimmed.fastq,metagenome_R2_trimmed.fastq --minid 0.96 --idfilter 0.98 -o MetaRib_SSU_out
```

## Evaluation signals

- The SSU_sequences output directory contains metagenome_SSU_sequences.fasta that exists and is valid FASTA.
- metagenome_SSU_sequences.fasta contains full-length/nearly full-length 16S/18S rRNA gene sequences reconstructed by both EMIRGE and MetaRib.
- The SortMeRNA .log reports the percentage of SSU reads filtered, confirming the EMIRGE branch input was correctly extracted while the MetaRib branch ran on unfiltered reads.
- MetaRib settings match the article's parameters (minid = 0.96, idfilter = 0.98), and EMIRGE ran with JOIN_TRESHOLD/IDENTITY = 1 and 120 iterations.
- Downstream taxonomic classification of the combined reconstructions reproduces the expected taxonomic resolution (RiboTaxa reported 100% and 83.7% F-measure at genus and species level with no false positives on the mock community).

## Limitations

- Chimera detection was not included because no chimeric sequences were detected during 16S rRNA gene reconstruction, but chimeric database sequences could carry over into SSU reconstructions if reads map across the full length of the chimera.
- A part of the observed diversity may originate from artificial in silico diversity created during full-length sequence reconstruction, and occasional small indel errors may occur in reconstructed sequences.
- High 16S rRNA conservation prevents discrimination of all species (e.g. certain Enterobacteriaceae and Lachnospiraceae), so some sequences stop at genus level.
- No 16S rRNA gene copy number normalization is applied, and RiboTaxa is slower than short-read classifiers such as Kraken2 and Centrifuge.

## Evidence

- [methods] MetaRib is run on the unfiltered high-quality reads with minid = 0.96 and idfilter = 0.98 in the RiboTaxa SSU reconstruction workflow.: [evidence span withheld — anchor status: not_found]
