---
name: protein-family-profiling-with-pfam-hmms
description: Use when you need Pfam protein family presence/absence profiles for nucleotide
  or protein FASTAs (draft genomes, metagenome bins, SAGs) to feed phenotype prediction
  or comparative genotype-phenotype inference.
when_to_use_negative:
- The input is already a Pfam domain table or other feature matrix — proceed directly
  to phenotype classification rather than re-scanning.
- 'You need trait predictions integrating transcriptomic or other omics evidence:
  Traitar currently uses only presence/absence of protein families.'
- 'The claim rests on complete absence of a trait in a highly incomplete draft genome
  (e.g. 60%-complete unFirm_1): absent predictions may simply reflect missing protein
  families in the input genome.'
edam_operation: http://edamontology.org/operation_0246
edam_topics:
- http://edamontology.org/topic_0176
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3299
tools:
- name: Traitar
  role: End-to-end framework that chains Prodigal gene calling, HMMER Pfam annotation,
    and 67-trait phenotype prediction from the resulting protein family profiles
  repo: https://github.com/hzi-bifo/traitar
- name: Prodigal
  role: Predicts open reading frames from nucleotide assemblies before Pfam scanning
    (from_nucleotides mode)
- name: HMMER (hmmsearch)
  role: Scans predicted coding sequences against Pfam 27.0 HMM models to build protein
    family profiles
- name: GNU parallel
  role: Enables parallel execution of the workflow via the -c parameter
- name: Pfam 27.0 (Pfam-A.hmm)
  role: Reference HMM library that must be fetched with 'traitar pfam' or registered
    with --local
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:9a4b42d0399632c762c1f732d9d38ed3c0d757d5a7e12fe676288300157a5b77
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hzi-bifo/traitar
  tools:
  - GNU parallel
  - HMMER (hmmsearch)
  - Pfam 27.0 (Pfam-A.hmm)
  - Prodigal
  - Python
  - Traitar
  - matplotlib
  - pandas
  - pip
  - scipy
  edam_operation: http://edamontology.org/operation_0246
  edam_topics:
  - http://edamontology.org/topic_0176
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 20
---

# protein-family-profiling-with-pfam-hmms

## Summary

Annotate bacterial genomes with Pfam 27.0 protein family profiles by running Prodigal gene calling followed by HMMER hmmsearch, as implemented inside the Traitar phenotype workflow. This profile is the input representation from which Traitar's phypat and phypat+PGL classifiers assign 67 phenotypes.

## When to use

Use when you need Pfam protein family presence/absence profiles for nucleotide or protein FASTAs (draft genomes, metagenome bins, SAGs) to feed phenotype prediction or comparative genotype-phenotype inference.

## When NOT to use

- The input is already a Pfam domain table or other feature matrix — proceed directly to phenotype classification rather than re-scanning.
- You need trait predictions integrating transcriptomic or other omics evidence: Traitar currently uses only presence/absence of protein families.
- The claim rests on complete absence of a trait in a highly incomplete draft genome (e.g. 60%-complete unFirm_1): absent predictions may simply reflect missing protein families in the input genome.

## Inputs

- Nucleotide FASTA assemblies (e.g. draft population genomes such as unClos_1 and unFirm_1) for the from_nucleotides mode
- Protein or nucleotide CDS FASTAs from external gene prediction for the from_genes mode
- Sample file (tab-separated: sample_file_name, sample_name, optional category column)
- Pfam 27.0 Pfam-A.hmm HMM models (downloaded via 'traitar pfam' or supplied locally)

## Outputs

- Gene prediction results in <out_dir>/gene_prediction (Prodigal ORF calls, when starting from nucleotides)
- Pfam protein family profiles in <out_dir>/pfam_annotation
- Phenotype prediction tables (predictions_majority-vote_combined.txt, flat versions) and heatmaps in <out_dir>/phenotype_prediction
- Phenotype-relevant protein family GFF tracks in phypat/feat_gffs and phypat+PGL/feat_gffs

## How to apply

