---
name: oligonucleotide-thermodynamic-stability-assessment
description: Use when designing degenerate or nondegenerate primers from aligned nucleotide
  sequences and candidate oligos must be filtered by hairpin/homodimer Tm, GC content
  bounds, and per-primer degeneracy limits before amplification testing.
when_to_use_negative:
- Input sequences are unaligned or in a non-FASTA format — EcoFunPrimer's select subcommand
  requires aligned FASTA input and cannot enumerate stable oligos from raw reads or
  protein sequences alone.
- The goal is ranking or prioritizing which genes to target (the upstream R-score/cumulative
  R-score threshold step) rather than assessing stability of already-designed candidate
  oligos.
- Experimental (wet-lab) validation of primer thermodynamics is required; this skill
  covers only computational/in silico assessment, and the authors note further experimental
  validation is required.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0632
tools:
- name: EcoFunPrimer
  role: Generates thermodynamically stable degenerate or nondegenerate primer pairs
    from aligned nucleotide sequences, applying hairpin/homodimer Tm, GC content,
    and degeneracy filters (PrimerDesign.jar select/screen subcommands).
  repo: https://github.com/rdpstaff/EcoFunPrimer
- name: MetaFunPrimer
  role: Pipeline that embeds EcoFunPrimer to design primers for ranked target genes,
    expands degenerate outputs to nondegenerate candidates, and performs in silico
    PCR and minimal-set selection.
  repo: https://github.com/jialiu232/MetaFunPrimer_paper_info
- name: Primer3
  role: Underlying primer design software upon which the PrimerDesign native code
    (compiled via make in the EcoFunPrimer setup) builds for oligo thermodynamic calculations.
  repo: https://github.com/rdpstaff/EcoFunPrimer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/msystems.00201-21
    title: 'MetaFunPrimer: an Environment-Specific, High-Throughput Primer Design
      Tool for Improved Quantification of Target Genes'
  source_content_hash: sha256:80633def1384cba3b1a9e3d6b6f3c2f72f282086a6dbee4b00ef01334b226a2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rdpstaff/EcoFunPrimer
  tools:
  - EcoFunPrimer
  - MetaFunPrimer
  - Primer3
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# Oligonucleotide thermodynamic stability assessment

## Summary

Assess candidate PCR/qPCR primers for thermodynamic stability — hairpin and homodimer melting temperatures, G+C content, and degeneracy — so that only thermodynamically stable oligos proceed to in silico PCR screening, as done by EcoFunPrimer within MetaFunPrimer.

## When to use

Use when designing degenerate or nondegenerate primers from aligned nucleotide sequences and candidate oligos must be filtered by hairpin/homodimer Tm, GC content bounds, and per-primer degeneracy limits before amplification testing.

## When NOT to use

- Input sequences are unaligned or in a non-FASTA format — EcoFunPrimer's select subcommand requires aligned FASTA input and cannot enumerate stable oligos from raw reads or protein sequences alone.
- The goal is ranking or prioritizing which genes to target (the upstream R-score/cumulative R-score threshold step) rather than assessing stability of already-designed candidate oligos.
- Experimental (wet-lab) validation of primer thermodynamics is required; this skill covers only computational/in silico assessment, and the authors note further experimental validation is required.

## Inputs

- Aligned nucleotide sequences of target genes in FASTA (.fasta/.fa) format
- User-defined design parameters: amplicon product length range, melting temperature ranges, degeneracy cap, GC content bounds (e.g., Table S7 for the amoA-AOB case study)

## Outputs

- Thermodynamically stable primer pairs (degenerate forms), e.g., 181 nondegenerate candidate pairs expanded from the amoA-AOB design
- Minimal nondegenerate primer set capturing maximum reference gene products (78 primer pairs for amoA-AOB)
- Optional 'screen' graph of coverage and entropy for each position of the input reference sequences

## How to apply

