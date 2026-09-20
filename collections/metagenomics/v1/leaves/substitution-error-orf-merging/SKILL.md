---
name: substitution-error-orf-merging
description: Use when predicting genes in metagenomic reads with substitution errors
  (e.g. Illumina data), where a single base error can introduce a spurious stop codon
  that fragments an otherwise correct ORF prediction.
when_to_use_negative:
- When reads are high-quality assembled contigs or complete genomes — use standard
  Glimmer, which routinely identifies >99% of genes in complete prokaryotic genomes
- When the dominant error mode is indels in homopolymer runs (454) and you specifically
  want frame-branching indel prediction rather than stop-codon substitution merging
- When computational cost is a hard constraint — the article notes Glimmer-MG is more
  computationally expensive than existing software
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Glimmer-MG
  role: Performs the ORF-based gene prediction, evaluating apparent stop codons as
    possible sequencing substitutions and merging adjacent ORFs accordingly
  repo: https://github.com/davek44/Glimmer-MG
- name: Illumina
  role: Sequencing platform whose reads carry the substitution errors this skill models
- name: FragGeneScan
  role: Prior error-aware gene prediction method used as a comparison baseline for
    identifying simulated error positions
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:49763810d958e34f8fd962879602403cceabbc066b846aee7e97b8e3be168515
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/davek44/Glimmer-MG
  repo_url: https://github.com/davek44/Glimmer-MG
  tools:
  - 454 Life Sciences
  - FragGeneScan
  - Glimmer-MG
  - Illumina
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Substitution-error ORF merging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Glimmer-MG's approach to Illumina-style substitution errors: adjacent predicted ORFs whose apparent stop codons are attributable to sequencing substitutions are merged so gene predictions track the true coding frame in error-prone short reads.

## When to use

Use when predicting genes in metagenomic reads with substitution errors (e.g. Illumina data), where a single base error can introduce a spurious stop codon that fragments an otherwise correct ORF prediction.

## When NOT to use

- When reads are high-quality assembled contigs or complete genomes — use standard Glimmer, which routinely identifies >99% of genes in complete prokaryotic genomes
- When the dominant error mode is indels in homopolymer runs (454) and you specifically want frame-branching indel prediction rather than stop-codon substitution merging
- When computational cost is a hard constraint — the article notes Glimmer-MG is more computationally expensive than existing software

## Inputs

- Metagenomic shotgun read sets (e.g. simulated Illumina reads with substitution errors, 454/Sanger reads with indel errors) with per-base quality values
- Trained Glimmer-MG gene prediction models parameterized by phylogenetic classification (Phymm) and clustering (Scimm)

## Outputs

- Gene predictions with sequencing-error annotations (substitution-derived stop codons and indel positions) marking where coding frames are corrected
- Per-configuration gene-prediction sensitivity and precision values (e.g. error_model_ablation.csv)

## How to apply

Run Glimmer-MG in substitution-error mode on read sets such as simulated Illumina metagenomic reads, which carry substitution errors rather than indel artifacts. Glimmer-MG evaluates apparent in-frame stop codons within candidate ORFs against a probabilistic model of sequencing error; when a stop codon is more plausibly explained by a base substitution than by a true gene termination, the flanking ORF predictions are merged into a single gene. This complements the indel branch model used for 454/Sanger reads (which splits ORFs into alternate reading frames at low-quality base calls such as homopolymer runs, capped at two indels per ORF and penalized by base-quality log-likelihoods). Decision point: use substitution mode for Illumina-style data and indel-branching mode for 454-style data, since the error modalities differ. Validate by computing gene-prediction sensitivity and precision against simulated ground truth; the paper reports that on simulated 454 reads with a 2.0% error rate, sensitivity is 73.2% with precision 54.7%, and that error modeling yields more accurate identification of simulated error positions than FragGeneScan.

## Related tools

- **Glimmer-MG** (Performs the ORF-based gene prediction, evaluating apparent stop codons as possible sequencing substitutions and merging adjacent ORFs accordingly) — https://github.com/davek44/Glimmer-MG
- **Illumina** (Sequencing platform whose reads carry the substitution errors this skill models)
- **FragGeneScan** (Prior error-aware gene prediction method used as a comparison baseline for identifying simulated error positions)

## Evaluation signals

- On simulated 454 reads with 2.0% error rate, gene-prediction sensitivity reaches ~73.2% with precision ~54.7%; large deviations suggest error modeling was misapplied
- Disabling indel/error modeling should drop sensitivity toward the ~41.9% ablation baseline on 454 reads at 2.0% error
- Merged ORFs should only span apparent stop codons attributable to substitutions — merged predictions spanning true gene boundaries inflate false positives and degrade precision
- Simulated error positions should be identified more accurately than FragGeneScan on the same benchmark reads

## Limitations

- Predicting genes on error-prone sequences, particularly those with indels, remains a difficult problem even with error modeling
- At 2.0% error on simulated 454 reads many genes are still predicted incorrectly (precision 54.7%)
- Substantial gains are more likely to come from reducing the error rate via improvements to assembly and base-calling accuracy rather than from computation

## Evidence

- [readme] Frame branching at low-quality base calls: [evidence span withheld — anchor status: external_doc]
