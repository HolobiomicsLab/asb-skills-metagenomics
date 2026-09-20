---
name: orf-frame-branching-error-correction
description: Use when predicting genes in raw metagenomic reads (454, Sanger, or Illumina)
  that carry indel or substitution sequencing errors, so that coding-frame tracking
  must tolerate insertions, deletions, and stop-codon-introducing substitutions.
when_to_use_negative:
- Input is assembled, high-accuracy contigs with negligible sequencing error — frame
  branching adds complexity without benefit
- Reads lack per-base quality values, undermining quality-likelihood penalization
  of indel candidates
- The task is taxonomic classification or clustering itself (use Phymm/Scimm directly
  rather than gene prediction with error correction)
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Glimmer-MG
  role: Primary gene predictor that performs ORF frame branching at indel candidates
    and stop-codon substitution merging; retrained per Scimm cluster using Phymm classifications
  repo: https://github.com/davek44/Glimmer-MG
- name: 454 Life Sciences
  role: Sequencing platform whose homopolymer-run indel artifacts motivate the frame-branching
    error model; simulated 454 reads at 2.0% error used for validation
- name: Illumina
  role: Sequencing platform whose substitution errors motivate the stop-codon-introducing
    substitution model and adjacent-ORF merging
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:99e2e1a0aa1b6d18f18315cfb2aef4f22f0de4f155b596d95d095f2157351652
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
  - Glimmer-MG
  - Illumina
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# orf-frame-branching-error-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Gene prediction on error-prone metagenomic reads in which each ORF branches into alternate reading frames at predicted indel positions (e.g. 454 homopolymer-run artifacts) and adjacent ORFs separated by apparent stop codons are merged when attributable to substitution errors, as implemented in Glimmer-MG.

## When to use

Use when predicting genes in raw metagenomic reads (454, Sanger, or Illumina) that carry indel or substitution sequencing errors, so that coding-frame tracking must tolerate insertions, deletions, and stop-codon-introducing substitutions.

## When NOT to use

- Input is assembled, high-accuracy contigs with negligible sequencing error — frame branching adds complexity without benefit
- Reads lack per-base quality values, undermining quality-likelihood penalization of indel candidates
- The task is taxonomic classification or clustering itself (use Phymm/Scimm directly rather than gene prediction with error correction)

## Inputs

- Metagenomic read sets (454, Sanger, or Illumina) in standard sequence format with per-base quality values
- Glimmer-MG configuration with indel-error modeling enabled (or substitution-error mode for Illumina reads)
- Phymm phylogenetic classifications and Scimm cluster assignments for model parameterization
- Simulated read ground truth (for evaluation against known gene coordinates)

## Outputs

- Predicted genes/ORFs with frame-branch corrections at indel candidate positions
- Predicted insertion, deletion, and stop-codon-introducing substitution error positions
- Merged ORF predictions across substitution-attributable stop codons
- Per-configuration sensitivity and precision values (e.g. error_model_ablation.csv)

## How to apply

Run Glimmer-MG on the read set with per-base quality values, enabling indel-error modeling so each predicted ORF branches into three reading frames (original plus two alternates) at low-quality base calls such as ends of 454 homopolymer runs, capped at two indels per ORF and penalized by base-quality log-likelihoods. For Illumina reads, run in substitution-error mode so that adjacent predicted ORFs whose apparent stop codons are attributable to sequencing substitutions are merged into a single gene. Pair this with Glimmer-MG's upstream Phymm-based phylogenetic classification and Scimm clustering, followed by in-cluster retraining on initial predictions, to parameterize the interpolated Markov models. To quantify the contribution of error modeling, re-run with indel modeling disabled and compare sensitivity/precision against simulated ground truth (the paper reports 454 sensitivity rising from ~41.9% to 73.2% with precision 54.7% at a 2.0% error rate).

## Related tools

- **Glimmer-MG** (Primary gene predictor that performs ORF frame branching at indel candidates and stop-codon substitution merging; retrained per Scimm cluster using Phymm classifications) — https://github.com/davek44/Glimmer-MG
- **454 Life Sciences** (Sequencing platform whose homopolymer-run indel artifacts motivate the frame-branching error model; simulated 454 reads at 2.0% error used for validation)
- **Illumina** (Sequencing platform whose substitution errors motivate the stop-codon-introducing substitution model and adjacent-ORF merging)

## Evaluation signals

- On simulated 454 reads at 2.0% error, sensitivity with indel modeling enabled should be ~73.2% with precision ~54.7%
- Ablation baseline with indel modeling disabled should show markedly lower 454 sensitivity (~41.9%), confirming the branching model's contribution
- Predicted indel positions should be more accurately identified than FragGeneScan's on the same simulated reads
- No ORF should exceed two predicted indels per ORF, and each branch should be penalized by base-quality log-likelihoods
- Adjacent ORF merges should occur only where the separating stop codon is plausibly a sequencing substitution

## Limitations

- Predicting genes on error-prone sequences, particularly those with indels, remains a difficult problem — precision on 2.0%-error 454 reads is only 54.7%, indicating many genes are predicted incorrectly
- Glimmer-MG is more computationally expensive than existing gene prediction software
- Substantial further gains likely depend on reducing raw error rates via improved assembly and base-calling rather than computational correction

## Evidence

- [readme] Finally, Glimmer-MG can predict insertions and deletions in the sequence by branching into a different frame at low quality base calls such as homopolymer runs in 454 sequences.: [evidence span withheld — anchor status: external_doc]
