---
name: flagellar-motility-gene-verification
description: Use when validating that a trained Traitar phenotype model links the
  correct Pfam families to flagellar motility, nitrate reduction, or l-arabinose phenotypes,
  e.g. reproducing Table 3 accessions from Table S6 rankings.
when_to_use_negative:
- The task is phenotype prediction for new genomes (use 'traitar phenotype' from nucleotides/genes
  instead) rather than inspecting or verifying phenotype model protein families.
- The input already consists of curated gene–phenotype associations (e.g. PICA ortholog-cluster
  data), since Traitar's Pfam-based ranking pipeline does not apply.
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0176
- http://edamontology.org/topic_0621
tools:
- name: Traitar
  role: Predicts phenotypes from Pfam profiles and exposes phenotype-relevant protein
    families per trait via its phypat and phypat+PGL classifiers and 'traitar show'
    inspection
  repo: https://github.com/hzi-bifo/traitar
- name: pip
  role: Installs the traitar package and its Python dependencies (scipy, matplotlib,
    pandas)
- name: HMMER
  role: Annotates coding sequences with Pfam families during the Traitar workflow,
    feeding the family rankings inspected here
- name: prodigal
  role: Predicts open reading frames from nucleotide input in the standard Traitar
    workflow prior to Pfam annotation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:1ff479d525d7e4a43843d5ac8bdc618960c32b95390f7be51a881d234ef51d50
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hzi-bifo/traitar
  tools:
  - GNU parallel
  - HMMER
  - Traitar
  - pip
  - prodigal
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0176
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# flagellar-motility-gene-verification

## Summary

Reproduce and verify Traitar's phenotype-to-Pfam associations for motility, nitrate-to-nitrite conversion, and l-arabinose fermentation by extracting top-ranked Pfam families (e.g. flagellar export chaperone FliS, PF02561) from the trained phypat/phypat+PGL model rankings and cross-checking accessions against the paper's reference list.

## When to use

Use when validating that a trained Traitar phenotype model links the correct Pfam families to flagellar motility, nitrate reduction, or l-arabinose phenotypes, e.g. reproducing Table 3 accessions from Table S6 rankings.

## When NOT to use

- The task is phenotype prediction for new genomes (use 'traitar phenotype' from nucleotides/genes instead) rather than inspecting or verifying phenotype model protein families.
- The input already consists of curated gene–phenotype associations (e.g. PICA ortholog-cluster data), since Traitar's Pfam-based ranking pipeline does not apply.

## Inputs

- Table S6 phenotype–Pfam family ranking output from trained phypat/phypat+PGL Traitar models
- Pfam 27.0 Pfam-A.hmm HMM models (downloaded or registered locally)
- Trained phenotype classification models for the 67 Traitar traits

## Outputs

- pfam_phenotype_rankings.tsv with verified phenotype–Pfam family–accession mappings for motility, nitrate-to-nitrite conversion, and l-arabinose fermentation
- List of top-ranked Pfam accessions per phenotype (e.g. PF02561 for motility) with mismatch flags

## How to apply

Install traitar via pip ('pip install traitar-<version>.tar.gz') with its Python dependencies (scipy, matplotlib, pandas) and register Pfam 27.0 HMM models, either auto-downloaded via 'traitar pfam <path>' or manually from ftp://ftp.ebi.ac.uk/pub/databases/Pfam/releases/Pfam27.0/Pfam-A.hmm.gz with 'traitar pfam --local <path>'. Load the Table S6 phenotype–Pfam ranking output from the trained phypat or phypat+PGL models and filter to the three phenotypes of interest: motility, nitrate-to-nitrite conversion, and l-arabinose fermentation. Extract the top-ranked Pfam families per phenotype, annotate each with its Pfam accession, and cross-check against the reference set: FliS PF02561 and MotB PF13677/CZB PF13682 (motility), Nar PF02665/PF02613, NapB PF03892, NapD PF03927 (nitrate-to-nitrite), and l-arabinose isomerase PF11762 plus arabinofuranosidase PF06964 (l-arabinose). Flag any mismatch and write the verified phenotype–family–accession mappings to pfam_phenotype_rankings.tsv.

## Related tools

- **Traitar** (Predicts phenotypes from Pfam profiles and exposes phenotype-relevant protein families per trait via its phypat and phypat+PGL classifiers and 'traitar show' inspection) — https://github.com/hzi-bifo/traitar
- **pip** (Installs the traitar package and its Python dependencies (scipy, matplotlib, pandas))
- **HMMER** (Annotates coding sequences with Pfam families during the Traitar workflow, feeding the family rankings inspected here)
- **prodigal** (Predicts open reading frames from nucleotide input in the standard Traitar workflow prior to Pfam annotation)

## Examples

```
pip install traitar-<version>.tar.gz --user && traitar pfam <path>
```

## Evaluation signals

- Every reported accession (PF02561, PF13677, PF13682, PF02665, PF02613, PF03892, PF03927, PF11762, PF06964) appears in pfam_phenotype_rankings.tsv under its correct phenotype.
- FliS (PF02561) is ranked among the top Pfam families for the motility phenotype, matching the paper's finding that it inhibits early polymerization of the flagellar filament FliC.
- A script_runs / contains_substring check on the output TSV confirms each phenotype–accession pair.
- The extracted families match the paper's Table 3 reference list exactly, with zero mismatches flagged during cross-check.

## Limitations

- Traitar currently uses only presence/absence of protein families; other omics data are not integrated into genotype–phenotype inference.
- Pfam 27.0 HMM models are not distributed with the package and must be downloaded or registered locally before any annotation or model inspection.
- For incomplete genomes, absence of a phenotype prediction may simply reflect missing protein families in the input genome.

## Evidence

- [results] Among the Pfam families important for classifying the motility phenotype, Traitar selected the flagellar export chaperone FliS (accession no. PF02561): [evidence span withheld — anchor status: not_found]
- [methods] Traitar requires Pfam 27.0 models: "traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded"
- [readme] traitar show inspects phenotype model families: [evidence span withheld — anchor status: external_doc]
- [readme] Traitar links protein families and predicted phenotypes: [evidence span withheld — anchor status: external_doc]
- [intro] we discuss protein families identified for several phenotypes: "we discuss the protein families Traitar identified for several phenotypes, namely, for motility, nitrate-to-nitrite conversion, and l-arabinose fermentation."
