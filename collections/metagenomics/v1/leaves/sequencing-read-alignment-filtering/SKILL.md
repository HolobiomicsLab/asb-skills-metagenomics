---
name: sequencing-read-alignment-filtering
description: Use when Illumina FASTQ reads (real or ART-simulated) must be mapped
  to a fungal taxonomic database and off-target bacterial mapping must be quantified
  or minimized via strict alignment-length and quality thresholds.
when_to_use_negative:
- Input is not Illumina next-generation sequencing reads — the article's ART simulation
  and bowtie2 filtering were designed and benchmarked for Illumina-formatted output
  only
- The goal is bacterial or general taxonomic profiling, not human mycobiome specificity
  assessment — FunOMIC-T is by design restricted to fungal sequences and would under-report
  bacteria
- Functional (protein-level) annotation is the goal — use the DIAMOND blastx route
  against FunOMIC-P (coverage >95%, identity >99%, e-value < 10e-10) rather than bowtie2
  read mapping to FunOMIC-T
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: FunOMIC
  role: Automated pipeline wrapping the bowtie2 alignment of reads to FunOMIC-T and
    downstream samtools filtering; invoked via FunOMIC.sh
  repo: https://github.com/ManichanhLab/FunOMIC
- name: bowtie2
  role: Short-read aligner mapping FASTQ reads against the 1.69 million fungal single-copy
    marker genes of FunOMIC-T
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: samtools
  role: Filters mapped reads, keeping only alignments covering >80% of read length
    with q-score >30
  repo: http://www.htslib.org/
- name: ART
  role: Simulates Illumina-formatted sequencing reads from bacterial and mixed mock-community
    genomes for specificity validation
- name: Diamond blastx
  role: Alternative alignment step for functional profiling against FunOMIC-P under
    coverage >95%, identity >99%, e-value < 10e-10 thresholds
  repo: https://github.com/bbuchfink/diamond/wiki
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:fb34bbcfd4b4bf3fac0250ac849c493f4ffb135f44fe59d51c9a83328cefa3bd
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
  - Diamond blastx
  - FunOMIC
  - bowtie2
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# sequencing-read-alignment-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns simulated or real Illumina next-generation sequencing reads to the FunOMIC-T fungal marker-gene database with bowtie2 and applies a strict samtools mapping filter (>80% of read length aligned, q-score >30) to enforce fungal specificity in mycobiome profiling.

## When to use

Use when Illumina FASTQ reads (real or ART-simulated) must be mapped to a fungal taxonomic database and off-target bacterial mapping must be quantified or minimized via strict alignment-length and quality thresholds.

## When NOT to use

- Input is not Illumina next-generation sequencing reads — the article's ART simulation and bowtie2 filtering were designed and benchmarked for Illumina-formatted output only
- The goal is bacterial or general taxonomic profiling, not human mycobiome specificity assessment — FunOMIC-T is by design restricted to fungal sequences and would under-report bacteria
- Functional (protein-level) annotation is the goal — use the DIAMOND blastx route against FunOMIC-P (coverage >95%, identity >99%, e-value < 10e-10) rather than bowtie2 read mapping to FunOMIC-T

## Inputs

- Illumina paired-end FASTQ reads (real metagenomes or ART-simulated, e.g., 999,021 bacterial reads from a mixed mock community)
- FunOMIC-T taxonomic database (1.69 million fungal single-copy marker genes) downloaded from https://manichanh.vhir.org/funomic/
- Optional FunOMIC-P functional database (3.4 million non-redundant fungal proteins)
- Bacterial database path for FunOMIC.sh (-a bacterialDB_path)

## Outputs

- BAM/mapping output of reads retained after the >80% alignment-length and q-score >30 samtools filter
- Bacterial mapping rate (e.g., <0.01% for the 903-genome mock community)
- Specificity value (false positive / (false positive + true negative), e.g., 0.9999) written to validation_metrics.csv

## How to apply

Simulate or obtain Illumina-formatted reads (e.g., ART for the 903-genome bacterial mock community and the mixed bacterial:fungal mock community with a ~1000:1 bacterial:fungal read ratio), then align them to the 1.69 million FunOMIC-T single-copy fungal marker genes through the FunOMIC pipeline using bowtie2 (FunOMIC.sh with -b FunOMIC-T_folder). Retain mapped reads only when more than 80% of the read length aligns to the reference with a q-score over 30, enforced via samtools filtering. For functional profiling against FunOMIC-P, DIAMOND blastx uses even stricter criteria (read coverage >95%, identity >99%, e-value < 10e-10). Compute the bacterial mapping rate and specificity as false positive / (false positive + true negative) to judge whether the filter suppressed bacterial off-targets.

## Related tools

- **FunOMIC** (Automated pipeline wrapping the bowtie2 alignment of reads to FunOMIC-T and downstream samtools filtering; invoked via FunOMIC.sh) — https://github.com/ManichanhLab/FunOMIC
- **bowtie2** (Short-read aligner mapping FASTQ reads against the 1.69 million fungal single-copy marker genes of FunOMIC-T) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **samtools** (Filters mapped reads, keeping only alignments covering >80% of read length with q-score >30) — http://www.htslib.org/
- **ART** (Simulates Illumina-formatted sequencing reads from bacterial and mixed mock-community genomes for specificity validation)
- **Diamond blastx** (Alternative alignment step for functional profiling against FunOMIC-P under coverage >95%, identity >99%, e-value < 10e-10 thresholds) — https://github.com/bbuchfink/diamond/wiki

## Examples

```
FunOMIC.sh -1 read1.fastq -2 read2.fastq -p output_prefix -o output_folder -a bacterialDB_path -b FunOMIC-T_folder -c FunOMIC-P_folder -t #_threads
```

## Evaluation signals

- The mixed mock community (~1000:1 bacterial:fungal read ratio) yields zero aligned reads out of 999,021 bacterial reads, i.e., specificity = 0.9999
- The 903-genome bacterial mock community (19,301,201 Illumina reads) shows a mapping rate to FunOMIC-T below 0.01%
- Only reads with >80% of their length aligned and q-score >30 are present in the retained mapping output
- Independent contamination check: <0.01% of the 1.69 million fungal marker genes map to the Unified Human Gastrointestinal Genome (UHGG)
- validation_metrics.csv is populated with the computed mapping rate and specificity values matching the stated validation thresholds

## Limitations

- Extraction and quality of single-copy marker genes depend on the completeness of available fungal genomes, giving lower fungal taxonomy coverage than amplicon databases
- High inter-kingdom conservation of protein-coding genes means bacterial contamination was not totally preventable, even with exceedingly strict mapping thresholds for fungal functional annotation
- No bacterial read filtration step is included before functional annotation; the authors propose adding it in a future update

## Evidence
