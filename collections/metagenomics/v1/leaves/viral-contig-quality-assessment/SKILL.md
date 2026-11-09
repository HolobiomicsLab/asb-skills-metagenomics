---
name: viral-contig-quality-assessment
description: Use when long assembled contigs from viral metagenomic samples must be
  evaluated for viral quality and completeness before dereplication into a non-redundant
  contig library.
when_to_use_negative:
- Input sequences are short contigs or raw reads rather than length-filtered assembled
  contigs
- The goal is taxonomic classification of already-isolated viral genomes rather than
  assessing metagenome-assembled viral contigs (ViroProfiler is specifically designed
  for classifying viruses in samples with isolated viruses)
- Sample contigs have not yet been merged and length-filtered, i.e., the assembly-pooling
  step upstream of quality assessment has not been performed
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: CheckV
  role: Assesses viral contig quality and completeness of the merged, length-filtered
    contig pool before dereplication
  repo: https://bitbucket.org/berkeleylab/checkv
- name: metaSPAdes
  role: Assembles quality-filtered reads per sample in metagenomic mode ('--meta'),
    producing the contigs that are then quality-assessed
  repo: https://github.com/ablab/spades
- name: ViroProfiler
  role: Containerized Nextflow pipeline that orchestrates assembly, CheckV-based quality
    assessment, and subsequent dereplication into the nrclib
  repo: https://github.com/deng-lab/viroprofiler
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:e8a28c87d603f4adac2537292fc29fb3ade17fe2377f1138ec4e5d5313d6fbcd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/deng-lab/viroprofiler
  tools:
  - CheckV
  - ViroProfiler
  - metaSPAdes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# viral-contig-quality-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assess the quality and completeness of viral contigs assembled from shotgun viral metagenomes (via metaSPAdes) using CheckV within the ViroProfiler pipeline, so that downstream dereplication and annotation operate on well-characterized viral sequences.

## When to use

Use when long assembled contigs from viral metagenomic samples must be evaluated for viral quality and completeness before dereplication into a non-redundant contig library.

## When NOT to use

- Input sequences are short contigs or raw reads rather than length-filtered assembled contigs
- The goal is taxonomic classification of already-isolated viral genomes rather than assessing metagenome-assembled viral contigs (ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses)
- Sample contigs have not yet been merged and length-filtered, i.e., the assembly-pooling step upstream of quality assessment has not been performed

## Inputs

- Long contigs assembled from quality-filtered reads (metaSPAdes '--meta' mode), merged across samples and length-filtered (FASTA)
- Sample-level assembled contig FASTA files from the ViroProfiler assembly step

## Outputs

- CheckV quality/completeness assessment table for each viral contig
- Quality-assessed viral contigs ready for >=95% ANI / >=85% AF dereplication into nrclib.fasta

## How to apply

Assemble quality-filtered reads per sample with metaSPAdes in metagenomic mode (ext.args = '--meta', 4 CPUs, 100 GB as configured in ViroProfiler's default config), merge contigs across samples, and filter by minimum length to remove short spurious sequences. Then run CheckV on the filtered long-contig pool (1 CPU, 30 GB memory) to evaluate viral contig quality and completeness, including detection of putative host regions flanking viral contigs as done in the MGV quality-control step. Use the CheckV assessment as a decision point before species-level ANI dereplication (>=95% ANI / >=85% AF per MIUViG), so that only assessed viral contigs feed the non-redundant contig library (nrclib.fasta) used for downstream taxonomy and functional annotation.

## Related tools

- **CheckV** (Assesses viral contig quality and completeness of the merged, length-filtered contig pool before dereplication) — https://bitbucket.org/berkeleylab/checkv
- **metaSPAdes** (Assembles quality-filtered reads per sample in metagenomic mode ('--meta'), producing the contigs that are then quality-assessed) — https://github.com/ablab/spades
- **ViroProfiler** (Containerized Nextflow pipeline that orchestrates assembly, CheckV-based quality assessment, and subsequent dereplication into the nrclib) — https://github.com/deng-lab/viroprofiler

## Examples

```
checkv end_to_end long_contigs.fasta checkv_out -t 1
```

## Evaluation signals

- CheckV output exists and reports a quality/completeness estimate for every contig in the input FASTA
- Only contigs that passed the minimum-length filter enter the CheckV assessment (short spurious sequences removed upstream)
- Assessed contigs subsequently dereplicate into species-level clusters at >=95% ANI / >=85% AF consistent with MIUViG guidelines
- nrclib.fasta contains exactly one representative per cluster with no duplicate contigs at the 95% ANI / 85% AF threshold
- CheckV process ran with its configured resources (1 CPU, 30 GB memory) per the pipeline's default config

## Limitations

- Excessive environmental contaminations in metagenome sequences could increase the pipeline running time and result in lower precision
- ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses; isolating viral fractions before sequencing is recommended for accurate estimation
- Quality assessment is only meaningful on assembled contigs of sufficient length; very short contigs are removed by the minimum-length filter and cannot be reliably evaluated

## Evidence

- [readme] CheckV quality control in the MGV supporting materials: [evidence span withheld — anchor status: not_found]
