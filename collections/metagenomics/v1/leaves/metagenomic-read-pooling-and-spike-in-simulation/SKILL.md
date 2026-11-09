---
name: metagenomic-read-pooling-and-spike-in-simulation
description: Use when benchmarking a targeted assembly or annotation pipeline (e.g.
  CasCollect) against a complete metagenome assembly, and a simulated community with
  known CRISPR/Cas content and background host contamination is needed from raw FASTQ
  reads.
when_to_use_negative:
- The input is an already assembled metagenome or contigs rather than raw paired-end
  FASTQ reads — CasCollect and this pooling workflow operate on unassembled reads.
- The goal is profiling a real environmental metagenome (e.g. the article's ground
  water, lake, permafrost, wetland datasets) rather than a controlled benchmark with
  known ground truth.
- The target gene set is not amenable to HMM-based seed identification and no user-supplied
  HMMs or repeat references are available to reconfigure CasCollect.
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Seqtk
  role: read stripping/preparation for pooling the isolate FASTQs and the mouse spike-in
    into the simulated metagenome
  repo: https://github.com/lh3/seqtk
- name: CasCollect
  role: targeted assembly pipeline benchmarked on the simulated metagenome in protein
    mode with --trim and --meta (HMMER seeds, VSEARCH expansion, metaSPAdes, CRISPRCasFinder)
  repo: https://github.com/sandialabs/CasCollect
- name: BBTools/bbduk
  role: read trimming/cleaning step inside the CasCollect run on the pooled data
  repo: https://jgi.doe.gov/data-and-tools/bbtools/
- name: VSEARCH
  role: seed-expansion identity matching (default 95%, 5 cycles) during CasCollect
    runs on the simulated metagenome
  repo: https://github.com/torognes/vsearch
- name: FragGeneScan-Plus
  role: ORF prediction on short reads in the metagenomic analysis chain referenced
    by the article
  repo: https://github.com/hallamlab/FragGeneScanPlus
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:26aaf7592a3999dcf3495da6c1689a62de173cd4c8d3d07b58bb5511df79ae93
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/seqtk
  tools:
  - BBTools/bbduk
  - CRISPRCasFinder
  - CasCollect
  - FragGeneScan-Plus
  - HMMER
  - PRICE
  - Seqtk
  - VSEARCH
  - bbduk
  - metaSPAdes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# metagenomic-read-pooling-and-spike-in-simulation

## Summary

Constructs a controlled in-silico metagenome by pooling paired-end Illumina isolate read sets (E. coli KLY, P. aeruginosa VA-134, S. pyogenes M39) with spiked-in mouse reads using Seqtk, so that targeted assemblers like CasCollect can be benchmarked against complete assembly baselines with a known ground-truth cas gene content.

## When to use

Use when benchmarking a targeted assembly or annotation pipeline (e.g. CasCollect) against a complete metagenome assembly, and a simulated community with known CRISPR/Cas content and background host contamination is needed from raw FASTQ reads.

## When NOT to use

