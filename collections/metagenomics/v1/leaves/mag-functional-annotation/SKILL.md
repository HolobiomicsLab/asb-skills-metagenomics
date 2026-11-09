---
name: mag-functional-annotation
description: Use when you have MAG or complete-genome FASTA files and need KEGG KO-level
  functional annotation plus Module completeness recap tables, especially for fragmented
  MAG assemblies where standard annotation tools miss orthologs.
when_to_use_negative:
- When the goal is only Module completeness evaluation with no HMM-based ortholog
  expansion — use `--skip_hmm` workflows instead, since HMM search changes the hit
  set.
- When sequences are already provided as KEGG-annotated KO lists without a genome
  (use `--no_genome`, which stops after KEGG MODULES Completeness Evaluation).
- When annotating very short contigs or heavily truncated genes, where recovery rates
  drop (most clearly at 70% truncation in the MAG dataset) and results are unreliable.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3174
tools:
- name: KEMET
  role: Command-line Python toolbox that evaluates KEGG Module completeness and expands
    annotations via profile HMM searches for missing KOs
  repo: https://github.com/Matteopaluh/KEMET
- name: eggNOG-Mapper
  role: Annotates Prodigal-predicted genes with KEGG KOs in default `.emapper.annotations`
    format consumed by KEMET via `-a eggnog`
- name: Prodigal
  role: Performs gene calling (v2.6.3, default options) on the genome/MAG FASTA prior
    to annotation
- name: GTDB-tk
  role: Used (v1.5.0) for taxonomic assignment of MAGs when selecting representative
    genomes
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:62f69f8fa2d766f80b0181794abaf76ba10d1d00b8010cde02ab543982a4c180
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Matteopaluh/KEMET
  repo_url: https://github.com/Matteopaluh/KEMET
  tools:
  - GTDB-tk
  - KEMET
  - Prodigal
  - Python
  - eggNOG-Mapper
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# mag-functional-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate MAGs and complete microbial genomes by calling genes with Prodigal, assigning KEGG KOs with eggNOG-mapper v2, and expanding the annotation with KEMET's profile HMM searches (default score-per-length threshold 0.43) to recover missing orthologs and summarize KEGG Module completeness.

## When to use

Use when you have MAG or complete-genome FASTA files and need KEGG KO-level functional annotation plus Module completeness recap tables, especially for fragmented MAG assemblies where standard annotation tools miss orthologs.

## When NOT to use

- When the goal is only Module completeness evaluation with no HMM-based ortholog expansion — use `--skip_hmm` workflows instead, since HMM search changes the hit set.
- When sequences are already provided as KEGG-annotated KO lists without a genome (use `--no_genome`, which stops after KEGG MODULES Completeness Evaluation).
- When annotating very short contigs or heavily truncated genes, where recovery rates drop (most clearly at 70% truncation in the MAG dataset) and results are unreliable.

## Inputs

- MAG or complete genome nucleotide FASTA file(s)
- eggNOG-mapper v2 output (.emapper.annotations) per genome (or kaas/kofamkoala KO annotation files)
- KEGG GENES reference database downloadable/updated via the KEGG API
- Optionally: NCBI nr database for translated BLASTp validation of HMM hits

## Outputs

- KEGG MODULES completeness recap tables summarizing metabolic potential
- HMM-search candidate missing-KO gene hits with score-per-length values
- Per-genome recovered KO annotations that can feed GSMM gapfilling
- Validation tables (e.g. threshold_tuning_results.csv, ko_mock_removal_tpr.csv)

## How to apply

First call genes with Prodigal v2.6.3 using default options, then functionally annotate the predicted proteins with eggNOG-mapper v2 (default parameters) keeping the `.emapper.annotations` extension. Run KEMET as `./kemet.py [FASTA_file] -a eggnog` to obtain KEGG MODULES recap tables, and add `--hmm_mode onebm` to search for missing KOs with profile HMMs; the default HMM quality filter `--threshold_value 0.43` differentiates legit hits, and this threshold was tuned against manually curated translated BLASTp hits from NCBI nr, retaining 4.6–7.5% of hits on two MAG datasets. If a MAG per species is selected, require ≥90% completeness and ≤5% contamination before downstream analysis. Validate recovery power with mock-removal controls: remove three KO annotations per genome and count correct gene/annotation recovery (reported 55% / 20-36 for MAGs, 67% / 8-12 for complete genomes on untruncated sequences). Expect recovery to drop for fragmented sequences (20–33% at 30% truncation, declining further at 70% truncation), reflecting the impact of truncation on gene prediction.

## Related tools

- **KEMET** (Command-line Python toolbox that evaluates KEGG Module completeness and expands annotations via profile HMM searches for missing KOs) — https://github.com/Matteopaluh/KEMET
- **eggNOG-Mapper** (Annotates Prodigal-predicted genes with KEGG KOs in default `.emapper.annotations` format consumed by KEMET via `-a eggnog`)
- **Prodigal** (Performs gene calling (v2.6.3, default options) on the genome/MAG FASTA prior to annotation)
- **GTDB-tk** (Used (v1.5.0) for taxonomic assignment of MAGs when selecting representative genomes)

## Examples

```
./kemet.py genomes/bin1.fasta -a eggnog --hmm_mode onebm --threshold_value 0.43 --log
```

## Evaluation signals

- Tuned threshold must retain 4.6–7.5% of BLASTp-verified hits across the two MAG datasets, matching the paper's reported range.
- Untruncated mock-removal test must reproduce 20/36 (55%) and 8/12 (67%) correct gene-and-annotation recoveries for MAGs and complete genomes.
- Truncation test recovery rates at 30% deletion must fall in the 20–33% range (42% assuming unbiased gene prediction) and decline monotonically at 70% truncation.
- Annotation files must retain the required default extension (`.emapper.annotations`) for KEMET's `-a eggnog` input parsing.
- Default HMM quality filter is score-per-length 0.43; any deviation must be recorded via `--threshold_value [VALUE]`.

## Limitations

- KO mock removal on untruncated sequences recovers only 55% (MAGs) and 67% (complete genomes) of removed annotations; many genuine missing KOs are not recovered.
- Gene truncation simulating fragmented MAG assemblies sharply reduces recovery (20–33% at 30% truncation; further decline at 70%), most clearly in the MAG dataset.
- MAGs are only suitable for this analysis at high quality: ≥90% completeness and ≤5% contamination when more than one MAG per species exists.
- The KEGG database used for HMM profiles must be kept up to date via the KEGG API; stale profiles may miss recent ortholog definitions.

## Evidence
