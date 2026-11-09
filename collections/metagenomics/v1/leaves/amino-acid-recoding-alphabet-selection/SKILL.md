---
name: amino-acid-recoding-alphabet-selection
description: Use when training Snekmer supervised models or clustering protein families
  (e.g., nitrogen cycling families or UniProt genomes) and you must choose among the
  7 AAR alphabets (sizes 2–20) and k=4–16 via config.yaml sweeps.
when_to_use_negative:
- Input is nucleotide sequence or an already-encoded feature/kmer table — the skill
  assumes raw amino acid FASTA input to the AAR-kmer recoding pipeline
- The goal is exact sequence similarity search or the highest-sensitivity de novo
  clustering regardless of method — MMseqs2 easy-cluster achieved higher V-scores
  than any Snekmer AAR/kmer parameterization on the 10-genome benchmark
- Only a single alphabet/k combination is of interest with no comparison — the value
  of this skill is the full 7x13 sweep and its validation against reported best combinations
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0091
tools:
- name: Snekmer
  role: Runs the supervised 'model' and unsupervised 'cluster' modes that recode proteins
    into AAR kmer vectors for each alphabet/kmer-length combination being evaluated
  repo: https://github.com/PNNL-CompBio/Snekmer
- name: Snakemake
  role: Orchestrates the per-combination Snekmer workflows; Snakemake arguments are
    passed through directly
  repo: https://snakemake.readthedocs.io
- name: bsf-jaccard-py
  role: Provides Jaccard similarity for the agglomerative-jaccard clustering used
    to score alternative alphabet/kmer parameterizations (falls back to scipy if unavailable)
  repo: https://github.com/PNNL-CompBio/bsf-jaccard-py
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad005
    title: 'Snekmer: a scalable pipeline for protein sequence fingerprinting based
      on amino acid recoding'
  source_content_hash: sha256:2158e95e49dfccf7a40c3fc7d7bb6b5bbf328d382fadfdbc3f89693b0f9177f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-CompBio/Snekmer
  tools:
  - Blazing Signature Filter
  - Python
  - Snakemake
  - Snekmer
  - bsf-jaccard-py
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# amino-acid-recoding-alphabet-selection

## Summary

Systematic selection of the amino acid recoding (AAR) alphabet (hydro, standard, solvacc, hydrocharge, hydrostruct, miqs, or no recoding) and kmer length (k=4–16) for Snekmer's supervised 'model' and unsupervised 'cluster' modes, judged by cross-validation ROC AUC/PR AUC and clustering V-scores.

## When to use

Use when training Snekmer supervised models or clustering protein families (e.g., nitrogen cycling families or UniProt genomes) and you must choose among the 7 AAR alphabets (sizes 2–20) and k=4–16 via config.yaml sweeps.

## When NOT to use

- Input is nucleotide sequence or an already-encoded feature/kmer table — the skill assumes raw amino acid FASTA input to the AAR-kmer recoding pipeline
- The goal is exact sequence similarity search or the highest-sensitivity de novo clustering regardless of method — MMseqs2 easy-cluster achieved higher V-scores than any Snekmer AAR/kmer parameterization on the 10-genome benchmark
- Only a single alphabet/k combination is of interest with no comparison — the value of this skill is the full 7x13 sweep and its validation against reported best combinations

## Inputs

- Protein family FASTA files with HMM-derived ground-truth labels (nitrogen cycling families)
- 10-genome UniProt/SwissProt protein FASTA dataset (46,908 proteins; genome list per Supplementary Table S2)
- config.yaml with 'k', 'alphabet' (0–5 or None), 'cv' (5), and a fixed 'random_state'

## Outputs

- 7 alphabets x 13 kmer-length performance table of cross-validation ROC AUC and PR AUC (mirrors Figure 3 / Supplementary Table S1)
- Per-family ROC AUC heatmap from unsupervised clustering (Figure 4)
- V-score comparison CSV (e.g., table3_vscore_comparison.csv) of Snekmer parameterizations vs MMseqs2 across GO, Pfam, InterPro, TIGRFAMs, COG annotations

## How to apply

