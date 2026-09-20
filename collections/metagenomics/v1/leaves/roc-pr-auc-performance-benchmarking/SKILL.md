---
name: roc-pr-auc-performance-benchmarking
description: Use when comparing supervised classification performance of protein family
  models across amino acid recoding alphabets and kmer lengths (e.g., k=4-16) to identify
  optimal encoding/kmer parameter combinations.
when_to_use_negative:
- Input sequences are not pre-annotated into known families — use Snekmer's unsupervised
  'cluster' mode or the Learn/Apply pipeline instead of supervised 'model' mode benchmarking
- The goal is annotating new sequences against pre-trained models (Snekmer 'search'/'apply'
  mode outputs snekmer_results.csv), not evaluating classifier performance
- Only a single alphabet/kmer configuration is of interest — a full cross-product
  sweep benchmark is unnecessary overhead
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3474
tools:
- name: Snekmer
  role: Runs the supervised model mode that recodes proteins into AAR kmer vectors
    and trains classifiers with cross-validation ROC AUC / PR AUC reporting
  repo: github.com/PNNL-CompBio/Snekmer
- name: Snakemake
  role: Orchestrates the snekmer model workflow so every alphabet x kmer-length combination
    is executed reproducibly
  repo: github.com/PNNL-CompBio/Snekmer
- name: Python
  role: Used to assemble per-run cross-validation scores into the single alphabet
    x kmer-length performance table
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad005
    title: 'Snekmer: a scalable pipeline for protein sequence fingerprinting based
      on amino acid recoding'
  source_content_hash: sha256:ccbc689a47e55e64834f63aeaaa74271b8e9bbe49aa04e3ec65b61bcc76997ef
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Snakemake
  - Snekmer
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# ROC-PR AUC performance benchmarking

## Summary

Systematically benchmarks supervised protein-family classifiers by sweeping amino acid recoding alphabets and kmer lengths in Snekmer's model mode, then assembling 5-fold cross-validation ROC AUC and PR AUC scores into a complete alphabet x kmer-length performance table.

## When to use

Use when comparing supervised classification performance of protein family models across amino acid recoding alphabets and kmer lengths (e.g., k=4-16) to identify optimal encoding/kmer parameter combinations.

## When NOT to use

- Input sequences are not pre-annotated into known families — use Snekmer's unsupervised 'cluster' mode or the Learn/Apply pipeline instead of supervised 'model' mode benchmarking
- The goal is annotating new sequences against pre-trained models (Snekmer 'search'/'apply' mode outputs snekmer_results.csv), not evaluating classifier performance
- Only a single alphabet/kmer configuration is of interest — a full cross-product sweep benchmark is unnecessary overhead

## Inputs

- Pre-annotated nitrogen cycling protein family sequences (training FASTA inputs)
- Snekmer config.yaml template (resources/config.yaml) with alphabet, k, cv, and random_state fields
- Snekmer GitHub repository with example nitrogen cycling family data

## Outputs

- Model-mode cross-validation reports with ROC AUC and PR AUC per family
- Complete 7 alphabets x 13 kmer lengths performance table (mirroring Figure 3 / Supplementary Table S1)

## How to apply

Clone the Snekmer repository and start from the resources/config.yaml template. For each of the 7 alphabet settings (0=hydro size 2, 1=standard size 7, 2=solvacc size 3, 3=hydrocharge size 3, 4=hydrostruct size 3, 5=miqs size 10, None=no recoding size 20) and each kmer length from 4 to 16, write a config.yaml specifying 'k' (int), 'alphabet' (str or int), 'cv' (5-fold cross-validation), and a fixed 'random_state' for reproducibility. Run 'snekmer model' so the Snakemake-driven workflow trains supervised classifiers on the pre-annotated nitrogen cycling protein family sequences. Collect cross-validation ROC AUC and PR AUC scores from the model-mode cross-validation reports and assemble them into a single performance table mirroring Figure 3 / Supplementary Table S1, with rows/columns for alphabet and kmer length. In this dataset, the best combinations were k=4 with the MIQS AAR alphabet and k=8 with the standard chemistry encoding, where the majority of models achieved ROC AUC of 1.0 (perfect classification).

## Related tools

- **Snekmer** (Runs the supervised model mode that recodes proteins into AAR kmer vectors and trains classifiers with cross-validation ROC AUC / PR AUC reporting) — github.com/PNNL-CompBio/Snekmer
- **Snakemake** (Orchestrates the snekmer model workflow so every alphabet x kmer-length combination is executed reproducibly) — github.com/PNNL-CompBio/Snekmer
- **Python** (Used to assemble per-run cross-validation scores into the single alphabet x kmer-length performance table)

## Examples

```
cd Snekmer && sed -i 's/^k:.*/k: 4/; s/^alphabet:.*/alphabet: miqs/; s/^cv:.*/cv: 5/' resources/config.yaml && snekmer model
```

## Evaluation signals

- Table completeness: 7 alphabets x 13 kmer lengths (k=4-16), every cell with finite ROC AUC and PR AUC
- MIQS alphabet at k=4 and Standard alphabet at k=8 both attain ROC AUC = 1.0, matching the reported finding
- Cross-validation is 5-fold with a fixed random_state, so re-running any cell reproduces the same score
- Scores match the structure of Figure 3 / Supplementary Table S1 when arranged by alphabet and kmer length
- All configs were validated against the resources/config.yaml template keys (k, alphabet, cv, random_state)

## Limitations

- Best-performing parameters may be dataset-specific: k=4/MIQS and k=8/standard reflect the nitrogen cycling families and may not transfer to other protein families
- Representing proteins as kmer sets uses an exact-match methodology that may be limited in terms of sensitivity and coverage; AAR mitigates but does not remove this
- Some unsupervised clustering paths require the BSF Blazing Signature Filter extension, otherwise falling back to scipy

## Evidence

- [abstract] Across the examined protein families, the best-performing parameter combination was k of 4 with MIQS AAR or a k of 8 for the standard chemistry encoding, for which the majority of models had an ROC AUC of 1.0: [evidence span withheld — anchor status: not_found]
- [intro] There are six operation modes for Snekmer: "There are six operation modes for Snekmer: `cluster`, `model`, `search`, `learn`, `apply`, and `motif`,"
- [results] An example config.yaml is included at resources/config.yaml: "An example ``config.yaml`` is included at `resources/config.yaml"
- [readme] Snekmer model CLI verified by README install check: "Verify installation:
``n  snekmer -h
"
