---
name: supervised-pair-classifier-training
description: Use when reproducing or retraining vRhyme's supervised pair classifiers
  from fragment sequence features, or when validating that regenerated .sav model
  files behave like the reference models in vRhyme/models/.
when_to_use_negative:
- Input is a whole metagenome rather than predicted viral scaffolds — vRhyme is built
  for viral sequences and is not meant to bin microbes
- Unsupervised metagenomic binning of bacteria/archaea is the goal — tools like MetaBat2,
  VAMB, CONCOCT, or BinSanity are the intended choices; note MetaBat2 outperformed
  vRhyme on a non-viral synthetic dataset
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: scikit-learn
  role: Provides the ExtraTreesClassifier and MLPClassifier used for supervised classification
    of same/different scaffold pairs; required version >= 0.23.0
  repo: https://scikit-learn.org/stable/install.html
- name: vRhyme
  role: Ships the trained model files (vRhyme/models/vRhyme_machine_model_ET.sav.gz
    and NN.sav) that reproduced classifiers must match in behavior
  repo: https://github.com/AnantharamanLab/vRhyme
- name: Pandas
  role: Handles the pair-distance feature table (vRhyme_machine_distances.tsv) used
    as training input
  repo: https://pandas.pydata.org/pandas-docs/stable/getting_started/install.html
- name: NumPy
  role: Numerical support for computing sequence-feature distances and feeding model
    training
  repo: https://numpy.org/install/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:eb36bcf28ad60a7f1dd5256905313d160931af56b9abfcf11f8ac9ecaf076432
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/AnantharamanLab/vRhyme
  tools:
  - NumPy
  - Pandas
  - Python
  - scikit-learn
  - vRhyme
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# supervised-pair-classifier-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train and save the hybrid scikit-learn ExtraTrees and MLP classifiers that vRhyme uses to classify scaffold pairs as 'same' vs 'different' viral genome origin, reproducing the shipped vRhyme_machine_model_ET.sav and vRhyme_machine_model_NN.sav models.

## When to use

Use when reproducing or retraining vRhyme's supervised pair classifiers from fragment sequence features, or when validating that regenerated .sav model files behave like the reference models in vRhyme/models/.

## When NOT to use

- Input is a whole metagenome rather than predicted viral scaffolds — vRhyme is built for viral sequences and is not meant to bin microbes
- Unsupervised metagenomic binning of bacteria/archaea is the goal — tools like MetaBat2, VAMB, CONCOCT, or BinSanity are the intended choices; note MetaBat2 outperformed vRhyme on a non-viral synthetic dataset

## Inputs

- Labeled training/testing set of 69,632 same/different scaffold pairs derived from 39,105 genome fragments
- Per-pair sequence-feature distance table (vRhyme_machine_distances.tsv) with k-mer/tetranucleotide frequency distances and GC content differences
- Reference model files from vRhyme/models/ (vRhyme_machine_model_ET.sav.gz, vRhyme_machine_model_NN.sav) for comparison

## Outputs

- Fitted ExtraTreesClassifier saved as vRhyme_machine_model_ET.sav (max_depth=10, max_features=7, n_estimators=1500)
- Fitted MLPClassifier saved as vRhyme_machine_model_NN.sav (alpha=0.001, hidden_layer_sizes=(5,25,50,75,100,100,75,50,25,5), learning_rate_init=0.0001, max_iter=1250)
- 'same'/'different' classification probabilities for held-out scaffold pairs

## How to apply

Fragment complete viral genomes from NCBI RefSeq/GenBank and IMG/VR into 39,105 scaffold fragments and construct 69,632 labeled same/different scaffold pairs. Compute per-pair sequence-feature distances (tetranucleotide/k-mer frequency distances and GC content differences; the features vRhyme uses with defaults --max_gc 0.20 and --min_kmer 0.60) and tabulate them as vRhyme_machine_distances.tsv. Train the ExtraTreesClassifier (scikit-learn >= 0.23.0, with NumPy and Pandas) using max_depth = 10, max_features = 7, n_estimators = 1500, and the MLPClassifier (neural network) using alpha = 0.001, beta_1 = 0.7, beta_2 = 0.8, hidden_layer_sizes = (5,25,50,75,100,100,75,50,25,5), learning_rate_init = 0.0001, max_iter = 1250; these iterative train/test settings were the most robust in vRhyme. Save the fitted models as vRhyme_machine_model_ET.sav and vRhyme_machine_model_NN.sav, mirroring the files shipped in the vRhyme/models/ folder (which ships gzipped, e.g. vRhyme_machine_model_ET.sav.gz, and must be gunzipped before install). Run the hybrid models on held-out pairs requiring the 'same' probability threshold to be met by both models before calling a pair 'same'.

## Related tools

- **scikit-learn** (Provides the ExtraTreesClassifier and MLPClassifier used for supervised classification of same/different scaffold pairs; required version >= 0.23.0) — https://scikit-learn.org/stable/install.html
- **vRhyme** (Ships the trained model files (vRhyme/models/vRhyme_machine_model_ET.sav.gz and NN.sav) that reproduced classifiers must match in behavior) — https://github.com/AnantharamanLab/vRhyme
- **Pandas** (Handles the pair-distance feature table (vRhyme_machine_distances.tsv) used as training input) — https://pandas.pydata.org/pandas-docs/stable/getting_started/install.html
- **NumPy** (Numerical support for computing sequence-feature distances and feeding model training) — https://numpy.org/install/

## Examples

```
from sklearn.ensemble import ExtraTreesClassifier; m = ExtraTreesClassifier(max_depth=10, max_features=7, n_estimators=1500); m.fit(X_pairs, y_labels)
```

## Evaluation signals

- Both saved .sav files (vRhyme_machine_model_ET.sav and vRhyme_machine_model_NN.sav) load successfully via pickle/joblib in scikit-learn >= 0.23.0
- Reproduced pair probabilities correctly discriminate 'same' vs 'different' scaffold pairs on held-out pairs, matching the behavior of the reference models in the vRhyme/models/ folder
- Both models must agree (each meeting the 'same' probability threshold) for a pair to be classified 'same' in the hybrid scheme
- Training feature table matches the vRhyme_machine_distances.tsv format with k-mer frequency distances and GC content differences within the --max_gc 0.20 / --min_kmer 0.60 defaults
- aux/test_vRhyme.py reports 'Success' statements for both machine learning models after installation

## Limitations

- vRhyme (and its classifiers) had difficulties separating multiple inactive (non-replicating) prophages from the same host genome
- Binning non-viral genomes is an anticipated limitation; on a non-viral synthetic dataset MetaBat2 performed better than vRhyme
- vRhyme is not meant to bin microbes, and performance on an entire metagenome as input has not been fully evaluated

## Evidence

- [methods] Iterative training and testing on the 39,105-fragment pair set yielded MLPClassifier and ExtraTreesClassifier as the most robust models: [evidence span withheld — anchor status: not_found]
