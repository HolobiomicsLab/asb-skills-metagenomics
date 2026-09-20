---
name: cross-tool-retention-comparison
description: Use when you need to quantify and compare false-removal rates (off-target
  loss of bacterial reads) of host-decontamination tools such as Hostile vs HRRT on
  simulated bacterial Illumina data.
when_to_use_negative:
- The goal is decontaminating real (meta)genomic data rather than evaluating or benchmarking
  tool precision — in that case run `hostile clean` directly on the sample FASTQs.
- The primary question is human read removal sensitivity (percent of real human reads
  removed, e.g. the 1000 Genomes Project >99.5% benchmark), not microbial read retention
  on simulated bacterial data.
- Input reads are non-Illumina (e.g. ONT long reads) or not derived from the FDA-ARGOS
  bacterial assemblies, so the published retention figures and ≥99.989% validation
  threshold do not apply.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0091
tools:
- name: Hostile
  role: Decontaminates the simulated paired Illumina reads in short-read mode and
    emits a JSON log (reads_in/reads_out) from which bacterial read retention is computed
  repo: https://github.com/bede/hostile
- name: Bowtie2
  role: Alignment backend Hostile invokes automatically for paired short-read cleaning
    with --sensitive defaults (-D 15 -R 2 -N 0 -L 22 -i S,1,1.15) and -k 1 against
    the human-t2t-hla index
- name: HRRT (sra-human-scrubber)
  role: Comparator host-removal tool run on the same simulated reads; its cleaned
    output provides the retention denominator for the cross-tool fold-difference
  repo: https://github.com/ncbi/sra-human-scrubber
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad728
    title: 'Hostile: accurate decontamination of microbial host sequences'
  source_content_hash: sha256:b85c0ddc0f44407c2532d2e268ea5546aac45007733f1577317251dfae3609eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bede/hostile
  tools:
  - Bowtie2
  - HRRT (sra-human-scrubber)
  - Hostile
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cross-tool-retention-comparison

## Summary

Benchmark how precisely host-read decontamination tools retain microbial sequence by simulating paired Illumina reads from the 985 FDA-ARGOS bacterial assemblies (BioProject 231221) with DWGSIM, cleaning them with Hostile and HRRT, and tabulating bacterial read retention per tool.

## When to use

Use when you need to quantify and compare false-removal rates (off-target loss of bacterial reads) of host-decontamination tools such as Hostile vs HRRT on simulated bacterial Illumina data.

## When NOT to use

- The goal is decontaminating real (meta)genomic data rather than evaluating or benchmarking tool precision — in that case run `hostile clean` directly on the sample FASTQs.
- The primary question is human read removal sensitivity (percent of real human reads removed, e.g. the 1000 Genomes Project >99.5% benchmark), not microbial read retention on simulated bacterial data.
- Input reads are non-Illumina (e.g. ONT long reads) or not derived from the FDA-ARGOS bacterial assemblies, so the published retention figures and ≥99.989% validation threshold do not apply.

## Inputs

- 985 reference grade FDA-ARGOS bacterial genome assemblies (BioProject 231221)
- DWGSIM-simulated paired Illumina FASTQ files (R1/R2) per bacterial assembly
- Hostile JSON clean logs with reads_in, reads_out, reads_removed, reads_removed_proportion fields
- HRRT (sra-human-scrubber) cleaned FASTQ output for the same simulated reads

## Outputs

- Per-tool bacterial read retention values (reads_out/reads_in) for Hostile and HRRT
- Fold-difference in bacterial reads incorrectly removed by HRRT vs Hostile (with 95% CI)
- Tabulated comparison file bacterial_retention_comparison.csv (Hostile 99.99989% vs HRRT 99.99752% in the published run)

## How to apply

