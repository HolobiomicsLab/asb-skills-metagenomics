---
name: pangenome-database-volume-reduction-analysis
description: Use when reproducing or benchmarking GeneMark-HM's efficiency claims,
  specifically the factor-15 database reduction and single-CPU run-time comparisons
  on metagenomic contigs.
when_to_use_negative:
- The goal is gene prediction accuracy evaluation (gene start error rates, TGA codon
  assignment) rather than computational efficiency of the pan-genome database
- The input metagenome is not from the human microbiome — the pipeline is parameterized
  for human microbiome data and pan-genome assignment rates drop for oral (89.0%)
  and skin (79.4%) metagenomes
- Multi-CPU benchmarking is required; the published targets (≈12 min, ~80% in BLASTn)
  are single-CPU conditions
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: GeneMark-HM
  role: Pipeline whose run-time and database memory footprint are being benchmarked
    on the 45 Mb test sequence
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: NCBI BLASTn
  role: Similarity search against the formatted pan-genome database; profiled as the
    dominant (~80%) fraction of GeneMark-HM wall-clock time
- name: MetaGeneMark
  role: Baseline gene predictor run on the same 45 Mb input for wall-clock comparison
    (target ≈10 s)
- name: Perl
  role: Language of the gm_hm.pl pipeline driver script invoked for the benchmark
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqab047
    title: 'GeneMark-HM: improving gene prediction in DNA sequences of human microbiome'
  source_content_hash: sha256:5bbe4a0ef6a2fd0eb2415b3c37d94f8d08cb82ae3571ebeb5f93b2a378c9dfb6
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# pangenome-database-volume-reduction-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks the GeneMark-HM pipeline's computational efficiency: quantifies the volume/memory reduction achieved by species-level pan-genome databases versus non-redundant SGB/tCluster gene sets, and measures wall-clock run-times of GeneMark-HM versus MetaGeneMark and MetaProdigal on a fixed 45 Mb metagenome sequence.

## When to use

Use when reproducing or benchmarking GeneMark-HM's efficiency claims, specifically the factor-15 database reduction and single-CPU run-time comparisons on metagenomic contigs.

## When NOT to use

- The goal is gene prediction accuracy evaluation (gene start error rates, TGA codon assignment) rather than computational efficiency of the pan-genome database
- The input metagenome is not from the human microbiome — the pipeline is parameterized for human microbiome data and pan-genome assignment rates drop for oral (89.0%) and skin (79.4%) metagenomes
- Multi-CPU benchmarking is required; the published targets (≈12 min, ~80% in BLASTn) are single-CPU conditions

## Inputs

- 45 Mb real metagenome test sequence (FASTA)
- GeneMark-HM pipeline from github.com/GeneProbe/GeneMark-HM
- refmod database of model files (http://geneprobe.net/genemark_hm/refmod.tar)
- ~20 Gb pan-genome FASTA from Amazon S3 storage
- Non-redundant SGB/tCluster gene set (for volume comparison baseline)

## Outputs

- runtime_benchmark.csv with measured run-times, BLASTn scanning fraction, and reduction factor
- output.gtf from gm_hm.pl on the 45 Mb test sequence
- Wall-clock times for GeneMark-HM (~12 min), MetaGeneMark (~10 s), MetaProdigal (~3.5 min)
- Pan-genome database volume/memory reduction factor (≈15)

## How to apply

Install GeneMark-HM from github.com/GeneProbe/GeneMark-HM, download the refmod model-file database and the ~20 Gb pan-genome FASTA, and format the pan database with makeblastdb using -parse_seqids and -dbtype nucl. Run gm_hm.pl on the 45 Mb test sequence with --cores 1 (plus --clean and --out output.gtf) and record the wall-clock time; published target is ~12 min. Run MetaGeneMark and MetaProdigal on the same 45 Mb input under single-CPU conditions, targeting published times of ~10 s and ~3.5 min respectively. Profile the GeneMark-HM execution to quantify the fraction of wall-clock time spent in the BLASTn similarity scan against the pan database (target ~80%), then compute the volume and memory reduction of the pan-genome database relative to the non-redundant SGB/tCluster gene set (target factor 15, Table 5). Write all measured values to runtime_benchmark.csv and compare against the published benchmarks.

## Related tools

- **GeneMark-HM** (Pipeline whose run-time and database memory footprint are being benchmarked on the 45 Mb test sequence) — https://github.com/GeneProbe/GeneMark-HM
- **NCBI BLASTn** (Similarity search against the formatted pan-genome database; profiled as the dominant (~80%) fraction of GeneMark-HM wall-clock time)
- **MetaGeneMark** (Baseline gene predictor run on the same 45 Mb input for wall-clock comparison (target ≈10 s))
- **Perl** (Language of the gm_hm.pl pipeline driver script invoked for the benchmark)

## Examples

```
nohup path_to/bin/gm_hm.pl --seq ../sequence.fasta --cores 1 --out output.gtf --clean > logfile &
```

## Evaluation signals

- GeneMark-HM single-CPU wall-clock time on the 45 Mb sequence is approximately 12 minutes
- MetaGeneMark and MetaProdigal complete in ≈10 s and ≈3.5 min respectively on the same input
- Profiled BLASTn scanning accounts for ~80% of total GeneMark-HM run-time
- Computed volume/memory reduction of the pan-genome database versus the non-redundant SGB/tCluster gene set is ≈15, matching Table 5
- runtime_benchmark.csv is populated with all three run-times, the BLASTn fraction, and the reduction factor

## Limitations

- The ~20 Gb pan-genome FASTA download from Amazon S3 can take significant time
- Published benchmarks are single-CPU; multithreading via --cores changes run-time comparisons
- Efficiency results depend on the pan-genome database built from human gut taxa; other metagenomes have lower assignment coverage

## Evidence

- [other] Construction of the pan-genomes reduced the memory footprint necessary for the GeneMark-HM operation by a factor of 15 (Table 5).: [evidence span withheld — anchor status: not_found]
- [readme] Multithreading is supported. Set "--cores" command line option according to available system resources.: [evidence span withheld — anchor status: external_doc]
