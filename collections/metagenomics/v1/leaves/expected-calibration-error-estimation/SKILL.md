---
name: expected-calibration-error-estimation
description: Use when evaluating a probabilistic 16S rRNA taxonomic classifier whose
  confidence must be verified against reported calibration numbers — e.g. reproducing
  DeepTaxa's per-rank ECE (domain 0.0002 → species 0.0242) on the 69,335-sequence
  Greengenes2 2024.09 test set.
when_to_use_negative:
- Only point-predictions or FASTA sequences without labels are available — ECE requires
  paired predictions and true taxonomic labels, so plain `deeptaxa predict` without
  a --taxonomy-file cannot produce it
- The input is a k-mer or alignment-based classifier (SINTAX, Kraken 2, DADA2) that
  does not emit calibrated softmax probabilities over all seven ranks
- Sequences come from reference databases outside the training vocabulary regime (e.g.
  SILVA, RDP) — the article notes cross-database performance remains untested, so
  calibration numbers would not be comparable to the 0.0242 reference
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: DeepTaxa
  role: Computes per-rank ECE alongside top-1 accuracy and weighted F1 when predict/evaluate
    is run with taxonomy labels on the Greengenes2 test set
  repo: github.com/systems-genomics-lab/deeptaxa
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:e02fcd7280c037ac854494ad780e9a3ccac08246466cea0a913e87062dd626b4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DNABERT-2
  - DeepTaxa
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Expected calibration error estimation

## Summary

Quantifies how well a hierarchical 16S rRNA classifier's softmax confidences match empirical correctness by binning confidence scores and computing expected calibration error (ECE) per taxonomic rank, alongside accuracy and F1. In DeepTaxa this yields species-level ECE ≈ 0.0242 on the Greengenes2 2024.09 test set.

## When to use

Use when evaluating a probabilistic 16S rRNA taxonomic classifier whose confidence must be verified against reported calibration numbers — e.g. reproducing DeepTaxa's per-rank ECE (domain 0.0002 → species 0.0242) on the 69,335-sequence Greengenes2 2024.09 test set.

## When NOT to use

- Only point-predictions or FASTA sequences without labels are available — ECE requires paired predictions and true taxonomic labels, so plain `deeptaxa predict` without a --taxonomy-file cannot produce it
- The input is a k-mer or alignment-based classifier (SINTAX, Kraken 2, DADA2) that does not emit calibrated softmax probabilities over all seven ranks
- Sequences come from reference databases outside the training vocabulary regime (e.g. SILVA, RDP) — the article notes cross-database performance remains untested, so calibration numbers would not be comparable to the 0.0242 reference

## Inputs

- Held-out test FASTA with known taxonomic labels (e.g. gg_2024_09_testing.fna.gz, 69,335 Greengenes2 2024.09 sequences, with gg_2024_09_testing.tsv.gz labels)
- Trained classifier checkpoint emitting per-rank softmax probabilities (e.g. deeptaxa-full-length-v2.pt or all five seed checkpoints)

## Outputs

- Per-rank ECE values (domain through species, e.g. species ECE ≈ 0.0242) written to species_metrics.csv or the deeptaxa predict evaluation output
- Confidence-bin calibration tables (10 equal-width bins per rank)

## How to apply

Run the classifier's predict/evaluate mode on the held-out FASTA with its taxonomy labels so per-sequence predictions and true ranks are available; in DeepTaxa this means softmax at temperature T = 1.0 over the seven rank classifier heads of HybridCNNBERTClassifier. For each taxonomic rank, bin the top-1 softmax confidence into 10 equal-width bins, then compute ECE as the sum over bins of bin weight × |mean confidence − bin accuracy|. Report ECE next to per-rank top-1 accuracy and weighted F1 in species_metrics.csv; the published per-rank ECE ordering runs from domain (0.0002) through genus (0.0145) up to species (0.0242). If multiple seed checkpoints are available, average the per-rank softmax probabilities of all five seeds (e.g. seeds 42, 123, 456, 789, 1011) before binning, since soft-vote ensembling improves both accuracy and calibration. Validate by checking the species ECE lands near the reported 0.0242 and that ECE decreases monotonically from the species rank toward coarser ranks.

## Related tools

- **DeepTaxa** (Computes per-rank ECE alongside top-1 accuracy and weighted F1 when predict/evaluate is run with taxonomy labels on the Greengenes2 test set) — github.com/systems-genomics-lab/deeptaxa

## Examples

```
deeptaxa predict --fasta-file gg_2024_09_testing.fna.gz --taxonomy-file gg_2024_09_testing.tsv.gz --checkpoint deeptaxa-full-length-v2.pt --output-dir ../deeptaxa-outputs/evaluation
```

## Evaluation signals

- Species-rank ECE reproduces within a small tolerance of the published 0.0242 (article, 3-seed mean; README v2 table also lists 0.0242)
- Per-rank ECE ordering matches the published table: domain 0.0002, phylum 0.0022, class 0.0024, order 0.0057, family 0.0074, genus 0.0145, species 0.0242
- ECE is only emitted when ground-truth taxonomy labels are supplied; schema check that the output contains per-rank accuracy, F1, and ECE columns
- ECE is computed with 10 equal-width confidence bins over softmax confidences at T = 1.0 — changing binning or temperature should be flagged as a protocol deviation

## Limitations

- ECE is a binned scalar; the workflow also reports entropy of the softmax distributions as a complementary prediction-uncertainty signal
- Calibration figures are specific to the Greengenes2 2024.09 test set; the article states performance across other reference databases (SILVA, RDP) and independent environmental/clinical datasets remains to be tested
- Checkpoint numbers differ by release: the article reports 92.96% accuracy / 0.0212-scale metrics from a 3-seed mean while the README v2 reports 92.95% (5-seed mean) — cite the matching release when comparing ECE

## Evidence

- [abstract] with F1 above 0.99 from domain through class and a species-level expected calibration error of 0.0242: "with F1 above 0.99 from domain through class and a species-level expected calibration error of 0.0242"
- [readme] adds per-rank accuracy, F1, ECE to the output: [evidence span withheld — anchor status: external_doc]
- [readme] average the per-rank softmax probabilities of all five seed checkpoints for a region and take the argmax at each rank: [evidence span withheld — anchor status: external_doc]
