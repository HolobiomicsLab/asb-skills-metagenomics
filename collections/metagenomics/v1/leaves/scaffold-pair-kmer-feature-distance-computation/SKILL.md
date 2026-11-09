---
name: scaffold-pair-kmer-feature-distance-computation
description: Use when building labeled same/different scaffold-pair training data
  or reproducing vRhyme's supervised classifiers, which require nucleotide feature
  distance values (k-mer frequencies, GC content) as input features per pair.
when_to_use_negative:
- Input scaffolds are not predicted viral sequences — vRhyme is built for viral scaffolds
  and is not meant to bin microbes
- The goal is binning a real metagenome rather than model training/reproduction —
  vRhyme computes features internally and the pair-distance table is only needed for
  the supervised classifier construction workflow
edam_operation: http://edamontology.org/operation_0284
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3473
tools:
- name: vRhyme
  role: Defines the nucleotide feature set and thresholds (--max_gc 0.20, --min_kmer
    0.60) that the per-pair distances must replicate; ships the trained models these
    features feed
  repo: https://github.com/AnantharamanLab/vRhyme
- name: Python
  role: Implementation language for computing and tabulating the pair distances
- name: Pandas
  role: Tabulates the per-pair distance values into vRhyme_machine_distances.tsv
- name: NumPy
  role: Numerical computation of k-mer frequency and GC content distances
- name: scikit-learn
  role: Consumes the distance feature matrix to train ExtraTreesClassifier and MLPClassifier
    pair classifiers
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:25149966bb66250d6da29297162b6824fe32acd8d6df5a07b29e62e50252060c
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
  edam_operation: http://edamontology.org/operation_0284
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# scaffold-pair-kmer-feature-distance-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-pair sequence-feature distances (tetranucleotide/k-mer frequency distances and GC content differences) between scaffold pairs, which vRhyme uses as features for supervised machine learning classification of 'same' versus 'different' genome origin.

## When to use

Use when building labeled same/different scaffold-pair training data or reproducing vRhyme's supervised classifiers, which require nucleotide feature distance values (k-mer frequencies, GC content) as input features per pair.

## When NOT to use

- Input scaffolds are not predicted viral sequences — vRhyme is built for viral scaffolds and is not meant to bin microbes
- The goal is binning a real metagenome rather than model training/reproduction — vRhyme computes features internally and the pair-distance table is only needed for the supervised classifier construction workflow

## Inputs

- Fragmented viral genome FASTA sequences (39,105 genome fragments from NCBI RefSeq/GenBank and IMG/VR in the reference build)
- Labeled scaffold-pair list (69,632 same/different pairs)

## Outputs

- vRhyme_machine_distances.tsv — per-pair sequence-feature distance values (k-mer/tetranucleotide frequency distances and GC content differences)

## How to apply

Start from fragmented viral genomes (39,105 fragments in the reference construction) assembled into 69,632 labeled same/different scaffold pairs. For each pair, compute sequence-feature distances: tetranucleotide/k-mer frequency distances and GC content differences — the same nucleotide features vRhyme uses at runtime with defaults --max_gc 0.20 and --min_kmer 0.60. Tabulate the per-pair distance values into a table named vRhyme_machine_distances.tsv using NumPy and Pandas. These distances then serve as the input feature matrix for training the ExtraTreesClassifier and MLPClassifier models saved as vRhyme_machine_model_ET.sav and vRhyme_machine_model_NN.sav. Validate by checking the table's row count matches the pair count and that distance distributions discriminate same-genome pairs from different-genome pairs consistent with the trained models.

## Related tools

- **vRhyme** (Defines the nucleotide feature set and thresholds (--max_gc 0.20, --min_kmer 0.60) that the per-pair distances must replicate; ships the trained models these features feed) — https://github.com/AnantharamanLab/vRhyme
- **Python** (Implementation language for computing and tabulating the pair distances)
- **Pandas** (Tabulates the per-pair distance values into vRhyme_machine_distances.tsv)
- **NumPy** (Numerical computation of k-mer frequency and GC content distances)
- **scikit-learn** (Consumes the distance feature matrix to train ExtraTreesClassifier and MLPClassifier pair classifiers)

## Examples

```
import pandas as pd; df = pd.read_csv('vRhyme_machine_distances.tsv', sep='\t')
```

## Evaluation signals

- Row count of vRhyme_machine_distances.tsv equals the number of labeled pairs (69,632 in the reference build)
- Same-genome pairs show k-mer frequency distances below the --min_kmer 0.60-style threshold while different-genome pairs are separated by GC content differences consistent with --max_gc 0.20 defaults
- The distance matrix reproduces 'same'/'different' probabilities when passed to the saved vRhyme_machine_model_ET.sav and vRhyme_machine_model_NN.sav models
- Features cover both tetranucleotide/k-mer frequency distances and GC content differences, matching the nucleotide features vRhyme classifies on
- Table loads cleanly with Pandas (tab-separated, consistent numeric columns, no missing distance values)

## Limitations

- Sequence-feature similarity alone cannot separate scaffolds that sound the same but come from different genomes — coverage variance comparisons are needed to disambiguate them
- Distance features are computed on genome fragments, so distance distributions depend on how viral genomes were fragmented into the 39,105 fragments
- vRhyme had difficulties separating multiple inactive (non-replicating) prophages from the same host genome, where these nucleotide features are less discriminative

## Evidence

- [methods] tetranucleotide/k-mer frequency distances and GC content differences, the features vRhyme uses with defaults --max_gc 0.20 and --min_kmer 0.60: [evidence span withheld — anchor status: not_found]
