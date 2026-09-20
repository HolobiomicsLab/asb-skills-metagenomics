---
name: phage-host-prediction-interpretation
description: Use when you have quality-assessed viral contigs (e.g., identified by
  VirSorter2/DeepVirFinder/VIBRANT and checked with CheckV) and need to know which
  bacteria they infect, or when reproducing host-based findings such as phage–host
  links in UC vs healthy viromes.
when_to_use_negative:
- Input is isolated viral fractions/single-virus samples rather than metagenomic contigs
  — the article notes ViroProfiler is 'specifically designed for classifying viruses
  in samples with isolated viruses', and host prediction requires assembled community
  contigs.
- Sequences carry excessive environmental contamination — this 'could increase the
  running time of the pipeline and result in lower precision', degrading host-prediction
  reliability.
- Only genome taxonomy or AMG annotation is needed with no host attribution goal —
  host prediction is a costly step (300 GB memory in the validation run) that is unnecessary
  for taxonomy-only questions.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: ViroProfiler
  role: Containerized Nextflow pipeline orchestrating viral identification and host
    prediction steps
  repo: https://github.com/deng-lab/viroprofiler
- name: iPhoP
  role: Host prediction engine executed via the VIRALHOST_IPHOP process (12 CPUs,
    300 GB in the UC validation)
- name: VirSorter2
  role: Upstream viral contig identification feeding the host-prediction step
  repo: https://github.com/jiarong/VirSorter2
- name: VIBRANT
  role: Upstream viral contig identification feeding the host-prediction step
  repo: https://github.com/AnantharamanLab/VIBRANT
- name: DeepVirFinder
  role: Upstream viral contig identification feeding the host-prediction step
  repo: https://github.com/jessieren/DeepVirFinder
- name: CheckV
  role: Quality/completeness assessment of contigs before host prediction
- name: metaSPAdes
  role: De novo (--meta) assembly producing contigs that host prediction is run against
  repo: https://github.com/ablab/spades
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:c8e411e995f14592febdf0ce95b3c3224eb28bf4b5c3c73adc871b6b73b837eb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/deng-lab/viroprofiler
  repo_url: https://github.com/deng-lab/viroprofiler
  tools:
  - CheckV
  - DeepVirFinder
  - FASTP
  - MMseqs2
  - VIBRANT
  - VirSorter2
  - ViroProfiler
  - iPhoP
  - metaSPAdes
  - vConTACT2
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# phage-host-prediction-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predict bacterial hosts for viral contigs assembled from shotgun virome data using ViroProfiler's host-prediction module (iPHoP), then interpret predicted host assignments biologically — e.g., counting phages infecting Bacteroidia in ulcerative-colitis versus healthy fecal samples.

## When to use

Use when you have quality-assessed viral contigs (e.g., identified by VirSorter2/DeepVirFinder/VIBRANT and checked with CheckV) and need to know which bacteria they infect, or when reproducing host-based findings such as phage–host links in UC vs healthy viromes.

## When NOT to use

- Input is isolated viral fractions/single-virus samples rather than metagenomic contigs — the article notes ViroProfiler is 'specifically designed for classifying viruses in samples with isolated viruses', and host prediction requires assembled community contigs.
- Sequences carry excessive environmental contamination — this 'could increase the running time of the pipeline and result in lower precision', degrading host-prediction reliability.
- Only genome taxonomy or AMG annotation is needed with no host attribution goal — host prediction is a costly step (300 GB memory in the validation run) that is unnecessary for taxonomy-only questions.

## Inputs

- Non-redundant viral contigs (FASTA) from ViroProfiler assembly and viral-identification steps
- Raw shotgun virome reads (FASTQ, e.g., SRA downloads from the UC fecal virome study)
- ViroProfiler Nextflow config file specifying resources (e.g., 12 CPUs, 300 GB for VIRALHOST_IPHOP)

## Outputs

- Per-contig predicted host taxa with confidence scores (iPHoP output table)
- Counts of phages predicted to infect target host taxa (e.g., Bacteroidia) per sample group (UC vs healthy)
- Summary table combining host counts with viral contig counts and family-level composition

## How to apply

Run ViroProfiler (deng-lab/viroprofiler, Nextflow DSL2, Docker/Singularity containerized) up to the host-prediction stage: FASTP trimming, host read removal (DECONTAM), metaSPAdes (--meta) assembly, viral contig identification with VirSorter2, DeepVirFinder and VIBRANT, then quality assessment with CheckV. Invoke the iPHoP host-prediction process (VIRALHOST_IPHOP in the ViroProfiler workflow; the UC validation run used 12 CPUs and 300 GB memory) on the non-redundant viral contig set; iPHoP integrates multiple signals to output per-contig predicted host taxa with confidence scores. Filter predictions to the host class/taxon of interest (e.g., Bacteroidia) using the confidence thresholds encoded in iPHoP output, and tally predicted hosts per condition (UC vs healthy). Compile host counts alongside contig counts and taxonomy (vConTACT2, MMseqs2) into a summary table for comparison against the published validation values (e.g., 761 vs 183 viral contigs) and Figure 3.

## Related tools

- **ViroProfiler** (Containerized Nextflow pipeline orchestrating viral identification and host prediction steps) — https://github.com/deng-lab/viroprofiler
- **iPhoP** (Host prediction engine executed via the VIRALHOST_IPHOP process (12 CPUs, 300 GB in the UC validation))
- **VirSorter2** (Upstream viral contig identification feeding the host-prediction step) — https://github.com/jiarong/VirSorter2
- **VIBRANT** (Upstream viral contig identification feeding the host-prediction step) — https://github.com/AnantharamanLab/VIBRANT
- **DeepVirFinder** (Upstream viral contig identification feeding the host-prediction step) — https://github.com/jessieren/DeepVirFinder
- **CheckV** (Quality/completeness assessment of contigs before host prediction)
- **metaSPAdes** (De novo (--meta) assembly producing contigs that host prediction is run against) — https://github.com/ablab/spades

## Examples

```
nextflow run deng-lab/viroprofiler -profile docker -c uc_viroprofiler.config --input samples.csv --iphop_cpus 12 --iphop_memory '300 GB'
```

## Evaluation signals

- Host-prediction output table covers all viral contigs submitted, with per-contig predicted host and confidence score fields populated.
- Reproduced Bacteroidia-infecting phage counts in UC vs healthy samples match the validation summary table and Figure 3 of the source study.
- Downstream resource accounting shows the VIRALHOST_IPHOP process ran within the configured limits (12 CPUs, 300 GB) without failure.
- Viral contig inputs to host prediction reproduce the improved discovery rate (761 contigs vs 183 in the original UC study) when the same 20 samples are processed.
- Host-attributable counts are consistent with independently computed family-level composition (Podoviridae proportions) from vConTACT2/MMseqs2 taxonomy.

## Limitations

- ViroProfiler is 'specifically designed for classifying viruses in samples with isolated viruses', limiting host-prediction scope for some sample types.
- Excessive environmental contaminations in metagenome sequences 'could increase the running time of the pipeline and result in lower precision', so isolating viral fractions before sequencing is recommended.
- The discussion does not name benchmark datasets or comparison tools for host prediction specifically; validation relies on previously published experimental viral metagenome data.

## Evidence