Input aligned nucleotide sequences in FASTA format to EcoFunPrimer (PrimerDesign.jar, embedded in MetaFunPrimer) via the 'select' subcommand; optionally run the 'screen' subcommand first to inspect coverage and entropy per alignment position. Enforce thermodynamic stability filters: maximum hairpin temperature (--hairMax, default 35) and maximum homodimer temperature (--homoMax, default 35), plus a G+C content window (--GCFilterMin 0.45 to --GCFilterMax 0.70). Constrain enumeration with oligo position bounds (e.g., forwardMinPos/forwardMaxPos defaults of 100/120) and a per-primer degeneracy cap (--degenMax; the amoA-AOB case study allowed at most 6 degenerate forms, vs. the tool default of 8); each degenerate pair can be expanded into all nondegenerate forward/reverse combinations for candidate enumeration. Apply these filters BEFORE in silico PCR, where a pair is scored as amplifying a gene product only when both forward and reverse primers achieve a 100% match to the reference sequence; surviving pairs are then reduced to a minimal set capturing the maximum number of reference gene products.

## Related tools

- **EcoFunPrimer** (Generates thermodynamically stable degenerate or nondegenerate primer pairs from aligned nucleotide sequences, applying hairpin/homodimer Tm, GC content, and degeneracy filters (PrimerDesign.jar select/screen subcommands).) — https://github.com/rdpstaff/EcoFunPrimer
- **MetaFunPrimer** (Pipeline that embeds EcoFunPrimer to design primers for ranked target genes, expands degenerate outputs to nondegenerate candidates, and performs in silico PCR and minimal-set selection.) — https://github.com/jialiu232/MetaFunPrimer_paper_info
- **Primer3** (Underlying primer design software upon which the PrimerDesign native code (compiled via make in the EcoFunPrimer setup) builds for oligo thermodynamic calculations.) — https://github.com/rdpstaff/EcoFunPrimer

## Examples

```
java -Xmx8g -jar PrimerDesign/dist/PrimerDesign.jar -i amoA_AOB_aligned.fasta --hairMax 35 --homoMax 35 --GCFilterMin 0.45 --GCFilterMax 0.70 --degenMax 6
```

## Evaluation signals

- Every output primer pair satisfies the stability thresholds: hairpin Tm ≤ 35, homodimer Tm ≤ 35, GC content within 0.45–0.70, and per-primer degeneracy within the user cap (6 in the amoA-AOB case study vs. default 8).
- Degenerate outputs expand to the expected number of nondegenerate candidate pairs (181 for amoA-AOB), and the minimal-set step reduces these to the reported 78 pairs.
- In silico PCR against the original user-provided reference gene set passes only for pairs where both forward and reverse primers achieve a 100% match to a sequence.
- The 'screen' subcommand output shows plausible coverage and entropy profiles across alignment positions before enumeration begins.
- The final primer set achieves the expected reference coverage (94% of amoA-AOB genes observed in the 1,550 soil metagenomes, vs. 49% for previously published primers).

## Limitations

- EcoFunPrimer is still in development, per its README.
- Degeneracy is recommended to be no higher than 10 per primer pair (--degenMax default 8); higher degeneracy risks unwieldy assays, and assayMax (default 20) caps the number of degenerate primer pairs per assay.
- Validation is computational only; the authors note that further experimental validation is required before trusting designed primers.
- Highly diverse functional genes may require numerous primer pairs and numerous qPCR runs, potentially exceeding HT-qPCR platform capacity.

## Evidence

- [readme] Thermodynamically stable primer pairs with stability and degeneracy thresholds: [evidence span withheld — anchor status: external_doc]
- [readme] Hairpin and homodimer maximum temperatures default to 35: [evidence span withheld — anchor status: external_doc]
- [readme] GC content filter bounds and degeneracy cap: [evidence span withheld — anchor status: external_doc]
- [results] EcoFunPrimer generates degenerate or nondegenerate primers based on user inputs: "EcoFunPrimer is a primer design tool which is capable of generating degenerate or nondegenerate primers based on input genes and parameters defined by the user."
- [methods] 100% forward and reverse match criterion for in silico PCR success: [evidence span withheld — anchor status: not_found]
