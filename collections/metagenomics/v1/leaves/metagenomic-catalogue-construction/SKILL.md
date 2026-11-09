---
name: metagenomic-catalogue-construction
description: Use when short-read metagenomic (FASTQ.GZ) data from mixed microbial
  communities must be mined for novel insertion sequences and consolidated into a
  non-redundant FASTA catalogue for downstream annotation or genome querying.
when_to_use_negative:
- The target ISs lack inverted terminal repeats or have ITRs shorter than the 25 nt
  default (ITRs as short as 10 nt are missed at default settings) — Palidis only detects
  ISs with ITRs
- ISs are flanked outside the 500–3000 bp proximity range (e.g. regions lengthened
  by nested insertion events) — these are missed by Palidis
- The input is isolate genomes to annotate rather than mixed-community metagenomic
  reads — use the released ISC as a reference database instead of rebuilding it
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: Palidis
  role: Nextflow pipeline that detects ISs by identifying palindromic ITRs (>=25 nt)
    via maximal exact matching, filters by IS length (500-3000 bp), and clusters candidate
    ITRs with CD-HIT-EST
  repo: https://github.com/blue-moon22/palidis4
- name: pal-MEM
  role: Efficient maximal exact matching engine (modified from E-MEM) used inside
    Palidis to find inverted terminal repeat sequences in large metagenomic read files
  repo: https://github.com/blue-moon22/pal-MEM
- name: CD-HIT-EST
  role: Clusters candidate ITRs per sample and clusters pooled per-sample IS sequences
    at -c 0.95 to produce the non-redundant catalogue
- name: COBS
  role: Compact bit-sliced signature index of 661,405 microbial genomes queried to
    predict the origins of catalogue ISs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:bb1c947d9d4a9c1a837bce47b361d4776cb88ef0a2bb488e06bb2ae2a9c4cf87
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/palidis4
  tools:
  - CD-HIT-EST
  - COBS
  - Palidis
  - pal-MEM
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# metagenomic-catalogue-construction

## Summary

Build a non-redundant catalogue of insertion sequences (ISs) from mixed microbial community metagenomes by detecting palindromic inverted terminal repeats (ITRs ≥25 nt) with Palidis, then pooling and clustering detections across samples with CD-HIT-EST to emit a reference catalogue like ISC v1.0.0.

## When to use

Use when short-read metagenomic (FASTQ.GZ) data from mixed microbial communities must be mined for novel insertion sequences and consolidated into a non-redundant FASTA catalogue for downstream annotation or genome querying.

## When NOT to use

- The target ISs lack inverted terminal repeats or have ITRs shorter than the 25 nt default (ITRs as short as 10 nt are missed at default settings) — Palidis only detects ISs with ITRs
- ISs are flanked outside the 500–3000 bp proximity range (e.g. regions lengthened by nested insertion events) — these are missed by Palidis
- The input is isolate genomes to annotate rather than mixed-community metagenomic reads — use the released ISC as a reference database instead of rebuilding it

## Inputs

- Paired-end short-read metagenomic FASTQ.GZ files (e.g. Human Microbiome Project oral and stool samples, download links in Data S1)
- Per-sample assembled contigs (FASTA) referenced in the palidis4 manifest
- Tab-delimited manifest with lane_id, read_directory, sample_id, contigs_path headers

## Outputs

- Non-redundant insertion sequence catalogue in FASTA format (e.g. ISC v1.0.0: 879 unique ISs)
- Per-sample IS information table (IS_name, sample_id, contig, is_start, is_end, description with InterPro/PANTHER transposase annotations)
- Transposase annotations (e.g. 87 unique transposases via Interproscan/Prodigal)

## How to apply

