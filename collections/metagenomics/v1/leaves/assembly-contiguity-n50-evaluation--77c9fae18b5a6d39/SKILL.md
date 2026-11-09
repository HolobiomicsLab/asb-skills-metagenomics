---
name: assembly-contiguity-n50-evaluation
description: Use when de novo assembled 10x Linked-Read (Supernova2) assemblies must
  be compared across parameter settings (total coverage C, physical coverage CF, read
  coverage per CR) using N50 and NA50 contiguity/accuracy metrics.
when_to_use_negative:
- Assemblies from long-read technologies (PacBio, ONT) or non-barcode data where the
  10x parameter space and NA50 reference-based framing do not apply
- No high-quality reference genome is available and only N50 (which the authors note
  is misleading) can be computed, precluding accuracy-aware NA50 comparison
- The goal is variant calling benchmarking (e.g. against the GIAB benchmark) rather
  than assembly contiguity/accuracy assessment
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3673
tools:
- name: Supernova2
  role: Produces the diploid de novo assemblies whose contig/scaffold N50 and NA50
    are evaluated
- name: LRTK-SIM
  role: Generates simulated Linked-Read FASTQ libraries across the C/CF/CR parameter
    sweep used to build the assemblies being evaluated
  repo: https://github.com/zhanglu295/LRTK-SIM
- name: Evaluate_diploid_assembly
  role: Jupyter notebook workflow (Evaluate_assembly_performance.ipynb) for computing
    and comparing assembly evaluation metrics
  repo: https://github.com/zhanglu295/Evaluate_diploid_assembly
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giz141
    title: Assessment of human diploid genome assembly with 10x Linked-Reads data
  source_content_hash: sha256:729d0434b717c66095403b6d4fe46358c52af62d4d7ccc9c0f61cae8bd5a159d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhanglu295/LRTK-SIM
  tools:
  - Evaluate_diploid_assembly
  - LRTK-SIM
  - Supernova2
  - gen_fasta.py
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# assembly-contiguity-n50-evaluation

## Summary

Evaluate diploid Supernova2 assemblies of 10x Linked-Read data using contiguity metrics (contig and scaffold N50) and the accuracy-aware NA50 metric, which penalizes contiguity achieved by misassembled sequence. This skill reproduces the Zhang et al. 2019 framework for comparing assemblies built from libraries with different coverage parameters.

## When to use

Use when de novo assembled 10x Linked-Read (Supernova2) assemblies must be compared across parameter settings (total coverage C, physical coverage CF, read coverage per CR) using N50 and NA50 contiguity/accuracy metrics.

## When NOT to use

- Assemblies from long-read technologies (PacBio, ONT) or non-barcode data where the 10x parameter space and NA50 reference-based framing do not apply
- No high-quality reference genome is available and only N50 (which the authors note is misleading) can be computed, precluding accuracy-aware NA50 comparison
- The goal is variant calling benchmarking (e.g. against the GIAB benchmark) rather than assembly contiguity/accuracy assessment

## Inputs

- Supernova2 de novo assembly outputs (contigs and scaffolds) from simulated or real 10x Linked-Read libraries
- Paired-end FASTQ libraries simulated by LRTK-SIM (X_S1_L001_R1_001.fastq.gz / X_S1_L001_R2_001.fastq.gz) or real Chromium data
- hg38 human reference sequence (for NA50 accuracy-aware comparison)

## Outputs

- Contig N50 values per assembly (e.g. 621.4–758.1 kb across the simulated coverage sweep)
- Scaffold N50 values per assembly
- NA50 accuracy-aware contiguity values per assembly (e.g. 587.5–713.3 kb)
- Comparison of metrics across libraries/parameter combinations

## How to apply

After running Supernova2 de novo assembly on each simulated or real Linked-Read library (e.g. the eight libraries spanning total coverage C = 16x to 78x), compute contig and scaffold N50 as standard contiguity metrics. For accuracy-aware comparison against the hg38 human reference, also compute NA50, which discounts contig length contributed by misassembled sequence; this is only possible when a high-quality reference exists. Evaluate trends across the parameter sweep rather than single values: in the simulation, contig N50 rose from 621.4 to 758.1 kb and NA50 from 587.5 to 713.3 kb as total coverage C increased beyond 56x. Use the Evaluate_assembly_performance.ipynb workflow in the Evaluate_diploid_assembly repository for the metric computation. Correct application is judged by monotonic N50/NA50 increases with C within the broadly optimal parameter ranges (CF between 332x and 823x for a given C) and by agreement between N50 and NA50 trends.

## Related tools

- **Supernova2** (Produces the diploid de novo assemblies whose contig/scaffold N50 and NA50 are evaluated)
- **LRTK-SIM** (Generates simulated Linked-Read FASTQ libraries across the C/CF/CR parameter sweep used to build the assemblies being evaluated) — https://github.com/zhanglu295/LRTK-SIM
- **Evaluate_diploid_assembly** (Jupyter notebook workflow (Evaluate_assembly_performance.ipynb) for computing and comparing assembly evaluation metrics) — https://github.com/zhanglu295/Evaluate_diploid_assembly

## Examples

```
python LRTK-SIM.py ./diploid_config  # then run Supernova2 on each lib_X/*.fastq.gz pair and evaluate N50/NA50 per Evaluate_assembly_performance.ipynb
```

## Evaluation signals

- Contig and scaffold N50 must increase monotonically with total sequencing coverage C across the simulated sweep, reproducing the reported contig N50 shift from 621.4 to 758.1 kb
- NA50 should track the contiguity trend, shifting from 587.5 to 713.3 kb beyond C = 56x; a large N50–NA50 gap signals misassembly-driven contiguity
- Assemblies built at broadly optimal parameters should place ~80% of the genome in a diploid state
- Assembly quality should worsen when physical coverage CF exceeds ~1,000x for a given C, providing a negative-control check on the metric trends
- Comparison metrics for real data (NA12878, NA24385) should approximate the trends observed in the simulated parameter-matched libraries

## Limitations

- NA50 comparison is only possible because a high-quality human reference genome exists; without one, quality assessment relies on N50, which is misleading
- No alternative assembler to Supernova2 was available for comparison, limiting interpretation of assembly-quality effects
- The simulation pipeline (gen_fasta.py) supports SNVs only, so simulated diploid references lack other variant classes
- Long DNA fragments could significantly extend phase blocks but decreased contig contiguity, so contiguity metrics alone do not capture phasing benefits

## Evidence

- [results] assembly contiguity increased with total coverage C: [evidence span withheld — anchor status: not_found]
- [discussion] na50-requires-reference: "which we were able to compare to NA50 because there exists a high-quality human reference genome"
- [discussion] n50-misleading: [evidence span withheld — anchor status: not_found]
- [readme] sim-data-tested-by-supernova: "The simulated data have been tested and accepted by 10X official pipelines [Long Ranger](https://support.10xgenomics.com/genome-exome/software/pipelines/latest/what-is-long-ranger) and [Supernova]"
- [abstract] assembly-quality-trends: "We found that assembly quality could be improved by increasing the total sequencing coverage (C) and keeping physical coverage of DNA fragments (CF) or read coverage per fragment (CR) within broad"
