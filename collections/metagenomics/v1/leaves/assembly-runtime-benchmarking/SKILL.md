---
name: assembly-runtime-benchmarking
description: Use when a newly proposed assembly or gene-targeting pipeline needs quantitative
  comparison of runtimes and contig outputs against a standard complete-assembly baseline
  on identical sequencing datasets.
when_to_use_negative:
- The goal is the biological discovery itself rather than performance comparison of
  assembly strategies
- Input is already assembled contigs or a finished genome — runtime benchmarking of
  read-to-assembly pipelines is not applicable
- The target genes lack profile HMMs or reference sequences, so no targeted-assembly
  arm can be configured for the benchmark
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: CasCollect
  role: Targeted assembly pipeline being benchmarked (seed generation, VSEARCH read
    expansion, SPAdes assembly, CRISPRCasFinder annotation)
  repo: https://github.com/sandialabs/CasCollect
- name: SPAdes
  role: Complete de novo assembly baseline run on the full trimmed read set per isolate
- name: metaSPAdes
  role: Metagenome-mode baseline assembler used in place of SPAdes for the pooled
    metagenome benchmark
- name: PRICE
  role: Third assembler baseline for runtime and contig-count comparison on the simulated
    metagenome
- name: HMMER
  role: Queries 120 Cas protein family profile HMMs to identify cas genes in contigs
    from all pipelines; runtime component of CasCollect
- name: VSEARCH
  role: Read subset expansion at 95% identity; number of cycles (-cycle) affects targeted-assembly
    runtime
  repo: https://github.com/torognes/vsearch
- name: bbduk
  role: Quality trimming and adapter removal (--trim) applied before both benchmark
    arms
  repo: https://jgi.doe.gov/data-and-tools/bbtools/
- name: Seqtk
  role: Read preparation (stripping) when building the simulated metagenome benchmark
    input
  repo: https://github.com/lh3/seqtk
- name: CRISPRCasFinder
  role: Annotates contigs to verify cas operon and CRISPR array recovery in the benchmark
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:e12bc0921e7353fdca13bb8d939ad1c212a82fe14069db97e68273123e734dab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sandialabs/CasCollect
  tools:
  - CRISPRCasFinder
  - CasCollect
  - HMMER
  - PRICE
  - SPAdes
  - Seqtk
  - VSEARCH
  - bbduk
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# assembly-runtime-benchmarking

## Summary

Benchmarking skill for comparing targeted assembly pipelines (e.g. CasCollect) against conventional complete assembly baselines by wall-clock runtime and assembled contig counts on the same input read sets, optionally including third-party assemblers like PRICE.

## When to use

Use when a newly proposed assembly or gene-targeting pipeline needs quantitative comparison of runtimes and contig outputs against a standard complete-assembly baseline on identical sequencing datasets.

## When NOT to use

- The goal is the biological discovery itself rather than performance comparison of assembly strategies
- Input is already assembled contigs or a finished genome — runtime benchmarking of read-to-assembly pipelines is not applicable
- The target genes lack profile HMMs or reference sequences, so no targeted-assembly arm can be configured for the benchmark

## Inputs

- Paired-end Illumina genomic DNA FASTQ reads for bacterial isolates (e.g. SRR1424625, SRR2939129, SRR5280756)
- Simulated metagenome FASTQ (pooled isolate reads with mouse reads SRR1752459 spiked in, prepared with Seqtk)
- Pipeline parameters (e.g. -cycle, -cpu, -mem, --trim, --meta flags)

## Outputs

- Per-isolate wall-clock runtimes for each pipeline
- Contig counts per pipeline applying the 500 bp size cutoff
- Comparison table (e.g. cascollect_metagenome_benchmark.csv) of runtime and contig counts
- Annotated contigs verifying cas gene and CRISPR array recovery

## How to apply