Download the 985 reference grade FDA-ARGOS bacterial genomes from BioProject 231221 and simulate paired Illumina reads from each assembly with DWGSIM. Run `hostile clean --fastq1 <r1> --fastq2 <r2>` in paired short-read mode so Bowtie2 aligns against the default `human-t2t-hla` index (T2T-CHM13v2.0 + IPD-IMGT/HLA v3.51) with its `--sensitive` parameters (-D 15 -R 2 -N 0 -L 22 -i S,1,1.15) and -k 1, capturing the JSON log. Run HRRT (ncbi/sra-human-scrubber, `scrub.sh`) on the identical simulated reads. Compute per-tool bacterial retention as reads_out/reads_in using the Hostile JSON log fields (reads_in, reads_out, reads_removed, reads_removed_proportion) and the HRRT output, then save a comparison table (bacterial_retention_comparison.csv). Validate that Hostile retains at least the published ≥99.989% of simulated bacterial reads and report the fold-difference in incorrectly removed reads vs HRRT.

## Related tools

- **Hostile** (Decontaminates the simulated paired Illumina reads in short-read mode and emits a JSON log (reads_in/reads_out) from which bacterial read retention is computed) — https://github.com/bede/hostile
- **Bowtie2** (Alignment backend Hostile invokes automatically for paired short-read cleaning with --sensitive defaults (-D 15 -R 2 -N 0 -L 22 -i S,1,1.15) and -k 1 against the human-t2t-hla index)
- **HRRT (sra-human-scrubber)** (Comparator host-removal tool run on the same simulated reads; its cleaned output provides the retention denominator for the cross-tool fold-difference) — https://github.com/ncbi/sra-human-scrubber

## Examples

```
hostile clean --fastq1 <r1> --fastq2 <r2>
```

## Evaluation signals

- Hostile must retain ≥99.989% of simulated bacterial reads (reads_out/reads_in from the JSON log), matching the published abstract figure
- Expected published result: Hostile 99.99989% retention vs HRRT 99.99752% retention on the same DWGSIM-simulated FDA-ARGOS reads
- HRRT should remove a mean of 21 times (95% CI: 9, 30) more bacterial reads than Hostile in the fold-difference calculation
- Each Hostile run must produce a parseable JSON log whose reads_in equals the DWGSIM read count per assembly
- bacterial_retention_comparison.csv must contain one retention row per tool per assembly and reproduce the reported totals when aggregated

## Limitations

- HRRT masks identified human reads with 'N' by default (rather than dropping them), so read-counting conventions must be harmonized when computing its retention
- The HRRT k-mer database contains k-mers from all Eukaryota human RefSeq records minus non-Eukaryota RefSeq, making it aggressive about identifying human-origin reads
- Earlier Hostile versions could corrupt empty fastq.gz files when no reads remain after decontamination, and index checksum verification was only added in the latest release (issues #20, #24)
- Only the human-t2t-hla and human-t2t-hla-argos985-mycob140 index performances were evaluated in the paper; masked-index variants (≥99.997% retention) exist but were benchmarked separately

## Evidence

- [abstract] retains at least 99.989% of simulated bacterial reads: "removes at least 99.6% of real human reads and retains at least 99.989% of simulated bacterial reads"
- [abstract] 21-43 times fewer bacterial reads than the comparator: "Hostile removes 21%-23% more human short reads and 21-43 times fewer bacterial reads, typically in less time"
- [methods] decontamination throughput used in the benchmark: "bacterial Illumina reads were decontaminated at 32Mbp/s (210k reads/sec) and bacterial ONT reads at 22Mbp/s, using 8 alignment threads"
- [methods] FDA-ARGOS bacterial genome panel used for simulation: "985 reference grade bacterial genomes](https://www.ncbi.nlm.nih.gov/bioproject/231221) including common human pathogens"
- [readme] default index composition cleaned against in the comparison: "The default index `human-t2t-hla` comprises [T2T-CHM13v2.0](https://www.ncbi.nlm.nih.gov/assembly/11828891) and [IPD-IMGT/HLA](https://www.ebi.ac.uk/ipd/imgt/hla/) v3.51"
