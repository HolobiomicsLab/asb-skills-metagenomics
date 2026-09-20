---
name: fungal-functional-profiling
description: Use when performing fungal functional profiling of shotgun metagenomic
  reads against the FunOMIC-P protein database, especially when distinguishing fungal
  signal from dominant bacterial backgrounds.
when_to_use_negative:
- Reads should be taxonomically profiled first with FunOMIC-T if the question is species/strain
  identification — FunOMIC-T gave 0.9999 specificity on mock bacterial reads, whereas
  FunOMIC-P retains some bacterial contamination due to conserved proteins
- Input is amplicon data (ITS) rather than shotgun metagenomic reads; FunOMIC is designed
  for next-generation sequencing read mapping to protein/marker databases
- Bacterial functional profiling is the goal — FunOMIC-P is a fungal-only protein
  database and the pipeline treats bacterial reads as background to be excluded
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0659
tools:
- name: FunOMIC
  role: Automated pipeline that wraps read mapping against the FunOMIC-P functional
    database and post-mapping filtering
  repo: https://github.com/ManichanhLab/FunOMIC
- name: Diamond blastx v2.0.8
  role: Translated nucleotide-to-protein aligner used to map reads to FunOMIC-P with
    e-value < 10e-10
  repo: https://github.com/bbuchfink/diamond
- name: FLASH2
  role: Merges overlapping paired-end reads prior to database mapping in the FunOMIC
    workflow
  repo: https://github.com/dstreett/FLASH2
- name: samtools
  role: Alignment file handling within the FunOMIC pipeline
  repo: http://www.htslib.org/
- name: bowtie2
  role: Short-read aligner used in the FunOMIC pipeline (e.g. taxonomic mapping against
    FunOMIC-T)
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: KEGGREST
  role: R/Bioconductor package used by FunOMIC to attach KEGG functional annotations
  repo: https://www.bioconductor.org/packages/release/bioc/html/KEGGREST.html
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:a6cdcdf128423ef6e8133bf12b2eb7ec6cd7d5d6efd3f8b1fe180b3ae8fda35f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ManichanhLab/FunOMIC
  repo_url: https://github.com/ManichanhLab/FunOMIC
  tools:
  - ART
  - Diamond blastx v2.0.8
  - FLASH2
  - FunOMIC
  - KEGGREST
  - bowtie2
  - samtools
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# fungal-functional-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Functional annotation of mycobiome sequencing reads by mapping them against the FunOMIC-P database (3.4 million non-redundant fungal proteins) using Diamond blastx, with strict identity/coverage/e-value thresholds tuned to suppress bacterial cross-mapping.

## When to use

Use when performing fungal functional profiling of shotgun metagenomic reads against the FunOMIC-P protein database, especially when distinguishing fungal signal from dominant bacterial backgrounds.

## When NOT to use

- Reads should be taxonomically profiled first with FunOMIC-T if the question is species/strain identification — FunOMIC-T gave 0.9999 specificity on mock bacterial reads, whereas FunOMIC-P retains some bacterial contamination due to conserved proteins
- Input is amplicon data (ITS) rather than shotgun metagenomic reads; FunOMIC is designed for next-generation sequencing read mapping to protein/marker databases
- Bacterial functional profiling is the goal — FunOMIC-P is a fungal-only protein database and the pipeline treats bacterial reads as background to be excluded

## Inputs

- Paired-end Illumina FASTQ files (-1 read1.fastq -2 read2.fastq)
- FunOMIC-P functional database (3,413,239 non-redundant fungal proteins, downloaded from https://manichanh.vhir.org/funomic/)
- Bacterial decoy database (bacterialDB_path, -a flag)

## Outputs

- Fungal functional annotation of metagenomic reads (KEGG-based functional assignments via pipeline modules using KEGGREST)
- Filtered Diamond blastx alignment results passing coverage >95%, identity >99%, e-value < 10e-10

## How to apply

Run the FunOMIC pipeline (github.com/ManichanhLab/FunOMIC) on paired-end reads, downloading the FunOMIC-P database from https://manichanh.vhir.org/funomic/ and invoking FunOMIC.sh with the -c flag pointing to the FunOMIC-P folder. Reads are mapped with Diamond blastx v2.0.8 with an e-value < 10e-10; hits are then filtered on read coverage >95% and identity percentage >99%, which the paper validated as optimal via a nine-threshold sweep (>90/95/99% identity x >90/95/99% coverage) on ART-simulated mock community reads, yielding ~70% retained fungal read hits. Mapped reads are retained only if >80% of their length aligns with q-score >30. Because inter-kingdom protein conservation makes some bacterial contamination unavoidable, apply this skill after (or alongside) taxonomic profiling with FunOMIC-T, which showed specificity of 0.9999 against bacterial mock reads.

## Related tools

- **FunOMIC** (Automated pipeline that wraps read mapping against the FunOMIC-P functional database and post-mapping filtering) — https://github.com/ManichanhLab/FunOMIC
- **Diamond blastx v2.0.8** (Translated nucleotide-to-protein aligner used to map reads to FunOMIC-P with e-value < 10e-10) — https://github.com/bbuchfink/diamond
- **FLASH2** (Merges overlapping paired-end reads prior to database mapping in the FunOMIC workflow) — https://github.com/dstreett/FLASH2
- **samtools** (Alignment file handling within the FunOMIC pipeline) — http://www.htslib.org/
- **bowtie2** (Short-read aligner used in the FunOMIC pipeline (e.g. taxonomic mapping against FunOMIC-T)) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **KEGGREST** (R/Bioconductor package used by FunOMIC to attach KEGG functional annotations) — https://www.bioconductor.org/packages/release/bioc/html/KEGGREST.html

## Examples

```
FunOMIC.sh -1 sample_R1.fastq -2 sample_R2.fastq -p sample -o out/ -a bacterialDB/ -b FunOMIC-T/ -c FunOMIC-P/ -t 16
```

## Evaluation signals

- The output alignment table contains hits filtered at read coverage >95%, identity percentage >99%, and e-value < 10e-10
- Threshold sweep validation: the coverage >95% / identity >99% combination should give the highest fungal read hit pass rate (~70%) among the nine combinations
- Specificity check: simulated bacterial reads (e.g. from a 903-genome / 458-species mock) should show near-zero mapping to fungal databases; FunOMIC-T gave <0.01% mapping and specificity 0.9999
- Mapped reads retained must satisfy >80% aligned length and q-score >30
- FunOMIC.sh completes with the -c FunOMIC-P folder resolved and produces per-sample functional profiles

## Limitations

- High inter-kingdom conservation of protein-coding genes means bacterial contamination of functional annotation was not totally preventable, even with strict mapping thresholds
- No bacterial read filtration step is included before functional annotation in the current release; the authors propose adding it in a future update
- Extraction and quality of fungal marker genes depend on completeness of available fungal genomes, limiting taxonomic/functional coverage relative to amplicon databases

## Evidence
