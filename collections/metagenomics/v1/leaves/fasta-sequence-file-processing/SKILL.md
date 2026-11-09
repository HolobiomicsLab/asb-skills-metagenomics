---
name: fasta-sequence-file-processing
description: Use when the input is a user-supplied protein multi-FASTA file (e.g.
  .faa from complete genomes or metagenomes) that must be screened against a curated
  HMM library and assigned protein-family/type predictions.
when_to_use_negative:
- Input is nucleotide sequence or non-FASTA formatted records — DiSCo requires FASTA
  protein sequences and will not translate or reformat them
- Input is already a filtered/annotated feature table rather than raw protein sequences
  — DiSCo starts from hmmsearch over the FASTA set, so pre-filtered tables bypass
  its model-specific cutoffs
- The goal is de novo functional annotation outside the Dsr-dependent dissimilatory
  sulphur pathway — the HMM library is curated specifically for Dsr proteins
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0621
tools:
- name: DiSCo
  role: Perl 5 script that takes the FASTA protein input (-i), runs hmmsearch against
    the HMM library, filters hits, and emits the hitlist and optional FASTA sequence
    output (-o)
  repo: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- name: hmmsearch 3.1b2
  role: HMMER3 search of the FASTA protein set against the DiSCo HMM library with
    default parameters
  repo: http://hmmer.org/
- name: Perl 5
  role: Language in which DiSCo.pl and the filter script are implemented; the filter
    script parses the hmmsearch output
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1099/mgen.0.000603
    title: 'DiSCo: a sequence-based type-specific predictor of Dsr-dependent dissimilatory
      sulphur metabolism in microbial data'
  source_content_hash: sha256:2f3c4d560af0a4e35a432d4954a1e8d70fa55769d9ab7e3abeadd971dc4803e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
  tools:
  - DiSCo
  - Perl 5
  - hmmbuild 3.1b2
  - hmmsearch 3.1b2
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# fasta-sequence-file-processing

## Summary

Prepare and feed FASTA-formatted protein sequence files into the DiSCo pipeline (DiSCo.pl) so that hmmsearch (HMMER 3.1b2) can identify Dsr-dependent dissimilatory sulphur metabolism proteins and classify their predicted enzyme type, with optional FASTA extraction of hits.

## When to use

Use when the input is a user-supplied protein multi-FASTA file (e.g. .faa from complete genomes or metagenomes) that must be screened against a curated HMM library and assigned protein-family/type predictions.

## When NOT to use

- Input is nucleotide sequence or non-FASTA formatted records — DiSCo requires FASTA protein sequences and will not translate or reformat them
- Input is already a filtered/annotated feature table rather than raw protein sequences — DiSCo starts from hmmsearch over the FASTA set, so pre-filtered tables bypass its model-specific cutoffs
- The goal is de novo functional annotation outside the Dsr-dependent dissimilatory sulphur pathway — the HMM library is curated specifically for Dsr proteins

## Inputs

- Protein sequence file in FASTA format (user-supplied, e.g. example.faa)
- Complete genomes protein sequences in FASTA format (for genome-scale runs / jackknife)
- DiSCo HMM library (curated hidden Markov models)
- thresholds.txt (tab-separated table: model, score, E-value)

## Outputs

- Filtered DiSCo hitlist (tab-separated by default; columns include Sequence identifier, HMM name, Predicted enzyme type, Score, Bias, E-value, Accuracy, domain coordinates, annotation, DsrC-type notes)
- FASTA file of hit sequences (when the -o flag is used)
- Per-sequence classification into protein family and predicted type

## How to apply

Supply the protein sequences as a FASTA file to DiSCo.pl with the -i flag (e.g. `perl DiSCo.pl -i example.faa`), which first runs hmmsearch (HMMER v3.1b2, default parameters, 1 thread by default via -n) of the FASTA set against the DiSCo HMM library. Hits passing the HMMER-specific conditional and independent E-value cutoff of <=10^-10 and accuracy >=0.5, with bias one order of magnitude lower than score, are filtered by the Perl 5 filter script and assigned via the best-hit strategy (each hit must also fulfil its model-specific score/E-value from thresholds.txt). Write the filtered hitlist with the -p prefix (default TAB separator, alternatives via -s 1/2/3) and optionally extract hit sequences back into a FASTA outfile with the -o flag. The same FASTA handling applies in jackknife validation: leave-one-out rebuilt HMMs are searched against the complete-genomes FASTA set and compared against the full-model reference hitlist.

## Related tools

- **DiSCo** (Perl 5 script that takes the FASTA protein input (-i), runs hmmsearch against the HMM library, filters hits, and emits the hitlist and optional FASTA sequence output (-o)) — https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- **hmmsearch 3.1b2** (HMMER3 search of the FASTA protein set against the DiSCo HMM library with default parameters) — http://hmmer.org/
- **Perl 5** (Language in which DiSCo.pl and the filter script are implemented; the filter script parses the hmmsearch output)

## Examples

```
perl DiSCo.pl -i example.faa -p example -s 1 -o
```

## Evaluation signals

- Output hitlist columns match the documented 20-column schema (Sequence identifier, HMM name, Predicted enzyme type, ..., DsrC-type notes)
- All retained hits satisfy the filter: conditional and independent E-value <=10^-10, accuracy >=0.5, and bias one order of magnitude lower than score
- Each assigned hit passes its model-specific score/E-value from thresholds.txt, and only the best-scoring model per input sequence is kept
- With -o, the extracted FASTA sequences correspond one-to-one to the hitlist entries
- In jackknife mode, leave-one-out results reproduce the full-model reference (expected 0.9% assignment loss, 5 of 545, and no type-prediction changes)

## Limitations

- DiSCo requires hmmsearch to be in the user's $PATH and depends on HMMER3; no dependency auto-installation is provided
- hmmsearch runs with 1 thread by default (-n must be set explicitly to parallelize)
- The HMM library only covers Dsr-dependent dissimilatory sulphur metabolism proteins; hits to paralogous families are resolved solely by model-specific cutoffs and best-hit scoring

## Evidence

- [abstract] It takes user-supplied protein sequences and lists the identified proteins and their classification in terms of protein family and predicted type.: "It takes user-supplied protein sequences and lists the identified proteins and their classification in terms of protein family and predicted type."
- [readme] This tool runs a set of FASTA formatted protein sequences against the curated set of hidden Markov models (HMMs).: "This tool runs a set of FASTA formatted protein sequences against the curated set of hidden Markov models (HMMs)."
- [intro] The first step of DiSCo.pl consists of an hmmsearch of a given protein sequences set in FASTA format against the DiSCo HMM library: [evidence span withheld — anchor status: not_found]
- [intro] DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5: "DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5"
- [methods] #tab seperated table with user-defined cut-offs: column 1: model, column 2: score, column 3: E-value: "#tab seperated table with user-defined cut-offs: column 1: model, column 2: score, column 3: E-value"
