---
name: dsr-sulphur-metabolism-protein-typing
description: Use when you have user-supplied protein sequences in FASTA format (complete
  genomes or metagenomic assemblies) and need to detect Dsr proteins and predict enzyme
  direction/type automatically, without phylogenetic tree reconstruction.
when_to_use_negative:
- You need phylogenetic placement or evolutionary relationships of DsrAB sequences
  rather than automatic type classification — DiSCo replaces, not reproduces, phylogenetic
  reconstruction
- Your marker question concerns non-Dsr metabolic pathways — the tool's HMMs cover
  only Dsr-dependent dissimilatory sulphur metabolism proteins
- You rely on a single marker protein (e.g. DsrD) alone for distinguishing dissimilatory
  sulphate-reducers from other sulphur oxidizers — the paper warns this can lead to
  incorrect assignments
edam_operation: http://edamontology.org/operation_3092
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_0196
tools:
- name: DiSCo 1.0
  role: Standalone Perl 5 tool that runs hmmsearch against the DiSCo HMM library and
    assigns protein family and predicted enzyme type via the filter script's best-hit
    strategy
  repo: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- name: hmmsearch 3.1b2
  role: HMMER3 search of the FASTA protein set against the DiSCo HMM library; thread
    count adjustable with -n
- name: hmmbuild 3.1b2
  role: Rebuilds the DsrABCMK HMM profiles (e.g. for jackknife/leave-one-out robustness
    testing) with default parameters
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_002
  - task_005
  source_papers:
  - doi: 10.1099/mgen.0.000603
    title: 'DiSCo: a sequence-based type-specific predictor of Dsr-dependent dissimilatory
      sulphur metabolism in microbial data'
  source_content_hash: sha256:6ee4cf274faa469a839dba50a65cbe23692f7df353a5c6dd126e06ff39408580
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
  edam_operation: http://edamontology.org/operation_3092
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# dsr-sulphur-metabolism-protein-typing

## Summary

Identify and classify Dsr-dependent dissimilatory sulphur metabolism proteins (DsrA/B/CMK, DsrC, DsrD, DsrEFH, DsrL, DsrO) and predict their enzyme type by running protein sequences in FASTA format against the curated DiSCo HMM library with hmmsearch and a best-hit Perl filter.

## When to use

Use when you have user-supplied protein sequences in FASTA format (complete genomes or metagenomic assemblies) and need to detect Dsr proteins and predict enzyme direction/type automatically, without phylogenetic tree reconstruction.

## When NOT to use

- You need phylogenetic placement or evolutionary relationships of DsrAB sequences rather than automatic type classification — DiSCo replaces, not reproduces, phylogenetic reconstruction
- Your marker question concerns non-Dsr metabolic pathways — the tool's HMMs cover only Dsr-dependent dissimilatory sulphur metabolism proteins
- You rely on a single marker protein (e.g. DsrD) alone for distinguishing dissimilatory sulphate-reducers from other sulphur oxidizers — the paper warns this can lead to incorrect assignments

## Inputs

- Input protein sequence file in FASTA format (-i, e.g. example.faa) from complete genomes or (meta)genomic assemblies
- DiSCo HMM library (bundled, default current directory; alternative path via -m)
- Model-specific cutoffs table (thresholds.txt: column 1 model, column 2 score, column 3 E-value; via -t)

## Outputs

- Filtered DiSCo hitlist (disco_hitlist.tsv) with per-hit sequence identifier, HMM name, predicted enzyme type, score, bias, E-values, accuracy, domain coordinates, and DsrC cysteine-type notes, using the configured separator (TAB or comma)
- Optional FASTA file of DiSCo hit sequences for downstream analyses (-o flag)

## How to apply

Assemble input protein sequences as a FASTA file and run DiSCo.pl (Perl 5), which first performs an hmmsearch (HMMER v3.1b2) of the sequences against the DiSCo HMM library, then applies the bundled Perl filter script. Retain only hits meeting the HMMER-specific cutoffs: conditional and independent E-value ≤10^-10, accuracy ≥0.5, and bias one order of magnitude lower than score. Apply the best-hit strategy: each hit must fulfil its model-specific cutoff (tab-separated thresholds.txt with model, score, E-value columns), then, among all hits to the same input sequence, keep the best-scoring model. Output a filtered hitlist (-p filename prefix) with the chosen separator (-s 0=TAB default, 1=comma, 2=semicolon, 3=space), optionally exporting hit sequences as FASTA with -o; the hitlist contains protein family and predicted type per hit. Scale large scans with multiple hmmsearch threads (-n).

## Related tools

- **DiSCo 1.0** (Standalone Perl 5 tool that runs hmmsearch against the DiSCo HMM library and assigns protein family and predicted enzyme type via the filter script's best-hit strategy) — https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- **hmmsearch 3.1b2** (HMMER3 search of the FASTA protein set against the DiSCo HMM library; thread count adjustable with -n)
- **hmmbuild 3.1b2** (Rebuilds the DsrABCMK HMM profiles (e.g. for jackknife/leave-one-out robustness testing) with default parameters)

## Examples

```
perl DiSCo.pl -i example.faa -p example -s 1 -o
```

## Evaluation signals

- disco_hitlist.tsv exists and is delimited as configured (TAB by default, or comma with -s 1) and contains protein family and predicted type columns per hit
- Every reported hit fulfils the cutoffs: conditional and independent E-value ≤1e-10, accuracy ≥0.5, bias one order of magnitude below score
- Only one (best-scoring, model-cutoff-passing) model is assigned per input sequence
- Benchmark on the 1187-genome independent test set reproduces the reported metrics, e.g. DsrO false discovery rate of 0.06 versus 0.8 (blast) and 0.6 (rBBH), with minimal accuracy 0.98 for complete genomes and 0.93 for metagenomic records
- Jackknife (leave-one-out) rebuilt models change ≤0.9% of assignments (5 of 545) and no hit changes its predicted type

## Limitations

- Some paralogous marker proteins (e.g. DsrEFH, ~100 amino acids) are small, complicating careful distinction between paralogous families
- Presence of a reductive-type DsrAB complex alone is not a good proxy for use of oxidized sulphur compounds as electron acceptors, and some organisms with reductive-type Dsr-systems are actually dSDM
- Using DsrD as a single marker may lead to incorrect assignments since it is not essential for sulphate reduction
- DiSCo depends on HMMER3 (hmmsearch must be in $PATH); per-hit assignments depend on user-defined cutoffs in thresholds.txt, and any specificity assumes those cutoffs are maintained

## Evidence

- [intro] DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5: "DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5"
- [intro] This filter step assigns DiSCo hits based on the best hit strategy: "This filter step assigns DiSCo hits based on the best hit strategy: (i) a hit needs to fulfil the model specific cutoff; (ii) all hits to the same input sequence are compared"
- [intro] The first step of DiSCo.pl consists of an hmmsearch: [evidence span withheld — anchor status: not_found]
- [readme] This tool runs a set of FASTA formatted protein sequences against the curated set of hidden Markov models: "This tool runs a set of FASTA formatted protein sequences against the curated set of hidden Markov models (HMMs)."
- [intro] HMM were created using hmmbuild: "HMM were created using hmmbuild [74] (v. 3.1b2) with default parameters"
