---
name: hmm-based-protein-classification
description: Use when input is a protein sequence set in FASTA format and the goal
  is automatic detection and enzyme-type prediction of proteins belonging to known
  paralogous families, where blast-based or phylogenetic assignment is too slow or
  error-prone.
when_to_use_negative:
- Input is nucleotide sequence (e.g. raw reads or contigs) rather than a protein FASTA
  set — DiSCo requires user-supplied protein sequences
- The goal is de novo phylogenetic reconstruction of DsrAB rather than automatic family/type
  assignment from sequence data
- Proteins of interest fall outside the Dsr-dependent dissimilatory sulphur metabolism
  families covered by the DiSCo HMM library
edam_operation: http://edamontology.org/operation_0414
edam_topics:
- http://edamontology.org/topic_0623
- http://edamontology.org/topic_0080
tools:
- name: DiSCo
  role: Perl 5 driver that runs hmmsearch against the curated HMM library and filters/assigns
    hits via the best-hit strategy
  repo: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- name: hmmsearch 3.1b2
  role: HMMER3 profile search engine executed by DiSCo.pl against the HMM library;
    must be in $PATH
- name: hmmbuild 3.1b2
  role: Rebuilds HMM profiles (e.g. for jackknife leave-one-out stability testing)
    with default parameters
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_004
  - task_001
  - task_003
  - task_002
  source_papers:
  - doi: 10.1099/mgen.0.000603
    title: 'DiSCo: a sequence-based type-specific predictor of Dsr-dependent dissimilatory
      sulphur metabolism in microbial data'
  source_content_hash: sha256:fb558b815643753146e35013c56fb5102a334efede3a1ce64def660b92fb8739
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
  tools:
  - DiSCo
  - DiSCo 1.0
  - blastp
  - hmmbuild 3.1b2
  - hmmsearch 3.1b2
  edam_operation: http://edamontology.org/operation_0414
  edam_topics:
  - http://edamontology.org/topic_0623
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# hmm-based-protein-classification

## Summary

Identify and type-classify proteins from FASTA-formatted protein data by searching them against a curated profile HMM library with HMMER3 hmmsearch, then filtering and assigning hits via a best-hit strategy. This is the core mechanism of DiSCo, a Perl 5 tool for Dsr-dependent dissimilatory sulphur metabolism protein classification.

## When to use

Use when input is a protein sequence set in FASTA format and the goal is automatic detection and enzyme-type prediction of proteins belonging to known paralogous families, where blast-based or phylogenetic assignment is too slow or error-prone.

## When NOT to use

- Input is nucleotide sequence (e.g. raw reads or contigs) rather than a protein FASTA set — DiSCo requires user-supplied protein sequences
- The goal is de novo phylogenetic reconstruction of DsrAB rather than automatic family/type assignment from sequence data
- Proteins of interest fall outside the Dsr-dependent dissimilatory sulphur metabolism families covered by the DiSCo HMM library

## Inputs

- Protein sequences in FASTA format (e.g. genome proteomes or metagenomic predicted proteins)
- DiSCo HMM library (HMMER3 profiles, default location ./)
- Perl 5 filter script (default location ./, overridable via -f)
- Tab-separated thresholds.txt with user-defined per-model cut-offs (model, score, E-value)

## Outputs

- Filtered DiSCo hitlist (tab-separated by default; columns include sequence identifier, HMM name, predicted enzyme type, score, bias, E-value, accuracy, domain coordinates, input sequence annotation, DsrC-type notes)
- Optional FASTA file of hit sequences (via -o flag)

## How to apply

Run DiSCo.pl with -i pointing to the input protein FASTA; its first step is an hmmsearch (HMMER v3.1b2, default parameters) of the sequence set against the DiSCo HMM library. Apply the filter script's criteria: hits must fulfil the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5, and a bias one order of magnitude lower than their score. Then apply the best-hit assignment strategy: each hit must pass its model-specific cutoff from the tab-separated thresholds table (column 1: model, column 2: score, column 3: E-value), and all hits to the same input sequence are compared so only the best-scoring model is assigned. Write the filtered hitlist with the -p prefix using the chosen separator (default TAB; -s 1 for comma), and optionally export hit sequences in FASTA format with the -o flag. Use -n to allocate multiple hmmsearch threads when scanning large (meta)genomic sets, and -m/-f/-t to override the HMM library, filter script, or user-defined cutoffs.

## Related tools

- **DiSCo** (Perl 5 driver that runs hmmsearch against the curated HMM library and filters/assigns hits via the best-hit strategy) — https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- **hmmsearch 3.1b2** (HMMER3 profile search engine executed by DiSCo.pl against the HMM library; must be in $PATH)
- **hmmbuild 3.1b2** (Rebuilds HMM profiles (e.g. for jackknife leave-one-out stability testing) with default parameters)

## Examples

```
perl DiSCo.pl -i example.faa -p example -s 1 -o
```

## Evaluation signals

- Every reported hit fulfils the cutoffs: conditional and independent E-value ≤10−10, accuracy ≥0.5, and bias one order of magnitude below score
- Each input sequence receives at most one assigned model under the best-hit strategy, and that hit passes its model-specific score/E-value cutoff from thresholds.txt
- Hitlist is delimited as configured (TAB default, or comma with -s 1) and contains the expected columns per hit, including protein family and predicted enzyme type
- On the complete genomes benchmark, assignments reach the reported minimal accuracy of 0.98 compared against curated reference labels
- Leave-one-out jackknife rebuilds of the HMMs should reproduce results with only ~0.9% (5 of 545) assignments lost and no type-prediction changes

## Limitations

- Dependent on HMMER3: the hmmsearch executable must be present in the user's $PATH
- Classification depends on user-defined per-model cut-offs (thresholds.txt); rebuilt or altered HMM libraries require keeping the original model-specific score/E-value thresholds for comparability
- Distinction between paralogous families is hampered by the small size (~100 amino acids) of some marker proteins (e.g. DsrEFH), so assignments rest on HMM discrimination rather than phylogeny

## Evidence

- [intro] DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5: "DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5"
- [intro] The first step of DiSCo.pl consists of an hmmsearch of a given protein sequences set in FASTA format against the DiSCo HMM library: [evidence span withheld — anchor status: not_found]
- [methods] #tab seperated table with user-defined cut-offs: column 1: model, column 2: score, column 3: E-value: "#tab seperated table with user-defined cut-offs: column 1: model, column 2: score, column 3: E-value"
- [readme] This tool runs a set of FASTA formatted protein sequences against the curated set of hidden Markov models (HMMs).: "This tool runs a set of FASTA formatted protein sequences against the curated set of hidden Markov models (HMMs)."
