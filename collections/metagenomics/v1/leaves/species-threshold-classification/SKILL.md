---
name: species-threshold-classification
description: Use when assigning species-level labels to MAGs or draft genome assemblies
  relative to a curated reference genome set, requiring explicit AQ/ANI similarity
  thresholds rather than Mash distance alone.
when_to_use_negative:
- Genomes that are complete isolates with existing RefSeq taxonomy — a database lookup
  suffices and Mash/MUMmer thresholds add no labeling power
- When only Mash distance is available without a whole-genome alignment — Mash distance
  alone cannot enforce the AQ > 60% / ANI > 95% dual criterion
- Classification of short sequencing reads rather than assembled genomes — use a read-level
  classifier such as DeepMicrobes instead
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: Mash
  role: MinHash-sketches all MAG and HGR genomes; mash dist identifies each MAG's
    closest relative by lowest distance before alignment
- name: MUMmer
  role: nucmer/dnadiff alignment of each MAG against its closest relative to compute
    aligned-query fraction (AQ) and ANI
- name: R
  role: Summarizing mashdiff results and writing the per-MAG species assignment table
  repo: https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:c0490f4bfa18c6e593ecbc9fe84ac1632c6161455a150439f152541bb0af0fa4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/MicrobeLab/DeepMicrobes
  tools:
  - DeepMicrobes
  - MUMmer
  - Mash
  - R
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# species-threshold-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns species labels to gut-derived MAGs (3,269 assemblies from ERP108418) by finding each MAG's closest relative among the 2,505 HGR training genomes via lowest Mash distance, then labeling the MAG as the same species only when MUMmer-based aligned-query fraction (AQ) > 60% and ANI > 95%.

## When to use

Use when assigning species-level labels to MAGs or draft genome assemblies relative to a curated reference genome set, requiring explicit AQ/ANI similarity thresholds rather than Mash distance alone.

## When NOT to use

- Genomes that are complete isolates with existing RefSeq taxonomy — a database lookup suffices and Mash/MUMmer thresholds add no labeling power
- When only Mash distance is available without a whole-genome alignment — Mash distance alone cannot enforce the AQ > 60% / ANI > 95% dual criterion
- Classification of short sequencing reads rather than assembled genomes — use a read-level classifier such as DeepMicrobes instead

## Inputs

- 3,269 MAG assemblies from ERP108418 in FASTA format (with .fa extension for mashdiff.sh)
- 2,505 HGR training genome FASTA files (Table S1 metadata)
- Mash sketch file (.msh) of the reference genomes, generated with mash sketch
- MUMmer 3.23 alignment output (dnadiff/nucmer .tab or .delta files) per MAG-closest-relative pair

## Outputs

- mag_species_assignment.tsv: one row per MAG (3,269 rows) with columns for closest-relative genome, AQ (% query aligned), ANI, and same-species label
- Optional parsed mashdiff output (*dbname_parsed.tab): Query name / Reference name / Ref length / % Ref covered / Query length / % Query aligned / ANI / Mash distance

## How to apply

Collect the 3,269 MAG FASTA assemblies and the 2,505 HGR training genome FASTAs, then sketch every genome with Mash (MinHash k-mer sketches) to enable rapid pairwise distance estimation. Run mash dist between each MAG sketch and all HGR sketches to identify each MAG's closest relative genome. Align each MAG against its closest HGR relative with MUMmer 3.23 (nucmer/dnadiff-style) and compute the aligned-query fraction (AQ) and average nucleotide identity (ANI) from the alignment; the MGS-gut mashdiff pipeline reports '% Query aligned' and 'ANI' columns for exactly this purpose. Apply the paper's dual-threshold decision rule: label a MAG as the same species as its closest relative only when AQ > 60% AND ANI > 95%; pairs falling in the 10–60% aligned / ANI < 95% regime are treated as distinct (the paper used this window to define false-positive species classifications). Write per-MAG closest-relative, AQ, ANI, and label columns to mag_species_assignment.tsv.

## Related tools

- **Mash** (MinHash-sketches all MAG and HGR genomes; mash dist identifies each MAG's closest relative by lowest distance before alignment)
- **MUMmer** (nucmer/dnadiff alignment of each MAG against its closest relative to compute aligned-query fraction (AQ) and ANI)
- **R** (Summarizing mashdiff results and writing the per-MAG species assignment table) — https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts

## Examples

```
mashdiff.sh -i genome_folder/ -r reference.msh -s hgr -p mag
```

## Evaluation signals

- Output table mag_species_assignment.tsv exists with exactly 3,269 rows (one per MAG from ERP108418)
- AQ (% query aligned), ANI, and species-label columns are present and non-missing in every row
- Same-species labels occur only for MAG-relative pairs with AQ > 60% AND ANI > 95% — no label violates either threshold
- Each reported closest relative is the HGR genome with the lowest mash dist value across all 2,505 sketches for that MAG
- Pairs reported in the 10–60% aligned / ANI < 95% range are labeled as different species (false-positive control regime in Table S6)

## Limitations

- Fragmented MAGs reduce the aligned-query fraction, so incomplete assemblies may fail the AQ > 60% cutoff even when they belong to the same species
- mashdiff.sh was tested only with Mash v2.0, MUMmer v3.23, and Python 2.7, and is coded for LSF cluster environments
- Newly added species require updating the reference sketch/alignment set (and, for the downstream DeepMicrobes classifier, retraining the entire deep neural network)

## Evidence

- [supplementary] Table S6 defines the same-species boundary regime using aligned fraction and ANI: [evidence span withheld — anchor status: external_doc]
- [readme] The MGS-gut mashdiff pipeline performs the closest-relative whole-genome alignment this skill relies on: [evidence span withheld — anchor status: external_doc]
- [readme] The parsed mashdiff output reports the AQ and ANI values used for the dual-threshold decision: [evidence span withheld — anchor status: not_found]
- [supplementary] The benchmark MAGs are reads simulated from 3,269 gut-derived MAGs excluded from training: [evidence span withheld — anchor status: external_doc]
