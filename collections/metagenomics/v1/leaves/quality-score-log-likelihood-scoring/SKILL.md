---
name: quality-score-log-likelihood-scoring
description: Use when predicting genes on error-prone short reads (454/Sanger with
  indels, Illumina with substitutions) where per-base quality values are available
  and frame shifts must be tolerated rather than treated as true stop codons.
when_to_use_negative:
- Input is high-quality assembled contigs or complete genomes where standard Glimmer
  is effective (the paper notes Glimmer identifies >99% of genes in complete prokaryotic
  genomes)
- Reads lack per-base quality values, since branch penalties depend on base-quality
  log-likelihoods
- Sequences are so error-prone that indel modeling cannot recover predictions (the
  article notes predicting genes on error-prone sequences with indels remains a difficult
  problem)
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0102
tools:
- name: Glimmer-MG
  role: 'Implements the indel/substitution error models: branches ORFs into alternate
    frames at low-quality base calls penalized by base-quality log-likelihoods, capped
    at two indels per ORF'
  repo: github.com/davek44/Glimmer-MG
- name: 454 Life Sciences
  role: Read/sequencing platform whose homopolymer-run indel artifacts define the
    low-quality candidate sites this scoring targets
- name: Illumina
  role: Read/sequencing platform whose substitution errors are handled in the complementary
    stop-codon-merging mode
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:7ea7a0fe40bd5c0ce0ec70c758b6ba12c41e3e4377c36cd626b5a0c9686920a2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/davek44/Glimmer-MG
  tools:
  - 454 Life Sciences
  - Glimmer-MG
  - Illumina
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Quality-score log-likelihood scoring for indel error prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores candidate indel sites in metagenomic reads by base-quality log-likelihoods so that ORF-based gene prediction (Glimmer-MG) can branch into alternate reading frames at low-quality base calls (e.g. 454 homopolymer runs) instead of truncating genes at frame-shifting errors.

## When to use

Use when predicting genes on error-prone short reads (454/Sanger with indels, Illumina with substitutions) where per-base quality values are available and frame shifts must be tolerated rather than treated as true stop codons.

## When NOT to use

- Input is high-quality assembled contigs or complete genomes where standard Glimmer is effective (the paper notes Glimmer identifies >99% of genes in complete prokaryotic genomes)
- Reads lack per-base quality values, since branch penalties depend on base-quality log-likelihoods
- Sequences are so error-prone that indel modeling cannot recover predictions (the article notes predicting genes on error-prone sequences with indels remains a difficult problem)

## Inputs

- Metagenomic read sets with per-base quality values (454/Sanger reads carrying homopolymer-run indel artifacts, Illumina reads carrying substitution errors)
- Nucleotide sequence files (e.g. FASTA) for gene prediction
- Simulated ground-truth gene annotations for validation runs

## Outputs

- Predicted genes with inferred indel positions and corrected reading frames
- Branch decisions per ORF (original frame vs. two alternate frames) with quality log-likelihood penalties
- Per-configuration sensitivity and precision values (e.g. error_model_ablation.csv)

## How to apply

Traverse each predicted ORF in Glimmer-MG and flag low-quality base calls as indel candidates, with 454 homopolymer-run ends treated as prime candidates. For each candidate, branch the ORF into three alternate readings — the original frame plus the two shifted frames — implying an insertion or deletion, capped at two indels per ORF. Penalize each branch by the log-likelihood of the base quality values at the candidate sites so that high-quality stop codons are kept while low-quality apparent frameshifts are crossed. In substitution-error mode on Illumina reads, merge adjacent ORFs whose apparent stop codons are attributable to sequencing substitutions rather than true gene boundaries. Validate by re-running the same simulations with indel modeling disabled (ablation) and comparing sensitivity/precision against the simulated ground truth.

## Related tools

- **Glimmer-MG** (Implements the indel/substitution error models: branches ORFs into alternate frames at low-quality base calls penalized by base-quality log-likelihoods, capped at two indels per ORF) — github.com/davek44/Glimmer-MG
- **454 Life Sciences** (Read/sequencing platform whose homopolymer-run indel artifacts define the low-quality candidate sites this scoring targets)
- **Illumina** (Read/sequencing platform whose substitution errors are handled in the complementary stop-codon-merging mode)

## Evaluation signals

- On simulated 454 reads at a 2.0% error rate, sensitivity should rise from the no-indel-modeling baseline (~41.9%) to ~73.2% with indel modeling enabled
- With indel modeling enabled, precision on simulated 454 reads at 2.0% error should be ~54.7%
- No ORF receives more than two predicted indels (branching is capped at two per ORF)
- Predicted indel positions should match simulated indel locations more accurately than FragGeneScan on the same reads
- High-quality stop codons should be retained (not crossed), while only low-quality base-call sites trigger frame branching

## Limitations

- Predicting genes on error-prone sequences, particularly those with indels, remains a difficult problem (many genes on simulated 454 reads at 2.0% error are still predicted incorrectly, precision 54.7%)
- Glimmer-MG is more computationally expensive than existing software due to branching and multiple iterations
- Substantial further gains are more likely to come from reducing the error rate via improved assembly and base calling rather than from the prediction algorithm itself

## Evidence

- [readme] README: branching into a different frame at low quality base calls such as homopolymer runs in 454 sequences: [evidence span withheld — anchor status: external_doc]
- [other] Source card workflow: indels capped at two per ORF and penalized by base-quality log-likelihoods: [evidence span withheld — anchor status: not_found]
