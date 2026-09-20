---
name: degenerate-primer-design
description: Use when aligned nucleotide sequences of target functional genes (e.g.,
  amoA-AOB FASTA from ranked/clustered gene sets) must be converted into a minimal
  set of primer pairs that captures maximal diversity of a reference gene set.
when_to_use_negative:
- Input sequences are unaligned or not in FASTA format — EcoFunPrimer's select subcommand
  requires aligned nucleotide sequences
- The goal is ranking/prioritizing gene targets from metagenomes rather than designing
  primers (use the upstream ranking stage instead)
- Only a handful of closely related sequences must be targeted and a single degenerate
  consensus primer suffices; the high-throughput minimal-set machinery is unnecessary
edam_operation: http://edamontology.org/operation_0308
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_0654
tools:
- name: EcoFunPrimer
  role: Embedded primer design tool that generates thermodynamically stable degenerate
    or nondegenerate primers from aligned input genes and user-defined parameters
  repo: https://github.com/rdpstaff/EcoFunPrimer
- name: MetaFunPrimer
  role: Pipeline wrapper that runs EcoFunPrimer on ranked genes, performs in silico
    PCR evaluation with a 100% forward-and-reverse match criterion, and selects the
    minimal primer set capturing maximum reference gene products
  repo: github.com/jialiu232/MetaFunPrimer
- name: Primer3
  role: Underlying primer design software upon which MetaFunPrimer builds; EcoFunPrimer
    compiles primer3 native code
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1128/msystems.00201-21
    title: 'MetaFunPrimer: an Environment-Specific, High-Throughput Primer Design
      Tool for Improved Quantification of Target Genes'
  source_content_hash: sha256:f14636218519db2479f1f793a3fadc228e7a78287ff52135232a2c0e0be3362f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rdpstaff/EcoFunPrimer
  tools:
  - EcoFunPrimer
  - MetaFunPrimer
  - Primer3
  edam_operation: http://edamontology.org/operation_0308
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# degenerate-primer-design

## Summary

Designs thermodynamically stable degenerate and nondegenerate primer pairs from aligned nucleotide sequences of environmentally abundant target genes using EcoFunPrimer within MetaFunPrimer, then enumerates nondegenerate forms and evaluates them via in silico PCR against a reference gene set.

## When to use

Use when aligned nucleotide sequences of target functional genes (e.g., amoA-AOB FASTA from ranked/clustered gene sets) must be converted into a minimal set of primer pairs that captures maximal diversity of a reference gene set.

## When NOT to use

- Input sequences are unaligned or not in FASTA format — EcoFunPrimer's select subcommand requires aligned nucleotide sequences
- The goal is ranking/prioritizing gene targets from metagenomes rather than designing primers (use the upstream ranking stage instead)
- Only a handful of closely related sequences must be targeted and a single degenerate consensus primer suffices; the high-throughput minimal-set machinery is unnecessary

## Inputs

- Aligned nucleotide sequences of ranked target genes in FASTA format (e.g., .fasta/.fa from the upstream gene ranking stage)
- User-defined design parameters (amplicon product length range, melting temperature range, degeneracy limit, e.g., Table S7)
- Reference gene set (e.g., curated amoA-AOB sequences from Fungene v9.6, HMM score >400 and coverage >70.2%) for in silico PCR evaluation

## Outputs

- Minimal set of nondegenerate primer pairs (e.g., 78 amoA-AOB pairs, after 181 nondegenerate candidate pairs)
- In silico PCR evaluation results per candidate pair (100% forward-and-reverse match criterion)
- Coverage metrics per primer set size (e.g., coverage_curve.csv relating set size k to fraction of reference genes amplified)

## How to apply

Load the aligned nucleotide sequences of ranked target genes (e.g., 720 soil-abundant amoA-AOB genes) together with user-defined design parameters such as amplicon product length and melting temperature ranges (per Table S7). Run EcoFunPrimer's 'select' subcommand within MetaFunPrimer, which produces thermodynamically stable primer pairs and allows multiple degenerate output forms per primer pair, respecting per-primer degeneracy limits (e.g., at most 6 degenerate forms per primer; README default degenMax is 8). Convert all degenerate outputs to their nondegenerate forms to enumerate every possible forward/reverse candidate pair (e.g., 181 nondegenerate candidates expanding to a minimal set of 78 pairs). Evaluate each candidate by in silico PCR against the original reference gene set, counting amplification as successful only when both forward and reverse primers achieve a 100% match. Apply MetaFunPrimer's minimal-set selection to output the smallest number of primer pairs capturing the maximum number of reference gene products.

## Related tools

- **EcoFunPrimer** (Embedded primer design tool that generates thermodynamically stable degenerate or nondegenerate primers from aligned input genes and user-defined parameters) — https://github.com/rdpstaff/EcoFunPrimer
- **MetaFunPrimer** (Pipeline wrapper that runs EcoFunPrimer on ranked genes, performs in silico PCR evaluation with a 100% forward-and-reverse match criterion, and selects the minimal primer set capturing maximum reference gene products) — github.com/jialiu232/MetaFunPrimer
- **Primer3** (Underlying primer design software upon which MetaFunPrimer builds; EcoFunPrimer compiles primer3 native code)

## Examples

```
java -Xmx8g -jar /path/to/PrimerDesign.jar --degenMax 6 -i ranked_amoA_genes.fasta
```

## Evaluation signals

- Final yield matches the reported minimal set size (78 nondegenerate amoA-AOB primer pairs on the 720 ranked genes)
- Every output primer pair passes the 100% forward-and-reverse in silico PCR match criterion against the reference gene set
- Full-set in silico coverage reaches ~94% (676/720) of soil-abundant amoA-AOB genes, versus 49% (356/720) for previously published primers
- Each primer respects the degeneracy constraint (at most 6 degenerate forms per primer in the case study; README recommends degenMax no higher than 10)
- Coverage-versus-set-size curve is monotone in cumulative gene capture and the total set stays below the goal of fewer than 100 primer pairs

## Limitations

- The tool was still in development at the time of the README; only computational/in silico validation was provided and further experimental validation is required
- Highly diverse functional genes may yield numerous primer pairs requiring numerous qPCR runs, potentially exceeding HT-qPCR platform capacity
- Design reliability depends on input reference quality; existing primers detect a limited range of phylogenetically diverse genes and often result in misinterpretation

## Evidence

- [results] EcoFunPrimer generates degenerate or nondegenerate primers based on input genes and user parameters: "EcoFunPrimer is a primer design tool which is capable of generating degenerate or nondegenerate primers based on input genes and parameters defined by the user."
- [methods] 100% forward and reverse match criterion for in silico PCR: [evidence span withheld — anchor status: not_found]
- [methods] minimal primer set selection: "MetaFunPrimer outputs the minimal number of primer sets to capture the maximum number of reference gene products"
- [readme] EcoFunPrimer select subcommand description: [evidence span withheld — anchor status: external_doc]
- [readme] degeneracy parameter guidance: "degenMax (default:
                                                        8) - Maximum
                                                        degeneracy per"
