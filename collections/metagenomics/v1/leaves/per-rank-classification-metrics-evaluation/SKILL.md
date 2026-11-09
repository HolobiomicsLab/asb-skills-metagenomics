---
name: per-rank-classification-metrics-evaluation
description: Use when DeepTaxa (or comparable hierarchical 16S rRNA classifier) predictions
  on labeled test sequences must be scored per taxonomic rank, e.g. reproducing the
  Greengenes2 2024.09 species-level accuracy/F1/ECE benchmarks.
when_to_use_negative:
- Input is an unlabeled FASTA with no ground-truth taxonomy file — `deeptaxa predict`
  will classify but cannot emit accuracy/F1/ECE
- 'The goal is similarity-stratified degradation analysis: per-rank metrics must be
  joined with BLAST/VSEARCH nearest-neighbor identity buckets first, so this skill
  alone is insufficient'
- Input comes from an alternative reference database (SILVA, RDP) or a different marker
  gene — the article notes DeepTaxa was trained and evaluated primarily on Greengenes2
  and cross-database performance remains untested
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: DeepTaxa
  role: Produces per-sequence per-rank predictions and, when given a taxonomy file,
    computes per-rank accuracy, F1, and ECE via `deeptaxa predict --taxonomy-file`
  repo: https://github.com/systems-genomics-lab/deeptaxa
- name: scikit-learn
  role: Computes weighted F1-score and accuracy per rank when metrics are recomputed
    manually from joined prediction/label tables
- name: ensemble_predict.py
  role: Averages per-rank softmax probabilities across the five seed checkpoints before
    taking argmax, yielding ensemble per-rank metrics (full-length ensemble species
    F1 0.9330 vs single-seed 0.9213)
  repo: https://github.com/systems-genomics-lab/deeptaxa
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:71873380bda748d241a2e57d10fe364da21ba3e4528f4967cdaae58f29226166
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - BLAST
  - DeepTaxa
  - VSEARCH
  - ensemble_predict.py
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# per-rank-classification-metrics-evaluation

## Summary

Computes classification metrics (accuracy, weighted F1, expected calibration error) separately at each of the seven taxonomic ranks (domain through species) for DeepTaxa predictions on the Greengenes2 2024.09 held-out test set. Because DeepTaxa emits all ranks in one forward pass, evaluation must decompose performance per rank rather than report a single aggregate score.

## When to use

Use when DeepTaxa (or comparable hierarchical 16S rRNA classifier) predictions on labeled test sequences must be scored per taxonomic rank, e.g. reproducing the Greengenes2 2024.09 species-level accuracy/F1/ECE benchmarks.

## When NOT to use

- Input is an unlabeled FASTA with no ground-truth taxonomy file — `deeptaxa predict` will classify but cannot emit accuracy/F1/ECE
- The goal is similarity-stratified degradation analysis: per-rank metrics must be joined with BLAST/VSEARCH nearest-neighbor identity buckets first, so this skill alone is insufficient
- Input comes from an alternative reference database (SILVA, RDP) or a different marker gene — the article notes DeepTaxa was trained and evaluated primarily on Greengenes2 and cross-database performance remains untested

## Inputs

- gg_2024_09_testing.fna.gz — Greengenes2 2024.09 held-out test FASTA (69,335 sequences)
- gg_2024_09_testing.tsv.gz — ground-truth per-sequence taxonomy labels for the test set
- DeepTaxa checkpoint (e.g. deeptaxa-full-length-v2.pt, 76.4M parameters, or per-seed v2 checkpoints)
- Per-sequence per-rank prediction output from deeptaxa predict (or scripts/ensemble_predict.py soft-vote probabilities)

## Outputs

- Per-rank table of accuracy, F1, and ECE for domain, phylum, class, order, family, genus, species
- Per-epoch/aggregate metrics JSON from training runs (deeptaxa_<uuid>_epoch<N>.json with loss, accuracy, F1, precision, recall per rank)
- Cross-seed mean and standard deviation of species-level accuracy/F1 (e.g. 92.95%, 5-seed mean; species std 0.0004 F1)

