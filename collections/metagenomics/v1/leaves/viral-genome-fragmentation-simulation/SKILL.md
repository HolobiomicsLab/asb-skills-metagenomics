---
name: viral-genome-fragmentation-simulation
description: 'Use when you need labeled training data for viral scaffold binning models:
  complete viral genomes must be converted into metagenome-like fragments and paired
  as ''same'' vs ''different'' genome origin.'
when_to_use_negative:
- The input is an entire metagenome rather than predicted viral sequences — vRhyme
  is built to run on viral sequences/scaffolds and whole-metagenome performance 'has
  not been fully evaluated'.
- The goal is binning non-viral (microbial) genomes — vRhyme 'is not meant to bin
  microbes' and this is an anticipated limitation.
- Only pre-binned scaffolds are available and no labeled pairs need to be generated
  from complete genomes.
edam_topics:
- http://edamontology.org/topic_0114
tools:
- name: vRhyme
  role: Source of the fragmentation/feature-distance scheme and reference models folder;
    the trained models are deployed in vRhyme for scaffold pair classification
  repo: https://github.com/AnantharamanLab/vRhyme
- name: scikit-learn
  role: Implements the ExtraTreesClassifier and MLPClassifier trained on the simulated
    fragment pairs (version >= 0.23.0)
- name: Python
  role: Implementation language for genome fragmentation, pair construction, and model
    training scripts
- name: NumPy
  role: Numerical handling of feature distance matrices used as classifier input
- name: Pandas
  role: Tabulation and loading of vRhyme_machine_distances.tsv pair feature distances
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:308b32939c23734883848251f6d4740b72644621306656ac6515d1837257c6f0
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0114
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# viral-genome-fragmentation-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulates metagenome fragmentation by downloading complete viral genomes from NCBI RefSeq/GenBank and IMG/VR, cutting them into scaffold fragments (39,105 total), and constructing labeled same/different scaffold pairs (69,632 pairs) to train vRhyme's supervised classifiers (ExtraTrees and MLP).

## When to use

Use when you need labeled training data for viral scaffold binning models: complete viral genomes must be converted into metagenome-like fragments and paired as 'same' vs 'different' genome origin.

## When NOT to use

- The input is an entire metagenome rather than predicted viral sequences — vRhyme is built to run on viral sequences/scaffolds and whole-metagenome performance 'has not been fully evaluated'.
- The goal is binning non-viral (microbial) genomes — vRhyme 'is not meant to bin microbes' and this is an anticipated limitation.
- Only pre-binned scaffolds are available and no labeled pairs need to be generated from complete genomes.

## Inputs

- Complete viral genomes from NCBI RefSeq, GenBank, and IMG/VR (FASTA nucleotide sequences)
- Per-pair sequence-feature distances (tetranucleotide/k-mer frequency distances and GC content differences), tabulated as vRhyme_machine_distances.tsv

## Outputs

- 39,105 scaffold fragments derived from complete viral genomes
- Labeled training/testing set of 69,632 same/different scaffold pairs
- vRhyme_machine_distances.tsv feature distance table
- Fitted scikit-learn models: vRhyme_machine_model_ET.sav and vRhyme_machine_model_NN.sav

## How to apply

Download complete viral genomes from NCBI RefSeq/GenBank and IMG/VR and fragment them into scaffold fragments to yield 39,105 fragments in total. Construct the labeled training/testing set of 69,632 same/different scaffold pairs, labeling pairs by whether both fragments originate from the same genome. Compute per-pair sequence-feature distance values — tetranucleotide/k-mer frequency distances and GC content differences, the features vRhyme uses with defaults --max_gc 0.20 and --min_kmer 0.60 — and tabulate them as vRhyme_machine_distances.tsv. Train scikit-learn (version >= 0.23.0) ExtraTreesClassifier (max_depth=10, max_features=7, n_estimators=1500) and MLPClassifier (alpha=0.001, beta_1=0.7, beta_2=0.8, hidden_layer_sizes=(5,25,50,75,100,100,75,50,25,5), learning_rate_init=0.0001, max_iter=1250) on the labeled pairs, saving the fitted models as vRhyme_machine_model_ET.sav and vRhyme_machine_model_NN.sav. The rationale is that iterative training/testing on this simulated pair set produced the most robust classifiers for classifying 'same' versus 'different' scaffold pairs in vRhyme.

## Related tools

- **vRhyme** (Source of the fragmentation/feature-distance scheme and reference models folder; the trained models are deployed in vRhyme for scaffold pair classification) — https://github.com/AnantharamanLab/vRhyme
- **scikit-learn** (Implements the ExtraTreesClassifier and MLPClassifier trained on the simulated fragment pairs (version >= 0.23.0))
- **Python** (Implementation language for genome fragmentation, pair construction, and model training scripts)
- **NumPy** (Numerical handling of feature distance matrices used as classifier input)
- **Pandas** (Tabulation and loading of vRhyme_machine_distances.tsv pair feature distances)

## Examples

```
from sklearn.ensemble import ExtraTreesClassifier; model = ExtraTreesClassifier(max_depth=10, max_features=7, n_estimators=1500).fit(X_pairs, y_same_different)
```

## Evaluation signals

- The fragment set contains exactly 39,105 scaffold fragments and the labeled pair set contains 69,632 same/different scaffold pairs.
- vRhyme_machine_distances.tsv is produced with per-pair tetranucleotide/k-mer frequency distances and GC content differences.
- Both saved model files (vRhyme_machine_model_ET.sav and vRhyme_machine_model_NN.sav) load successfully with scikit-learn.
- Reproduced pair probabilities correctly discriminate same vs. different scaffold pairs, requiring the probability output from both models to meet the given threshold.
- Behavior matches the reference models shipped in the vRhyme models/ folder.

## Limitations

- vRhyme had difficulties separating multiple inactive (non-replicating) prophages from the same host genome, which limits the separability of 'same'-genome fragment pairs in some regimes.
- Binning non-viral genomes is an anticipated limitation, so simulated fragments must come from viral genomes.
- Version-sensitive dependencies: scikit-learn >= 0.23.0, NumPy >= 1.17.0, and Pandas >= 1.0.0 are required for reproducible model training and .sav loading.

## Evidence

- [methods] workflow: [evidence span withheld — anchor status: not_found]
- [methods] methods: [evidence span withheld — anchor status: not_found]
