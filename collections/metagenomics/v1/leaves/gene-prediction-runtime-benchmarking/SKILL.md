---
name: gene-prediction-runtime-benchmarking
description: Use when comparing computational cost or database footprint of gene finders
  on metagenomic contigs, or when reproducing the GeneMark-HM efficiency results (12
  min on 45 Mb, ~80% in BLASTn, 15x pan-genome reduction).
when_to_use_negative:
- The goal is gene prediction accuracy (sensitivity/specificity of starts, TGA codon
  roles) rather than computational cost — use accuracy reproduction workflows instead.
- Input is not a human microbiome metagenome; the pan-genome database and pipeline
  parameters are optimized for human microbiome contigs.
- Multi-CPU or cluster throughput benchmarking is required — published targets (GeneMark-HM
  ~12 min, MetaGeneMark ~10 s, MetaProdigal ~3.5 min) were measured single-CPU.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: GeneMark-HM
  role: Pipeline being benchmarked; run via gm_hm.pl on the 45 Mb test sequence with
    --cores 1
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: MetaGeneMark
  role: Comparison gene finder run on the same input with published target run-time
    of ~10 s
- name: NCBI BLASTn
  role: Similarity scan against the pan-genome database; makeblastdb builds the index
    and profiling attributes ~80% of run-time to this step
- name: Perl
  role: Runtime dependency of the gm_hm.pl driver (Perl 5.10 with Parallel::ForkManager)
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqab047
    title: 'GeneMark-HM: improving gene prediction in DNA sequences of human microbiome'
  source_content_hash: sha256:0373054d9bf0ebadb5343a409074357d5d90bd2496f5ec7c1263a06449f17330
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/GeneProbe/GeneMark-HM
  repo_url: https://github.com/GeneProbe/GeneMark-HM
  tools:
  - GeneMark-HM
  - MetaGeneMark
  - NCBI BLASTn
  - Perl
  - compp
  - makeblastdb
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# gene-prediction-runtime-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks wall-clock run-times of gene prediction pipelines (GeneMark-HM vs MetaGeneMark and MetaProdigal) on a fixed metagenomic sequence under identical single-CPU conditions, and quantifies the volume/memory reduction achieved by species-level pan-genome databases (factor ~15, Table 5).

## When to use

Use when comparing computational cost or database footprint of gene finders on metagenomic contigs, or when reproducing the GeneMark-HM efficiency results (12 min on 45 Mb, ~80% in BLASTn, 15x pan-genome reduction).

## When NOT to use

- The goal is gene prediction accuracy (sensitivity/specificity of starts, TGA codon roles) rather than computational cost — use accuracy reproduction workflows instead.
- Input is not a human microbiome metagenome; the pan-genome database and pipeline parameters are optimized for human microbiome contigs.
- Multi-CPU or cluster throughput benchmarking is required — published targets (GeneMark-HM ~12 min, MetaGeneMark ~10 s, MetaProdigal ~3.5 min) were measured single-CPU.

## Inputs

- 45 Mb real metagenome test sequence (FASTA)
- GeneMark-HM pipeline installed with refmod.tar model library (~/.gmhmmp2_key)
- ~20 Gb pan-genome FASTA (pan.fasta) formatted as a nucleotide BLAST database
- MetaGeneMark and MetaProdigal executables

## Outputs

- runtime_benchmark.csv with measured wall-clock times, BLASTn scanning fraction, and database reduction factor
- output.gtf gene predictions from gm_hm.pl
- Per-step timing logs from verbose pipeline execution

## How to apply

Install GeneMark-HM from github.com/GeneProbe/GeneMark-HM, download the model-file database (refmod.tar) and the ~20 Gb pan-genome FASTA, and build the BLAST index with makeblastdb -parse_seqids -dbtype nucl. Run gm_hm.pl on a fixed 45 Mb metagenome test sequence with --cores 1 and record wall-clock time (published target ~12 min), then run MetaGeneMark (~10 s) and MetaProdigal (~3.5 min) on the same input under the same single-CPU conditions. Profile the GeneMark-HM execution to attribute the fraction of run-time spent in BLASTn similarity scanning against the pan database (published target ~80%). Finally compute the volume/memory reduction of the constructed pan-genome database relative to the non-redundant SGB/tCluster gene set, which the paper reports as a factor of 15 (Table 5), and write all measurements to a runtime_benchmark.csv.

## Related tools

- **GeneMark-HM** (Pipeline being benchmarked; run via gm_hm.pl on the 45 Mb test sequence with --cores 1) — https://github.com/GeneProbe/GeneMark-HM
- **MetaGeneMark** (Comparison gene finder run on the same input with published target run-time of ~10 s)
- **NCBI BLASTn** (Similarity scan against the pan-genome database; makeblastdb builds the index and profiling attributes ~80% of run-time to this step)
- **Perl** (Runtime dependency of the gm_hm.pl driver (Perl 5.10 with Parallel::ForkManager))

## Examples

```
nohup path_to/bin/gm_hm.pl --seq ../sequence.fasta --cores 1 --out output.gtf --clean > logfile &
```

## Evaluation signals

- GeneMark-HM wall-clock time on the 45 Mb sequence is ≈12 min with --cores 1
- MetaGeneMark and MetaProdigal run in ≈10 s and ≈3.5 min respectively on the same input
- Profiling shows ~80% of GeneMark-HM run-time spent in BLASTn scanning against the pan database
- Pan-genome database volume/memory reduction vs the non-redundant SGB/tCluster gene set is ≈15 (Table 5)
- runtime_benchmark.csv exists and contains all measured run-times, the BLASTn fraction, and the reduction factor

## Limitations

- Published timings are single-CPU wall-clock times; multithreaded runs (--cores > 1) will not match them.
- The ~20 Gb pan-genome download can take significant time and the pipeline creates temporary working folders unless --clean is used.
- Results depend on the reference pan-genome database quality; the reduction factor is specific to the human microbiome pan-genome construction and may not transfer to other taxonomic scopes.

## Evidence

- [results] Construction of the pan-genomes reduced the memory footprint necessary for the GeneMark-HM operation by a factor of 15 (Table 5).: [evidence span withheld — anchor status: not_found]
- [readme] Multithreading is supported. Set "--cores" command line option according to available system resources.: [evidence span withheld — anchor status: external_doc]
