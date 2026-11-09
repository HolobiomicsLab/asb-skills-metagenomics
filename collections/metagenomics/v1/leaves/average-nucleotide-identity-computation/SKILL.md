---
name: average-nucleotide-identity-computation
description: Use when MAG or genome FASTA assemblies must be assigned species labels
  relative to a reference genome database via closest-relative search plus AQ/ANI
  alignment thresholds.
when_to_use_negative:
- The input is metagenomic sequencing reads to be classified, not genome assemblies
  — read-level classification is the job of DeepMicrobes, not Mash/MUMmer genome comparison.
- Query genomes are so distant from all references that AQ <= 60% or ANI < 95% — the
  criterion only supports same-species labeling, not de novo species delineation or
  higher-rank assignment.
- A pre-computed taxonomy for the assemblies already exists (e.g., CheckM tree_qa
  output from checkm_assessment.sh) and re-deriving labels via closest-relative alignment
  would be redundant.
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: Mash
  role: MinHash sketching and mash dist to identify each MAG's closest relative reference
    genome by lowest distance (tested v2.0 in mashdiff.sh)
- name: MUMmer
  role: Whole-genome alignment (nucmer/dnadiff) of each MAG against its closest relative
    to compute aligned query fraction (AQ) and ANI (tested v3.23 in mashdiff.sh)
- name: mashdiff.sh
  role: Wrapper pipeline that runs the Mash closest-relative selection and MUMmer
    whole-genome alignment, emitting the parsed dnadiff/Mash results table
  repo: https://github.com/Finn-Lab/MGS-gut
- name: R
  role: Used for R scripts evaluating mashdiff results (e.g., mashdiff-hist_scatter_fig1a.R,
    mashdiff-counts_fig1b.R) and downstream label/threshold handling
  repo: https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:d96a9003e0d58d1d43df76f9ca6f8476bb04ca9859dc5ce6849208ad7c481840
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/MicrobeLab/DeepMicrobes
  repo_url: https://github.com/Finn-Lab/MGS-gut
  tools:
  - DeepMicrobes
  - MUMmer
  - Mash
  - R
  - mashdiff.sh
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# average-nucleotide-identity-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns species labels to MAG assemblies by first finding each MAG's closest relative among the 2505 human gut reference (HGR) genomes with Mash MinHash distances, then computing aligned query fraction (AQ) and ANI from MUMmer 3.23 whole-genome alignments, labeling same-species only when AQ > 60% and ANI > 95%.

## When to use

Use when MAG or genome FASTA assemblies must be assigned species labels relative to a reference genome database via closest-relative search plus AQ/ANI alignment thresholds.

## When NOT to use

- The input is metagenomic sequencing reads to be classified, not genome assemblies — read-level classification is the job of DeepMicrobes, not Mash/MUMmer genome comparison.
- Query genomes are so distant from all references that AQ <= 60% or ANI < 95% — the criterion only supports same-species labeling, not de novo species delineation or higher-rank assignment.
- A pre-computed taxonomy for the assemblies already exists (e.g., CheckM tree_qa output from checkm_assessment.sh) and re-deriving labels via closest-relative alignment would be redundant.

## Inputs

- 3269 gut-derived MAG assemblies in FASTA format (ERP108418)
- 2505 HGR training reference genome FASTA files
- Mash sketch file (.msh) generated with mash sketch from the reference genomes

## Outputs

- mag_species_assignment.tsv with one row per MAG containing closest-relative, AQ, ANI, and species-label columns (3269 rows expected)
- per-query *dbname_parsed.tab files with columns: Query name / Reference name / Ref length / % Ref covered / Query length / % Query aligned / ANI / Mash distance

## How to apply

Collect the MAG assemblies (e.g., 3269 high-quality gut-derived MAGs from ERP108418) and the 2505 HGR training genome FASTA files. Sketch every genome with mash sketch (MinHash of k-mers; README tested Mash v2.0) and run mash dist between each MAG and all reference sketches to find the closest relative by lowest Mash distance. Align each MAG against its closest reference with MUMmer 3.23 (nucmer/dnadiff, wrapped by the mashdiff.sh script from Finn-Lab/MGS-gut) and compute the aligned-query fraction (AQ, '% Query aligned') and ANI from the alignment. Apply the species-assignment criterion used in the DeepMicrobes benchmark: label a MAG as the same species as its closest relative only when AQ > 60% and ANI > 95%; pairs with ANI < 95% (e.g., the 121 out-of-database genomes in Table S6) are treated as different species. Write per-MAG closest-relative, Mash distance, AQ, ANI, and species-label fields to mag_species_assignment.tsv.

## Related tools

- **Mash** (MinHash sketching and mash dist to identify each MAG's closest relative reference genome by lowest distance (tested v2.0 in mashdiff.sh))
- **MUMmer** (Whole-genome alignment (nucmer/dnadiff) of each MAG against its closest relative to compute aligned query fraction (AQ) and ANI (tested v3.23 in mashdiff.sh))
- **mashdiff.sh** (Wrapper pipeline that runs the Mash closest-relative selection and MUMmer whole-genome alignment, emitting the parsed dnadiff/Mash results table) — https://github.com/Finn-Lab/MGS-gut
- **R** (Used for R scripts evaluating mashdiff results (e.g., mashdiff-hist_scatter_fig1a.R, mashdiff-counts_fig1b.R) and downstream label/threshold handling) — https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts

## Examples

```
mashdiff.sh -i genome_folder/ -r reference.msh -s db_name -p output_prefix
```

## Evaluation signals

- mag_species_assignment.tsv exists with exactly one row per MAG (3269 rows) and contains AQ, ANI, and species-label fields for every row.
- Same-species labels occur only for rows with AQ > 60% and ANI > 95%; no row violating either threshold carries a same-species label.
- Each MAG is paired with exactly one closest reference genome, the one with the lowest mash dist value across the 2505 HGR sketches.
- Parsed per-query output columns match the documented header: Query name / Reference name / Ref length / % Ref covered / Query length / % Query aligned / ANI / Mash distance.
- ANI values fall within [0, 100] and AQ values within [0, 100]; Mash distances are within [0, 1].

## Limitations

- mashdiff.sh and checkm_assessment.sh were coded for LSF cluster environments and require Python 2.7; the pipeline may not run unchanged on modern schedulers or Python 3.
- Same-species assignment is conservative and binary: pairs with AQ <= 60% or ANI < 95% (as for the 121 genomes of species absent from RefSeq/training in Table S6) are not labeled as the same species even if closely related.
- Fragmented MAG assemblies can depress '% Query aligned' (AQ), causing genuinely same-species pairs to fall below the AQ > 60% cutoff and be mislabeled as different species.

## Evidence

- [readme] Compare a set of genomes against a reference database. Selects best representative for whole-genome alignment.: [evidence span withheld — anchor status: external_doc]
- [readme] Output columns of mashdiff.sh results: [evidence span withheld — anchor status: not_found]
- [intro] 2505-species training repertoire from the article intro: "we trained DeepMicrobes on the previously defined complete bacterial repertoire of the human gut microbiota (2). The repertoire is composed of 2505 species"
- [readme] 3269 benchmark MAGs listed in DeepMicrobes-data supplementary data: [evidence span withheld — anchor status: external_doc]
- [supplementary] Table S6 false-positive criterion for out-of-database genomes: [evidence span withheld — anchor status: external_doc]
