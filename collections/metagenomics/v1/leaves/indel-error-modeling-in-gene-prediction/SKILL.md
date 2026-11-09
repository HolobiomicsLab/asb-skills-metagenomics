---
name: indel-error-modeling-in-gene-prediction
description: Use when predicting protein-coding genes in error-prone sequencing reads
  (especially 454 reads with homopolymer-run indel artifacts and per-base quality
  values) where indel errors shift the reading frame and truncate ORFs.
when_to_use_negative:
- Input reads are high-accuracy Illumina reads where the dominant error mode is substitution,
  not indel — use Glimmer-MG's stop-codon substitution mode instead of indel branching
- Assembled contigs or complete genomes where few indel errors remain; the plain Glimmer
  framework already identifies >99% of genes in complete prokaryotic genomes
- The goal is only classifier training or clustering (Phymm/Scimm) with no gene prediction
  step
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: Glimmer-MG
  role: Gene predictor that performs ORF traversal with indel-error branching into
    alternate reading frames at low-quality base calls, capped at two indels per ORF
  repo: https://github.com/davek44/Glimmer-MG
- name: FragGeneScan
  role: Prior method used as a comparator to verify that Glimmer-MG identifies simulated
    indel positions more accurately
- name: 454 Life Sciences
  role: Sequencing platform whose homopolymer-run indel artifacts define the error
    model being simulated and tested
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:502a7e0dc135f0135a0aedd99e45e4b6c7f8e2a0298c704590884f1433839dd5
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# indel-error-modeling-in-gene-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A Glimmer-MG technique that predicts insertion and deletion sequencing errors in raw metagenomic reads by branching each ORF into alternate reading frames at low-quality base calls (e.g. 454 homopolymer runs), so gene predictions track the true coding frame despite indel errors.

## When to use

Use when predicting protein-coding genes in error-prone sequencing reads (especially 454 reads with homopolymer-run indel artifacts and per-base quality values) where indel errors shift the reading frame and truncate ORFs.

## When NOT to use

- Input reads are high-accuracy Illumina reads where the dominant error mode is substitution, not indel — use Glimmer-MG's stop-codon substitution mode instead of indel branching
- Assembled contigs or complete genomes where few indel errors remain; the plain Glimmer framework already identifies >99% of genes in complete prokaryotic genomes
- The goal is only classifier training or clustering (Phymm/Scimm) with no gene prediction step

## Inputs

- Metagenomic read sets (FASTA/FASTQ) with per-base quality values, e.g. simulated 454/Sanger reads carrying homopolymer-run indel errors at a 2.0% error rate
- Simulated ground-truth gene annotations for sensitivity/precision evaluation
- Glimmer-MG-trained interpolated Markov models (IMMs) parameterized by Phymm phylogenetic classification and Scimm clusters

## Outputs

- Glimmer-MG gene predictions that track coding frames across predicted indels, including predicted insertion/deletion positions
- Per-configuration gene-prediction sensitivity and precision values (e.g. error_model_ablation.csv comparing indel-modeling-on vs. off)
- Predicted indel positions for comparison against simulated indel ground truth (e.g. versus FragGeneScan)

## How to apply

Run Glimmer-MG on reads carrying per-base quality values and enable indel-error modeling, which traverses each predicted ORF, flags low-quality base calls as indel candidates, and splits the ORF into three branches: the original frame plus two alternate frames implying insertion or deletion predictions. Cap the search at two indels per ORF and penalize branch choices by base-quality log-likelihoods so high-quality stop codons are not misinterpreted as errors. For Illumina reads, run Glimmer-MG in substitution-error mode instead, merging adjacent predicted ORFs whose apparent stop codons are attributable to sequencing substitutions. Establish an ablation baseline by re-running the same reads with indel modeling disabled, then compute sensitivity and precision against the known ground truth. On simulated 454 reads at 2.0% error, expect sensitivity to rise from ~41.9% (no indel model) to ~73.2% with indel modeling, at precision ~54.7%.

## Related tools

- **Glimmer-MG** (Gene predictor that performs ORF traversal with indel-error branching into alternate reading frames at low-quality base calls, capped at two indels per ORF) — https://github.com/davek44/Glimmer-MG
- **FragGeneScan** (Prior method used as a comparator to verify that Glimmer-MG identifies simulated indel positions more accurately)
- **454 Life Sciences** (Sequencing platform whose homopolymer-run indel artifacts define the error model being simulated and tested)

## Evaluation signals

- On simulated 454 reads at 2.0% error rate, sensitivity with indel modeling enabled reaches ~73.2% versus a ~41.9% no-indel-modeling ablation baseline, at precision ~54.7%
- Indel-modeling-on runs must show a large sensitivity delta over the ablation run on the same simulated read sets; if not, quality values or indel branching are not being consumed
- Predicted indel positions are more accurately recovered than by FragGeneScan on the same simulated data
- Branching must be capped at two indels per ORF, and predicted indel sites should correlate with low-quality base calls and homopolymer-run ends
- Predictions on error-free reads should not be degraded by indel modeling (no spurious frame branches at high-quality bases)

## Limitations

- Predicting genes on error-prone sequences, particularly those with indels, remains a difficult problem — precision on 2.0%-error 454 reads is only 54.7%, indicating many genes are predicted incorrectly
- Glimmer-MG is more computationally expensive than existing gene prediction software
- Substantial further gains are expected from reducing the error rate via improved assembly and base-calling rather than from computation

## Evidence

- [readme] README indel branching: [evidence span withheld — anchor status: external_doc]
