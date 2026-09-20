---
name: inverted-terminal-repeat-palindrome-detection
description: Use when input is paired-end, short-read metagenomic FASTQ.GZ data (or
  mixed microbial genomes) and the goal is to discover novel insertion sequences via
  their inverted terminal repeats, e.g. to build or query an IS catalogue.
when_to_use_negative:
- The target ISs lack inverted terminal repeats or have ITRs shorter than 10 nt (Palidis
  is only equipped to discover ISs with ITRs, and the catalogue uses a 25 nt minimum)
- The ISs of interest are longer than 3000 bp or lengthened by nested insertion events,
  pushing ITRs outside the 500–3000 bp proximity range
- Input is isolate genome data intended for annotation against ISC built from healthy
  human oral cavity and stool metagenomes only — the catalogue's scope is limited
  to those sources
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0091
tools:
- name: Palidis (palidis4)
  role: Nextflow pipeline that performs palindromic detection of insertion sequences
    via maximal exact matching, ITR distance filtering, CD-HIT-EST clustering, and
    transposase annotation
  repo: https://github.com/blue-moon22/palidis4
- name: pal-MEM
  role: Efficient computation of Maximal Exact Matches (modified from E-MEM) to identify
    inverted terminal repeats in large metagenomic read files
  repo: https://github.com/blue-moon22/pal-MEM
- name: CD-HIT-EST
  role: Clusters pooled IS sequences at 0.95 identity threshold to remove redundancy
    and build the non-redundant catalogue
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:e69c9bf48665c25a34f03127c097325f21ea922d63b1d7f605cb50605f5ed376
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/palidis4
  tools:
  - CD-HIT-EST
  - Palidis
  - Palidis (palidis4)
  - pal-MEM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# inverted-terminal-repeat-palindrome-detection

## Summary

Discovers novel insertion sequences (ISs) in short-read metagenomic data by detecting palindromic inverted terminal repeats (ITRs) via efficient maximal exact matching, then filtering candidates by ITR length (>=25 nt) and IS size (500–3000 bp) before clustering into a non-redundant catalogue.

## When to use

Use when input is paired-end, short-read metagenomic FASTQ.GZ data (or mixed microbial genomes) and the goal is to discover novel insertion sequences via their inverted terminal repeats, e.g. to build or query an IS catalogue.

## When NOT to use

- The target ISs lack inverted terminal repeats or have ITRs shorter than 10 nt (Palidis is only equipped to discover ISs with ITRs, and the catalogue uses a 25 nt minimum)
- The ISs of interest are longer than 3000 bp or lengthened by nested insertion events, pushing ITRs outside the 500–3000 bp proximity range
- Input is isolate genome data intended for annotation against ISC built from healthy human oral cavity and stool metagenomes only — the catalogue's scope is limited to those sources

## Inputs

- Paired-end short-read metagenomic FASTQ.GZ files (e.g. Human Microbiome Project raw reads)
- Per-sample assembled contigs (FASTA) referenced in a tab-delimited manifest with lane_id, read_directory, sample_id, contigs_path
- Optional reference IS databases (ISfinder or a COBS index of 661,405 microbial genomes) for origin prediction

## Outputs

- FASTA file of insertion sequences per sample
- Per-IS information table (IS_name, sample_id, contig, is_start, is_end, description with InterPro/PANTHER transposase accessions)
- Non-redundant IS catalogue (e.g. Insertion Sequence Catalogue v1.0.0 with 879 unique ISs after CD-HIT-EST clustering at 0.95 identity)

## How to apply

Run Palidis (a Nextflow pipeline) on each metagenome with default parameters: it pre-processes FASTQ.GZ reads, uses pal-MEM (modified from E-MEM) for efficient maximal exact matching to find repeat sequences, maps reads against assemblies with Bowtie2, and applies distance filters to get candidate ITRs. Detect ISs as palindromic ITRs of at least 25 nt flanking regions in the 500–3000 bp proximity range; cluster candidate ITRs with CD-HIT-EST (sequence identity threshold -c 0.95) and filter by cluster concordance to emit final insertion sequences. Key tunable parameters include --min_itr_length (default 25), --max_itr_length (default 50), --kmer_length (default 15), --min_is_len (default 500), and --max_is_len (default 3000); lowering ITR length or k-mer thresholds increases sensitivity but is more computationally intensive. The rationale is that ISs commonly carry characteristic inverted repeats, so palindromic MEMs flanking a correctly-sized inter-repeat region are strong IS candidates that can be annotated for transposases (Prodigal + InterProScan) and pooled into a non-redundant catalogue.

## Related tools

- **Palidis (palidis4)** (Nextflow pipeline that performs palindromic detection of insertion sequences via maximal exact matching, ITR distance filtering, CD-HIT-EST clustering, and transposase annotation) — https://github.com/blue-moon22/palidis4
- **pal-MEM** (Efficient computation of Maximal Exact Matches (modified from E-MEM) to identify inverted terminal repeats in large metagenomic read files) — https://github.com/blue-moon22/pal-MEM
- **CD-HIT-EST** (Clusters pooled IS sequences at 0.95 identity threshold to remove redundancy and build the non-redundant catalogue)

## Examples

```
nextflow palidis.nf --manifest manifest.tsv --batch_name hmp_batch -c configs/conf/<name_of_config>.config
```

## Evaluation signals

- Applying the workflow to 264 HMP metagenomes must yield 2517 ISs on 1837 contigs across 218 of 264 samples
- After CD-HIT-EST pooling at -c 0.95, the catalogue must contain 879 unique ISs with 87 unique transposases, matching released ISC v1.0.0 at github.com/blue-moon22/ISC
- 519 of the 879 unique ISs must be novel (absent from previously characterized ISs)
- All reported ISs should have ITRs of 25 or more nucleotides and fall within the 500–3000 bp length range at default settings
- The pipeline's regression test must pass: ./tests/regression_tests.sh prints 'Test passed. All outputs expected.'

## Limitations

- Palidis only discovers ISs with ITRs, so the ISC contains only common DDE types of ISs with ITRs; ISs lacking ITRs are missed
- ITRs as short as ten nucleotides are missed at the default 25 nt minimum threshold, and lowering the threshold (or k-mer length) is more computationally intensive
- ISs flanked by ITRs beyond the 500–3000 bp proximity range (e.g. regions lengthened by nested insertion events) are missed; raising the maximum IS length is a planned future improvement
- ISC v1.0.0 was built only from healthy human oral cavity and stool metagenomes, limiting its use as an annotation reference to those sources

## Evidence

- [intro] we present a tool called Palidis (palindromic detection of insertion sequences) that finds ISs using an efficient maximal exact matching algorithm: "we present a tool called Palidis (palindromic detection of insertion sequences) that finds ISs using an efficient maximal exact matching algorithm"
- [abstract] Applying Palidis to 264 human metagenomes identifies 879 unique insertion sequences, with 519 being novel: "Applying Palidis to 264 human metagenomes identifies 879 unique insertion sequences, with 519 being novel and not previously characterized."
- [discussion] the catalogue currently contains ISs with ITRs that are 25 or greater nucleotides in length as generated by Palidis: "the catalogue currently contains ISs with ITRs that are 25 or greater nucleotides in length as generated by Palidis, although ITRs can be as short as ten nucleotides in length"
- [readme] --min_itr_length    Minimum length of ITR. (Default: 25): "--min_itr_length    Minimum length of ITR. (Default: 25)"
- [readme] pal-MEM finds inverted terminal repeats (ITRs) in large genomes, including metagenomes.: [evidence span withheld — anchor status: external_doc]
