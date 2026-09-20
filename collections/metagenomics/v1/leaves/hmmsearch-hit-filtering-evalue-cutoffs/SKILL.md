---
name: hmmsearch-hit-filtering-evalue-cutoffs
description: Use when assigning HMM library hits to input protein sequences after
  hmmsearch and you must apply model-specific score/E-value cutoffs and keep only
  the -scoring model per sequence.
when_to_use_negative:
- Input is raw nucleotide or non-protein data — DiSCo expects a user-supplied protein
  sequence set in FASTA format
- Hits were produced by a similarity tool other than hmmsearch against the DiSCo HMM
  library (e.g. blastp results), since the HMMER-specific conditional/independent
  E-value and accuracy fields will not exist
- The goal is de novo functional annotation of proteins outside Dsr-dependent dissimilatory
  sulphur metabolism — the HMM library and cutoffs are specific to Dsr pathway families
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0623
- http://edamontology.org/topic_3299
tools:
- name: DiSCo
  role: Standalone Perl pipeline whose filter step implements the HMMER cutoff filtering
    and best-hit model assignment
  repo: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- name: hmmsearch 3.1b2
  role: Produces the HMMER hit output (score, bias, E-value, conditional/independent
    E-value, accuracy) that the filter parses
  repo: http://hmmer.org/
- name: Perl 5
  role: Language of DiSCo.pl and the filter script (-f option allows an alternative
    filter script location)
  repo: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1099/mgen.0.000603
    title: 'DiSCo: a sequence-based type-specific predictor of Dsr-dependent dissimilatory
      sulphur metabolism in microbial data'
  source_content_hash: sha256:d6aa4232411ac8479df625b137886043a4894a60a30ed10658b6f0ed97449d63
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
  tools:
  - DiSCo
  - Perl 5
  - hmmsearch 3.1b2
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0623
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# hmmsearch hit filtering with E-value cutoffs and best-hit model assignment

## Summary

A filter step from the DiSCo pipeline that parses hmmsearch (HMMER v3.1b2) output and assigns DiSCo hits to input protein sequences using HMMER-specific E-value/accuracy/bias cutoffs plus a per-model cutoff table and a best-hit strategy.

## When to use

Use when assigning HMM library hits to input protein sequences after hmmsearch and you must apply model-specific score/E-value cutoffs and keep only the best-scoring model per sequence.

## When NOT to use

- Input is raw nucleotide or non-protein data — DiSCo expects a user-supplied protein sequence set in FASTA format
- Hits were produced by a similarity tool other than hmmsearch against the DiSCo HMM library (e.g. blastp results), since the HMMER-specific conditional/independent E-value and accuracy fields will not exist
- The goal is de novo functional annotation of proteins outside Dsr-dependent dissimilatory sulphur metabolism — the HMM library and cutoffs are specific to Dsr pathway families

## Inputs

- hmmsearch output (HMMER v3.1b2) from searching a FASTA protein sequence file against the DiSCo HMM library
- Tab-separated model cutoff table (columns: model, score, E-value), e.g. thresholds.txt via -t

## Outputs

- Filtered DiSCo hitlist (filename prefix via -p, separator via -s; default TAB) listing sequence identifier, HMM name, predicted enzyme type, scores, E-values, accuracy and domain coordinates
- Optional FASTA outfile of extracted hit sequences via -o

## How to apply

Run hmmsearch (HMMER v3.1b2, default parameters) on a FASTA protein sequence set against the curated DiSCo HMM library, then parse the output with the Perl filter script. Retain only hits fulfilling the HMMER-specific conditional and independent E-value cutoff of <=10^-10 and accuracy >=0.5, and discard hits whose bias is not one order of magnitude lower than their score. Next require each hit to pass its model-specific cutoff from a tab-separated table (column 1: model, column 2: score, column 3: E-value; per Table S6, customizable via -t thresholds.txt), and apply the best-hit strategy: compare all hits to the same input sequence and assign the best-scoring model. Write the filtered hitlist with the -p filename prefix and chosen separator (-s; default TAB), optionally extracting hit sequences to FASTA with -o. The rationale is to prevent false positives while disambiguating paralogous protein families via model-specific thresholds rather than a single global cutoff.

## Related tools

- **DiSCo** (Standalone Perl pipeline whose filter step implements the HMMER cutoff filtering and best-hit model assignment) — https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- **hmmsearch 3.1b2** (Produces the HMMER hit output (score, bias, E-value, conditional/independent E-value, accuracy) that the filter parses) — http://hmmer.org/
- **Perl 5** (Language of DiSCo.pl and the filter script (-f option allows an alternative filter script location)) — https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo

## Examples

```
perl DiSCo.pl -i example.faa -p example -s 1 -o
```

## Evaluation signals

- Every retained hit has conditional and independent E-value <=10^-10 and accuracy >=0.5
- Every retained hit's bias is one order of magnitude lower than its score
- Each assigned hit passes the model-specific score and E-value cutoff from the thresholds table (model, score, E-value columns)
- At most one model is assigned per input sequence (best-hit invariant)
- Output hitlist matches the -p prefix and -s separator (default TAB), and columns follow the documented 20-column output format

## Limitations

- Cutoffs are model-specific and were optimized to retrieve known Dsr pathway lineages; thresholds may exclude divergent or novel families not represented in the training set
- The best-hit strategy assigns only the single best-scoring model per input sequence, potentially discarding secondary domain hits or multi-family membership
- The filter is HMMER3-specific — it relies on hmmsearch report fields (bias, conditional/independent E-value, accuracy) that other search tools do not emit

## Evidence

- [intro] HMMER-specific cutoffs: "DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5"
- [intro] best-hit strategy: "This filter step assigns DiSCo hits based on the best hit strategy: (i) a hit needs to fulfil the model specific cutoff; (ii) all hits to the same input sequence are compared"
- [intro] hmmsearch first step: [evidence span withheld — anchor status: not_found]
- [methods] cutoff table format: "#tab seperated table with user-defined cut-offs: column 1: model, column 2: score, column 3: E-value"
- [readme] output columns: "Conditional *E*-value
   - Independent *E*-value
   - Domain score
   - Domain bias"