Download the raw reads (e.g. the 264 Human Microbiome Project metagenomes via the links in Data S1) and run Palidis v3.1.0 (Nextflow pipeline palidis4) with default parameters on each sample: it pre-processes FASTQ.GZ reads, uses pal-MEM maximal exact matching to find repeats, maps reads to assemblies with Bowtie2, filters candidate ITRs by distance and IS length (min_is_len 500, max_is_len 3000 bp), and clusters candidate ITRs with CD-HIT-EST. Collect the per-sample detections (in the HMP run: 2517 ISs on 1837 contigs across 218 of 264 samples), then pool the per-sample IS FASTA sequences and cluster them with CD-HIT-EST at a sequence identity threshold of -c 0.95 to remove redundancy. Emit the non-redundant catalogue (Insertion Sequence Catalogue v1.0.0 with 879 unique ISs, 87 unique transposases) and optionally predict IS origins by querying against ISfinder or a COBS index of 661,405 microbial genomes. Validate by confirming the catalogue matches the released ISC at github.com/blue-moon22/ISC.

## Related tools

- **Palidis** (Nextflow pipeline that detects ISs by identifying palindromic ITRs (>=25 nt) via maximal exact matching, filters by IS length (500-3000 bp), and clusters candidate ITRs with CD-HIT-EST) — https://github.com/blue-moon22/palidis4
- **pal-MEM** (Efficient maximal exact matching engine (modified from E-MEM) used inside Palidis to find inverted terminal repeat sequences in large metagenomic read files) — https://github.com/blue-moon22/pal-MEM
- **CD-HIT-EST** (Clusters candidate ITRs per sample and clusters pooled per-sample IS sequences at -c 0.95 to produce the non-redundant catalogue)
- **COBS** (Compact bit-sliced signature index of 661,405 microbial genomes queried to predict the origins of catalogue ISs)

## Examples

```
nextflow palidis.nf --manifest manifest.tsv --batch_name hmp_batch -c configs/conf/sanger.config
```

## Evaluation signals

- Catalogue contains exactly 879 unique insertion sequences with 87 unique transposases (matches ISC v1.0.0 released at github.com/blue-moon22/ISC)
- Per-sample detection totals match the published run: 2517 ISs on 1837 contigs across 218 of 264 samples
- 519 of the 879 catalogue ISs are novel (not found in ISfinder or the COBS index of 661,405 microbial genomes)
- Every catalogue entry carries ITRs of >=25 nt and an IS length within the 500-3000 bp window
- palidis4 regression test passes: `./tests/regression_tests.sh` prints 'Test passed. All outputs expected.'

## Limitations

- ISC only contains common DDE types of ISs with ITRs; Palidis cannot discover ISs lacking ITRs
- ITRs as short as ten nucleotides are missed at default settings (minimum ITR length threshold is 25 nt); lowering the threshold and k-mer length is more computationally intensive on complex communities
- ISs lengthened by nested insertion events may fall outside the 500-3000 bp proximity range and be missed; increasing maximum IS length is planned for future ISC iterations
- ISC v1.0.0 was built only from healthy human oral cavity and stool metagenomes, limiting its use as an annotation reference to those sources

## Evidence

- [abstract] Applying Palidis to 264 human metagenomes identifies 879 unique insertion sequences, with 519 being novel and not previously characterized.: "Applying Palidis to 264 human metagenomes identifies 879 unique insertion sequences, with 519 being novel and not previously characterized."
- [readme] Efficient maximal exact matching to get repeat sequences: "2. Efficient maximal exact matching to get repeat sequences using [pal-MEM](https://github.com/blue-moon22/pal-MEM) [`palmem`]"
- [readme] k-mer and length defaults: "--min_is_len        Minimum length of insertion sequence. (Default: 500)  --max_is_len        Maximum length of insertion sequence. (Default: 3000)"
- [discussion] ITR length limitation: "the catalogue currently contains ISs with ITRs that are 25 or greater nucleotides in length as generated by Palidis, although ITRs can be as short as ten nucleotides in length"
- [intro] Pool and cluster identified ISs: "These ISs can then be pooled and clustered to create a non-redundant catalogue of ISs."
