---
name: long-read-error-tolerance-interpretation
description: Use when evaluating k-mer-based metagenome comparison tools on simulated
  long-read (PacBio) datasets — e.g. the CAMI 'toy' HMP samples — where you must judge
  whether clustering-by-origin errors stem from the error model rather than the algorithm.
when_to_use_negative:
- Input samples are real short-read data with low error rates, where any misclassification
  cannot be attributed to a PacBio-style error model and this tolerance-interpretation
  logic does not apply
- The goal is a single pairwise comparison rather than an all-vs-all distance matrix
  suitable for body-site clustering
- Data-reduction approaches (sketching-only or presence/absence metrics) are the accepted
  standard, since the article reports such methods 'greatly diminish the resolution'
  of comparative analyses — but if resolution loss is tolerable, a lighter-weight
  workflow is preferable
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Libra
  role: Computes the all-vs-all k-mer cosine-similarity distance matrix (COSINESIMILARITY,
    LOGARITHM weighting, MAP mode) over the long-read samples
  repo: https://github.com/iychoi/libra
- name: Mash
  role: Indexes each sample by unique k-mers and emits the pairwise MinHash distance
    matrix as an independent comparison baseline
- name: Simka
  role: Computes Jaccard-ab and Bray-Curtis distance matrices over the same samples
    as a third independent baseline
- name: Hadoop
  role: Execution framework on which Libra's preprocess and core (scoring) MapReduce
    jobs are run via 'hadoop jar libra-all.jar'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:509e61132a5eca53a14eff97a1bcbbf86686e9d64861697d68aad0e20dc96cea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Hadoop
  - Libra
  - Mash
  - Simka
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# long-read-error-tolerance-interpretation

## Summary

Validates whether all-vs-all metagenome distance tools (Mash, Libra, Simka) cluster simulated PacBio long-read samples correctly despite elevated sequencing error, by clustering each tool's distance matrix and attributing any residual body-site misclassifications to the long-read error rate rather than to tool failure.

## When to use

Use when evaluating k-mer-based metagenome comparison tools on simulated long-read (PacBio) datasets — e.g. the CAMI 'toy' HMP samples — where you must judge whether clustering-by-origin errors stem from the error model rather than the algorithm.

## When NOT to use

- Input samples are real short-read data with low error rates, where any misclassification cannot be attributed to a PacBio-style error model and this tolerance-interpretation logic does not apply
- The goal is a single pairwise comparison rather than an all-vs-all distance matrix suitable for body-site clustering
- Data-reduction approaches (sketching-only or presence/absence metrics) are the accepted standard, since the article reports such methods 'greatly diminish the resolution' of comparative analyses — but if resolution loss is tolerable, a lighter-weight workflow is preferable

## Inputs

- Simulated CAMI 'toy' Pacific Biosciences (PacBio) long-read FASTA/FASTQ dataset of 49 HMP-derived metagenome samples
- Known body-site origin labels for each sample (ground-truth metadata)

## Outputs

- Pairwise MinHash distance matrix from Mash
- Log-weighted cosine similarity distance matrix from Libra HM scoring (written to /score_dir)
- Jaccard-ab and Bray-Curtis distance matrices from Simka
- Per-tool hierarchical clusterings of samples colored/labelled by body site, with shared-error-rate interpretation of misclassifications

## How to apply

Run the same sample set (e.g. 49 HMP-derived CAMI toy PacBio samples) through each tool to obtain comparable distance matrices: Mash's MinHash distance matrix, Libra's log-weighted cosine similarity matrix, and Simka's Jaccard-ab and Bray-Curtis matrices. For Libra, first preprocess FASTA/FASTQ reads with a k-mer index (default k=20 in the README) via 'hadoop jar libra-all.jar preprocess -k 20 -o /index_dir /source_dir', then score with the default COSINESIMILARITY algorithm, LOGARITHM weighting, and MAP mode. Hierarchically cluster each distance matrix and label samples by known body-site origin. The decision point is interpreting shared misclassifications: if the same small set of samples is misplaced by all three tools, attribute it to the increased PacBio error rate rather than to a tool-specific flaw, because the article validates Libra's distance metric precisely on simulated long- and short-read metagenomes to confirm sensitivity to community composition and abundance.

## Related tools

- **Libra** (Computes the all-vs-all k-mer cosine-similarity distance matrix (COSINESIMILARITY, LOGARITHM weighting, MAP mode) over the long-read samples) — https://github.com/iychoi/libra
- **Mash** (Indexes each sample by unique k-mers and emits the pairwise MinHash distance matrix as an independent comparison baseline)
- **Simka** (Computes Jaccard-ab and Bray-Curtis distance matrices over the same samples as a third independent baseline)
- **Hadoop** (Execution framework on which Libra's preprocess and core (scoring) MapReduce jobs are run via 'hadoop jar libra-all.jar')

## Examples

```
hadoop jar libra-all.jar core -w LOGARITHM -o /score_dir /index_dir
```

## Evaluation signals

- Every tool's clustering (Mash, Libra, Simka) places samples broadly by body site, matching the paper's reported result on the CAMI HMP toy PacBio dataset
- Any misclassifications are small and shared across all three tools, consistent with attribution to the PacBio error rate rather than a single tool's metric
- Libra HM scoring completed with the default COSINESIMILARITY algorithm, LOGARITHM weighting, and MAP mode, producing a full all-vs-all score matrix in /score_dir
- Libra preprocess index built with k-mer size 20 (default) over all 49 samples with no missing indices before scoring

## Limitations

- PacBio's increased error rate causes small body-site misclassifications that are shared across all tools and cannot be eliminated by tool choice
- Libra's distance metric was validated on simulated metagenomes (CAMI toy PacBio, HMP), so performance on real long-read environmental data is extrapolation
- Scoring algorithm, weighting, and run mode default to COSINESIMILARITY/LOGARITHM/MAP; changing them changes the distance matrix and invalidates direct comparison with the paper's clustering

## Evidence

- [intro] CAMI toy dataset is the long-read benchmark: "the simulated Critical Assessment of Metagenome Interpretation (CAMI) “toy” Pacific Biosciences (PacBio) dataset"
- [intro] Long-read validation rationale: "We validate this distance metric using simulated metagenomes (from both short- and long-read technologies) to show that Libra has exceptional sensitivity in distinguishing complex mixed microbiomes."
- [readme] Default scoring algorithm and alternatives: "s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
- [readme] Weighting options used in HM scoring: "w : weighting algorithm. LOGARITHM (default) | BOOLEAN | NATURAL"
- [abstract] All reads vs data reduction: "Libra uses all of the reads to calculate k-mer abundance in a Hadoop architecture that can scale to any size dataset to enable global-scale analyses"
