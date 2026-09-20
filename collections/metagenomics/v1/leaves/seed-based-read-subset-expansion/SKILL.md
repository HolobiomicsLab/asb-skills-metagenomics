---
name: seed-based-read-subset-expansion
description: Use when raw Illumina paired-end FASTQ reads (isolate genomes or metagenomes)
  must be assembled around cas genes/CRISPR arrays without complete genome or metagenome
  assembly, e.g. when runtime or compute limits full metaSPAdes assembly.
when_to_use_negative:
- Input is already assembled contigs rather than raw unassembled sequencing reads
  — CasCollect is designed for raw FASTQ input
- A complete, unbiased metagenome assembly is required for purposes beyond CRISPR/Cas
  detection (e.g. strain-resolved binning of non-Cas loci)
- Data is not short paired-end Illumina reads (CasCollect targets short DNA sequencing
  reads; read lengths tested were 90–226 bp)
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: CasCollect
  role: Python wrapper pipeline implementing seed generation, read subset expansion,
    targeted assembly and annotation
  repo: https://github.com/sandialabs/CasCollect
- name: HMMER
  role: Identifies seed reads matching the default 120 Cas protein family profile
    HMMs
- name: VSEARCH
  role: Performs read subset expansion by identity matching (95% default, five cycles)
    and seed matching to CRISPR repeats in DNA mode
  repo: https://github.com/torognes/vsearch
- name: metaSPAdes
  role: Assembles the expanded read subset in place of SPAdes for metagenomic data
- name: CRISPRCasFinder
  role: Searches targeted assembly contigs for CRISPR arrays and cas genes during
    annotation
- name: BBTools (bbduk/bbmerge)
  role: Quality trimming, adapter removal, contaminant read removal and overlapping
    read merging as an optional pre-filtering step
  repo: https://jgi.doe.gov/data-and-tools/bbtools/
- name: Seqtk
  role: Read stripping utility used within the pipeline
  repo: https://github.com/lh3/seqtk
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:3512b1c810f84603b6853a0e9ce405c58f12f39bca965b95cedd2960ade1a083
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sandialabs/CasCollect
  tools:
  - BBTools
  - BBTools (bbduk/bbmerge)
  - CRISPRCasFinder
  - CasCollect
  - HMMER
  - PRICE
  - Seqtk
  - VSEARCH
  - bbduk
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# Seed-based read subset expansion

## Summary

CasCollect's targeted assembly strategy: seed reads matching cas genes (via HMMER against 120 Cas protein HMMs) or CRISPR array repeats (via VSEARCH in DNA mode) prime iterative VSEARCH expansion cycles (default five cycles at 95% identity) that pull in overlapping reads, so only CRISPR/Cas loci are assembled instead of the full metagenome.

## When to use

Use when raw Illumina paired-end FASTQ reads (isolate genomes or metagenomes) must be assembled around cas genes/CRISPR arrays without complete genome or metagenome assembly, e.g. when runtime or compute limits full metaSPAdes assembly.

## When NOT to use

- Input is already assembled contigs rather than raw unassembled sequencing reads — CasCollect is designed for raw FASTQ input
- A complete, unbiased metagenome assembly is required for purposes beyond CRISPR/Cas detection (e.g. strain-resolved binning of non-Cas loci)
- Data is not short paired-end Illumina reads (CasCollect targets short DNA sequencing reads; read lengths tested were 90–226 bp)

## Inputs

- Raw paired-end Illumina FASTQ sequencing reads (metagenomic or isolate)
- Default set of 120 Cas protein family profile HMMs (protein mode) or user-defined CRISPR array repeat FASTA sequences (DNA mode)
- Optional contaminant reference genome (default: human genome) for --clean

## Outputs

- Targeted assembled contigs spanning cas operons and/or CRISPR arrays
- Annotated cas genes and CRISPR arrays identified by CRISPRCasFinder on the targeted contigs

## How to apply

