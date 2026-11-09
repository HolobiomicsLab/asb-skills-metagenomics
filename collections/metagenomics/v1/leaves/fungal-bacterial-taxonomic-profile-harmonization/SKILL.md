---
name: fungal-bacterial-taxonomic-profile-harmonization
description: Use when paired fungal and bacterial taxonomic abundance tables (e.g.
  FunOMIC-T output and MetaPhlAn profiles) from the same metagenomes must be subset
  to matched cohorts and combined for inter-kingdom correlation analysis.
when_to_use_negative:
- Input is already a single joint feature table or edge list with per-cohort partitions
  applied
- The analysis goal is within-kingdom (fungus-fungus or bacterium-bacterium) association
  structure rather than inter-kingdom interactions
- Fungal profiles were generated with amplicon (ITS) methods rather than FunOMIC-T
  mapping, since database-specific filtering steps and abundance scales would not
  be comparable
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: FunOMIC
  role: Provides the fungal taxonomic profiles (FunOMIC-T database) that constitute
    one side of the harmonized inter-kingdom table
  repo: https://github.com/ManichanhLab/FunOMIC
- name: bowtie2
  role: Maps metagenomic reads to the FunOMIC-T marker-gene database to produce fungal
    abundance profiles
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: MetaPhlAn
  role: Generates the bacterial taxonomic profiles (v3.0.9) paired with fungal profiles
    per sample
- name: SparCC
  role: Runs per-cohort inter-kingdom correlation estimation on the harmonized fungal-bacterial
    profiles
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:749e03518c7c95406b5bd2afeb22274f79608bc1410c311f27120cce1deaf6f3
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
  - FunOMIC
  - MetaPhlAn
  - SparCC
  - bowtie2
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# fungal-bacterial-taxonomic-profile-harmonization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Harmonizes fungal taxonomic profiles generated with the FunOMIC pipeline (FunOMIC-T database, bowtie2 mapping) with bacterial taxonomic profiles from MetaPhlAn v3.0.9 so the two kingdom-level tables can be jointly analyzed, e.g. with SparCC correlation networks, on the same sample set.

## When to use

Use when paired fungal and bacterial taxonomic abundance tables (e.g. FunOMIC-T output and MetaPhlAn profiles) from the same metagenomes must be subset to matched cohorts and combined for inter-kingdom correlation analysis.

## When NOT to use

- Input is already a single joint feature table or edge list with per-cohort partitions applied
- The analysis goal is within-kingdom (fungus-fungus or bacterium-bacterium) association structure rather than inter-kingdom interactions
- Fungal profiles were generated with amplicon (ITS) methods rather than FunOMIC-T mapping, since database-specific filtering steps and abundance scales would not be comparable

## Inputs

- Fungal taxonomic profile table (species-level abundances from FunOMIC-T mapping with bowtie2)
- Bacterial taxonomic profile table (species-level relative abundances from MetaPhlAn v3.0.9)
- Sample metadata table (continent, country, sample source, disease status)
- Sample identifier column shared by all three tables

## Outputs

- Per-cohort inter-kingdom edge-list table with columns fungus, bacterium, rho, p-value, cohort
- Summary block with counts of significant correlations per cohort (expected: 4,184 Danish, 3,471 Spanish) and strongest functional correlation coefficients (expected rho = 0.06 and rho = -0.13)
- Subsetted, sample-matched fungal and bacterial abundance tables per cohort

## How to apply

Load the fungal taxonomic profiles obtained by mapping sequencing reads to the FunOMIC-T database with bowtie2 (keeping mapped reads with >80% aligned length and q-score over 30) and the bacterial profiles from MetaPhlAn v3.0.9, together with sample metadata. Intersect the two tables on shared sample IDs and subset both to the target cohorts using metadata fields such as continent, country, sample source, and disease status (e.g. healthy Danish and healthy Spanish gut cohorts). Align the taxonomic feature axes (fungi vs. bacteria) into a joint feature table or keep them separate per kingdom as required by the downstream correlation tool. Run SparCC separately per cohort to estimate inter-kingdom correlations, then filter pairs to significant links using p-values < 0.001 and an absolute correlation coefficient over 0.05. Assemble per-cohort edge lists (fungus, bacterium, rho, p-value, cohort) and verify counts against reference values (e.g. 4,184 significant inter-kingdom correlations in the Danish healthy cohort and 3,471 in the Spanish cohort).

## Related tools

- **FunOMIC** (Provides the fungal taxonomic profiles (FunOMIC-T database) that constitute one side of the harmonized inter-kingdom table) — https://github.com/ManichanhLab/FunOMIC
- **bowtie2** (Maps metagenomic reads to the FunOMIC-T marker-gene database to produce fungal abundance profiles) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **MetaPhlAn** (Generates the bacterial taxonomic profiles (v3.0.9) paired with fungal profiles per sample)
- **SparCC** (Runs per-cohort inter-kingdom correlation estimation on the harmonized fungal-bacterial profiles)

## Examples

```
FunOMIC.sh -1 read1.fastq -2 read2.fastq -p output_prefix -o output_folder -a bacterialDB_path -b FunOMIC-T_folder -c FunOMIC-P_folder -t #_threads
```

## Evaluation signals

- Sample ID sets of the fungal and bacterial tables match exactly after harmonization (no samples dropped silently)
- Significant-edge counts (p < 0.001, |rho| > 0.05) reproduce the reported values: 4,184 for the Danish healthy gut cohort and 3,471 for the Spanish cohort
- Strongest functional correlation coefficients equal the expected rho = 0.06 and rho = -0.13
- Candida albicans negative correlations with short-chain-fatty-acid-producing bacteria appear among the retained edges
- Edge-list schema check: every row has non-null fungus, bacterium, rho, p-value, and cohort fields

## Limitations

- Extraction and quality of single-copy marker genes rely on the completeness of available fungal genomes, which may result in lower fungal taxonomy coverage compared with amplicon databases
- High inter-kingdom conservation of protein-coding genes means bacterial contamination of functional profiles was not totally preventable even with strict mapping thresholds
- No bacterial read filtration step is included before functional annotation in the current pipeline version

## Evidence
