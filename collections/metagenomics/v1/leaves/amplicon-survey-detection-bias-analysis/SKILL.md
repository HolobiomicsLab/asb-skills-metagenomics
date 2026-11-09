---
name: amplicon-survey-detection-bias-analysis
description: Use when you have curated 16S rRNA gene sequences (e.g. assembled and
  intron-stripped) and need to estimate which taxa a specific primer pair would fail
  to amplify in a cultivation-independent amplicon survey.
when_to_use_negative:
- Input is raw metagenomic reads rather than curated, intron-stripped 16S rRNA gene
  sequences — insertion sequences (up to 2,004 bp) must be removed first or binding
  calls will be confounded
- The question is abundance estimation in amplicon data itself, not detection-bias
  assessment of primer coverage
- Sequences do not span the 515-806 E. coli coordinate region — partial sequences
  cannot support a valid binding call
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0637
tools:
- name: USEARCH
  role: De-redundantize curated 16S rRNA gene sequences by clustering at 97% sequence
    identity with cluster_smallmem (query_cov 0.50, target_cov 0.50)
- name: Primer Prospector
  role: Assess binding of the 515F and 806R primer pair against each retained 16S
    sequence with default parameters
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:8ead435057351ab24bd944ae2a38fb07e74cf88e13fbeda4356cb6a4f2e30dce
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Primer Prospector
  - USEARCH
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# amplicon-survey-detection-bias-analysis

## Summary

Quantifies what fraction of bacterial 16S rRNA gene sequences would fail to bind common amplicon-survey primers (515F/806R), per lineage and overall. This skill reproduces the primer-evasion analysis from Brown et al. 2015, which showed 515F/806R would miss ~50% of Microgenomates and Saccharibacteria, 60% of WWE3, 100% of WS6, and ~20% of all bacteria sampled.

## When to use

Use when you have curated 16S rRNA gene sequences (e.g. assembled and intron-stripped) and need to estimate which taxa a specific primer pair would fail to amplify in a cultivation-independent amplicon survey.

## When NOT to use

- Input is raw metagenomic reads rather than curated, intron-stripped 16S rRNA gene sequences — insertion sequences (up to 2,004 bp) must be removed first or binding calls will be confounded
- The question is abundance estimation in amplicon data itself, not detection-bias assessment of primer coverage
- Sequences do not span the 515-806 E. coli coordinate region — partial sequences cannot support a valid binding call

## Inputs

- Curated assembled 16S rRNA gene sequences (e.g. FASTA; intron/insertion removed, >=800 bp)
- Primer sequences for the amplicon-survey pair being tested (e.g. 515F and 806R)
- Lineage/phylum assignments (e.g. Microgenomates, Saccharibacteria, WWE3, WS6) for each 16S sequence

## Outputs

- primer_binding_analysis.csv — per-lineage and overall primer-evasion percentages (e.g. ~50% Microgenomates, ~50% Saccharibacteria, 60% WWE3, 100% WS6)
- Per-sequence primer binding outcomes (bind / no-bind) for each primer of the pair

## How to apply

First curate the 16S rRNA gene sequences: remove any introns or large insertions (>=10 bp) and, as in the source study, cluster at 97% sequence identity using USEARCH cluster_smallmem with query_cov 0.50 and target_cov 0.50 to de-redundantize the 1,543 assembled bacterial 16S genes. Retain only cluster representatives spanning the 515-806 region of the E. coli 16S rRNA gene, excluding incomplete sequences, since primer binding cannot be assessed on partial coverage. Then assess binding of the 515F and 806R primers to each retained sequence with Primer Prospector using default mismatch parameters. Finally, aggregate per-sequence binding outcomes into per-lineage evasion rates (Microgenomates, Saccharibacteria, WWE3, WS6) and an all-bacteria overall rate, written out as a structured table such as primer_binding_analysis.csv. The rationale is that divergent 16S sequences and self-splicing introns in CPR organisms cause amplicon surveys to systematically miss members of the candidate phyla radiation.

## Related tools

- **USEARCH** (De-redundantize curated 16S rRNA gene sequences by clustering at 97% sequence identity with cluster_smallmem (query_cov 0.50, target_cov 0.50))
- **Primer Prospector** (Assess binding of the 515F and 806R primer pair against each retained 16S sequence with default parameters)

## Examples

```
usearch -cluster_smallmem curated_16s.fa -id 0.97 -query_cov 0.50 -target_cov 0.50 -uc 16s_clusters.uc
```

## Evaluation signals

- Computed evasion rates reproduce the reported pattern: ~50% of Microgenomates, ~50% of Saccharibacteria, 60% of WWE3, and 100% of WS6 sequences fail to bind 515F/806R
- Overall evasion across all bacterial 16S sequences in the study is ~20%, including organisms outside the CPR
- Only cluster representatives spanning the 515-806 region of the E. coli 16S rRNA gene are scored; incomplete sequences are excluded
- Clustering de-redundancy used 97% sequence identity with query_cov and target_cov of 0.50 in USEARCH cluster_smallmem
- Insertions >=10 bp were removed before analysis so binding calls reflect the spliced 16S sequence

## Limitations

- 31% of CPR 16S rRNA genes encode large (>=10 bp; max 2,004 bp) insertion sequences that must be removed, since they are probably self-splicing introns not retained in transcribed RNAs
- Binding calls apply to the sampled sequences; evasion percentages are study-specific and should not be extrapolated globally without additional sampling
- The conclusion is probabilistic ('would probably not bind'); Primer Prospector default parameters define the mismatch tolerance, and different stringency changes evasion rates

## Evidence

- [abstract] Primer binding analysis showed that primers extensively used in microbial surveys (515F and 806R 16 ) would probably not bind to 16S rRNA genes: "Primer binding analysis showed that primers extensively used in microbial surveys (515F and 806R 16 ) would probably not bind to 16S rRNA genes"
- [methods] Assembled 16S rRNA gene sequences were clustered at 97% sequence identity using USEARCH (cluster_smallmem: "Assembled 16S rRNA gene sequences were clustered at 97% sequence identity using USEARCH (cluster_smallmem"
