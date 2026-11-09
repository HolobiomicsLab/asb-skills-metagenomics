---
name: sequencing-error-modeling-gene-calling
description: Use when gene-calling on unassembled metagenomic reads from error-prone
  platforms (e.g. 454 with homopolymer-run indels), where frameshifts in raw reads
  break standard gene predictors.
when_to_use_negative:
- Input is already high-quality assembled consensus sequence with a low residual error
  rate, where explicit error modeling adds computation without benefit
- The task is standalone sequencing-error correction or base calling rather than gene
  prediction — Glimmer-MG models errors only to keep gene predictions in frame
- Computational cost is the binding constraint, since Glimmer-MG is more computationally
  expensive than existing gene prediction software
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Glimmer-MG
  role: Performs the final gene prediction pass with integrated insertion/deletion
    and stop-codon-introducing substitution error modeling, branching coding frame
    at low-quality base calls such as 454 homopolymer runs
  repo: https://github.com/davek44/Glimmer-MG
- name: Glimmer
  role: Underlying IMM-based gene prediction framework (1st- through 8th-order 3-periodic
    nonhomogeneous Markov models) on which the error modeling is layered
  repo: https://github.com/davek44/Glimmer-MG
- name: 454 Life Sciences
  role: Error-prone read source whose homopolymer-run indels motivate the frame-branching
    error model
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:aaed22d96949fe37ef013d40e7ca24120caede08f45dd52e2885202338b3c3d8
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
  - Glimmer
  - Glimmer-MG
  - Phymm
  - Scimm
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# sequencing-error-modeling-gene-calling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Model insertion, deletion, and stop-codon-introducing substitution errors directly during metagenomic gene prediction (as implemented in Glimmer-MG) so that predicted coding frames can track through raw error-prone reads, e.g. 454 homopolymer-run indels, without requiring prior error correction.

## When to use

Use when gene-calling on unassembled metagenomic reads from error-prone platforms (e.g. 454 with homopolymer-run indels), where frameshifts in raw reads break standard gene predictors.

## When NOT to use

- Input is already high-quality assembled consensus sequence with a low residual error rate, where explicit error modeling adds computation without benefit
- The task is standalone sequencing-error correction or base calling rather than gene prediction — Glimmer-MG models errors only to keep gene predictions in frame
- Computational cost is the binding constraint, since Glimmer-MG is more computationally expensive than existing gene prediction software

## Inputs

- Error-prone metagenomic sequence reads (e.g. simulated 454 reads or real 454 human gut microbiome reads) in FASTA/FASTQ
- Initial Glimmer IMM gene predictions and per-cluster retrained IMM, RBS, and start-codon models from the classification/clustering stages

## Outputs

- gene_predictions.gff — predicted coding features for all input sequences, with predicted insertion, deletion, and substitution error positions tracked along the coding frame
- Simulated/estimated indel positions within predicted genes

## How to apply

Run Glimmer-MG after the classification (Phymm) and clustering (Scimm/PhyScimm) stages, but fold error handling into the final prediction pass rather than correcting reads first. The predictor explicitly branches into a different coding frame at low-quality base calls—most notably homopolymer runs in 454 sequences—modeling insertions, deletions, and stop-codon-introducing substitutions as events that can be predicted alongside the genes themselves. This works in combination with Glimmer-MG's probabilistic model of gene length and start/stop codon presence, which handles truncated genes on short fragments where an error or the read end truncates the ORF. Rationale: sequencing errors shift or prematurely terminate coding frames, so explicitly modeling them lets the IMM-based prediction stay in the correct frame through error-prone raw sequence, and the reported simulated indel positions are compared against truth to show more accurate indel identification than FragGeneScan.

## Related tools

- **Glimmer-MG** (Performs the final gene prediction pass with integrated insertion/deletion and stop-codon-introducing substitution error modeling, branching coding frame at low-quality base calls such as 454 homopolymer runs) — https://github.com/davek44/Glimmer-MG
- **Glimmer** (Underlying IMM-based gene prediction framework (1st- through 8th-order 3-periodic nonhomogeneous Markov models) on which the error modeling is layered) — https://github.com/davek44/Glimmer-MG
- **454 Life Sciences** (Error-prone read source whose homopolymer-run indels motivate the frame-branching error model)

## Evaluation signals

- gene_predictions.gff exists, parses as valid GFF, and contains predicted features for all input sequences
- On simulated 454 reads with 2.0% error, reported sensitivity/precision should be in the ballpark of Glimmer-MG's 73.2% sensitivity and 54.7% precision; precision well above this on raw error-prone reads may indicate error modeling was not actually exercised
- Simulated indel positions reported by the predictor align more accurately with truth than FragGeneScan's on the same reads
- Predicted genes remain in a consistent coding frame through low-quality base calls/homopolymer runs instead of truncating at the first frameshift
- Truncated (partial) genes at read ends are reported with the probabilistic gene-length and start/stop-codon model rather than discarded

## Limitations

- Predicting genes on error-prone sequences, particularly those with indels, remains a difficult problem; at 2.0% simulated 454 error, precision is only 54.7%
- Glimmer-MG is more computationally expensive than existing gene prediction software
- Substantial further gains depend on reducing the error rate via improved assembly and base calling rather than computation
- No software version stated for Glimmer-MG or compared tools in the article

## Evidence

- [readme] Glimmer-MG can predict insertions and deletions in the sequence by branching into a different frame at low quality base calls such as homopolymer runs in 454 sequences.: [evidence span withheld — anchor status: external_doc]
