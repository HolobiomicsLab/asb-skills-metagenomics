---
name: hybrid-neural-network-extra-trees-probability-classification
description: Use when classifying pairs of viral scaffolds/fragments as same or different
  genome origin from sequence-feature distances (k-mer/tetranucleotide frequencies,
  GC content), or when retraining/loading vRhyme's .sav ML models.
when_to_use_negative:
- Input is an entire metagenome rather than predicted viral scaffolds — vRhyme is
  built to run on viral sequences/scaffolds and performance on whole metagenomes has
  not been fully evaluated
- The goal is binning non-viral (microbial) genomes — an anticipated limitation of
  vRhyme, and MetaBat2 outperformed vRhyme on non-viral synthetic data
- Only one input sample is available and coverage variance cannot be leveraged, or
  you need unsupervised clustering (e.g. VAMB variational autoencoders or CONCOCT
  Gaussian mixture models) instead of supervised pair classification
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: scikit-learn
  role: Provides the MLPClassifier and ExtraTreesClassifier used to train and apply
    the hybrid same/different scaffold-pair classifier (version >= 0.23.0)
- name: vRhyme
  role: Hosts the pre-trained models (vRhyme/models/vRhyme_machine_model_ET.sav) and
    defaults (--max_gc 0.20, --min_kmer 0.60) that define the feature space for pair
    classification
  repo: https://github.com/AnantharamanLab/vRhyme
- name: NumPy
  role: Numerical dependency for feature matrix manipulation during training and prediction
- name: Pandas
  role: Reads and tabulates vRhyme_machine_distances.tsv feature distance values for
    training
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:5068c5438b7a3a566fdfa52b041d681ec99d9ab875008b1b1a1a62e8538b3c4f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/AnantharamanLab/vRhyme
  repo_url: https://github.com/AnantharamanLab/vRhyme
  tools:
  - NumPy
  - Pandas
  - Python
  - scikit-learn
  - vRhyme
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# hybrid-neural-network-extra-trees-probability-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces and applies vRhyme's hybrid supervised classification scheme, in which scikit-learn MLPClassifier (neural network) and ExtraTreesClassifier models are trained on 69,632 same/different scaffold pairs built from 39,105 fragmented viral genomes, and their probability outputs are combined to classify scaffold pairs as 'same' or 'different' genome origin.

## When to use

Use when classifying pairs of viral scaffolds/fragments as same or different genome origin from sequence-feature distances (k-mer/tetranucleotide frequencies, GC content), or when retraining/loading vRhyme's .sav ML models.

## When NOT to use

- Input is an entire metagenome rather than predicted viral scaffolds — vRhyme is built to run on viral sequences/scaffolds and performance on whole metagenomes has not been fully evaluated
- The goal is binning non-viral (microbial) genomes — an anticipated limitation of vRhyme, and MetaBat2 outperformed vRhyme on non-viral synthetic data
- Only one input sample is available and coverage variance cannot be leveraged, or you need unsupervised clustering (e.g. VAMB variational autoencoders or CONCOCT Gaussian mixture models) instead of supervised pair classification

## Inputs

- FASTA files of complete viral genomes from NCBI RefSeq/GenBank and IMG/VR, fragmented into scaffold fragments (39,105 total)
- Labeled training/testing set of 69,632 'same'/'different' scaffold pairs
- vRhyme_machine_distances.tsv: per-pair sequence-feature distance values (tetranucleotide/k-mer frequency distances, GC content differences)

## Outputs

- vRhyme_machine_model_ET.sav — fitted ExtraTreesClassifier pickle model
- vRhyme_machine_model_NN.sav — fitted MLPClassifier pickle model
- 'same'/'different' classification probabilities for scaffold pairs (hybrid, both models must meet threshold)

## How to apply

