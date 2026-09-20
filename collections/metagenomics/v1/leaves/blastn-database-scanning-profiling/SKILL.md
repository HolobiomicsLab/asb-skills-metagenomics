---
name: blastn-database-scanning-profiling
description: Use when benchmarking GeneMark-HM run-time on a real metagenome FASTA
  (~45 Mb) with one CPU, quantifying the BLASTn fraction of wall-clock time against
  the pan database, or measuring the volume/memory reduction of species-level pan-genomes
  vs. the non-redundant SGB/tCluster gene set.
when_to_use_negative:
- Input metagenomes are not from the human microbiome — the pan-genome database was
  built specifically for human gut, oral, and skin microbiome species, and assignment
  coverage drops to 89.0% (oral) and 79.4% (skin).
- You need gene predictions rather than computational-efficiency measurements — this
  skill benchmarks run-time and database size, it does not evaluate annotation accuracy
  (use the simulated-metagenome accuracy workflow instead).
- The pipeline is run multi-threaded — published run-time targets (~12 min vs MetaGeneMark
  ~10 s and MetaProdigal ~3.5 min) hold only under single-CPU conditions.
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: GeneMark-HM
  role: Pipeline being profiled; its BLASTn similarity scan against the pan-genome
    database is the step whose wall-clock fraction is measured
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: NCBI BLASTn
  role: Similarity search engine used inside the pipeline to scan initial gene predictions
    against the pan-genome gene database; profiling isolates its time share
- name: MetaGeneMark
  role: Single-CPU run-time baseline (~10 s on the 45 Mb test sequence) for the efficiency
    comparison
- name: MetaProdigal
  role: Single-CPU run-time baseline (~3.5 min on the 45 Mb test sequence) for the
    efficiency comparison
- name: Perl
  role: Language of the gm_hm.pl pipeline entry script invoked for the benchmark
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqab047
    title: 'GeneMark-HM: improving gene prediction in DNA sequences of human microbiome'
  source_content_hash: sha256:8da7b94cefeca4191481048eadb1a9068f98028ba6ea88efb6f853cd85dd788b
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
  - MetaProdigal
  - NCBI BLASTn
  - Perl
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# blastn-database-scanning-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Profile a GeneMark-HM run to quantify how much of its wall-clock time is spent in the BLASTn similarity scan against the ~20 Gb human microbiome pan-genome database, and benchmark single-CPU run-times against MetaGeneMark and MetaProdigal on a 45 Mb test metagenome. This grounds the paper's computational-efficiency claim that pan-genome construction reduces the GeneMark-HM model database volume/memory by a factor of ~15 (Table 5).

## When to use

Use when benchmarking GeneMark-HM run-time on a real metagenome FASTA (~45 Mb) with one CPU, quantifying the BLASTn fraction of wall-clock time against the pan database, or measuring the volume/memory reduction of species-level pan-genomes vs. the non-redundant SGB/tCluster gene set.

## When NOT to use

- Input metagenomes are not from the human microbiome — the pan-genome database was built specifically for human gut, oral, and skin microbiome species, and assignment coverage drops to 89.0% (oral) and 79.4% (skin).
- You need gene predictions rather than computational-efficiency measurements — this skill benchmarks run-time and database size, it does not evaluate annotation accuracy (use the simulated-metagenome accuracy workflow instead).
- The pipeline is run multi-threaded — published run-time targets (~12 min vs MetaGeneMark ~10 s and MetaProdigal ~3.5 min) hold only under single-CPU conditions.

## Inputs

- Real metagenome assembly FASTA (~45 Mb test sequence)
- Pan-genome sequences database (~20 Gb FASTA, from Amazon S3 storage)
- refmod database of GeneMark.hmm-2 model files
- Non-redundant SGB/tCluster gene set (reference for the reduction-factor comparison)

## Outputs