- The input is an already assembled metagenome or contigs rather than raw paired-end FASTQ reads — CasCollect and this pooling workflow operate on unassembled reads.
- The goal is profiling a real environmental metagenome (e.g. the article's ground water, lake, permafrost, wetland datasets) rather than a controlled benchmark with known ground truth.
- The target gene set is not amenable to HMM-based seed identification and no user-supplied HMMs or repeat references are available to reconfigure CasCollect.

## Inputs

- E. coli KLY paired-end genomic DNA FASTQ reads (SRR1424625, 90 bp)
- P. aeruginosa VA-134 paired-end genomic DNA FASTQ reads (SRR2939129, 100 bp)
- S. pyogenes M39 paired-end genomic DNA FASTQ reads (SRR5280756, 226 bp)
- mouse sequencing FASTQ reads for spike-in (SRR1752459)
- 120 Cas protein family profile HMMs shipped with CasCollect

## Outputs

- pooled simulated metagenome FASTQ (interleaved or paired-end) with spiked-in mouse reads
- CasCollect targeted assembly contigs with CRISPRCasFinder cas/CRISPR annotations
- comparison table of runtimes and contig counts (e.g. cascollect_metagenome_benchmark.csv: 1.6 h / 723 contigs vs 2.5 h / 10467 vs 56)

## How to apply

Pool the paired-end bacterial isolate genomic read sets (E. coli KLY, P. aeruginosa VA-134, S. pyogenes M39, with reported read lengths of 90, 100 and 226 bp) and spike in mouse sequencing reads as contamination, preparing/normalizing reads with Seqtk. Run CasCollect in protein mode with --trim and --meta on the pooled metagenome (default 120 Cas protein HMMs queried by HMMER, 5 VSEARCH seed-expansion cycles at 95% identity, metaSPAdes assembly, CRISPRCasFinder annotation) and record wall-clock runtime and contig counts. Run a complete metaSPAdes assembly of the identical pooled input as the full-assembly baseline, and optionally a PRICE run, applying the same 500 bp contig cutoff before annotation. Acceptance is judged by approximate recovery of the reported benchmarks (CasCollect 1.6 h vs 2.5 h complete assembly, ~36% runtime reduction; 723 vs 10467 vs 56 contigs) and by HMMER/CRISPRCasFinder confirmation that all previously known cas genes from the three source isolates are recovered and cas operons are correctly assembled.

## Related tools

- **Seqtk** (read stripping/preparation for pooling the isolate FASTQs and the mouse spike-in into the simulated metagenome) — https://github.com/lh3/seqtk
- **CasCollect** (targeted assembly pipeline benchmarked on the simulated metagenome in protein mode with --trim and --meta (HMMER seeds, VSEARCH expansion, metaSPAdes, CRISPRCasFinder)) — https://github.com/sandialabs/CasCollect
- **BBTools/bbduk** (read trimming/cleaning step inside the CasCollect run on the pooled data) — https://jgi.doe.gov/data-and-tools/bbtools/
- **VSEARCH** (seed-expansion identity matching (default 95%, 5 cycles) during CasCollect runs on the simulated metagenome) — https://github.com/torognes/vsearch
- **FragGeneScan-Plus** (ORF prediction on short reads in the metagenomic analysis chain referenced by the article) — https://github.com/hallamlab/FragGeneScanPlus

## Examples

```
seqtk sample -s100 read1.fq 10000 > sub1.fq
```

## Evaluation signals

- CasCollect runtime on the pooled metagenome approximates the reported 1.6 h and complete metaSPAdes approximates 2.5 h (~36% reduction).
- Contig counts approximate the reported values: 723 (CasCollect), 10467 (complete assembly), 56 (PRICE) at the 500 bp annotation cutoff.
- HMMER/CRISPRCasFinder annotation recovers all previously known cas genes from all three source isolates (E. coli KLY, P. aeruginosa VA-134, S. pyogenes M39).
- Cas operons are assembled as intact, correctly annotated contigs rather than fragmented cas gene hits.
- CasCollect software dependency check (Check.py) reports all required tools (BBTools, Seqtk, FragGeneScan, HMMER, VSEARCH, SPAdes, CRISPRCasFinder) present in path before the run.

## Limitations

- This is a simulated, low-complexity community (three isolates plus mouse spike-in), so runtime and contig-count gains (~36% here) may not transfer to complex real metagenomes where the article reports near two-orders-of-magnitude improvement for cas-gene identification/annotation timing.
- No software version numbers or accession IDs are given for the tools or datasets, so exact reproducibility of timings is not guaranteed.
- CasCollect expects raw Illumina reads and depends on seven third-party programs being installed in the user's path; missing software must be fetched and installed before the pipeline runs.

## Evidence

- [results] Pooling of reads to build simulated metagenome: "we pooled together the E. coli KLY, P. aeruginosa VA-134 and S. pyogenes M39 bacterial isolate genome sequencing reads. Mouse sequencing reads were spiked in"
- [readme] Seqtk role in read preparation: [evidence span withheld — anchor status: not_found]
- [readme] Seqtk subsampling invocation: [evidence span withheld — anchor status: external_doc]
- [readme] CasCollect operates on unassembled reads: "CasCollect is a pipeline for the detection of Cas genes and CRISPR arrays from unassembled raw illumina sequencing reads."
- [results] Simulated metagenome benchmark results: "Both CasCollect and complete assembly pipelines were able to identify all known previously identified cas genes within each of the three bacterial isolate genomes."
