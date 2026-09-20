---
name: chimera-sequence-filtering
description: Use when processing raw 16S/ITS amplicon reads (e.g. 454 or Illumina
  FASTQ) into OTU tables and the dataset may contain chimeric sequences — e.g. synthetic
  benchmark sets with 20% chimeras — so chimeras must be excluded before clustering.
when_to_use_negative:
- Input is already a filtered OTU abundance table or representative-sequence FASTA
  — chimera removal has already been applied upstream
- The clustering strategy is closed-reference against a curated database that already
  excludes chimeric references rather than de-novo OTU inference
- Reads have not been quality filtered or dereplicated — run the expected-error (EE)
  based quality filtering and 100%-identity dereplication first, as chimeric de-novo
  checks assume abundant unique templates
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0080
tools:
- name: MICCA
  role: Pipeline whose de-novo OTU workflow performs UCHIME-based de-novo chimera
    removal on dereplicated reads before greedy clustering
  repo: https://github.com/compmetagen/micca
- name: VSEARCH
  role: Backend engine wrapped by MICCA's filter and otu commands to execute the sequence
    filtering steps
- name: UPARSE
  role: Alternative pipeline used as a benchmark for chimera-retention rates (15.6%
    and 15.5% on 16S-10 and ITS-10)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/srep09743
    title: 'MICCA: a complete and accurate software for taxonomic profiling of metagenomic
      data'
  source_content_hash: sha256:f4cfcba922b0fa749b33d24e25232f173f230307b86c1644b380903d74cfadc6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/compmetagen/micca
  tools:
  - MICCA
  - UPARSE
  - VSEARCH
  - swarm
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# chimera-sequence-filtering

## Summary

De-novo chimera removal step in an amplicon OTU pipeline (as implemented in MICCA with UCHIME in de-novo mode), applied to dereplicated 16S reads so that chimeric templates are discarded before greedy OTU clustering.

## When to use

Use when processing raw 16S/ITS amplicon reads (e.g. 454 or Illumina FASTQ) into OTU tables and the dataset may contain chimeric sequences — e.g. synthetic benchmark sets with 20% chimeras — so chimeras must be excluded before clustering.

## When NOT to use

- Input is already a filtered OTU abundance table or representative-sequence FASTA — chimera removal has already been applied upstream
- The clustering strategy is closed-reference against a curated database that already excludes chimeric references rather than de-novo OTU inference
- Reads have not been quality filtered or dereplicated — run the expected-error (EE) based quality filtering and 100%-identity dereplication first, as chimeric de-novo checks assume abundant unique templates

## Inputs

- Preprocessed/dereplicated amplicon reads in FASTA/FASTQ with per-unique-sequence abundances (e.g. 16S-10 dataset reads)
- Abundance-sorted unique-sequence file produced by exact string matching dereplication at 100% identity

## Outputs

- Non-chimeric unique-sequence set (chimeric templates discarded) feeding greedy OTU clustering
- Final OTU abundance table (otu_table.csv) and representative centroid sequences (rep_seqs.fasta) with chimeric reads excluded

## How to apply

Dereplicate the preprocessed reads at 100% sequence identity first, counting the abundance of each unique sequence, since MICCA starts clustering from the most abundant reads individuated after the dereplication step. Then run chimera detection in UCHIME de-novo mode on the dereplicated sequences and discard chimeric templates; in MICCA this is handled by its filtering/OTU commands which wrap VSEARCH. Validate on a benchmark such as the 16S-10 dataset (454 reads with a realistic error model including 20% chimera sequences) by confirming chimeric reads are excluded from the final OTU abundance table and centroid FASTA. Judge success by comparing retained-chimera rates and OTU-count convergence against other pipelines: UPARSE retained 15.6% and 15.5% chimeric reads on 16S-10 and ITS-10, while MICCA converged to ~77% of the real OTU number.

## Related tools

- **MICCA** (Pipeline whose de-novo OTU workflow performs UCHIME-based de-novo chimera removal on dereplicated reads before greedy clustering) — https://github.com/compmetagen/micca
- **VSEARCH** (Backend engine wrapped by MICCA's filter and otu commands to execute the sequence filtering steps)
- **UPARSE** (Alternative pipeline used as a benchmark for chimera-retention rates (15.6% and 15.5% on 16S-10 and ITS-10))

## Examples

```
micca filter -i derep_seqs.fasta -o nonchimera_seqs.fasta
```

## Evaluation signals

- On the 16S-10 benchmark (20% chimeric sequences), the final OTU table must contain no (or minimal) chimeric reads
- Estimated OTU count should converge to approximately 77% of the real number of OTUs, consistent with MICCA's reported behaviour on 16S-10
- Per-OTU abundances in otu_table.csv must match per-cluster read counts of the greedy clustering seeded from the most-abundant dereplicated reads
- Chimera-retention rate should be substantially lower than UPARSE's reported 15.6% (16S-10) and 15.5% (ITS-10)
- rep_seqs.fasta must contain exactly one centroid sequence per cluster and no discarded chimeric templates

## Limitations

- Validation relies on simulated datasets with known OTU composition (16S-10, ITS-10, 16S-R); the article notes that the lack of official and well-studied benchmark datasets makes it generally difficult to assess pipeline accuracy
- De-novo chimera filtering requires prior dereplication; applied to raw non-dereplicated reads it will not behave as reported
- Even the best pipelines discard real signal: MICCA converges to only ~77% of the real OTU number, so absence of chimeras does not imply complete OTU recovery

## Evidence

- [results] UPARSE obtained the worst results in terms of number of chimeric reads retained on both 16S-10 and ITS-10 datasets: "UPARSE obtained the worst results in terms of number of chimeric reads retained on both 16S-10 and ITS-10 datasets (15.6% and 15.5% respectively)"
- [results] Benchmark datasets contain 20% chimeras: "we have analysed the 16S-10 and ITS-10 datasets generated using a realistic error model for 454 reads and including 20% of chimera sequences"
- [discussion] Dereplication precedes clustering via most-abundant reads: "starting the clustering process from the most abundant reads individuated after the dereplication step"
- [results] MICCA-FAST dereplication: "we show the number of OTUs estimated by MICCA and MICCA-FAST, with the latter implementing an exact string matching dereplication algorithm"
- [readme] VSEARCH backend for MICCA filtering/OTU commands: "VSEARCH (doi: 10.7717/peerj.2584) used in ``classify``, ``filter``, ``mergepairs``, ``otu`` and ``msa`` commands"
