---
name: cross-validation-model-evaluation
description: Use when benchmarking supervised protein family classifiers built on
  amino acid recoding kmer vectors, and you need cross-validated ROC AUC/PR AUC to
  compare alphabets and kmer lengths.
when_to_use_negative:
- The task is unsupervised de novo family determination — use Snekmer's 'cluster'
  mode with agglomerative/density Jaccard clustering instead of model-mode cross-validation.
- You need probabilistic annotation of new sequences against existing families — use
  the 'search' or 'apply' mode (results/apply/snekmer_results.csv with Score/delta/Confidence),
  not classifier re-training and CV scoring.
- Input proteins have no family annotations, so supervised training labels are unavailable.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: Snekmer
  role: 'Supervised model-mode pipeline: recodes proteins into AAR kmer vectors, trains
    classifiers with configured ''cv'' and ''alphabet''/''k'' settings, and emits
    cross-validation score reports'
  repo: https://github.com/PNNL-CompBio/Snekmer
- name: Snakemake
  role: Orchestrates the snekmer model workflow across alphabet x kmer-length combinations;
    Snakemake arguments pass through directly
  repo: https://github.com/PNNL-CompBio/Snekmer
- name: Python
  role: Runtime (3.11+) for installing Snekmer via venv/pip and assembling the ROC
    AUC / PR AUC performance table
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad005
    title: 'Snekmer: a scalable pipeline for protein sequence fingerprinting based
      on amino acid recoding'
  source_content_hash: sha256:9f1052a65e4868464332e5b6fd798c2ee6ed06b104c4a3be8116bb5a18fffe33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-CompBio/Snekmer
  tools:
  - Python
  - Snakemake
  - Snekmer
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# cross-validation-model-evaluation

## Summary

Evaluates Snekmer supervised (model mode) AAR-kmer protein family classifiers using 5-fold cross-validation, collecting ROC AUC and PR AUC across 7 recoding alphabets (including MIQS and standard chemistry) and kmer lengths k=4–16 to identify optimal encoding/kmer combinations such as MIQS at k=4 or standard chemistry at k=8.

## When to use

Use when benchmarking supervised protein family classifiers built on amino acid recoding kmer vectors, and you need cross-validated ROC AUC/PR AUC to compare alphabets and kmer lengths.

## When NOT to use

- The task is unsupervised de novo family determination — use Snekmer's 'cluster' mode with agglomerative/density Jaccard clustering instead of model-mode cross-validation.
- You need probabilistic annotation of new sequences against existing families — use the 'search' or 'apply' mode (results/apply/snekmer_results.csv with Score/delta/Confidence), not classifier re-training and CV scoring.
- Input proteins have no family annotations, so supervised training labels are unavailable.

## Inputs

- Pre-annotated nitrogen cycling protein family sequences (FASTA) from the Snekmer repository example data
- resources/config.yaml template from github.com/PNNL-CompBio/Snekmer
- Alphabet setting (int 0–5 or None) and kmer length k (4–16) parameters

## Outputs

- Per-combination cross-validation ROC AUC scores from model-mode cross-validation reports
- Per-combination cross-validation PR AUC scores
- Assembled performance table of ROC AUC / PR AUC by alphabet x kmer length (mirroring Figure 3 / Supplementary Table S1)

## How to apply

Clone github.com/PNNL-CompBio/Snekmer and, for each of the 7 alphabet settings (0=hydro size 2, 1=standard size 7, 2=solvacc size 3, 3=hydrocharge size 3, 4=hydrostruct size 3, 5=miqs size 10, None/no recoding size 20) and each k from 4 to 16, write a config.yaml setting 'k' (int), 'alphabet' (str or int), 'cv' set to 5-fold cross-validation, and a fixed 'random_state' for reproducibility. Run 'snekmer model' so the Snakemake-driven pipeline trains supervised classifiers on the pre-annotated nitrogen cycling protein family sequences (33 families). Collect cross-validation ROC AUC and PR AUC scores from the model-mode cross-validation reports for every alphabet x kmer-length combination and assemble them into a performance table mirroring Figure 3 / Supplementary Table S1. Judge correctness by verifying the table is complete (7 alphabets x 13 kmer lengths, all entries finite) and that MIQS at k=4 and the Standard chemistry alphabet at k=8 attain ROC AUC = 1.0 as reported for the majority of models.

## Related tools

- **Snekmer** (Supervised model-mode pipeline: recodes proteins into AAR kmer vectors, trains classifiers with configured 'cv' and 'alphabet'/'k' settings, and emits cross-validation score reports) — https://github.com/PNNL-CompBio/Snekmer
- **Snakemake** (Orchestrates the snekmer model workflow across alphabet x kmer-length combinations; Snakemake arguments pass through directly) — https://github.com/PNNL-CompBio/Snekmer
- **Python** (Runtime (3.11+) for installing Snekmer via venv/pip and assembling the ROC AUC / PR AUC performance table)

## Examples

```
git clone https://github.com/PNNL-CompBio/Snekmer.git; cd Snekmer; pip install -r requirements.txt; pip install .; snekmer model
```

## Evaluation signals

- Completeness check: performance table covers 7 alphabets x 13 kmer lengths (k=4–16) with finite ROC AUC and PR AUC for every cell
- Reproduces the reported optimum: MIQS alphabet (alphabet=5) at k=4 and Standard chemistry (alphabet=1) at k=8 give ROC AUC = 1.0 for the majority of the 33 nitrogen cycling family models
- config.yaml contains the required keys: 'k' (int), 'alphabet' (str or int), 'cv' (int), and a fixed 'random_state' for reproducibility
- Scores are read from the model-mode cross-validation reports produced by 'snekmer model', not from apply/search cosine-similarity outputs
- Repeated runs with the fixed 'random_state' yield identical CV scores

## Limitations

- Exact-match kmer representation may be limited in sensitivity and coverage; AAR partially compensates by linking sequences with distant sequence similarity, so results depend on alphabet choice
- Perfect (ROC AUC = 1.0) CV scores were observed for only the best parameter combinations and the majority (not necessarily all) of models; other alphabet/k combinations underperform
- Model evaluation is 5-fold cross-validation on pre-annotated families; it does not measure performance on truly novel query families as the search/apply cosine-similarity modes would

## Evidence

- [abstract] Supervised model mode evaluates AAR-kmer classifiers by CV: "Here, we describe Snekmer, a software tool for recoding proteins into AAR kmer vectors and performing either (i) construction of supervised classification models trained on input protein families"
- [results] Model output scores collected from cross-validation reports: [evidence span withheld — anchor status: not_found]
- [intro] Supervised pipeline generates models from AAR-kmer representations: "(2) generates supervised machine learning models, or (3) searches sequences against pre-trained models to determine probabilistic annotations."
- [methods] Snakemake drives and is configurable through the workflow: "Snakemake arguments are passed through directly to Snakemake (they are not Snekmer-specific)."
- [readme] Python/Snakemake installation requirements for running model mode: "Requirements:
- Python 3.11+
- A working C/C++ toolchain may be required for some dependencies depending on your platform.
- Snekmer orchestrates workflows via Snakemake"
