---
name: command-line-pipeline-execution
description: Use when you need to phenotype bacterial genome assemblies (nucleotide
  or protein FASTA) from the command line with Traitar, e.g. to reproduce present/absent
  calls for metagenome-recovered population genomes like unClos_1 and unFirm_1.
when_to_use_negative:
- Input is already a Pfam annotation or phenotype table rather than genome sequences
  — skip the gene-calling pipeline and compare tables directly
- Only a handful of phenotypes of interest are known and a manual metabolic reconstruction
  already exists — Traitar complements rather than replaces a manual reconstruction
  such as Frank et al.'s Table 4
- The plant biomass trait is needed — the plant biomass predictor prototype is not
  included in the released Traitar software
edam_operation: http://edamontology.org/operation_3092
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Traitar
  role: 'Main command-line pipeline: gene calling, Pfam annotation, and 67-phenotype
    prediction'
  repo: https://github.com/hzi-bifo/traitar
- name: Prodigal
  role: Open reading frame prediction invoked by the from_nucleotides workflow
- name: HMMER (hmmsearch)
  role: Scans predicted coding sequences against Pfam 27.0 HMM models for protein
    family profiles
- name: GNU parallel
  role: Enables parallel execution via the -c parameter (e.g. -c 2)
- name: pip
  role: Recommended installer for traitar rather than manually cloning the repository
- name: Docker
  role: Alternative execution route via the aweimann/traitar container image
  repo: https://hub.docker.com/r/aweimann/traitar
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:d2f914f167b44431a07446d50ae38750805f4c335eed7afaedc977e8e627e909
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hzi-bifo/traitar
  tools:
  - Docker
  - GNU parallel
  - HMMER (hmmsearch)
  - Prodigal
  - Python
  - Traitar
  - matplotlib
  - pandas
  - pip
  - scipy
  edam_operation: http://edamontology.org/operation_3092
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# command-line-pipeline-execution

## Summary

Executes the Traitar command-line phenotype prediction pipeline end to end: installing traitar via pip, fetching Pfam 27.0 HMM models, verifying external dependencies (Prodigal, hmmsearch, GNU parallel), and running the 'traitar phenotype' workflow to predict 67 phenotypes from genome sequences.

## When to use

Use when you need to phenotype bacterial genome assemblies (nucleotide or protein FASTA) from the command line with Traitar, e.g. to reproduce present/absent calls for metagenome-recovered population genomes like unClos_1 and unFirm_1.

## When NOT to use

- Input is already a Pfam annotation or phenotype table rather than genome sequences — skip the gene-calling pipeline and compare tables directly
- Only a handful of phenotypes of interest are known and a manual metabolic reconstruction already exists — Traitar complements rather than replaces a manual reconstruction such as Frank et al.'s Table 4
- The plant biomass trait is needed — the plant biomass predictor prototype is not included in the released Traitar software

## Inputs

- Nucleotide FASTA genome assemblies (e.g. biogas reactor population genomes unClos_1 and unFirm_1) for the from_nucleotides route
- Protein/gene FASTA or pre-computed genes for the from_genes route
- Tab-separated sample file with sample_file_name, sample_name, and optional category columns (mandatory header row)
- Pfam 27.0 Pfam-A.hmm HMM models (downloaded via 'traitar pfam' or registered locally)

## Outputs

- Gene prediction results in <out_dir>/gene_prediction
- Pfam protein family annotations in <out_dir>/pfam_annotation
- Phenotype predictions for 67 traits in <out_dir>/phenotype_prediction (predictions_majority-vote_combined.txt, predictions_flat_majority-votes_combined.txt)
- Heatmaps: heatmap_phypat.png, heatmap_phypat_ggl.png, heatmap_comb.png
- Phenotype-relevant protein family GFF tracks in phypat/feat_gffs and phypat+PGL/feat_gffs

## How to apply

