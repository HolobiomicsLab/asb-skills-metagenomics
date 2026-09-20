---
name: usco-presence-absence-matrix-construction
description: Use when a set of related MAGs/species-level genome bins (e.g. SGB009,
  SGB034, SGB080) must be screened for shared absence of universal single-copy orthologs
  to infer lineage-specific metabolic gene loss.
when_to_use_negative:
- Input genomes have not passed quality screening (article required ≥50% completeness
  and <5% contamination) — missing USCOs in low-quality MAGs may reflect incompleteness,
  not gene loss
- The goal is general gene-family clustering or pan-genome construction across all
  genes, not assessment of universal single-copy ortholog presence/absence
- Assessing genome quality itself (BUSCO-based completeness/contamination scoring)
  rather than mining a shared-loss pattern across a lineage
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_3697
tools:
- name: BUSCO
  role: Runs in genome mode (-m genome) against bacteria_odb10 to report single-copy,
    duplicated, fragmented and missing statuses for the 124 USCOs; full_table.tsv
    is parsed to build the matrix
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:331f6fdfa5aba6fe4db698f2d9dd3a1f67d165af905a3992fb482bc026f29d64
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BUSCO
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# usco-presence-absence-matrix-construction

## Summary

Constructs a genome-by-gene presence/absence matrix of the 124 universal single-copy orthologs (USCOs) from BUSCO bacteria_odb10 full_table.tsv outputs, to detect gene losses shared across a lineage (e.g. Saccharofermentanales in the VMGC) and flag missing genes in de novo purine biosynthesis.

## When to use

Use when a set of related MAGs/species-level genome bins (e.g. SGB009, SGB034, SGB080) must be screened for shared absence of universal single-copy orthologs to infer lineage-specific metabolic gene loss.

## When NOT to use

- Input genomes have not passed quality screening (article required ≥50% completeness and <5% contamination) — missing USCOs in low-quality MAGs may reflect incompleteness, not gene loss
- The goal is general gene-family clustering or pan-genome construction across all genes, not assessment of universal single-copy ortholog presence/absence
- Assessing genome quality itself (BUSCO-based completeness/contamination scoring) rather than mining a shared-loss pattern across a lineage

## Inputs

- Genome FASTA files of target species-level genome bins (e.g. Saccharofermentanales SGB009, SGB034, SGB080 MAGs from VMGC_prokaryote_SGB.tar.gz / VMGC_prokaryote_MAG.tar.gz, Zenodo record 10457006)
- BUSCO 5.4.2 with the bacteria_odb10 lineage database (124 USCOs for prokaryotes)

## Outputs

- Per-genome BUSCO full_table.tsv reports classifying each of the 124 USCOs as single-copy, duplicated, fragmented or missing
- Genome-by-USCO presence/absence matrix (complete single-copy USCO = present; fragmented/missing = absent)
- List of USCOs absent in all target genomes, with functional annotations (e.g. KEGG pathway assignment)

## How to apply

Retrieve the target genome FASTA files (e.g. Saccharofermentanales SGB009, SGB034, SGB080 and related MAGs from VMGC Zenodo record 10457006) and run BUSCO 5.4.2 in genome mode (-m genome, quiet mode -q) against the bacteria_odb10 lineage, which contains the 124 USCOs for prokaryotes. Parse each run's full_table.tsv to classify each of the 124 USCOs as single-copy, duplicated, fragmented or missing per genome. Assemble a genome-by-USCO matrix encoding complete single-copy USCOs as present and fragmented/missing USCOs as absent. Identify USCOs absent in every genome in the set and annotate them by functional category (e.g. KEGG pathways) to flag those involved in de novo purine biosynthesis. Rationale: USCOs are near-universally conserved single-copy prokaryotic markers, so their consistent absence across a whole order is a strong signal of genuine gene loss rather than assembly incompleteness (the article's quality screening already enforced ≥50% completeness and <5% contamination).

## Related tools

- **BUSCO** (Runs in genome mode (-m genome) against bacteria_odb10 to report single-copy, duplicated, fragmented and missing statuses for the 124 USCOs; full_table.tsv is parsed to build the matrix)

## Examples

```
busco -m genome -q --lineage_dataset bacteria_odb10 -i SGB009.fa -o busco_SGB009; grep -E "Missing|Fragmented" busco_SGB009/busco_down/run_bacteria_odb10/full_table.tsv
```

## Evaluation signals

- Matrix has exactly one row per Saccharofermentanales genome and exactly 124 USCO columns from bacteria_odb10
- Reproduces the reported pattern that nine USCO genes are absent in all Saccharofermentanales genomes, consistent with Fig. 3c and Extended Data Fig. 4c
- Five of the nine absent USCOs encode enzymes involved in de novo purine biosynthesis per KEGG annotation
- Each USCO status in the matrix is traceable to a corresponding line in the BUSCO run's full_table.tsv (single-copy vs fragmented/missing)
- Genomes used pass the article's quality thresholds (≥50% completeness, <5% contamination) so absence reflects gene loss rather than incompleteness

## Limitations

- CheckM2 completeness may be underestimated for some genomes, which could confound presence/absence interpretation; the article notes 'the completeness of A. indicium and A. nucleatus might be underestimated in the current CheckM2 algorithm'
- Fragmented USCOs are encoded as absent, so truly present but partially assembled genes will be counted as missing
- Findings apply to VMGC MAGs; the article notes vaginal microbial sequence capture is incomplete, especially for non-western populations, so other lineages/datasets may differ

## Evidence

- [abstract] Similar assessment results were also revealed by other algorithms such as CheckM and BUSCO (based on 124 universal single-copy orthologs (USCOs) for prokaryotes32): "Similar assessment results were also revealed by other algorithms such as CheckM and BUSCO (based on 124 universal single-copy orthologs (USCOs) for prokaryotes32)"
- [abstract] yielded 18,570 prokaryotic genomes with ≥50% completeness and <5% contamination: "yielded 18,570 prokaryotic genomes with ≥50% completeness and <5% contamination"
- [readme] The microbial sequence data for VMGC is stored at https://zenodo.org/records/10457006: "The microbial sequence data for VMGC is stored at https://zenodo.org/records/10457006"
- [abstract] Approximately 67.4% of protein-coding genes could be accommodated in the Kyoto Encyclopedia of Genes and Genomes (KEGG) database21: "Approximately 67.4% of protein-coding genes could be accommodated in the Kyoto Encyclopedia of Genes and Genomes (KEGG) database21"
