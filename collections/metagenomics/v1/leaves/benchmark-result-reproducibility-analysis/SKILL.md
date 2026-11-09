---
name: benchmark-result-reproducibility-analysis
description: Use when validating or reproducing published computational-efficiency
  claims for a metagenomic gene-prediction pipeline — specifically wall-clock timing
  on a fixed-length test sequence, time-in-scan profiling, and database size reduction
  factors.
when_to_use_negative:
- The goal is accuracy evaluation of gene prediction (start/stop errors, TGA coding)
  rather than computational-efficiency reproduction — use an annotation-accuracy protocol
  instead
- Input is not a human-microbiome metagenome assembly; pan-genome assignment rates
  and databases are specific to human gut/oral/skin metagenomes
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: GeneMark-HM
  role: Pipeline whose run-time, BLASTn scanning fraction, and pan-genome database
    footprint are benchmarked; invoked via gm_hm.pl
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: MetaGeneMark
  role: Reference gene predictor run on the same 45 Mb input under single-CPU conditions
    as the ≈10 s published benchmark comparison
- name: NCBI BLASTn
  role: Similarity-search backend against the pan-genome database whose fraction of
    total run-time (~80%) and database size reduction (factor 15) are quantified;
    database built with makeblastdb
- name: MetaProdigal
  role: Secondary reference predictor timed on the same input against the ≈3.5 min
    published benchmark
- name: Perl
  role: Language of the gm_hm.pl pipeline driver script being executed and timed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqab047
    title: 'GeneMark-HM: improving gene prediction in DNA sequences of human microbiome'
  source_content_hash: sha256:dee0f0e9f4c42e137222a173a2843bd904d59d5d6e1262bc3b47b9eb1563be2d
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# benchmark-result-reproducibility-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce published performance benchmarks for the GeneMark-HM gene prediction pipeline by measuring single-CPU wall-clock run-times, profiling the BLASTn similarity-scanning fraction, and computing the pan-genome database volume/memory reduction factor against stated published targets.

## When to use

Use when validating or reproducing published computational-efficiency claims for a metagenomic gene-prediction pipeline — specifically wall-clock timing on a fixed-length test sequence, time-in-scan profiling, and database size reduction factors.

## When NOT to use

- The goal is accuracy evaluation of gene prediction (start/stop errors, TGA coding) rather than computational-efficiency reproduction — use an annotation-accuracy protocol instead
- Input is not a human-microbiome metagenome assembly; pan-genome assignment rates and databases are specific to human gut/oral/skin metagenomes

## Inputs

- 45 Mb real metagenome test sequence in FASTA format
- refmod database of GeneMark model files (http://geneprobe.net/genemark_hm/refmod.tar)
- ~20 Gb pan-genome sequence FASTA from the GeneMark-HM distribution (Amazon S3)
- Installed GeneMark-HM pipeline, MetaGeneMark, and MetaProdigal executables

## Outputs

- runtime_benchmark.csv containing measured run-times, BLASTn scanning fraction, and database reduction factor
- GeneMark-HM prediction output (output.gtf, aa.fasta, nt.fasta) from the timing run

## How to apply

Install the GeneMark-HM pipeline from github.com/GeneProbe/GeneMark-HM, download the refmod model-file database and the ~20 Gb pan-genome FASTA, and format the pan-genome with makeblastdb (-parse_seqids, -dbtype nucl). Run gm_hm.pl on the 45 Mb real metagenome test sequence with --cores 1 and --clean, recording wall-clock time; repeat under identical single-CPU conditions with MetaGeneMark and MetaProdigal. Profile the GeneMark-HM execution to quantify the fraction of run-time spent in BLASTn scanning against the pan-genome database, and compute the volume and memory reduction of the pan-genome database relative to the non-redundant SGB/tCluster gene set. Compare all measurements against the published targets (MetaGeneMark ≈10 s, MetaProdigal ≈3.5 min, GeneMark-HM ≈12 min, ~80% in BLASTn scanning, reduction factor ≈15) and write them to runtime_benchmark.csv.

## Related tools

- **GeneMark-HM** (Pipeline whose run-time, BLASTn scanning fraction, and pan-genome database footprint are benchmarked; invoked via gm_hm.pl) — https://github.com/GeneProbe/GeneMark-HM
- **MetaGeneMark** (Reference gene predictor run on the same 45 Mb input under single-CPU conditions as the ≈10 s published benchmark comparison)
- **NCBI BLASTn** (Similarity-search backend against the pan-genome database whose fraction of total run-time (~80%) and database size reduction (factor 15) are quantified; database built with makeblastdb)
- **MetaProdigal** (Secondary reference predictor timed on the same input against the ≈3.5 min published benchmark)
- **Perl** (Language of the gm_hm.pl pipeline driver script being executed and timed)

## Examples

```
nohup path_to/bin/gm_hm.pl --seq ../sequence.fasta --cores 1 --aa aa.fasta --nt nt.fasta --out output.gtf --clean > logfile &
```

## Evaluation signals

- Measured MetaGeneMark run-time ≈10 s on the 45 Mb input with --cores 1
- Measured MetaProdigal run-time ≈3.5 min under the same single-CPU conditions
- Measured GeneMark-HM run-time ≈12 min, with ~80% of wall-clock time attributed to BLASTn scanning against the pan-genome database
- Computed pan-genome database volume/memory reduction relative to the non-redundant SGB/tCluster gene set equals a factor of ≈15 (cf. Table 5)
- runtime_benchmark.csv contains all measured run-times, scanning fraction, and reduction factor with values matching published targets

## Limitations

- Benchmarks are single-CPU wall-clock measurements; multithreading (e.g. --cores 4) changes run-times and must be disabled for comparison to published targets
- The ~20 Gb pan-genome database download can take considerable time and memory footprint depends on the pan-genome construction relative to the non-redundant SGB/tCluster gene set
- Results depend on external tables (e.g. Table 5) not contained inline, and no version numbers were stated for GeneMark-HM, GeneMarkS-2, MetaGeneMark, or MetaProdigal
- Timing comparisons assume the test sequence is a real metagenome assembly; the pipeline is validated for human microbiome contigs (up to 99.4% pan-genome assignment in gut, lower in oral/skin)

## Evidence

- [other] Construction of the pan-genomes reduced the memory footprint necessary for the GeneMark-HM operation by a factor of 15 (Table 5).: [evidence span withheld — anchor status: not_found]
- [readme] pan-genome download size: "Pan-genome file is ~20 Gb in size and download can take some time."
- [readme] install instructions: "git clone https://github.com/GeneProbe/GeneMark-HM.git"
- [abstract] similarity search step: "Genes initially predicted in each contig served as queries for a fast similarity search through the pan-genome database."
- [readme] pipeline invocation: [evidence span withheld — anchor status: external_doc]
