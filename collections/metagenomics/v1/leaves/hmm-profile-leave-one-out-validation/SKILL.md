---
name: hmm-profile-leave-one-out-validation
description: Use when a curated HMM-based classifier (e.g. the DsrABCMK model set
  behind DiSCo) needs its stability tested against training-set composition, i.e.
  checking whether model assignments or type predictions change when single training
  sequences are dropped.
when_to_use_negative:
- When the goal is to re-tune or re-optimize model thresholds — this skill deliberately
  keeps the original user-defined cut-offs fixed to isolate training-set effects
- When input is not protein sequence data in FASTA format; DiSCo runs protein sets
  against its curated HMM library, not nucleotide data
- When testing robustness to algorithm parameters or cutoff choices rather than to
  training-set composition — a different sensitivity design is needed
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: DiSCo
  role: 'Pipeline being stress-tested: DiSCo.pl runs the rebuilt leave-one-out HMM
    library against the protein FASTA input via hmmsearch and filters hits with the
    best-hit strategy'
  repo: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- name: hmmbuild
  role: Rebuilds each HMM profile (v. 3.1b2, default parameters) from the DsrABCMK
    training set minus one sequence per jackknife iteration
  repo: http://hmmer.org/
- name: hmmsearch
  role: HMMER v. 3.1b2 search of the protein sequence set against each rebuilt HMM
    library with default parameters and 1 thread
  repo: http://hmmer.org/
- name: Perl 5
  role: Language in which DiSCo.pl and the filter script are implemented
  repo: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1099/mgen.0.000603
    title: 'DiSCo: a sequence-based type-specific predictor of Dsr-dependent dissimilatory
      sulphur metabolism in microbial data'
  source_content_hash: sha256:3c190e97b61a2d4925aac5181d825e609edc07922b8457fe22840331e6664066
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
  tools:
  - DiSCo
  - Perl 5
  - hmmbuild
  - hmmbuild 3.1b2
  - hmmsearch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# HMM profile leave-one-out (jackknife) validation

## Summary

Robustness assessment for curated HMM model sets: each training sequence is excluded one at a time, the HMM profile is rebuilt with hmmbuild, and the original user-defined score/E-value cut-offs are kept fixed so downstream assignments can be compared against the full-model reference hitlist.

## When to use

Use when a curated HMM-based classifier (e.g. the DsrABCMK model set behind DiSCo) needs its stability tested against training-set composition, i.e. checking whether model assignments or type predictions change when single training sequences are dropped.

## When NOT to use

- When the goal is to re-tune or re-optimize model thresholds — this skill deliberately keeps the original user-defined cut-offs fixed to isolate training-set effects
- When input is not protein sequence data in FASTA format; DiSCo runs protein sets against its curated HMM library, not nucleotide data
- When testing robustness to algorithm parameters or cutoff choices rather than to training-set composition — a different sensitivity design is needed

## Inputs

- List of training sequences per model (e.g. Table S6 of the DiSCo supplementary material)
- Complete genomes protein sequences in FASTA format
- DiSCo HMM library (DsrABCMK model set) and filter script (script.pl)
- thresholds.txt: tab-separated user-defined cut-offs (column 1: model, column 2: score, column 3: E-value)

## Outputs

- Per-exclusion filtered DiSCo hitlists (tab-separated, default separator) listing sequence identifier, HMM name, predicted enzyme type, score, bias, E-value, accuracy and domain details
- Optional FASTA file of hit sequences (via the -o flag)
- Validation summary: number/proportion of assignments lost relative to the full model and any type-prediction changes

## How to apply

Read the list of training sequences (e.g. Table S6 for DsrABCMK) and the complete-genomes protein set in FASTA format. For each model in the set, exclude one training sequence at a time and rebuild the HMM profile with hmmbuild (v. 3.1b2, default parameters); crucially, do NOT re-estimate thresholds — reuse the model-specific score and E-value columns of the thresholds.txt file unchanged. Run DiSCo.pl with the rebuilt HMM library (passed via -m) against the complete genomes dataset using hmmsearch (hmmer v. 3.1b2, default parameters, 1 thread) and the filter script's best-hit strategy, keeping the same conditional/independent E-value ≤10−10 and accuracy ≥0.5 hit filters. Collect the filtered tab-separated hitlist (optionally exporting hit sequences in FASTA with -o) and compare leave-one-out assignments to the full-model reference by count of lost assignments and by whether any hit changed its predicted enzyme type.

## Related tools

- **DiSCo** (Pipeline being stress-tested: DiSCo.pl runs the rebuilt leave-one-out HMM library against the protein FASTA input via hmmsearch and filters hits with the best-hit strategy) — https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- **hmmbuild** (Rebuilds each HMM profile (v. 3.1b2, default parameters) from the DsrABCMK training set minus one sequence per jackknife iteration) — http://hmmer.org/
- **hmmsearch** (HMMER v. 3.1b2 search of the protein sequence set against each rebuilt HMM library with default parameters and 1 thread) — http://hmmer.org/
- **Perl 5** (Language in which DiSCo.pl and the filter script are implemented) — https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo

## Examples

```
hmmbuild model.jackknife.hmm train_minus_one.sto && perl DiSCo.pl -i genomes.faa -m model.jackknife.hmm -t thresholds.txt -s 0 -o
```

## Evaluation signals

- Reference outcome for the DiSCo DsrABCMK models: a loss rate of 0.9% (five of 545 assignments lost) and zero type-prediction changes across all leave-one-out runs (Table S8)
- Thresholds in thresholds.txt (model-specific score and E-value columns) are byte-identical between full-model and leave-one-out runs
- Each rebuilt profile must be traceable to exactly N−1 training sequences for a model with N training sequences
- Filtered hits still satisfy hmmer-specific conditional and independent E-value ≤10−10, accuracy ≥0.5, and bias one order of magnitude lower than score
- Leave-one-out hitlists are directly diffable against the full-model reference hitlist because separator (-s 0, TAB) and column schema are kept at defaults

## Limitations

- Jackknife results hold only when cut-offs are frozen; if thresholds are re-optimized after each exclusion, results are no longer comparable to the 0.9% / no-type-change reference
- Lost assignments in individual jackknife runs reflect borderline hits near the model cut-off, so the loss rate is a lower-bound sensitivity measure of training-set dependence
- DiSCo requires hmmsearch to be in $PATH and depends on HMMER3; threads default to 1, so large genome sets take substantial wall time on a personal computer

## Evidence

- [supplementary] Jackknife leave-one-out procedure with hmmbuild: [evidence span withheld — anchor status: not_found]
- [supplementary] Keep user-defined cut-offs unchanged: [evidence span withheld — anchor status: not_found]
- [supplementary] Reference validation result for DsrABCMK: [evidence span withheld — anchor status: not_found]
- [intro] HMM profiles built with hmmbuild default parameters: "HMM were created using hmmbuild [74] (v. 3.1b2) with default parameters"
- [intro] DiSCo hit filtering thresholds: "DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5"
- [readme] README: tool implemented in Perl and dependent on HMMER3: "All programs are written in perl (version 5) and dependent on HMMER3. The script requires hmmsearch to be in your $PATH."
- [readme] README: thresholds.txt format and -m/-t flags: "-t </path/thresholds.txt>	#tab seperated table with user-defined cut-offs: column 1: model, column 2: score, column 3: E-value"