For each of the 7 alphabet settings (0=hydro size 2, 1=standard size 7, 2=solvacc size 3, 3=hydrocharge size 3, 4=hydrostruct size 3, 5=miqs size 10, None=no recoding size 20) and each k from 4 to 16, write a config.yaml specifying 'k' (int), 'alphabet' (str or int), 'cv' (5-fold), and a fixed 'random_state', then run 'snekmer model' or 'snekmer cluster' through the Snakemake workflow. For supervised selection, collect cross-validation ROC AUC and PR AUC from the model-mode reports and assemble a 7x13 performance table mirroring Figure 3/Supplementary Table S1 — the article's best combinations are k=4 with MIQS or k=8 with the standard chemistry encoding, both reaching ROC AUC 1.0. For unsupervised family-structure agreement, cluster the AAR-kmer vectors with the 'correlation' method and compute per-family ROC AUC against HMM labels; expect average AUC > 0.9 in more than half of the alphabet/kmer combinations. For large-scale clustering (10-genome, 46,908-protein UniProt set), compare parameterizations (miqs k=4, solvacc k=8, hydro k=14) using agglomerative-jaccard with complete linkage and dist_thresh 95, computing V-scores (harmonic mean of homogeneity and completeness) against GO/Pfam/InterPro/TIGRFAMs/COG annotations and benchmarking against MMseqs2 easy-cluster.

## Related tools

- **Snekmer** (Runs the supervised 'model' and unsupervised 'cluster' modes that recode proteins into AAR kmer vectors for each alphabet/kmer-length combination being evaluated) — https://github.com/PNNL-CompBio/Snekmer
- **Snakemake** (Orchestrates the per-combination Snekmer workflows; Snakemake arguments are passed through directly) — https://snakemake.readthedocs.io
- **bsf-jaccard-py** (Provides Jaccard similarity for the agglomerative-jaccard clustering used to score alternative alphabet/kmer parameterizations (falls back to scipy if unavailable)) — https://github.com/PNNL-CompBio/bsf-jaccard-py

## Examples

```
# config.yaml
alphabet: 5  # miqs; alternatives 0-4 (hydro, standard, solvacc, hydrocharge, hydrostruct) or None
k: 4
cv: 5
random_state: 42

$ snekmer model
```

## Evaluation signals

- Grid completeness: the performance table contains all 7 alphabets x 13 kmer lengths, each with finite ROC AUC and PR AUC
- MIQS (alphabet 5) at k=4 and the Standard alphabet at k=8 both attain cross-validation ROC AUC = 1.0 as reported
- Unsupervised check: the reproduced heatmap shows average ROC AUC > 0.9 in more than half of the alphabet/kmer-length combinations
- Clustering benchmark ordering: the best Snekmer V-score (hydro, k=14) is somewhat lower than the MMseqs2 easy-cluster V-score in the Table 3 comparison
- Fixed 'random_state' and 5-fold 'cv' yield reproducible scores across reruns of the same config.yaml

## Limitations

- Representing proteins as kmer sets uses an exact-match methodology that may be limited in terms of sensitivity and coverage; AAR mitigates this by linking sequences with distant sequence similarity, but alphabet choice trades alphabet size (2–20) against information retained
- Snekmer clustering V-scores were somewhat lower than MMseqs2 on the 46,908-protein 10-genome benchmark, even at the best parameterization (k=14, hydrophobicity AAR)
- Selection results are dataset-dependent: nitrogen cycling families favored MIQS k=4 / Standard k=8 for supervised models, whereas the 10-genome clustering favored hydro k=14

## Evidence

- [abstract] Here, we describe Snekmer, a software tool for recoding proteins into AAR kmer vectors and performing either (i) construction of supervised classification models: "Here, we describe Snekmer, a software tool for recoding proteins into AAR kmer vectors and performing either (i) construction of supervised classification models"
- [intro] Snekmer is a software package designed to reduce the representation of protein sequences by combining amino acid reduction (AAR) with the kmer approach.: "Snekmer is a software package designed to reduce the representation of protein sequences by combining amino acid reduction (AAR) with the kmer approach."
- [results] An example ``config.yaml`` is included at `resources/config.yaml: "An example ``config.yaml`` is included at `resources/config.yaml <https://github.com/PNNL-CompBio/Snekmer/blob/main/resources/config.yaml>`_"
- [methods] Agglomerative clustering using Jaccard distance (default; requires BSF or falls back to scipy): "Agglomerative clustering using Jaccard distance (default; requires BSF or falls back to scipy)"
- [readme] There are six operation modes for Snekmer: `cluster`, `model`, `search`, `learn`, `apply`, and `motif`: "There are six operation modes for Snekmer: `cluster`, `model`, `search`, `learn`, `apply`, and `motif`"
