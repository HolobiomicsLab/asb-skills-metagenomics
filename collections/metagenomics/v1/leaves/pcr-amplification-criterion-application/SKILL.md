---
name: pcr-amplification-criterion-application
description: Use when evaluating in silico whether a set of candidate primers (degenerate
  or expanded to nondegenerate forms) successfully amplifies reference gene sequences,
  and when comparing coverage of new primers against published primer sets.
when_to_use_negative:
- When the goal is de novo thermodynamic primer design from an alignment rather than
  evaluating amplification of existing primers (use EcoFunPrimer's select workflow
  instead)
- When primer-template mismatches are tolerated in the study design — the criterion
  here requires 100% forward and reverse match and will underestimate amplification
  otherwise
- When the input is experimental qPCR amplification data rather than reference gene
  sequences; this skill is strictly in silico
tools:
- name: MetaFunPrimer
  role: Performs the in silico PCR evaluation of designed primers against reference
    genes using the 100% forward-and-reverse match criterion and outputs the minimal
    primer set capturing maximum reference gene products
  repo: https://github.com/jialiu232/MetaFunPrimer_paper_info.git
- name: EcoFunPrimer
  role: Embedded in MetaFunPrimer; generates the degenerate or nondegenerate primer
    pairs that are then evaluated under the amplification criterion
  repo: https://github.com/rdpstaff/EcoFunPrimer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/msystems.00201-21
    title: 'MetaFunPrimer: an Environment-Specific, High-Throughput Primer Design
      Tool for Improved Quantification of Target Genes'
  source_content_hash: sha256:e7c4afaa02ff59a862adf2a531b71a6de2f12499ae4b9e66d86f9a37e93a4787
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jialiu232/MetaFunPrimer_paper_info.git
  tools:
  - EcoFunPrimer
  - MetaFunPrimer
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# pcr-amplification-criterion-application

## Summary

Applies MetaFunPrimer's in silico PCR criterion — a primer pair counts as amplifying a reference gene product only if both forward and reverse primers achieve a 100% match against the sequence — to quantify the detection coverage of a designed primer set (e.g., 78 amoA-AOB pairs) relative to previously published primers.

## When to use

Use when evaluating in silico whether a set of candidate primers (degenerate or expanded to nondegenerate forms) successfully amplifies reference gene sequences, and when comparing coverage of new primers against published primer sets.

## When NOT to use

- When the goal is de novo thermodynamic primer design from an alignment rather than evaluating amplification of existing primers (use EcoFunPrimer's select workflow instead)
- When primer-template mismatches are tolerated in the study design — the criterion here requires 100% forward and reverse match and will underestimate amplification otherwise
- When the input is experimental qPCR amplification data rather than reference gene sequences; this skill is strictly in silico

## Inputs

- Designed primer pairs (nondegenerate, or degenerate primers expanded to all nondegenerate forms)
- Reference functional gene nucleotide sequences (e.g., amoA-AOB from Fungene v9.6 after HMM score >400 and coverage >70.2% filtering)
- Optionally, previously published primer sequences for coverage benchmarking

## Outputs

- Per-pair or per-set list of amplified reference gene products under the 100% forward-and-reverse match criterion
- Cumulative coverage fraction of the reference gene set (e.g., coverage_curve.csv of coverage vs. primer-set size)
- Comparison table of new vs. published primer coverage (e.g., 94% vs. 49%)

## How to apply

Load the designed primer pairs (e.g., the 78 nondegenerate amoA-AOB pairs) and expand any degenerate primers to all nondegenerate forms. Run in silico PCR of each primer pair against the reference gene set (e.g., amoA-AOB sequences curated from Fungene v9.6, filtered at HMM score >400 and >70.2% HMM coverage), applying the strict rule that a pair amplifies a gene product only when both the forward and reverse primers achieve a 100% match against a sequence. Count amplified reference gene products per primer pair or per set, then compute cumulative coverage as the fraction of reference genes detected. MetaFunPrimer itself reports the minimal number of primer sets needed to capture the maximum number of reference gene products, which lets you build a coverage-vs-primer-set-size curve (e.g., 78 pairs reaching 94% = 676/720 soil-abundant amoA-AOB genes vs. 49% = 356/720 for published primers).

## Related tools

- **MetaFunPrimer** (Performs the in silico PCR evaluation of designed primers against reference genes using the 100% forward-and-reverse match criterion and outputs the minimal primer set capturing maximum reference gene products) — https://github.com/jialiu232/MetaFunPrimer_paper_info.git
- **EcoFunPrimer** (Embedded in MetaFunPrimer; generates the degenerate or nondegenerate primer pairs that are then evaluated under the amplification criterion) — https://github.com/rdpstaff/EcoFunPrimer

## Examples

```
java -Xmx8g -jar /path/to/PrimerDesign.jar -i amoA_AOB_aligned.fasta
```

## Evaluation signals

- Every reported amplification call satisfies both forward and reverse 100% match against the reference sequence — spot-check individual gene/primer alignments
- Full-set coverage of the 78 amoA-AOB pairs reproduces the reported ~94% (676/720) against the soil-abundant reference genes
- Published-primer benchmark reproduces ~49% (356/720) coverage, providing a positive-control comparison
- Coverage values are recorded for every subset size evaluated (e.g., coverage_curve.csv has one coverage value per k), and coverage is monotonic non-decreasing as primer pairs are added
- Degenerate primers were expanded to all nondegenerate forms before matching, so no degenerate IUPAC codes remain in evaluated sequences

## Limitations

- Further experimental validation is required; only computational/in silico validation is provided by the pipeline
- For highly diverse functional genes, MetaFunPrimer may design numerous primer pairs requiring numerous qPCR runs, potentially exceeding even HT-qPCR platform capacity
- The 100% match criterion is strict — any single mismatch in either primer disqualifies amplification, which may underestimate real-world amplification

## Evidence

- [methods] a pair of primers was considered to amplify a gene product successfully if both the forward and reverse primers achieve a 100% match against a sequence: [evidence span withheld — anchor status: not_found]
- [methods] MetaFunPrimer outputs the minimal number of primer sets to capture the maximum number of reference gene products: "MetaFunPrimer outputs the minimal number of primer sets to capture the maximum number of reference gene products"
- [abstract] these amoA-AOB primers can cover 94% of the amoA-AOB genes observed in the 1,550 soil metagenomes compared with a 49% estimated coverage by previously published primers: "these amoA-AOB primers can cover 94% of the amoA-AOB genes observed in the 1,550 soil metagenomes compared with a 49% estimated coverage by previously published primers"
- [methods] filtering for a Hidden Markov Model (HMM) search score of >400 and HMM coverage over 70.2% amino acid similarity: "filtering for a Hidden Markov Model (HMM) search score of >400 and HMM coverage over 70.2% amino acid similarity"
- [readme] The Primer Design terminal program produces thermodynamically stable primer pairs for qPCR from an input of aligned nucleotide sequences: [evidence span withheld — anchor status: external_doc]