Install traitar from the Python packaging index with pip after installing python-scipy, python-matplotlib, python-pandas, and optionally python-virtualenv. Fetch the Pfam 27.0 HMM models with 'traitar pfam <download folder>' or register a manually downloaded Pfam-A.hmm folder with 'traitar pfam --local', and verify that prodigal, hmmsearch, and GNU parallel are on the PATH. Run the standard workflow with 'traitar phenotype <in dir> <sample file> from_nucleotides <out_dir>', which triggers gene calling with Prodigal, Pfam annotation with HMMer, and phenotype prediction for the 67 traits; use 'from_genes' if gene prediction was already done externally, and pass '-c N' to parallelize with GNU parallel. The sample file is tab-separated with columns sample_file_name, sample_name, and optional category, with a mandatory header row. Results land in <out_dir>/gene_prediction, <out_dir>/pfam_annotation, and <out_dir>/phenotype_prediction as heatmaps (heatmap_phypat.png, heatmap_phypat_ggl.png) and tab-separated tables such as predictions_majority-vote_combined.txt where 0=absent, 1=phypat-only, 2=phypat+PGL-only, 3=supported by both.

## Related tools

- **Traitar** (Main command-line pipeline: gene calling, Pfam annotation, and 67-phenotype prediction) — https://github.com/hzi-bifo/traitar
- **Prodigal** (Open reading frame prediction invoked by the from_nucleotides workflow)
- **HMMER (hmmsearch)** (Scans predicted coding sequences against Pfam 27.0 HMM models for protein family profiles)
- **GNU parallel** (Enables parallel execution via the -c parameter (e.g. -c 2))
- **pip** (Recommended installer for traitar rather than manually cloning the repository)
- **Docker** (Alternative execution route via the aweimann/traitar container image) — https://hub.docker.com/r/aweimann/traitar

## Examples

```
traitar phenotype <in_dir> samples.txt from_nucleotides <out_dir> -c 2
```

## Evaluation signals

- Output directory contains the three expected subfolders: gene_prediction, pfam_annotation, and phenotype_prediction
- predictions_majority-vote_combined.txt encodes calls as 0 (absent), 1 (phypat-only), 2 (phypat+PGL-only), 3 (both classifiers)
- For unClos_1/unFirm_1 reproduction, Traitar should recall 87.5% (6/7) of reconstruction-inferred present phenotypes and agree on 81.8% (9/11) of absent phenotypes vs. the Frank et al. manual reconstruction
- Package example data run ('traitar phenotype ... from_genes ... -c 2' on Listeria grayi DSM_20601 and Listeria ivanovii WSLC3009) completes within ~5 minutes
- Heatmap files (heatmap_phypat.png, heatmap_phypat_ggl.png, heatmap_comb.png) are generated with hierarchical clustering dendrograms for phenotypes and samples

## Limitations

- For incomplete genomes (e.g. unFirm_1 at ~60% completeness), an absent phenotype prediction may simply reflect missing protein families in the input genome
- Genotype-phenotype inference uses only presence/absence of protein families; other omics data are not integrated (proposed only as future work)
- The plant biomass predictor is not included in the released Traitar software
- In the Docker container there is a problem with parallel usage so the -c option is not guaranteed to work, and output files are owned by root
- The from_genes GFF feature-track option is experimental and gene-id consistency across img/RefSeq/Prodigal sources is not guaranteed

## Evidence

- [readme] traitar phenotype <in dir>  <sample file> from_nucleotides <out_dir>: "will trigger the standard workflow of Traitar, which is to predict open reading frames with Prodigal, annotate the coding sequences"
- [methods] We strongly encourage you to install traitar with pip rather than by manually cloning the repository.: "We strongly encourage you to install traitar with pip rather than by manually cloning the repository."
- [intro] An easy-to-use, fully automated software framework for prediction of a broad range of traits from only the genome sequence: "an easy-to-use, fully automated software framework for the accurate prediction of currently 67 phenotypes directly from a genome sequence"
- [readme] The -c parameter sets the number of processes used e.g. -c 2: [evidence span withheld — anchor status: external_doc]
- [discussion] One obvious limitation being for incomplete genomes, the absence of a phenotype prediction may be due to the absence of the relevant protein families: "for incomplete genomes, the absence of a phenotype prediction may be due to the absence of the relevant protein families from the input genomes."
- [methods] traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded: "traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded"
