---
name: best-hit-model-assignment-per-sequence
description: Use when post-processing hmmsearch output against a profile HMM library
  where one input sequence may match several models and a single, -supported profile
  assignment per sequence is required.
when_to_use_negative:
- Hits have already been filtered and reduced to one assignment per input sequence
  — re-applying the best-hit step would be redundant
- The analysis goal is to survey all paralogous families or multiple model matches
  per sequence rather than a single best type assignment per input protein
- Input is nucleotide or raw (meta)genomic data rather than protein sequences in FASTA
  format
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0623
- http://edamontology.org/topic_0080
tools:
- name: DiSCo
  role: Standalone Perl tool whose filter script implements the HMMER cutoff filtering
    and best-hit model assignment per input sequence
  repo: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- name: hmmsearch 3.1b2
  role: Produces the raw per-sequence HMM hits (scores, bias, E-values, accuracy)
    that the filter script parses and assigns
- name: Perl 5
  role: Implementation language of DiSCo.pl and the filter script performing the best-hit
    assignment
  repo: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1099/mgen.0.000603
    title: 'DiSCo: a sequence-based type-specific predictor of Dsr-dependent dissimilatory
      sulphur metabolism in microbial data'
  source_content_hash: sha256:e0c2c265ed80b7b3657560a8b5b960d54a603254892331f01f93b7d3bd122c39
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
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# best-hit-model-assignment-per-sequence

## Summary

Assigns DiSCo HMM hits to input protein sequences using a best-hit strategy: each hit must first pass its model-specific cutoff, then all hits to the same input sequence are compared and only the best-scoring model is retained.

## When to use

Use when post-processing hmmsearch output against a profile HMM library where one input sequence may match several models and a single, best-supported profile assignment per sequence is required.

## When NOT to use

- Hits have already been filtered and reduced to one assignment per input sequence — re-applying the best-hit step would be redundant
- The analysis goal is to survey all paralogous families or multiple model matches per sequence rather than a single best type assignment per input protein
- Input is nucleotide or raw (meta)genomic data rather than protein sequences in FASTA format

## Inputs

- hmmsearch (HMMER v3.1b2) output from a FASTA protein sequence file searched against the DiSCo HMM library
- Tab-separated table of model-specific cutoffs: column 1 = model, column 2 = score, column 3 = E-value (default per Table S6; user-defined via -t)

## Outputs

- Filtered DiSCo hitlist (filename prefix via -p; TAB, comma, semicolon, or space separated per -s) listing sequence identifier, HMM name, predicted enzyme type, scores, E-values, accuracy, domain coordinates, and annotation
- Optional FASTA file of hit sequences via the -o flag

## How to apply

After running hmmsearch (HMMER v3.1b2, default parameters) on a FASTA protein file against the DiSCo HMM library, parse the output with the Perl filter script. First retain only hits fulfilling the HMMER-specific conditional and independent E-value cutoff of ≤10^-10, an accuracy of ≥0.5, and discard hits whose bias is not one order of magnitude lower than their score. Then apply the best-hit strategy: require each hit to pass its model-specific cutoff (a tab-separated table of model, score, E-value; per Table S6, overridable via the -t flag), compare all hits to the same input sequence, and keep only the best-scoring model. Write the filtered hitlist with the -p filename prefix using the chosen separator (default TAB), optionally extracting hit sequences to FASTA with -o. This prevents ambiguous or multi-model assignments for a single sequence.

## Related tools

- **DiSCo** (Standalone Perl tool whose filter script implements the HMMER cutoff filtering and best-hit model assignment per input sequence) — https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- **hmmsearch 3.1b2** (Produces the raw per-sequence HMM hits (scores, bias, E-values, accuracy) that the filter script parses and assigns)
- **Perl 5** (Implementation language of DiSCo.pl and the filter script performing the best-hit assignment) — https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo

## Examples

```
perl DiSCo.pl -i example.faa -p example -s 1 -o
```

## Evaluation signals

- Every reported hit passes both the global HMMER filters (conditional and independent E-value ≤10^-10, accuracy ≥0.5, bias one order of magnitude lower than score) and its model-specific score/E-value cutoff
- Each input sequence identifier appears at most once in the filtered hitlist, with the highest-scoring passing model
- No hit is retained whose bias is within one order of magnitude of its score
- Output columns match the documented schema (sequence identifier, HMM name, predicted enzyme type, score, bias, E-value, accuracy, conditional/independent E-value, domain coordinates, annotation, DsrC-type note)
- Benchmarked against genomic records, assignments achieve minimal accuracy of 0.98 for complete genomes and 0.93 for metagenomic records

## Limitations

- Using DsrD as a single marker, or relying on phylogenies alone, can lead to incorrect assignments (e.g. reductive-type Dsr-system organisms that are actually dSDM); the best-hit HMM strategy mitigates but does not remove these biological ambiguities
- Distinction between paralogous families is hampered by the small size (~100 amino acids) of some marker proteins such as DsrEFH
- User-defined cutoffs supplied via -t replace the optimized default thresholds (Table S6) and must be provided as a tab-separated model/score/E-value table

## Evidence

- [intro] This filter step assigns DiSCo hits based on the best hit strategy: (i) a hit needs to fulfil the model specific cutoff; (ii) all hits to the same input sequence are compared: "This filter step assigns DiSCo hits based on the best hit strategy: (i) a hit needs to fulfil the model specific cutoff; (ii) all hits to the same input sequence are compared"
- [intro] DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5: "DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5"
- [readme] -t </path/thresholds.txt>	#tab seperated table with user-defined cut-offs: column 1: model, column 2: score, column 3: E-value: "-t </path/thresholds.txt>	#tab seperated table with user-defined cut-offs: column 1: model, column 2: score, column 3: E-value"
- [intro] The first step of DiSCo.pl consists of an hmmsearch of a given protein sequences set in FASTA format against the DiSCo HMM library: [evidence span withheld — anchor status: not_found]