## How to apply

Run `deeptaxa predict` with both the FASTA file (gg_2024_09_testing.fna.gz, 69,335 sequences) and the ground-truth taxonomy file (gg_2024_09_testing.tsv.gz) so the tool attaches per-rank accuracy, F1, and ECE to the output. Predictions classify each sequence into all seven ranks (domain through species) in a single forward pass, so aggregate metrics per rank are computed against the matching ground-truth column of the taxonomy TSV. Weighted F1 is the article's headline metric — species-level F1 of 0.9212 with accuracy 92.96% (3-seed mean) — and per-rank ECE quantifies calibration (species ECE 0.0242). To average over seeds, evaluate the seed-42/123/456/789/1011 checkpoints separately (or use the 5-seed soft-vote ensemble in scripts/ensemble_predict.py) and report means and cross-seed standard deviations. If recomputing metrics outside the CLI, join per-sequence species predictions with the taxonomy TSV in pandas and use scikit-learn's f1_score/accuracy per rank column.

## Related tools

- **DeepTaxa** (Produces per-sequence per-rank predictions and, when given a taxonomy file, computes per-rank accuracy, F1, and ECE via `deeptaxa predict --taxonomy-file`) — https://github.com/systems-genomics-lab/deeptaxa
- **scikit-learn** (Computes weighted F1-score and accuracy per rank when metrics are recomputed manually from joined prediction/label tables)
- **ensemble_predict.py** (Averages per-rank softmax probabilities across the five seed checkpoints before taking argmax, yielding ensemble per-rank metrics (full-length ensemble species F1 0.9330 vs single-seed 0.9213)) — https://github.com/systems-genomics-lab/deeptaxa

## Examples

```
deeptaxa predict --fasta-file ../deeptaxa-data/greengenes/gg_2024_09_testing.fna.gz --taxonomy-file ../deeptaxa-data/greengenes/gg_2024_09_testing.tsv.gz --checkpoint ../deeptaxa-data/models/deeptaxa-full-length-v2.pt --output-dir ../deeptaxa-outputs/evaluation
```

## Evaluation signals

- Species-level accuracy lands at 92.95–92.96% with weighted F1 ~0.9208–0.9212 (5-seed mean, single-seed respectively) on the 69,335-sequence test set
- Species-level ECE ≈ 0.0242 and rank-ordered metric degradation from domain (F1 0.9998) to species (F1 0.9208); F1 above 0.99 through class rank
- 5-seed ensemble species F1 (0.9330) exceeds single-seed (0.9213) for full-length checkpoints
- Cross-seed standard deviation ≤0.0004–0.0008 F1 at every rank — larger variance indicates pipeline or seed-selection errors
- All seven ranks present in output table with monotonically decreasing accuracy from domain to species

## Limitations

- Reported numbers are for a single stratified train/validation/test split with three (paper) or five (README) seeds, not k-fold cross-validation, per the article's stated limitations
- Metrics are specific to Greengenes2 2024.09; the article notes performance across SILVA, RDP, or independent environmental/clinical datasets remains untested
- The 5-seed README metrics (92.95%, F1 0.9208) and 3-seed paper metrics (92.96%, F1 0.9212) differ slightly; reproduce with the matching seed set
- Checkpoints use PyTorch pickle serialization and must be downloaded only from the official Hugging Face repository

## Evidence

- [abstract] Species-level headline metrics: "DeepTaxa achieves species-level accuracy of 92.96% and F1 of 0.9212 (3-seed mean; cross-seed standard deviation ≤0.0008 F1 at every rank)"
- [abstract] Per-rank F1 profile and calibration: "with F1 above 0.99 from domain through class and a species-level expected calibration error of 0.0242"
- [readme] Evaluation command semantics: [evidence span withheld — anchor status: external_doc]
- [readme] Evaluation scope: [evidence span withheld — anchor status: external_doc]
- [methods] Test set provenance: "The dataset was partitioned into training (277 336 sequences, 80%) and test (69 335 sequences, 20%) sets using stratified sampling to preserve label distributions."
