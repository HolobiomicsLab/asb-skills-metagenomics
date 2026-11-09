---
name: genotype-phenotype-association-interpretation
description: Use when a trained Traitar phenotype model (phypat or phypat+PGL) yields
  a ranked list of phenotype-relevant Pfam families and you need to assign biological
  meaning, accessions, and phenotype-to-family mappings for a trait of interest.
when_to_use_negative:
- The goal is trait prediction for new genomes rather than interpreting which Pfam
  families drive already-trained phenotype models
- Phenotype labels or protein family annotations come from tools other than Traitar's
  Pfam-based phypat/phypat+PGL models (e.g., PICA's clusters of orthologous genes),
  since rankings and accessions are Traitar-specific
- The phenotype of interest is not among the 67 phenotypes Traitar predicts
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0176
- http://edamontology.org/topic_0621
tools:
- name: Traitar
  role: Produces the phenotype classification models and the phenotype-relevant protein
    family rankings that this skill interprets; also inspects model features via 'traitar
    show'
  repo: https://github.com/hzi-bifo/traitar
- name: pip
  role: Installs the traitar package and its Python dependencies (scipy, matplotlib,
    pandas)
- name: HMMER
  role: Underlying annotator that assigns Pfam families to predicted coding sequences,
    providing the protein family profiles used by Traitar's classifiers
- name: prodigal
  role: Predicts open reading frames from nucleotide FASTA input as the first step
    of the standard Traitar workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:a00cfa8df2703a36f619fe5512285895d81096b045328e0e03af5828f666501f
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
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0176
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# genotype-phenotype-association-interpretation

## Summary

Interprets the Pfam protein families that Traitar's phypat/phypat+PGL phenotype models rank as important for specific bacterial traits, linking families such as flagellar FliS (PF02561) to phenotypes like motility, nitrate-to-nitrite conversion, and l-arabinose fermentation.

## When to use

Use when a trained Traitar phenotype model (phypat or phypat+PGL) yields a ranked list of phenotype-relevant Pfam families and you need to assign biological meaning, accessions, and phenotype-to-family mappings for a trait of interest.

## When NOT to use

- The goal is trait prediction for new genomes rather than interpreting which Pfam families drive already-trained phenotype models
- Phenotype labels or protein family annotations come from tools other than Traitar's Pfam-based phypat/phypat+PGL models (e.g., PICA's clusters of orthologous genes), since rankings and accessions are Traitar-specific
- The phenotype of interest is not among the 67 phenotypes Traitar predicts

## Inputs

- Traitar phenotype model output: Table S6 phenotype–Pfam family rankings from phypat or phypat+PGL classifiers
- Pfam 27.0 HMM models (Pfam-A.hmm) used for protein family annotation
- Bacterial genome sequences (nucleotide FASTA) or pre-computed protein family profiles for re-running the traitar workflow

## Outputs

- Verified phenotype–Pfam family–accession mappings (e.g., pfam_phenotype_rankings.tsv)
- Top-ranked Pfam families per phenotype with accession numbers and functional annotations

## How to apply

Install traitar via pip together with its Python dependencies (scipy, matplotlib, pandas), then obtain the Pfam 27.0 HMM models via 'traitar pfam <path>' or register a local Pfam-A.hmm folder with 'traitar pfam --local'. Load the Table S6 phenotype–Pfam ranking output from the trained phypat/phypat+PGL models and filter the rankings to the phenotypes of interest (e.g., motility, nitrate-to-nitrite conversion, l-arabinose fermentation). Extract the top-ranked Pfam families per phenotype and annotate each with its Pfam accession (e.g., FliS PF02561, MotB PF13677, CZB PF13682, Nar PF02665/PF02613, NapB PF03892, NapD PF03927, l-arabinose isomerase PF11762, arabinofuranosidase PF06964). Cross-check extracted accessions against the reference list and write verified phenotype–family–accession mappings to an output table such as pfam_phenotype_rankings.tsv. The rationale is that Traitar links phenotype predictions to the Pfam families that contribute to classification, so top-ranked families reveal the mechanistic basis of each phenotype model.

## Related tools

- **Traitar** (Produces the phenotype classification models and the phenotype-relevant protein family rankings that this skill interprets; also inspects model features via 'traitar show') — https://github.com/hzi-bifo/traitar
- **pip** (Installs the traitar package and its Python dependencies (scipy, matplotlib, pandas))
- **HMMER** (Underlying annotator that assigns Pfam families to predicted coding sequences, providing the protein family profiles used by Traitar's classifiers)
- **prodigal** (Predicts open reading frames from nucleotide FASTA input as the first step of the standard Traitar workflow)

## Examples

```
pip install traitar-<version>.tar.gz --user; traitar pfam ~/pfam; traitar show 'Motility' --predictor phypat
```

## Evaluation signals

- Every reported reference accession (PF02561, PF13677, PF13682, PF02665, PF02613, PF03892, PF03927, PF11762, PF06964) appears in the output table under its correct phenotype (script_runs / contains_substring check)
- Each extracted Pfam family is annotated with a valid Pfam accession matching the Pfam 27.0 release
- The three phenotypes of interest (motility, nitrate-to-nitrite conversion, l-arabinose fermentation) each have non-empty top-ranked family lists in the filtered rankings
- The output table schema (phenotype, family, accession) matches the reference mapping with no mismatched phenotype–accession pairs
- Rankings reproduce the ordering reported in the paper's Table 3 for the selected phenotypes

## Limitations

- For incomplete genomes, the absence of a phenotype prediction may be due to the absence of the relevant protein families from the input genomes
- Genotype-phenotype inference with Traitar only takes into account the presence and absence of protein families; other omics data are not integrated
- The Pfam 27.0 HMM models are not distributed with the traitar package and must be downloaded or registered locally before annotation and model inspection

## Evidence

- [full_text] Traitar selected the flagellar export chaperone FliS (accession no. PF02561), which inhibits early polymerization of the flagellar filament FliC.: [evidence span withheld — anchor status: not_found]
- [intro] we discuss the protein families Traitar identified for several phenotypes, namely, for motility, nitrate-to-nitrite conversion, and l-arabinose fermentation.: "we discuss the protein families Traitar identified for several phenotypes, namely, for motility, nitrate-to-nitrite conversion, and l-arabinose fermentation"
- [methods] traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded: "traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded"
- [readme] Traitar can be used to inspect the protein families in each phenotype model: traitar show 'Glucose fermenter': [evidence span withheld — anchor status: external_doc]
- [discussion] At the moment, genotype-phenotype inference with Traitar only takes into account the presence and absence of protein families of the bacteria analyzed.: "At the moment, genotype-phenotype inference with Traitar only takes into account the presence and absence of protein families of the bacteria analyzed."
