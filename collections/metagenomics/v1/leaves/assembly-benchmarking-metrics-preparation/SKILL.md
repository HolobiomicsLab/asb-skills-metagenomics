---
name: assembly-benchmarking-metrics-preparation
description: 'Use when preparing assembler contigs (e.g. from results/assembly/<assembler>/)
  for reference-based benchmarking in LMAS: contigs must be length-filtered and mapped
  to a multi-FASTA of reference replicons before computing global and replicon-specific
  metrics.'
when_to_use_negative:
- Input is raw sequencing reads rather than assembler-generated contigs — this skill
  operates downstream of the de novo assembly step in LMAS
- No reference replicon multi-FASTA is available and only reference-free assembly
  metrics are required
- The goal is assembling new samples rather than evaluating existing assemblies; use
  the upstream LMAS assembly stage instead
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: LMAS
  role: Nextflow workflow orchestrating the contig filtering and reference mapping
    steps before metric computation
  repo: https://github.com/B-UMMI/LMAS
- name: BBTools
  role: Version 38.44 sequence-length filter producing the filtered set (FS) of contigs
    per --minLength
- name: minimap2
  role: Version 2.22 aligner indexing reference replicons and mapping FS contigs to
    them, one alignment file per sample
- name: Nextflow
  role: Workflow engine providing modularity, containerization, and process parallelization
    for reproducible execution
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giac122
    title: 'LMAS: evaluating metagenomic short <i>de novo</i> assembly methods through
      defined communities'
  source_content_hash: sha256:34aa46dda677e518826b733d85c74f61f5f9dabfe76e1aaae9ab4f092edd1a78
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/LMAS
  tools:
  - BBTools
  - LMAS
  - Nextflow
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# assembly-benchmarking-metrics-preparation

## Summary

Reconstructs the LMAS pre-metric preparation stage: filtering assembler-generated contig FASTA files down to the filtered set (FS) by minimum sequence length with BBTools v38.44, then aligning FS contigs to reference replicons with minimap2 v2.22 so reference-dependent quality metrics can be computed.

## When to use

Use when preparing assembler contigs (e.g. from results/assembly/<assembler>/) for reference-based benchmarking in LMAS: contigs must be length-filtered and mapped to a multi-FASTA of reference replicons before computing global and replicon-specific metrics.

## When NOT to use

- Input is raw sequencing reads rather than assembler-generated contigs — this skill operates downstream of the de novo assembly step in LMAS
- No reference replicon multi-FASTA is available and only reference-free assembly metrics are required
- The goal is assembling new samples rather than evaluating existing assemblies; use the upstream LMAS assembly stage instead

## Inputs

- Contig FASTA files per assembler from results/assembly/<assembler>/
- Single multi-FASTA file of reference replicons (--reference)
- --minLength parameter value (default: 1000 bp)

## Outputs

- Filtered set (FS) contig FASTA files containing only contigs meeting the --minLength threshold
- One minimap2 alignment file per sample mapping FS contigs to the reference replicons, for downstream global and reference-specific metrics

## How to apply

Load the contig FASTA files produced by each assembler in results/assembly/<assembler>/ together with the single multi-FASTA of reference replicons supplied via --reference. Apply the BBTools (v38.44) sequence-length filter with the minimum contig length set by the --minLength parameter (default 1000 bp) to retain only the filtered set (FS) — this mirrors the paper's observation that differences between assemblers attenuate after filtering for contigs <1 kbp. Index the reference replicons and align the FS contigs against them with minimap2 (v2.22) to obtain reference-dependent mappings. Write one alignment file per sample for downstream computation of global and reference replicon-specific performance metrics, which also feed the misassembly classification (chimera, insertion, deletion, inversion, rearrangement, translocation, duplication) and the mapped-reads threshold logic (--mapped_reads_threshold, the minimum percentage of a read aligning to a contig to be considered mapped).

## Related tools

- **LMAS** (Nextflow workflow orchestrating the contig filtering and reference mapping steps before metric computation) — https://github.com/B-UMMI/LMAS
- **BBTools** (Version 38.44 sequence-length filter producing the filtered set (FS) of contigs per --minLength)
- **minimap2** (Version 2.22 aligner indexing reference replicons and mapping FS contigs to them, one alignment file per sample)
- **Nextflow** (Workflow engine providing modularity, containerization, and process parallelization for reproducible execution)

## Examples

```
nextflow run B-UMMI/LMAS --reference replicons.fasta --minLength 1000
```

## Evaluation signals

- A valid minimap2 alignment file exists for every sample/assembler combination
- Every retained contig in the FS meets the --minLength length threshold (default 1000 bp)
- Alignment files are per-sample and reference-based, ready for downstream global and reference-specific metric computation
- Filtering for contigs <1 kbp reproduces the paper's expected attenuation of differences between genomic and metagenomic assemblers

## Limitations

- The --minLength default of 1000 bp attenuates differences between assemblers; results are sensitive to the chosen minimum contig length
- The mapped reads threshold (--mapped_reads_threshold, minimum percentage of a read aligning to a contig to be considered mapped) affects downstream metric interpretation
- Strain-level (SNP-level) discrimination is highly unlikely based solely on assembler-generated contigs, limiting the resolution of reference-based metrics

## Evidence

- [results] the differences between them tended to be attenuated after filtering for contigs <1 kbp: "the differences between them tended to be attenuated after filtering for contigs <1 kbp"
- [methods] Contigs > X section corresponds to --minLength: "The **X** in the **Contigs > X** section corresponds to the minimum contig size as defined in the ``--minLength`` parameter"
- [methods] mapped reads threshold definition: "--mapped_reads_threshold                Value for the minimum percentage of a read aligning to the
                                    contig to be considered as mapped"
- [discussion] LMAS integrates assembly and evaluation in one pipeline: "LMAS is the only software-integrating assembly and its evaluation into a single pipeline, guaranteeing the same conditions are met for all tools."
