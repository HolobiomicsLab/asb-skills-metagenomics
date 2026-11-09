---
name: phenotype-associated-protein-family-ranking
description: Use when Traitar (Pfam 27.0-based, phypat or phypat+PGL models) has been
  applied to genomes and you need the ranked Pfam families underlying a specific phenotype
  prediction, e.g. to interpret a trait assignment mechanistically.
when_to_use_negative:
- 'The genome of interest is highly incomplete: absence of a phenotype prediction
  may simply reflect missing protein families in the input genome, so family rankings
  would be misleading'
- You need trait prediction from data other than presence/absence of protein families
  - Traitar currently uses only Pfam presence/absence and does not integrate other
  omics data
- 'You want to predict plant biomass phenotypes: the plant biomass predictor is not
  included in the released Traitar software'
edam_operation: http://edamontology.org/operation_0415
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: traitar
  role: Trains phypat/phypat+PGL phenotype classifiers on Pfam profiles and exposes
    phenotype-relevant Pfam families via rankings and the 'traitar show' command
  repo: https://github.com/hzi-bifo/traitar
- name: HMMER
  role: Annotates predicted coding sequences with Pfam families that feed the phenotype
    models
- name: prodigal
  role: Predicts open reading frames from nucleotide input prior to Pfam annotation
- name: pip
  role: Installs the traitar package and its Python dependencies (scipy, matplotlib,
    pandas)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:564515534521884433d8c5348de0b981c30b24e9504d51a6baa17e5f0761e404
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hzi-bifo/traitar
  tools:
  - GNU parallel
  - HMMER
  - pip
  - prodigal
  - traitar
  edam_operation: http://edamontology.org/operation_0415
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# phenotype-associated-protein-family-ranking

## Summary

Extracts and verifies the Pfam protein families that Traitar's phypat/phypat+PGL classifiers rank as most important for specific phenotypes, such as motility (flagellar export chaperone FliS, PF02561), nitrate-to-nitrite conversion (Nar PF02665/PF02613, NapB PF03892, NapD PF03927), and l-arabinose fermentation (l-arabinose isomerase PF11762, arabinofuranosidase PF06964).

## When to use

Use when Traitar (Pfam 27.0-based, phypat or phypat+PGL models) has been applied to genomes and you need the ranked Pfam families underlying a specific phenotype prediction, e.g. to interpret a trait assignment mechanistically.

## When NOT to use

- The genome of interest is highly incomplete: absence of a phenotype prediction may simply reflect missing protein families in the input genome, so family rankings would be misleading
- You need trait prediction from data other than presence/absence of protein families - Traitar currently uses only Pfam presence/absence and does not integrate other omics data
- You want to predict plant biomass phenotypes: the plant biomass predictor is not included in the released Traitar software

## Inputs

- Traitar phenotype-Pfam feature ranking output (Table S6 / phypat and phypat+PGL model feature tables)
- Pfam 27.0 HMM models (Pfam-A.hmm.gz from ftp://ftp.ebi.ac.uk/pub/databases/Pfam/releases/Pfam27.0/Pfam-A.hmm.gz)
- Bacterial genome FASTAs (nucleotide) or pre-called gene FASTAs used to train/apply the models

## Outputs

- pfam_phenotype_rankings.tsv mapping each phenotype of interest to its ranked Pfam families and accessions
- Verified phenotype-family-accession associations (e.g. motility -> FliS PF02561; nitrate-to-nitrite -> Nar PF02665/PF02613; l-arabinose fermentation -> PF11762)

## How to apply

Install traitar via pip (pip install traitar-<version>.tar.gz) together with scipy, matplotlib and pandas, then register the Pfam 27.0 HMM models either via 'traitar pfam <path>' (auto-download of Pfam-A.hmm.gz) or with 'traitar pfam --local <path>'. Run or load the Table S6 phenotype-Pfam ranking output from the trained phypat/phypat+PGL models, filtering to the phenotypes of interest (e.g. motility, nitrate-to-nitrite conversion, l-arabinose fermentation). For each phenotype, extract the top-ranked Pfam families, annotate each with its Pfam accession, and cross-check against the reference list (FliS PF02561, MotB PF13677, CZB PF13682, Nar PF02665/PF02613, NapB PF03892, NapD PF03927, l-arabinose isomerase PF11762, arabinofuranosidase PF06964). The traitar 'show' command with --predictor can independently expose the majority features (Pfam families) contributing to a trait for a chosen classifier. Write the verified phenotype-family-accession mappings to a table (e.g. pfam_phenotype_rankings.tsv).

## Related tools

- **traitar** (Trains phypat/phypat+PGL phenotype classifiers on Pfam profiles and exposes phenotype-relevant Pfam families via rankings and the 'traitar show' command) — https://github.com/hzi-bifo/traitar
- **HMMER** (Annotates predicted coding sequences with Pfam families that feed the phenotype models)
- **prodigal** (Predicts open reading frames from nucleotide input prior to Pfam annotation)
- **pip** (Installs the traitar package and its Python dependencies (scipy, matplotlib, pandas))

## Examples

```
traitar show 'Glucose fermenter' --predictor phypat
```

## Evaluation signals

- Every reported accession (PF02561, PF13677, PF13682, PF02665, PF02613, PF03892, PF03927, PF11762, PF06964) appears in the output table under its correct phenotype (script_runs / contains_substring check)
- FliS PF02561 is recovered as a top-ranked Pfam family for the motility phenotype
- Nitrate-to-nitrite conversion rankings contain both Nar accessions (PF02665 and PF02613) plus NapB PF03892 and NapD PF03927
- l-arabinose fermentation rankings contain PF11762 (l-arabinose isomerase) and PF06964 (arabinofuranosidase)
- traitar show output for a phenotype contains the same majority Pfam features as the extracted ranking, for the same --predictor

## Limitations

- Genotype-phenotype inference with Traitar only takes into account the presence and absence of protein families; other omics data are not integrated
- For incomplete genomes, absence of a phenotype prediction may be due to the absence of the relevant protein families from the input genomes
- The plant biomass predictor is not included in the Traitar release
- Traitar requires the Pfam 27.0 HMM models, which are not distributed with the package and must be downloaded separately

## Evidence

- [intro] we discuss the protein families Traitar identified for several phenotypes, namely, for motility, nitrate-to-nitrite conversion, and l-arabinose fermentation.: "we discuss the protein families Traitar identified for several phenotypes, namely, for motility, nitrate-to-nitrite conversion, and l-arabinose fermentation."
- [methods] traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded: "traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded"
- [methods] pip install traitar-<version>.tar.gz --user: "pip install traitar-<version>.tar.gz --user"
- [readme] will show the majority features i.e. the Pfam families that contribute to the assignment of the trait Glucose fermenter: [evidence span withheld — anchor status: external_doc]
- [readme] Traitar will link the protein families and predicted phenotypes.: [evidence span withheld — anchor status: external_doc]
