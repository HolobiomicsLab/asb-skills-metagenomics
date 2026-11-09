---
name: 454-homopolymer-error-modeling
description: Use when gene-finding on 454 metagenomic reads where homopolymer-run
  artifacts cause frame-disrupting indels, or when simulating/validating 454 read
  error models against true gene coordinates.
when_to_use_negative:
- Input reads are error-free or from technologies without homopolymer artifacts (e.g.
  error-free Sanger or Illumina reads), where the 454 indel model adds no benefit
- Sequences are complete prokaryotic genomes, where standard Glimmer routinely identifies
  >99% of genes without error modeling
- A quick, computationally cheap prediction is required — the article notes Glimmer-MG
  is more computationally expensive than existing software
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Glimmer-MG
  role: Performs frame-branching gene prediction with insertion/deletion and stop
    codon-introducing substitution error modeling for 454 homopolymer artifacts
  repo: https://github.com/davek44/Glimmer-MG
- name: FragGeneScan
  role: Comparator gene predictor with its own error model, used as baseline for indel-prediction
    accuracy on 454 reads
- name: 454 Life Sciences
  role: Sequencing technology whose homopolymer-run error profile defines the error
    model being simulated and predicted
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:41831c9732b2007adf220054dde1f96a0832f2200a526c4bcd704db41bb19e20
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
  - Glimmer
  - Glimmer-MG
  - MetaGeneMark
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# 454-homopolymer-error-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Model 454-specific sequencing errors—particularly insertions/deletions arising from homopolymer runs—so that metagenomic gene predictions can track the correct coding frame on raw error-prone reads, as implemented in Glimmer-MG.

## When to use

Use when gene-finding on 454 metagenomic reads where homopolymer-run artifacts cause frame-disrupting indels, or when simulating/validating 454 read error models against true gene coordinates.

## When NOT to use

- Input reads are error-free or from technologies without homopolymer artifacts (e.g. error-free Sanger or Illumina reads), where the 454 indel model adds no benefit
- Sequences are complete prokaryotic genomes, where standard Glimmer routinely identifies >99% of genes without error modeling
- A quick, computationally cheap prediction is required — the article notes Glimmer-MG is more computationally expensive than existing software

## Inputs

- Raw 454 metagenomic sequencing reads (e.g. FASTA/FASTQ) from environmental samples such as the human gut microbiome (TS28, TS50)
- Reference GenBank genomes with known gene coordinates for simulation and ground-truth scoring
- Simulated 454 read sets generated with homopolymer-run indel error models at 0%–2% error rates

## Outputs

- Gene predictions with frame-consistent coordinates despite homopolymer-induced indels
- Nucleotide-level sensitivity and precision metrics, including 3'/5' and indel-prediction accuracy (e.g. 23.2% indel sensitivity / 63.8% indel precision at 1.0% 454 error)
- Comparison tables of sensitivity/precision per read technology and error rate

## How to apply

Fit an explicit error model in which low-quality base calls arise from homopolymer runs, causing insertions and deletions, and allow the gene predictor to branch into a different frame at these positions, also predicting stop codon-introducing substitution errors. In Glimmer-MG this is combined with phylogenetic classification (Phymm) and Scimm clustering with retraining before final predictions. When simulating evaluation data, generate 454 reads (e.g. with FlowSim from reference GenBank genomes under a Pareto abundance distribution) applying the 454 homopolymer error model at error rates spanning 0% to 2%. Score predictions against known gene coordinates at the nucleotide level, including 3'/5' sensitivity and precision; the paper's reference operating point is 73.2% sensitivity and 54.7% precision on simulated 454 reads with a 2.0% error rate. Rationale: error-aware frame branching lets predictions track coding frames in raw error-prone sequences and yields more accurate identification of simulated indel positions than FragGeneScan.

## Related tools

- **Glimmer-MG** (Performs frame-branching gene prediction with insertion/deletion and stop codon-introducing substitution error modeling for 454 homopolymer artifacts) — https://github.com/davek44/Glimmer-MG
- **FragGeneScan** (Comparator gene predictor with its own error model, used as baseline for indel-prediction accuracy on 454 reads)
- **454 Life Sciences** (Sequencing technology whose homopolymer-run error profile defines the error model being simulated and predicted)

## Evaluation signals

- On simulated 454 reads at 2.0% error rate, sensitivity and precision should approximate the reported 73.2% and 54.7%
- Indel-prediction sensitivity/precision at 1.0% error on 454 reads should reach the reported 23.2% and 63.8% within tolerance
- All computed sensitivity/precision values must fall within [0, 100] for every predictor and error-rate combination
- Glimmer-MG should show more accurate identification of simulated indel positions than FragGeneScan across all tested read lengths and error rates
- Retraining deltas should be positive and grow with error rate (e.g. 0.8% sensitivity gain without errors vs. 2.8% at 2.0% error on 454 reads)

## Limitations

- Predicting genes on error-prone sequences, particularly those with indels, remains a difficult problem — at 2.0% 454 error, precision of only 54.7% indicates many genes are predicted incorrectly
- Glimmer-MG is more computationally expensive than existing software
- Highly diverged organisms where optimal model parameters are unclear present another challenge
- Substantial further gains are more likely to come from reducing the error rate via improved assembly and base-calling rather than computation

## Evidence

- [readme] Finally, Glimmer-MG can predict insertions and deletions in the sequence by branching into a different frame at low quality base calls such as homopolymer runs in 454 sequences.: [evidence span withheld — anchor status: external_doc]
- [abstract] we model both insertion/deletion and substitution sequencing errors using a different approach than previous software: "we model both insertion/deletion and substitution sequencing errors using a different approach than previous software"
- [intro] Glimmer-MG predicts insertion, deletion and stop codon-introducing substitution errors in order to more closely track coding frames in raw error-prone sequences: "Glimmer-MG predicts insertion, deletion and stop codon-introducing substitution errors in order to more closely track coding frames in raw error-prone sequences"
- [discussion] On simulated 454 reads with a 2.0% error rate, Glimmer-MG's sensitivity is 73.2% and precision is 54.7%: "On simulated 454 reads with a 2.0% error rate, Glimmer-MG's sensitivity is 73.2% and precision is 54.7%"
