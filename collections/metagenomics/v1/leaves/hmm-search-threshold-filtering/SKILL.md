---
name: hmm-search-threshold-filtering
description: Use when you need to identify and type-classify Dsr-dependent dissimilatory
  sulphur metabolism proteins from FASTA protein data using profile HMMs with model-specific
  score and E-value cut-offs rather than phylogenetic assignment.
when_to_use_negative:
- Input is raw nucleotide data — DiSCo.pl requires FASTA protein sequences and does
  not translate.
- You need de novo phylogenetic placement of DsrAB sequences rather than fast HMM-based
  family/type assignment — the article contrasts DiSCo with time-consuming phylogenetic
  classification.
- Proteins fall outside the Dsr-dependent dissimilatory sulphur metabolism families
  covered by the DiSCo HMM library (e.g. unrelated paralogous markers or non-Dsr pathways).
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3299
tools:
- name: DiSCo 1.0
  role: Standalone Perl 5 genome-mining tool that orchestrates hmmsearch, applies
    the filter script's best-hit strategy and thresholds.txt cut-offs, and outputs
    the classified hitlist
  repo: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- name: hmmsearch 3.1b2
  role: HMMER3 search of the input protein FASTA set against the DiSCo HMM library
    with default parameters
- name: hmmbuild 3.1b2
  role: Rebuilds HMM profiles (default parameters) when retraining models, e.g. for
    jackknife leave-one-out resampling, with thresholds.txt cut-offs kept unchanged
- name: Perl 5
  role: Implementation language of DiSCo.pl and the filter script; requires hmmsearch
    in $PATH
  repo: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1099/mgen.0.000603
    title: 'DiSCo: a sequence-based type-specific predictor of Dsr-dependent dissimilatory
      sulphur metabolism in microbial data'
  source_content_hash: sha256:0e27afe0df296cc4f83524b97dc0c32259739c84cfbbc17508102b6d5313f3f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
  tools:
  - DiSCo
  - DiSCo 1.0
  - Perl 5
  - hmmbuild 3.1b2
  - hmmsearch 3.1b2
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# hmm-search-threshold-filtering

## Summary

DiSCo's two-stage identification filter: hmmsearch of FASTA protein input against the curated DiSCo HMM library (HMMER 3.1b2), followed by a Perl filter script that applies model-specific user-defined score/E-value cut-offs (thresholds.txt) and a best-hit strategy to assign each input sequence to one protein family and predicted enzyme type.

## When to use

Use when you need to identify and type-classify Dsr-dependent dissimilatory sulphur metabolism proteins from FASTA protein data using profile HMMs with model-specific score and E-value cut-offs rather than phylogenetic assignment.

## When NOT to use

- Input is raw nucleotide data — DiSCo.pl requires FASTA protein sequences and does not translate.
- You need de novo phylogenetic placement of DsrAB sequences rather than fast HMM-based family/type assignment — the article contrasts DiSCo with time-consuming phylogenetic classification.
- Proteins fall outside the Dsr-dependent dissimilatory sulphur metabolism families covered by the DiSCo HMM library (e.g. unrelated paralogous markers or non-Dsr pathways).

## Inputs

- Protein sequences in FASTA format (e.g. complete genomes protein sequences, -i flag)
- DiSCo HMM library (default: current directory; override with -m)
- thresholds.txt: tab-separated user-defined cut-offs (model, score, E-value; override with -t)
- Filter Perl script (default: current directory; override with -f)

## Outputs

- Filtered DiSCo hitlist, tab-separated by default (-s 0), with columns: sequence identifier, HMM name, predicted enzyme type, sequence/HMM lengths, score, bias, E-value, accuracy, alignment length, conditional/independent E-value, domain score/bias, domain coordinates, input annotation, DsrC-type note
- Optional FASTA file of hit sequences (-o flag)

## How to apply

Run DiSCo.pl with the input FASTA (-i); the first step executes hmmsearch (HMMER v. 3.1b2, default parameters, 1 thread by default via -n) against the DiSCo HMM library (override with -m). Hits must fulfil the hmmer-specific conditional and independent E-value ≤10−10, accuracy ≥0.5, and bias one order of magnitude lower than score. The filter script (-f) then applies model-specific user-defined cut-offs from the tab-separated thresholds.txt (-t: column 1 model, column 2 score, column 3 E-value) and compares all hits to the same input sequence under a best-hit strategy before assigning protein family and predicted enzyme type. Output is a tab-separated hitlist (default separator, changeable with -s) plus optional FASTA export of hit sequences with -o. Leave the thresholds.txt cut-offs unchanged when rebuilding HMMs (e.g. for jackknife leave-one-out hmmbuild resampling) so results stay comparable to the reference run.

## Related tools

- **DiSCo 1.0** (Standalone Perl 5 genome-mining tool that orchestrates hmmsearch, applies the filter script's best-hit strategy and thresholds.txt cut-offs, and outputs the classified hitlist) — https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- **hmmsearch 3.1b2** (HMMER3 search of the input protein FASTA set against the DiSCo HMM library with default parameters)
- **hmmbuild 3.1b2** (Rebuilds HMM profiles (default parameters) when retraining models, e.g. for jackknife leave-one-out resampling, with thresholds.txt cut-offs kept unchanged)
- **Perl 5** (Implementation language of DiSCo.pl and the filter script; requires hmmsearch in $PATH) — https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo

## Examples

```
perl DiSCo.pl -i genomes.faa -p disco_hits -t thresholds.txt -s 0 -n 1 -o
```

## Evaluation signals

- All reported hits satisfy E-value ≤10−10 (conditional and independent), accuracy ≥0.5, and bias one order of magnitude lower than score.
- Each retained hit also fulfils its model-specific cutoff from thresholds.txt (score and E-value columns).
- Only one best hit per input sequence is reported — all hits to the same input sequence are compared under the best-hit strategy.
- Output hitlist is tab-separated (default) with the 20 documented columns including HMM name, predicted enzyme type, and DsrC-type note.
- When applied with rebuilt (jackknife) HMMs and unchanged cut-offs, results should reproduce the reference: 0.9% assignment loss (5 of 545) and no type-prediction changes (Table S8).

## Limitations

- Requires HMMER3's hmmsearch available in $PATH; DiSCo defaults to a single hmmsearch thread (-n 1).
- HMM-based assignment depends on optimized model thresholds; using a marker like DsrD as a single marker, or judging reductive-type DsrAB presence alone, can lead to incorrect assignments.
- No explicit software version number is stated for DiSCo in the article's discussion, and supplementary content (e.g. Table S6/S8) must be retrieved separately.

## Evidence

- [intro] The first step of DiSCo.pl consists of an hmmsearch of a given protein sequences set in FASTA format against the DiSCo HMM library: [evidence span withheld — anchor status: not_found]
- [intro] DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5: "DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5"
- [readme] tab seperated table with user-defined cut-offs: column 1: model, column 2: score, column 3: E-value: "-t </path/thresholds.txt>	#tab seperated table with user-defined cut-offs: column 1: model, column 2: score, column 3: E-value"
- [readme] All programs are written in perl (version 5) and dependent on HMMER3. The script requires hmmsearch to be in your $PATH.: "All programs are written in perl (version 5) and dependent on HMMER3. 
The script requires hmmsearch to be in your $PATH."
- [intro] HMM were created using hmmbuild [74] (v. 3.1b2) with default parameters: "HMM were created using hmmbuild [74] (v. 3.1b2) with default parameters"