- runtime_benchmark.csv with wall-clock times for GeneMark-HM, MetaGeneMark, MetaProdigal
- Measured BLASTn scanning fraction of GeneMark-HM wall-clock time (~80%)
- Pan-genome database volume/memory reduction factor relative to SGB/tCluster gene set (Table 5)
- output.gtf gene predictions from the benchmark run

## How to apply

Clone github.com/GeneProbe/GeneMark-HM, download the refmod model-file database and the ~20 Gb pan-genome FASTA, and format the pan-genome with makeblastdb using -parse_seqids and -dbtype nucl. Run gm_hm.pl on the 45 Mb real metagenome test sequence with --cores 1 and record wall-clock time; run the same input through MetaGeneMark and MetaProdigal under identical single-CPU conditions for comparison against published targets of ~10 s and ~3.5 min. Profile the GeneMark-HM execution to determine the fraction of wall-clock time consumed by the BLASTn similarity scan against the pan database, where the model for GeneMark.hmm-2 is selected from initial gene predictions per contig; the published target is ~80%. Finally, compute the pan-genome database volume and memory footprint relative to the non-redundant SGB/tCluster gene set and compare to the published reduction factor of ~15. Write measured run-times, BLASTn fraction, and reduction factor to runtime_benchmark.csv.

## Related tools

- **GeneMark-HM** (Pipeline being profiled; its BLASTn similarity scan against the pan-genome database is the step whose wall-clock fraction is measured) — https://github.com/GeneProbe/GeneMark-HM
- **NCBI BLASTn** (Similarity search engine used inside the pipeline to scan initial gene predictions against the pan-genome gene database; profiling isolates its time share)
- **MetaGeneMark** (Single-CPU run-time baseline (~10 s on the 45 Mb test sequence) for the efficiency comparison)
- **MetaProdigal** (Single-CPU run-time baseline (~3.5 min on the 45 Mb test sequence) for the efficiency comparison)
- **Perl** (Language of the gm_hm.pl pipeline entry script invoked for the benchmark)

## Examples

```
nohup path_to/bin/gm_hm.pl --seq ../sequence.fasta --cores 1 --out output.gtf --verbose > logfile &
```

## Evaluation signals

- GeneMark-HM wall-clock time on the 45 Mb test sequence with --cores 1 is ~12 min, vs MetaGeneMark ~10 s and MetaProdigal ~3.5 min.
- Profiled BLASTn scanning fraction of GeneMark-HM wall-clock time is ~80%.
- Computed pan-genome database volume/memory reduction vs the non-redundant SGB/tCluster gene set is a factor of ~15, matching Table 5.
- Pan-genome database formats cleanly with makeblastdb -parse_seqids -dbtype nucl on the ~20 Gb FASTA.
- All measured values are written to runtime_benchmark.csv with one row per tool.

## Limitations

- The ~20 Gb pan-genome FASTA must be downloaded from Amazon S3 and 'download can take some time'; run-time benchmarks cannot start before both refmod and pan databases are installed.
- Pipeline accuracy and assignment-dependent behavior degrade when contigs cannot be assigned to a reference pan-genome; gut coverage reaches 99.4% but oral and skin metagenomes only reach 89.0% and 79.4%.
- No version numbers are stated for GeneMark-HM, GeneMark.hmm-2, GeneMarkS-2, MetaGeneMark, or MetaProdigal, so benchmark values are tied to the January 2021 release.

## Evidence

- [abstract] initial gene predictions scanned via BLASTn against pan database: "Genes initially predicted in each contig served as queries for a fast similarity search through the pan-genome database."
- [methods] pan-genome database size and download cost: "Pan-genome file is ~20 Gb in size and download can take some time."
- [readme] two databases core to the pipeline and taxa assignment step: [evidence span withheld — anchor status: external_doc]
- [readme] single-CPU benchmark invocation via gm_hm.pl with --cores option: [evidence span withheld — anchor status: external_doc]