For each isolate dataset (e.g. E. coli KLY, P. aeruginosa VA-134, S. pyogenes M39 paired-end reads), run the targeted pipeline — CasCollect.py in protein mode with --trim and, for simulated metagenomes, --meta, using -cycle 2 to reduce VSEARCH seed-expansion cycles from the default five — and record wall-clock runtime. Run the baseline as a complete SPAdes (or metaSPAdes for metagenomes) de novo assembly of the same trimmed reads, plus optionally PRICE as a third comparator. Apply the 500 bp contig size cutoff when tallying annotatable contigs for both pipelines, and verify with HMMER/CRISPRCasFinder that all previously known cas genes are recovered from contigs of each pipeline. Compare per-isolate runtimes (e.g. 23.3/17.4, 51.2/33.3, 21.6/31.5 min for targeted/complete) and contig counts (e.g. 19 vs 146, 111 vs 162, 89 vs 44), and for metagenomes expect roughly a 36% runtime reduction (1.6 h vs 2.5 h). Compile results into a summary table (CSV) with per-isolate runtime and contig-count columns.

## Related tools

- **CasCollect** (Targeted assembly pipeline being benchmarked (seed generation, VSEARCH read expansion, SPAdes assembly, CRISPRCasFinder annotation)) — https://github.com/sandialabs/CasCollect
- **SPAdes** (Complete de novo assembly baseline run on the full trimmed read set per isolate)
- **metaSPAdes** (Metagenome-mode baseline assembler used in place of SPAdes for the pooled metagenome benchmark)
- **PRICE** (Third assembler baseline for runtime and contig-count comparison on the simulated metagenome)
- **HMMER** (Queries 120 Cas protein family profile HMMs to identify cas genes in contigs from all pipelines; runtime component of CasCollect)
- **VSEARCH** (Read subset expansion at 95% identity; number of cycles (-cycle) affects targeted-assembly runtime) — https://github.com/torognes/vsearch
- **bbduk** (Quality trimming and adapter removal (--trim) applied before both benchmark arms) — https://jgi.doe.gov/data-and-tools/bbtools/
- **Seqtk** (Read preparation (stripping) when building the simulated metagenome benchmark input) — https://github.com/lh3/seqtk
- **CRISPRCasFinder** (Annotates contigs to verify cas operon and CRISPR array recovery in the benchmark)

## Examples

```
python CasCollect.py -fwd reads_1.fq -rev reads_2.fq -out cascollect_out --trim -cycle 2 -cpu 1 -mem 20
```

## Evaluation signals

- Runtimes of both pipelines are recorded per isolate and fall near reported values: 23.3/17.4 min (E. coli KLY), 51.2/33.3 min (P. aeruginosa VA-134), 21.6/31.5 min (S. pyogenes M39)
- On the simulated metagenome, CasCollect runtime is ~1.6 h vs ~2.5 h for complete assembly (~36% reduction) with contig counts near 723 vs 10467 vs 56 (CasCollect/complete/PRICE)
- Both pipelines recover all previously known cas genes from each of the three isolate genomes (verified via HMMER/CRISPRCasFinder)
- Contig counts are tallied consistently with the 500 bp annotation size cutoff for every pipeline
- Benchmark uses matched inputs: identical trimmed reads and identical CPU/RAM settings (e.g. 1 CPU, 20 Gb defaults) across arms

## Limitations

- The reported isolates showed only 'relatively similar timings' between CasCollect and complete assembly — the near two-orders-of-magnitude speedup claim applies to metagenomic datasets, not small isolate genomes
- The pipeline depends on third-party software (BBTools, Seqtk, FragGeneScan, HMMER, VSEARCH, SPAdes, CRISPRCasFinder) being installed in the user's path; missing tools must be auto-downloaded via Check.py
- No software version numbers for the tools were reported, making exact runtime reproduction environment-dependent

## Evidence

- [results] Runtimes 23.3/17.4 min: "with timings of 23.3/17.4, 51.2/33.3 and 21.6/31.5 min (Supplemental Figure S1B)"
- [results] Five-step pipeline: "The CasCollect pipeline was created from five distinct and optional steps: (i) read filtering, (ii) seed generation, (iii) read subset expansion, (iv) assembly and (v) annotation"
- [results] Contig cutoff: "Our Perl script for automating the annotation had set a size cutoff of 500 bp"
- [abstract] Metagenome speedup: "Benchmarking our targeted assembly pipeline demonstrates significantly improved timing by almost two orders of magnitude compared with conventional assembly and annotation"
- [readme] Required dependencies: "This pipeline relies on python and perl as well as requiring the installation of several pieces of software within the users path"
