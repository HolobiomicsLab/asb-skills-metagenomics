---
name: digital-normalization-of-reads
description: Use when preparing metagenomic/virome FASTQ reads for de-novo assembly
  where read depth is uneven and deep coverage would inflate assembly time or contig
  composition, e.g. prior to Virmap.pl or MEGAHIT.
when_to_use_negative:
- Input is already assembled contigs or a taxonomic table — normalization acts on
  raw reads only
- Reads are low-coverage or nearly uniform in coverage, where normalization offers
  little benefit and could discard informative depth before assembly
- A mapping-only workflow against a single reference genome is planned instead of
  de-novo assembly
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: VirMAP (Virmap.pl)
  role: Orchestrates dereplication and digital normalization of input reads via --useBbnorm
    before assembly
  repo: https://github.com/cmmr/VirMAP
- name: BBMap
  role: Provides the bbnorm normalization invoked with --useBbnorm
- name: khmer (normalize-by-median.py)
  role: Alternative digital normalization utility referenced by the article
  repo: https://github.com/dib-lab/khmer
- name: fasterq-dump
  role: Retrieves raw FASTQ reads from SRA prior to normalization
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:77975798259595fe9318d3ae6a58b76854680d4d659618f2c4c14fc5cb62691b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cmmr/VirMAP
  tools:
  - BBMap
  - DIAMOND
  - MEGAHIT
  - Perl
  - VirMAP
  - VirMAP (Virmap.pl)
  - fasterq-dump
  - khmer (normalize-by-median.py)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Digital normalization of reads

## Summary

Dereplicate and digitally normalize virome reads before assembly so that de-novo assembly (MEGAHIT) is not dominated by highly abundant sequences. VirMAP enables this via its --useBbnorm flag (BBMap) or khmer's normalize-by-median.py, using default parameters.

## When to use

Use when preparing metagenomic/virome FASTQ reads for de-novo assembly where read depth is uneven and deep coverage would inflate assembly time or contig composition, e.g. prior to Virmap.pl or MEGAHIT.

## When NOT to use

- Input is already assembled contigs or a taxonomic table — normalization acts on raw reads only
- Reads are low-coverage or nearly uniform in coverage, where normalization offers little benefit and could discard informative depth before assembly
- A mapping-only workflow against a single reference genome is planned instead of de-novo assembly

## Inputs

- Raw paired-end FASTQ reads from SRA (retrieved via fasterq-dump, e.g. SRR9875293_1.fastq)
- VirMAP per-sample output directory and VirmapRun.err stderr log
- VirMAP reference database files (Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd)

## Outputs

- Dereplicated and digitally normalized per-sample read sets feeding MEGAHIT assembly
- Per-sample VirMAP outputs (raw unfiltered taxa IDs and filtered table of taxa IDs)
- Aggregated per-sample metrics: viral contig megabases, contigs-per-taxon ratios, taxa-per-sample counts at 10%, 1%, and 0.1% subsampling

## How to apply

Retrieve raw reads with fasterq-dump (e.g. 'fasterq-dump -t /dev/shm -e 4 -O <outdir> <accession>'), then pass them through Virmap.pl with the --useBbnorm flag so input reads are dereplicated and digitally normalized using default parameters before the multi-kmer de-novo assembly with MEGAHIT (--useMegahit). Normalization is invoked per sample alongside the --sampleName flag and writes per-sample outputs (with stderr to VirmapRun.err) for downstream taxonomic classification via the bits-per-base scoring against the VirMAP database (Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd). The rationale is that normalization reduces the dominance of high-coverage regions so contigs from both abundant and low-coverage/divergent viruses are recovered, which is then reflected in viral contig megabases and taxa-per-sample counts (e.g. 231 vs 236 filtered taxa IDs per sample on PRJNA395784).

## Related tools

- **VirMAP (Virmap.pl)** (Orchestrates dereplication and digital normalization of input reads via --useBbnorm before assembly) — https://github.com/cmmr/VirMAP
- **BBMap** (Provides the bbnorm normalization invoked with --useBbnorm)
- **khmer (normalize-by-median.py)** (Alternative digital normalization utility referenced by the article) — https://github.com/dib-lab/khmer
- **fasterq-dump** (Retrieves raw FASTQ reads from SRA prior to normalization)

## Examples

```
Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq --useMegahit --useBbnorm --sampleName SRR9875293
```

## Evaluation signals

- Confirm the Virmap.pl command line includes --useBbnorm (and --useMegahit) and that stderr redirected to VirmapRun.err shows no normalization failures
- Verify reads were dereplicated before digital normalization (VirMAP dereplicates then normalizes with default parameters)
- Compare recovered viral contig megabases against article values (VirMAP 1.69/1.04 vs standard approach 4.3/4.0) as an assembly-quality invariant
- Check per-sample filtered taxa ID counts against reported averages (231 VirMAP vs 236 standard, filtered at 1000 bits/taxa ID; 423 unfiltered at 300 bits/contig)
- Verify the aggregated CSV contains metrics at 10%, 1%, and 0.1% subsampling levels for each of the four PRJNA395784 samples

## Limitations

- Normalization is applied with default parameters in VirMAP; no parameter tuning guidance or Supplementary Data 14 sensitivity/specificity settings are provided in the article text
- No software versions are stated for BBMap, khmer, MEGAHIT, or DIAMOND, so results may vary across versions
- Normalization alone does not guarantee accurate classification of low-coverage, highly divergent viruses; it must be paired with VirMAP's nucleotide + protein taxonomic scoring

## Evidence

- [discussion] Read dereplication and digital normalization: "input reads are dereplicated and digitally normalized18 using default parameters"
- [methods] Virmap.pl flags enabling normalization and assembly: "--useMegahit --useBbnorm --sampleName SRR9875293"
- [methods] khmer normalization tool referenced: "normalize-by-median.py (https://github.com/dib-lab/khmer)"
- [discussion] MEGAHIT as downstream assembler: "a multi-kmer de-novo assembly using either MEGAHIT21,22 or Tadpole19, or both"
