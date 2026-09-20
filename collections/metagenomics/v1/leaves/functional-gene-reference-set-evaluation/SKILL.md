---
name: functional-gene-reference-set-evaluation
description: Use when a curated reference gene set (e.g., amoA-AOB from Fungene v9.6)
  and candidate primer pairs exist, and you must quantify which subset size achieves
  maximum in silico amplification coverage relative to published primers.
when_to_use_negative:
- The goal is de novo primer design from metagenome-ranked gene targets rather than
  evaluating coverage of an existing primer set
- Input is amplicon sequencing data or a qPCR readout rather than reference gene sequences
  and primer sequences
- Primers will be assessed by thermodynamic or experimental validation only, without
  an in silico reference gene set to match against
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_0080
tools:
- name: MetaFunPrimer
  role: Pipeline whose output primer sets are evaluated; characterizes the number
    of primer pairs required to capture reference gene diversity and performs in silico
    PCR evaluation
  repo: https://github.com/jialiu232/MetaFunPrimer_paper_info.git
- name: EcoFunPrimer
  role: Embedded primer design/evaluation tool; its 'screen' subcommand graphs coverage
    and entropy for each position of the input reference sequences
  repo: https://github.com/rdpstaff/EcoFunPrimer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/msystems.00201-21
    title: 'MetaFunPrimer: an Environment-Specific, High-Throughput Primer Design
      Tool for Improved Quantification of Target Genes'
  source_content_hash: sha256:92adbe505079b8a70ad146ad08f635728234d0b00a562c48d87f48a555dd2651
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jialiu232/MetaFunPrimer_paper_info.git
  tools:
  - EcoFunPrimer
  - MetaFunPrimer
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# functional-gene-reference-set-evaluation

## Summary

Evaluate how many primer pairs are needed to capture the diversity of an environment-specific functional gene set (e.g., 78 nondegenerate amoA-AOB pairs covering 720 soil-abundant genes) by computing in silico PCR coverage curves of cumulative gene detection versus primer-set size.

## When to use

Use when a curated reference gene set (e.g., amoA-AOB from Fungene v9.6) and candidate primer pairs exist, and you must quantify which subset size achieves maximum in silico amplification coverage relative to published primers.

## When NOT to use

- The goal is de novo primer design from metagenome-ranked gene targets rather than evaluating coverage of an existing primer set
- Input is amplicon sequencing data or a qPCR readout rather than reference gene sequences and primer sequences
- Primers will be assessed by thermodynamic or experimental validation only, without an in silico reference gene set to match against

## Inputs

- Aligned nucleotide FASTA of reference functional gene sequences (e.g., amoA-AOB from RDP Fungene v9.6)
- Designed primer pairs (degenerate or nondegenerate), e.g., the 78 amoA-AOB pairs from MetaFunPrimer
- Previously published primer sequences for baseline coverage comparison

## Outputs

- coverage_curve.csv with cumulative in silico amplification fraction for each primer-set size k
- Per-position coverage and entropy plot (EcoFunPrimer screen) for the reference alignment
- Comparison table of designed vs. published primer coverage (e.g., 94% vs. 49%)

## How to apply

Load the designed primer pairs (expanding any degenerate primers to all nondegenerate forms) and the curated reference gene sequences (e.g., amoA-AOB filtered from Fungene v9.6 at HMM score >400 and coverage >70.2%). For each candidate primer-set size k, evaluate in silico PCR against the reference gene products using the criterion that a pair amplifies a gene only if both the forward and reverse primers achieve a 100% match against the sequence. Record the fraction of reference genes amplified for each k (out of, e.g., 720 soil-abundant amoA-AOB genes) and plot cumulative coverage versus primer-set size, annotating where coverage approaches the full-set value (94%, 676/720 for the 78-pair amoA-AOB set versus 49%, 356/720 for previously published primers). Rationale: MetaFunPrimer is explicitly designed to 'characterize the number of primer pairs that is required to capture the diversity in a given reference gene set', and EcoFunPrimer (embedded in MetaFunPrimer) offers a 'screen' subcommand that graphs coverage and entropy per position of the input reference alignment to guide this assessment.

## Related tools

- **MetaFunPrimer** (Pipeline whose output primer sets are evaluated; characterizes the number of primer pairs required to capture reference gene diversity and performs in silico PCR evaluation) — https://github.com/jialiu232/MetaFunPrimer_paper_info.git
- **EcoFunPrimer** (Embedded primer design/evaluation tool; its 'screen' subcommand graphs coverage and entropy for each position of the input reference sequences) — https://github.com/rdpstaff/EcoFunPrimer

## Examples

```
java -Xmx8g -jar PrimerDesign/dist/PrimerDesign.jar screen -i amoA_AOB_aligned.fasta
```

## Evaluation signals

- coverage_curve.csv exists and contains one coverage value per primer-subset size k
- Full-set (k=78) coverage matches the reported 94% (676/720 soil-abundant amoA-AOB genes)
- Published-primer baseline reproduces ~49% (356/720) under the same match criterion
- A gene is counted amplified only when both forward and reverse primers achieve a 100% match
- Coverage curve is monotonically non-decreasing in k and plateaus near the full-set value below ~100 primer pairs

## Limitations

- Only computational/in silico validation is provided; 'further experimental validation is required, but computational efforts can help determine which candidates to test experimentally'
- The 100% match criterion is strict, so single-mismatch amplicons are counted as non-detections and coverage may be underestimated
- Highly diverse functional genes may require numerous primer pairs, which may exceed HT-qPCR platform capacity

## Evidence

- [methods] 100% match criterion for in silico amplification: [evidence span withheld — anchor status: not_found]
- [abstract] Coverage comparison of designed vs published primers: "these amoA-AOB primers can cover 94% of the amoA-AOB genes observed in the 1,550 soil metagenomes compared with a 49% estimated coverage by previously published primers"
- [intro] Purpose of characterizing primer-set size: "characterizes the number of primer pairs that is required to capture the diversity in a given reference gene set"
- [readme] EcoFunPrimer screen subcommand for coverage/entropy plots: [evidence span withheld — anchor status: external_doc]
- [discussion] Computational validation caveat: "Similar to any primer design effort, further experimental validation is required, but computational efforts can help determine which candidates to test experimentally"
