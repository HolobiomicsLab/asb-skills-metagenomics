---
name: l-arabinose-fermentation-marker-identification
description: Use when a trained Traitar phypat/phypat+PGL model's phenotype-Pfam ranking
  output (Table S6) must be inspected to name the protein family markers underlying
  l-arabinose fermentation predictions.
when_to_use_negative:
- You need phenotype predictions for new genomes rather than interpretation of model
  feature importance — use 'traitar phenotype' prediction output instead
- The trait of interest is not among the 67 phenotypes covered by Traitar's trained
  models (e.g. the plant biomass predictor was not included in the release)
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3299
tools:
- name: Traitar
  role: Trains phenotype classifiers over Pfam profiles and exposes phenotype-relevant
    protein family rankings; 'traitar show' inspects majority features per trait
  repo: https://github.com/hzi-bifo/traitar
- name: pip
  role: Installs the traitar package and its Python dependencies
- name: HMMER
  role: Annotates predicted coding sequences with Pfam families that feed the phenotype
    models
- name: prodigal
  role: Predicts open reading frames from nucleotide input before Pfam annotation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:2775ff0dcd6fe6b54d675aef828924507eebc1b5a084644a14e1c65a1127c918
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
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# l-arabinose-fermentation-marker-identification

## Summary

Identifies the Pfam protein families that Traitar's phenotype classifiers rank as most important for the l-arabinose fermentation phenotype, e.g. l-arabinose isomerase (PF11762) and arabinofuranosidase (PF06964), linking genome-derived protein family content to a carbohydrate-utilization trait.

## When to use

Use when a trained Traitar phypat/phypat+PGL model's phenotype-Pfam ranking output (Table S6) must be inspected to name the protein family markers underlying l-arabinose fermentation predictions.

## When NOT to use

- You need phenotype predictions for new genomes rather than interpretation of model feature importance — use 'traitar phenotype' prediction output instead
- The trait of interest is not among the 67 phenotypes covered by Traitar's trained models (e.g. the plant biomass predictor was not included in the release)

## Inputs

- Trained Traitar phypat/phypat+PGL phenotype-Pfam ranking output (Table S6)
- Pfam 27.0 HMM models (Pfam-A.hmm)
- Bacterial genome sequences (nucleotide or protein FASTA) if re-running classification

## Outputs

- pfam_phenotype_rankings.tsv mapping l-arabinose fermentation to Pfam families with accessions (e.g. PF11762 l-arabinose isomerase, PF06964 arabinofuranosidase)

## How to apply

Install Traitar via pip (with scipy, matplotlib, pandas) and obtain the Pfam 27.0 HMM models using 'traitar pfam <path>' or by registering a local Pfam-A.hmm download with 'traitar pfam --local <path>'. Run or load the phenotype classification output and open the Table S6 phenotype-Pfam ranking produced by the trained phypat/phypat+PGL models, then filter the rankings to the l-arabinose fermentation phenotype. Extract the top-ranked Pfam families and annotate each with its Pfam accession, cross-checking against the reference list (l-arabinose isomerase PF11762, arabinofuranosidase PF06964). The rationale is that Traitar's genotype-phenotype inference uses only presence/absence of protein families, so top-ranked families in the model reveal the mechanistic basis of the trait prediction. Write the verified phenotype-family-accession mappings to pfam_phenotype_rankings.tsv.

## Related tools

- **Traitar** (Trains phenotype classifiers over Pfam profiles and exposes phenotype-relevant protein family rankings; 'traitar show' inspects majority features per trait) — https://github.com/hzi-bifo/traitar
- **pip** (Installs the traitar package and its Python dependencies)
- **HMMER** (Annotates predicted coding sequences with Pfam families that feed the phenotype models)
- **prodigal** (Predicts open reading frames from nucleotide input before Pfam annotation)

## Examples

```
traitar show 'l-Arabinose fermenter' --predictor phypat+PGL
```

## Evaluation signals

- Every reported accession (PF02561, PF13677, PF13682, PF02665, PF02613, PF03892, PF03927, PF11762, PF06964) appears in the output table under its correct phenotype
- PF11762 and PF06964 are present in the l-arabinose fermentation rows of pfam_phenotype_rankings.tsv
- Rankings are restricted to the three phenotypes of interest (motility, nitrate-to-nitrite conversion, l-arabinose fermentation) with no unfiltered rows
- A script_runs / contains_substring check passes for each accession-phenotype mapping

## Limitations

- Traitar's genotype-phenotype inference takes into account only presence/absence of protein families; other omics data are not integrated
- For incomplete genomes, absence of a phenotype prediction may simply reflect missing protein families in the input genome
- Phenotype labels derive from GIDEON and Bergey's Manual, so marker families inherit the noise of those phenotype databases

## Evidence

- [intro] we discuss the protein families Traitar identified for several phenotypes, namely, for motility, nitrate-to-nitrite conversion, and l-arabinose fermentation: "we discuss the protein families Traitar identified for several phenotypes, namely, for motility, nitrate-to-nitrite conversion, and l-arabinose fermentation"
- [results] Traitar selected the flagellar export chaperone FliS (accession no. PF02561): [evidence span withheld — anchor status: not_found]
- [methods] traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded: "traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded"
- [readme] Traitar will link the protein families and predicted phenotypes.: [evidence span withheld — anchor status: external_doc]
