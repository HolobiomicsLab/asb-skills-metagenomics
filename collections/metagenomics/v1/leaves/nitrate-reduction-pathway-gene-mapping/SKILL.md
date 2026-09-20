---
name: nitrate-reduction-pathway-gene-mapping
description: Use when bacterial genomes are being phenotyped with Traitar and you
  need to link the nitrate-to-nitrite conversion (nitrate reduction) phenotype to
  its supporting Pfam protein families and accessions.
when_to_use_negative:
- Input is already a phenotype prediction table and you only need the binary nitrate-reduction
  call — use Traitar's heatmap/prediction outputs directly instead of re-extracting
  feature rankings
- Genomes lack the relevant protein families (e.g., highly incomplete assemblies),
  since absence of a prediction may simply reflect missing Pfam families rather than
  true phenotype absence
- You need enzyme-level pathway reconstruction (e.g., distinguishing periplasmic Nap
  from membrane Nar operationally) rather than phenotype-level association of Pfam
  families
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0621
tools:
- name: traitar
  role: Predicts the nitrate-to-nitrite conversion phenotype and ranks Pfam families
    associated with it via the phypat and phypat+PGL classifiers
  repo: https://github.com/hzi-bifo/traitar
- name: pip
  role: Installs the traitar package and its Python dependencies (scipy, matplotlib,
    pandas)
- name: HMMER
  role: Annotates predicted coding sequences with Pfam 27.0 families that feed the
    phenotype models
- name: prodigal
  role: Predicts open reading frames from nucleotide FASTA input before Pfam annotation
- name: GNU parallel
  role: Enables parallel execution of the Traitar phenotype workflow via the -c parameter
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:13d8f9fb12db375100e1deb5a019f4322130139ef97a1676084fd4c86ce2d494
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
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# nitrate-reduction-pathway-gene-mapping

## Summary

Maps Traitar's phenotype-relevant Pfam protein families (e.g., Nar PF02665/PF02613, NapB PF03892, NapD PF03927) to the nitrate-to-nitrite conversion phenotype, using the phypat/phypat+PGL feature rankings from Traitar's 67-phenotype models trained on Pfam 27.0 annotations.

## When to use

Use when bacterial genomes are being phenotyped with Traitar and you need to link the nitrate-to-nitrite conversion (nitrate reduction) phenotype to its supporting Pfam protein families and accessions.

## When NOT to use

- Input is already a phenotype prediction table and you only need the binary nitrate-reduction call — use Traitar's heatmap/prediction outputs directly instead of re-extracting feature rankings
- Genomes lack the relevant protein families (e.g., highly incomplete assemblies), since absence of a prediction may simply reflect missing Pfam families rather than true phenotype absence
- You need enzyme-level pathway reconstruction (e.g., distinguishing periplasmic Nap from membrane Nar operationally) rather than phenotype-level association of Pfam families

## Inputs

- Bacterial genome sequences (nucleotide FASTA) or pre-called genes for Traitar phenotype prediction
- Pfam 27.0 HMM models (Pfam-A.hmm.gz) registered with Traitar
- Traitar Table S6 phenotype–Pfam feature ranking output from phypat or phypat+PGL models

## Outputs

- pfam_phenotype_rankings.tsv with verified nitrate-to-nitrite conversion phenotype-to-Pfam mappings (Nar PF02665, PF02613; NapB PF03892; NapD PF03927)
- Ranked list of nitrate-reduction-associated Pfam families with accessions

## How to apply

Install Traitar via pip (pip install traitar-<version>.tar.gz) with scipy, matplotlib, and pandas, and register the Pfam 27.0 HMM models either through 'traitar pfam <path>' auto-download or via 'traitar pfam --local <path>' pointing to Pfam-A.hmm.gz from the Pfam 27.0 release. Run the phenotype prediction workflow (Prodigal gene calling, HMMER Pfam annotation, phenotype classification) and load the Table S6 phenotype–Pfam ranking output from the trained phypat or phypat+PGL models. Filter the rankings to the nitrate-to-nitrite conversion phenotype and extract the top-ranked Pfam families, annotating each with its accession. Cross-check the extracted accessions against the reference list (Nar PF02665 and PF02613, NapB PF03892, NapD PF03927) and flag any mismatch before writing the verified phenotype–family–accession mappings to pfam_phenotype_rankings.tsv. The rationale is that Traitar's models identify protein families that discriminate phenotype-positive from phenotype-negative genomes, so the ranked features for nitrate-to-nitrite conversion reflect the enzymatic machinery (nitrate reductases) underlying the trait.

## Related tools

- **traitar** (Predicts the nitrate-to-nitrite conversion phenotype and ranks Pfam families associated with it via the phypat and phypat+PGL classifiers) — https://github.com/hzi-bifo/traitar
- **pip** (Installs the traitar package and its Python dependencies (scipy, matplotlib, pandas))
- **HMMER** (Annotates predicted coding sequences with Pfam 27.0 families that feed the phenotype models)
- **prodigal** (Predicts open reading frames from nucleotide FASTA input before Pfam annotation)
- **GNU parallel** (Enables parallel execution of the Traitar phenotype workflow via the -c parameter)

## Examples

```
traitar phenotype <in_dir> samples.txt from_nucleotides <out_dir> -c 2
```

## Evaluation signals

- All reported nitrate-reduction accessions (PF02665, PF02613, PF03892, PF03927) appear in pfam_phenotype_rankings.tsv under the nitrate-to-nitrite conversion phenotype, verified by a script_runs/contains_substring check
- No mismatch flags raised when cross-checking extracted accessions against the reference list
- Pfam annotation was produced with Pfam 27.0 HMM models (not another Pfam release), matching the version the Traitar models were trained on
- The phenotype predictions underlying the ranking were generated by phypat or phypat+PGL models, not unclassified features

## Limitations

- Genotype-phenotype inference with Traitar only takes into account the presence and absence of protein families; other omics data are not integrated
- For incomplete genomes, the absence of a nitrate-to-nitrite conversion prediction may be due to the absence of the relevant protein families from the input genomes
- Pfam 27.0 HMM models must be downloaded separately and are not distributed with the Traitar package

## Evidence

- [results] Traitar selected the flagellar export chaperone FliS (accession no. PF02561), which inhibits early polymerization of the flagellar filament FliC: [evidence span withheld — anchor status: not_found]
- [methods] traitar requires the Pfam 27.0 HMM models: "traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded"
- [methods] pip install traitar: "We strongly encourage you to install traitar with pip rather than by manually cloning the repository."
- [readme] phenotype prediction workflow: "which is to predict open reading frames with Prodigal, annotate the coding sequences provided as nucleotide FASTAs in the <in_dir> for all samples in <sample_file> with Pfam families using HMMer and"
- [discussion] limitation on omics integration: "At the moment, genotype-phenotype inference with Traitar only takes into account the presence and absence of protein families of the bacteria analyzed."