Fragment complete viral genomes from NCBI RefSeq/GenBank and IMG/VR into 39,105 scaffold fragments and construct the labeled set of 69,632 same/different scaffold pairs. Compute per-pair sequence-feature distance values (tetranucleotide/k-mer frequency distances and GC content differences, using vRhyme's defaults --max_gc 0.20 and --min_kmer 0.60) and tabulate them as vRhyme_machine_distances.tsv. Train the ExtraTreesClassifier (scikit-learn >= 0.23.0 with NumPy/Pandas; max_depth = 10, max_features = 7, n_estimators = 1500) and save it as vRhyme_machine_model_ET.sav; train the MLPClassifier (alpha = 0.001, beta_1 = 0.7, beta_2 = 0.8, hidden_layer_sizes = (5,25,50,75,100,100,75,50,25,5), learning_rate_init = 0.0001, max_iter = 1250) and save it as vRhyme_machine_model_NN.sav. Run both trained models on held-out pairs and require the probability output from BOTH models to meet the given threshold before calling a pair 'same' genome origin. The rationale is that the two model families complement each other in discriminating nucleotide feature compositions of same-genome scaffold pairs.

## Related tools

- **scikit-learn** (Provides the MLPClassifier and ExtraTreesClassifier used to train and apply the hybrid same/different scaffold-pair classifier (version >= 0.23.0))
- **vRhyme** (Hosts the pre-trained models (vRhyme/models/vRhyme_machine_model_ET.sav) and defaults (--max_gc 0.20, --min_kmer 0.60) that define the feature space for pair classification) — https://github.com/AnantharamanLab/vRhyme
- **NumPy** (Numerical dependency for feature matrix manipulation during training and prediction)
- **Pandas** (Reads and tabulates vRhyme_machine_distances.tsv feature distance values for training)

## Examples

```
from sklearn.neural_network import MLPClassifier; from sklearn.ensemble import ExtraTreesClassifier; nn = MLPClassifier(alpha=0.001, learning_rate_init=0.0001, max_iter=1250); et = ExtraTreesClassifier(max_depth=10, n_estimators=1500)
```

## Evaluation signals

- Unpickled vRhyme_machine_model_ET.sav and vRhyme_machine_model_NN.sav load without error (aux/test_vRhyme.py reports 'Success' for both machine learning models)
- Held-out pair probabilities from both models correctly discriminate 'same' vs 'different' scaffold pairs, matching the behavior of reference models in the vRhyme models/ folder
- Retrained hyperparameters match the published values (ExtraTrees: max_depth 10, max_features 7, n_estimators 1500; MLP: alpha 0.001, learning_rate_init 0.0001, max_iter 1250, hidden_layer_sizes (5,25,50,75,100,100,75,50,25,5))
- Pair-level call only issued when the probability outputs of BOTH the neural network and the extra-trees model meet the threshold (hybrid gating)
- Feature distance table contains per-pair values consistent with vRhyme defaults --max_gc 0.20 and --min_kmer 0.60

## Limitations

- vRhyme had difficulties separating multiple inactive (non-replicating) prophages from the same host genome
- vRhyme has an anticipated limitation in binning non-viral genomes; on a non-viral synthetic dataset MetaBat2 performed better than vRhyme
- The compressed model must be decompressed before use: vRhyme_machine_model_ET.sav.gz must be gunzipped after cloning before the model will load
- Scikit-learn version matters (>= 0.23.0) since .sav pickle models may not load across incompatible versions

## Evidence

- [methods] Iterative training and testing on the 39,105-fragment pair set yielded MLPClassifier (alpha = 0.001, beta_1 = 0.7, beta_2 = 0.8: [evidence span withheld — anchor status: not_found]
- [methods] Train the ExtraTreesClassifier (scikit-learn, version >= 0.23.0, with NumPy and Pandas) on the labeled pairs and save the fitted model as vRhyme_machine_model_ET.sav: [evidence span withheld — anchor status: not_found]
- [methods] confirm the saved .sav model files load successfully and the reproduced pair probabilities correctly discriminate same vs. different scaffold pairs: [evidence span withheld — anchor status: not_found]