First obtain the required Pfam 27.0 HMM models with 'traitar pfam <download folder>' or register a locally downloaded Pfam-A.hmm folder via 'traitar pfam --local <path>'. Verify Prodigal, hmmsearch, and GNU parallel are on the PATH; Traitar's standard workflow ('traitar phenotype ... from_nucleotides') predicts open reading frames with Prodigal, annotates coding sequences with Pfam families using HMMer, and emits per-genome protein family profiles. If gene prediction was already done externally, use the 'from_genes' entry point so analysis starts at the Pfam annotation step. Use the -c flag (e.g. -c 2) to parallelize, which requires GNU parallel. The Pfam profiles then serve as features for Traitar's 67-trait phenotype classifiers, e.g. for benchmarking against a manual metabolic reconstruction as done for the biogas reactor genomes unClos_1 and unFirm_1.

## Related tools

- **Traitar** (End-to-end framework that chains Prodigal gene calling, HMMER Pfam annotation, and 67-trait phenotype prediction from the resulting protein family profiles) — https://github.com/hzi-bifo/traitar
- **Prodigal** (Predicts open reading frames from nucleotide assemblies before Pfam scanning (from_nucleotides mode))
- **HMMER (hmmsearch)** (Scans predicted coding sequences against Pfam 27.0 HMM models to build protein family profiles)
- **GNU parallel** (Enables parallel execution of the workflow via the -c parameter)
- **Pfam 27.0 (Pfam-A.hmm)** (Reference HMM library that must be fetched with 'traitar pfam' or registered with --local)

## Examples

```
traitar phenotype <in_dir> samples.txt from_nucleotides <out_dir> -c 2
```

## Evaluation signals

- The workflow completes the full chain: gene_prediction, pfam_annotation, and phenotype_prediction directories are populated under the output directory.
- Pfam annotation used Pfam 27.0 models specifically — 'traitar pfam' fetch completed or a local Pfam-A.hmm folder was registered with 'traitar pfam --local'.
- Predicted phenotype calls reproduce published benchmark figures when compared to a reference reconstruction, e.g. recall 6/7 present phenotypes (87.5%) and 9/11 absent-phenotype agreement (81.8%) for unClos_1 and unFirm_1 vs. Frank et al.
- Encoded predictions in predictions_majority-vote_combined.txt are within the valid range 0–3 (0 absent, 1 phypat-only, 2 phypat+PGL-only, 3 both).
- Feature GFF tracks in feat_gffs link phenotype-relevant Pfam families back to Prodigal-called genes when starting from nucleotides.

## Limitations

- Pfam 27.0 HMM models are not distributed with the package and must be downloaded separately.
- Absence of a phenotype prediction on an incomplete genome may be due to absence of the relevant protein families from the input, not true biological absence.
- Genotype-phenotype inference with Traitar only takes into account presence and absence of protein families; other omics data are not integrated.
- The from_genes feature-track option is experimental and gene GFF id consistency across img/RefSeq/Prodigal sources is not guaranteed.
- The plant biomass predictor prototype is not included in the released Traitar software.

## Evidence

- [methods] traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded: "traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded"
- [readme] predict open reading frames with Prodigal, annotate the coding sequences ... with Pfam families using HMMer and finally predict phenotypes from the models for the 67 traits: "will trigger the standard workflow of Traitar, which is to predict open reading frames with Prodigal, annotate the coding sequences provided as nucleotide FASTAs in the <in_dir> for all samples in"
- [discussion] assigning 67 different phenotypes to bacteria on the basis of the protein family contents of their genomes: "assigning 67 different phenotypes to bacteria on the basis of the protein family contents of their genomes."
- [discussion] One obvious limitation being for incomplete genomes, the absence of a phenotype prediction may be due to the absence of the relevant protein families from the input genomes: "One obvious limitation being for incomplete genomes, the absence of a phenotype prediction may be due to the absence of the relevant protein families from the input genomes."
- [readme] Traitar provides the gene prediction results in <out_dir>/gene_prediction, the Pfam annotation in <out_dir>/pfam_annotation: [evidence span withheld — anchor status: external_doc]
