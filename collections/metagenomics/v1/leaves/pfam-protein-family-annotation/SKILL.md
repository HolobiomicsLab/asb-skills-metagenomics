---
name: pfam-protein-family-annotation
description: Use when nucleotide or protein FASTA sequences of bacterial genomes need
  Pfam protein family profiles as features for phenotype prediction or trait association
  analysis.
when_to_use_negative:
- Input already consists of Pfam annotation or a protein family presence/absence feature
  table — use the phenotype prediction models directly instead
- You need phenotype prediction only and have no genome sequences — the annotation
  step is a prerequisite, not the end goal
- Study requires genomic features beyond protein family presence/absence (e.g. other
  omics data); Traitar's inference currently only takes into account the presence
  and absence of protein families
edam_operation: http://edamontology.org/operation_0246
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0621
tools:
- name: Traitar
  role: Main framework orchestrating the Pfam annotation workflow (traitar phenotype
    command)
  repo: https://github.com/hzi-bifo/traitar
- name: Prodigal
  role: Predicts open reading frames from nucleotide FASTAs prior to HMMER annotation
- name: HMMER
  role: hmmsearch against Pfam 27.0 HMM models to annotate coding sequences with Pfam
    families
- name: pip
  role: Recommended installation method for traitar from the Python packaging index
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:fb20ef474b360c13543a44acf20500eef09ced71af19cb7db7ff5f2204ce7dd8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hzi-bifo/traitar
  tools:
  - GNU parallel
  - HMMER
  - Prodigal
  - Python
  - Traitar
  - hmmsearch
  - matplotlib
  - pip
  - prodigal
  - scipy
  edam_operation: http://edamontology.org/operation_0246
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# pfam-protein-family-annotation

## Summary

Annotate bacterial coding sequences with Pfam protein family presence/absence profiles using Traitar's workflow (Prodigal gene prediction + HMMER hmmsearch against Pfam 27.0), which serves as the feature basis for genotype-phenotype trait prediction across 67 phenotypes.

## When to use

Use when nucleotide or protein FASTA sequences of bacterial genomes need Pfam protein family profiles as features for phenotype prediction or trait association analysis.

## When NOT to use

- Input already consists of Pfam annotation or a protein family presence/absence feature table — use the phenotype prediction models directly instead
- You need phenotype prediction only and have no genome sequences — the annotation step is a prerequisite, not the end goal
- Study requires genomic features beyond protein family presence/absence (e.g. other omics data); Traitar's inference currently only takes into account the presence and absence of protein families

## Inputs

- Nucleotide FASTA files of assembled bacterial genomes (from_nucleotides mode)
- Protein/coding sequence FASTA files when gene prediction was already performed externally (from_genes mode)
- Sample file with mandatory header: sample_file_name, sample_name, optional category columns (tab-separated)
- Pfam 27.0 HMM models (Pfam-A.hmm), downloaded or registered locally

## Outputs

- Pfam annotation of coding sequences in <out_dir>/pfam_annotation
- Gene prediction results in <out_dir>/gene_prediction (Prodigal ORFs)
- Presence/absence Pfam protein family profiles used as classifier features for 67 phenotypes

## How to apply

Run Traitar's standard workflow via 'traitar phenotype <in_dir> <sample_file> from_nucleotides <out_dir>', which predicts open reading frames with Prodigal, then annotates the coding sequences with Pfam families using HMMER's hmmsearch against the Pfam 27.0 HMM models. The Pfam 27.0 models are not distributed with the package: obtain them with 'traitar pfam <download_folder>' or register a manually downloaded Pfam-A.hmm with 'traitar pfam --local <path to Pfam folder>'. If gene prediction was already done externally, use the 'from_genes' mode so analysis starts at the Pfam annotation step. Use the '-c' parameter (e.g. '-c 2') for parallel execution, which requires GNU parallel. Results land in <out_dir>/pfam_annotation, and the presence/absence protein family profiles then feed the phypat and phypat+PGL phenotype classifiers.

## Related tools

- **Traitar** (Main framework orchestrating the Pfam annotation workflow (traitar phenotype command)) — https://github.com/hzi-bifo/traitar
- **Prodigal** (Predicts open reading frames from nucleotide FASTAs prior to HMMER annotation)
- **HMMER** (hmmsearch against Pfam 27.0 HMM models to annotate coding sequences with Pfam families)
- **pip** (Recommended installation method for traitar from the Python packaging index)

## Examples

```
traitar phenotype <traitar_dir>/data/sample_data <traitar_dir>/data/sample_data/samples.txt from_genes <out_dir> -c 2
```

## Evaluation signals

- Output directory <out_dir>/pfam_annotation exists and contains Pfam family annotations for every sample in the sample file
- Annotated Pfam families derive from Pfam 27.0 models (verifiable by re-running 'traitar pfam' registration)
- Running 'traitar phenotype <traitar_dir>/data/sample_data ... from_genes <out_dir>' on packaged sample data completes within ~5 minutes for the two Listeria test genomes
- Resulting protein family profiles reproduce the downstream phenotype assignments to the 67 traits when fed to the phypat/phypat+PGL classifiers
- Gene prediction results appear in <out_dir>/gene_prediction when starting from nucleotides, confirming the Prodigal step ran before annotation

## Limitations

- Pfam 27.0 HMM models are not distributed with the traitar package and must be downloaded separately (or registered with --local)
- Parallel usage via '-c' requires GNU parallel; the README notes a problem with parallel usage in the Docker container so the option is not guaranteed to work there
- For incomplete genomes, absence of a protein family (and hence a trait) may simply reflect missing sequence in the input rather than true absence
- Traitar's inference only takes into account the presence and absence of protein families; no other omics data are integrated
- The Docker output directory is owned by root, requiring root access to inspect results

## Evidence

- [readme] traitar phenotype <in dir>  <sample file> from_nucleotides <out_dir> : "``traitar phenotype <in dir>  <sample file> from_nucleotides <out_dir> `` will trigger the standard workflow of Traitar, which is to predict open reading frames with Prodigal"
- [methods] Pfam 27.0 models must be downloaded: "traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded"
- [methods] Register local Pfam folder: "``traitar pfam --local <path to Pfam folder>`` to let traitar know where."
- [methods] Install with pip: "We strongly encourage you to install traitar with pip rather than by manually cloning the repository."
- [readme] Parallel usage requires GNU parallel: [evidence span withheld — anchor status: external_doc]
- [readme] from_genes mode starts at Pfam annotation: [evidence span withheld — anchor status: external_doc]