Run CasCollect with the --meta flag on metagenomic data (metaSPAdes replaces SPAdes). The pipeline first identifies seed reads using HMMER with the default 120 Cas protein family profile HMMs (protein mode) or VSEARCH against user-supplied CRISPR array repeat FASTAs (DNA mode, 105 repeats included). Seed reads then prime read subset expansion: VSEARCH searches for identity matches to recruit additional reads, with a default of five expansion cycles (-cycle) at 95% identity (-match), and matched reads are removed from the database between cycles to eliminate redundant searches. The expanded read subset is assembled with SPAdes/metaSPAdes, and contigs are annotated and searched with CRISPRCasFinder. This targeted approach required ~1.7 ± 0.3 h versus 50.6 ± 3.0 h for complete assembly plus annotation on four real metagenomes — roughly 3% of the time.

## Related tools

- **CasCollect** (Python wrapper pipeline implementing seed generation, read subset expansion, targeted assembly and annotation) — https://github.com/sandialabs/CasCollect
- **HMMER** (Identifies seed reads matching the default 120 Cas protein family profile HMMs)
- **VSEARCH** (Performs read subset expansion by identity matching (95% default, five cycles) and seed matching to CRISPR repeats in DNA mode) — https://github.com/torognes/vsearch
- **metaSPAdes** (Assembles the expanded read subset in place of SPAdes for metagenomic data)
- **CRISPRCasFinder** (Searches targeted assembly contigs for CRISPR arrays and cas genes during annotation)
- **BBTools (bbduk/bbmerge)** (Quality trimming, adapter removal, contaminant read removal and overlapping read merging as an optional pre-filtering step) — https://jgi.doe.gov/data-and-tools/bbtools/
- **Seqtk** (Read stripping utility used within the pipeline) — https://github.com/lh3/seqtk

## Examples

```
python CasCollect.py -1 reads_R1.fastq -2 reads_R2.fastq --meta -cycle 5 -match 95
```

## Evaluation signals

- Expanded read subset assembles contigs that CRISPRCasFinder identifies as complete cas operons and CRISPR arrays matching those recovered by complete assembly+annotation benchmarks
- Runtime of the targeted pipeline is roughly two orders of magnitude below complete metaSPAdes assembly plus annotation (~1.7 h vs ~50.6 h on the four tested metagenomes)
- On simulated metagenomic data (pooled E. coli KLY, P. aeruginosa VA-134, S. pyogenes M39 with mouse reads spiked in), all known cas genes are identified and cas operons correctly assembled
- Each expansion cycle's matched reads are removed from the database, so repeated VSEARCH searches do not re-report the same reads (no redundant matches across cycles)
- Tuning -match (default 95%) and -cycle (default 5) changes subset size; contigs below the 500 bp annotation cutoff are excluded

## Limitations

- Only reads near seed matches are assembled — loci without cas genes or known CRISPR repeats are not recovered
- DNA mode seed generation depends on user-supplied repeat sequences; isolated/novel repeats may be missed unless provided (105 repeats ship with CasCollect)
- No software version numbers stated for CasCollect or third-party tools in the article; results depend on default settings (1 cpu, 20 Gb RAM in the benchmark)
- Isolate benchmark runtimes could exceed complete assembly in some cases (e.g. 51.2 vs 33.3 min for P. aeruginosa VA-134), so speed gains are largest for large metagenomes

## Evidence

- [results] CasCollect identifies short DNA sequencing reads that match HMM profiles for cas protein genes: "we developed CasCollect. CasCollect identifies short DNA sequencing reads that match HMM profiles for cas protein genes"
- [results] Third-party programs underlying the pipeline: "This pipeline relies on several third-party programs that include: HMMER (21), VSEARCH (22), SPAdes (23) and CRISPRCasFinder (13)"
- [results] Five pipeline steps: "The CasCollect pipeline was created from five distinct and optional steps: (i) read filtering, (ii) seed generation, (iii) read subset expansion, (iv) assembly and (v) annotation"
- [results] Default expansion identity threshold: "The default identity match has been set to a moderately high value of 95%, however, this can be adjusted with the -match command."
- [results] Default number of expansion cycles: "The default number of these read subset expansion cycles is set at five and can be adjusted with the -cycle command."
- [results] Redundancy elimination between cycles: "This process eliminates redundant searches by removing reads from the database set that have been matched in the previous cycle"
- [results] Runtime advantage on real metagenomes: [evidence span withheld — anchor status: not_found]
- [readme] Tool requirements from README: "This pipeline relies on python and perl as well as requiring the installation of several pieces of software within the users path"
