---
name: 16s-rrna-primer-binding-assessment
description: Use when you have curated 16S rRNA gene sequences (e.g. assembled from
  metagenomes, ≥800 bp) and need to estimate which taxa a specific primer pair would
  miss in an amplicon survey.
when_to_use_negative:
- Input is a raw amplicon read set or OTU/feature table rather than curated full-length
  (≥800 bp) 16S rRNA gene sequences — binding cannot be assessed without sequence
  spanning the primer sites
- 16S sequences still contain large (≥10 bp) intron/insertion sequences that were
  not removed, since insertions within the primer region would spuriously inflate
  evasion rates
- Sequences that do not span the 515–806 E. coli-coordinated region — they cannot
  be scored for 515F/806R binding and should be excluded rather than counted as evading
edam_operation: http://edamontology.org/operation_0296
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0080
tools:
- name: USEARCH
  role: De-redundantizes assembled 16S rRNA gene sequences via cluster_smallmem at
    97% identity (query_cov 0.50, target_cov 0.50) before primer binding scoring
- name: Primer Prospector
  role: Evaluates 515F/806R primer binding against each de-redundantized 16S sequence
    representative with default parameters
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:a7f67ecb92822b608118d711c092fdd5d2717a1e94660618634c9ad625767f3c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Primer Prospector
  - USEARCH
  edam_operation: http://edamontology.org/operation_0296
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# 16S rRNA primer binding assessment

## Summary

A method for quantifying what fraction of assembled 16S rRNA gene sequences would evade detection by a given amplicon-survey primer pair (here 515F/806R), by first de-redundantizing sequences at 97% identity with USEARCH and then scoring primer binding with Primer Prospector. It quantifies amplification bias of common 16S primers against divergent lineages such as the CPR.

## When to use

Use when you have curated 16S rRNA gene sequences (e.g. assembled from metagenomes, ≥800 bp) and need to estimate which taxa a specific primer pair would miss in an amplicon survey.

## When NOT to use

- Input is a raw amplicon read set or OTU/feature table rather than curated full-length (≥800 bp) 16S rRNA gene sequences — binding cannot be assessed without sequence spanning the primer sites
- 16S sequences still contain large (≥10 bp) intron/insertion sequences that were not removed, since insertions within the primer region would spuriously inflate evasion rates
- Sequences that do not span the 515–806 E. coli-coordinated region — they cannot be scored for 515F/806R binding and should be excluded rather than counted as evading

## Inputs

- Curated, intron-removed bacterial 16S rRNA gene sequences (≥800 bp), e.g. assembled and error-corrected from metagenomes (1,543 genes in the source study)
- Primer sequences for the amplicon-survey pair of interest (515F and 806R)
- Taxonomic/lineage assignment (phylum or CPR lineage label) for each sequence

## Outputs

- Per-sequence primer binding outcomes (binds / does not bind) for each primer
- Per-lineage evasion rates (e.g. ~50% Microgenomates, ~50% Saccharibacteria, 60% WWE3, 100% WS6)
- Overall percentage of all bacterial sequences missed by the primer pair (~20% in the study)
- Structured summary table primer_binding_analysis.csv

## How to apply

First de-redundantize the curated assembled 16S rRNA gene sequences by clustering at 97% sequence identity with USEARCH cluster_smallmem, enforcing query_cov 0.50 and target_cov 0.50 so partial sequences do not merge incorrectly. Retain only cluster representatives that span the 515–806 region of the E. coli 16S rRNA gene, since sequences lacking that span cannot be judged for 515F/806R binding. Score binding of the 515F and 806R primers against each retained representative using Primer Prospector with default parameters, aggregating per-sequence outcomes into per-lineage evasion rates. Report evasion rates for lineages of interest (e.g. Microgenomates, Saccharibacteria, WWE3, WS6) plus the overall percentage of all bacterial sequences missed, and write the summary as primer_binding_analysis.csv (mirroring Extended Data Fig. 7 of Brown et al. 2015).

## Related tools

- **USEARCH** (De-redundantizes assembled 16S rRNA gene sequences via cluster_smallmem at 97% identity (query_cov 0.50, target_cov 0.50) before primer binding scoring)
- **Primer Prospector** (Evaluates 515F/806R primer binding against each de-redundantized 16S sequence representative with default parameters)

## Examples

```
usearch -cluster_smallmem curated_16s.fa -id 0.97 -query_cov 0.50 -target_cov 0.50 -uc clusters.uc
```

## Evaluation signals

- Computed per-lineage evasion rates reproduce the reported pattern: ~50% Microgenomates, ~50% Saccharibacteria, 60% WWE3, and 100% of WS6 sequences not bound by 515F/806R
- Overall evasion across all bacterial sequences in the dataset is approximately 20%, consistent with the paper's reported figure
- Only cluster representatives spanning the 515–806 region of the E. coli 16S rRNA gene are included in the denominator; incomplete sequences are excluded
- Clustering parameters match the paper: 97% sequence identity with query_cov 0.50 and target_cov 0.50 in USEARCH cluster_smallmem
- Output primer_binding_analysis.csv has one evasion-rate row per lineage plus an all-bacteria row, reproducing Extended Data Fig. 7

## Limitations

- Evasion percentages are specific to the sequences sampled in the study (groundwater CPR and other bacteria); rates are not universal across environments or datasets
- Primer binding predictions are probabilistic ('would probably not bind'); mismatch scoring with Primer Prospector default parameters may not capture marginal binding behavior
- Large (≥10 bp) insertions/introns in 16S rRNA genes (31% of CPR genes in this study) must be removed beforehand, as they are spliced out of transcribed RNAs and would confound binding assessment

## Evidence

- [full_text] Primer binding analysis showed that 515F and 806R would probably not bind to 16S rRNA genes of ∼50% of Microgenomates, ∼50% of Saccharibacteria, 60% of WWE3, and 100% of WS6 sequences sampled here.: [evidence span withheld — anchor status: not_found]
- [methods] Assembled 16S rRNA gene sequences were clustered at 97% sequence identity using USEARCH (cluster_smallmem: "Assembled 16S rRNA gene sequences were clustered at 97% sequence identity using USEARCH (cluster_smallmem"
- [abstract] In fact, these primers would probably miss ∼20% of all bacteria detected in this study, including organisms outside the CPR.: "In fact, these primers would probably miss ∼20% of all bacteria detected in this study, including organisms outside the CPR."
