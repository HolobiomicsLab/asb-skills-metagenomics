---
name: cross-platform-resistance-call-concordance-analysis
description: Use when the same clinical isolate has raw reads from both a high-error
  single-molecule platform (e.g. MinION) and a short-read platform (e.g. MiSeq) and
  you need to verify that genotypic resistance predictions agree across platforms.
when_to_use_negative:
- You only have reads from a single platform — there is no second platform call set
  to compare against, so a concordance analysis is undefined.
- The target organism is one where Mykrobe panels are unavailable or poorly characterized
  (the paper notes resistance mechanisms for such species require large training sets
  not yet available).
- You need per-drug diagnostic sensitivity/specificity estimates — the paper's N=1
  MinION test is explicitly 'only proof-of-principle' and cannot support performance
  statistics.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_2815
tools:
- name: Mykrobe predictor
  role: Genotypic AMR predictor (de Bruijn graph caller) run independently on each
    platform's raw reads to produce resistance calls
  repo: https://github.com/Mykrobe-tools/mykrobe
- name: Oxford Nanopore MinION
  role: Single-molecule sequencing platform supplying the high-error-rate test reads
    (~39x, 10% error model, base quality ignored)
- name: Illumina MiSeq
  role: Short-read platform supplying the reference resistance profile for the same
    isolate (~368x depth)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:89cfbe950a84b369d3061b31fb82abb3aa3ef06bfbf5b42d2e38c33c64ca6ab4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mykrobe-tools/mykrobe
  tools:
  - Illumina MiSeq
  - Mykrobe predictor
  - Oxford Nanopore MinION
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_2815
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cross-platform-resistance-call-concordance-analysis

## Summary

Runs Mykrobe predictor independently on ONT MinION and Illumina MiSeq reads of the same isolate and compares the resulting antibiotic-resistance calls drug-by-drug to quantify cross-platform concordance, as demonstrated in the paper's N=1 S. aureus proof-of-principle that yielded 12/12 identical calls.

## When to use

Use when the same clinical isolate has raw reads from both a high-error single-molecule platform (e.g. MinION) and a short-read platform (e.g. MiSeq) and you need to verify that genotypic resistance predictions agree across platforms.

## When NOT to use

- You only have reads from a single platform — there is no second platform call set to compare against, so a concordance analysis is undefined.
- The target organism is one where Mykrobe panels are unavailable or poorly characterized (the paper notes resistance mechanisms for such species require large training sets not yet available).
- You need per-drug diagnostic sensitivity/specificity estimates — the paper's N=1 MinION test is explicitly 'only proof-of-principle' and cannot support performance statistics.

## Inputs

- Raw ONT MinION 2D FASTQ reads of the isolate (~39x mean depth)
- Raw Illumina MiSeq FASTQ reads of the same isolate (~368x mean depth)
- Mykrobe predictor with the S. aureus species panel (12-antibiotic resistance panel)

## Outputs

- Per-platform Mykrobe resistance profiles across the 12-antibiotic panel
- Antibiotic-by-antibiotic concordance table (e.g. 12/12 agreement) comparing MinION vs MiSeq calls
- Identification of resistance-determinant SNPs (e.g. gyrA S84L) from each platform output

## How to apply

Obtain raw reads for both platforms from ENA/NCBI SRA, then run Mykrobe predictor (a de Bruijn graph genotype caller) on each dataset separately: the MinION 2D reads at ~39x mean depth with base quality scores ignored, and the MiSeq reads at ~368x mean depth. Tabulate the resistance profile across the 12-antibiotic S. aureus panel for each platform and compare antibiotic-by-antibiotic (e.g. in R). Verify concordant resistance-determinant calls — e.g. the gyrA S84L SNP for ciprofloxacin resistance — appear identically in both outputs, and check that neither platform reports false-positive resistance SNVs. Judge success by full agreement of calls between platforms, as the paper's N=1 test produced 'fully concordant results' (resistant to penicillin, methicillin, gentamicin, trimethoprim, erythromycin, ciprofloxacin and clindamycin; susceptible to fusidic acid) despite the ONT 10% error model.

## Related tools

- **Mykrobe predictor** (Genotypic AMR predictor (de Bruijn graph caller) run independently on each platform's raw reads to produce resistance calls) — https://github.com/Mykrobe-tools/mykrobe
- **Oxford Nanopore MinION** (Single-molecule sequencing platform supplying the high-error-rate test reads (~39x, 10% error model, base quality ignored))
- **Illumina MiSeq** (Short-read platform supplying the reference resistance profile for the same isolate (~368x depth))

## Examples

```
mykrobe predict --sample MDR-SA --species staph --output minion.json --format json --seq minion_2d_reads.fq.gz
```

## Evaluation signals

- Concordance table shows 12/12 identical resistance calls between MinION and MiSeq for the S. aureus isolate.
- The known ciprofloxacin resistance determinant gyrA S84L is called identically in both platform outputs.
- No additional resistance SNPs are falsely reported by either platform (no platform-specific false positives).
- MinION run reaches sufficient depth (~39x mean) for Mykrobe's genotype calling to converge on a single resistance profile.
- Species/resistance panel assignment is consistent across both runs (same isolate, same species call).

## Limitations

- The MinION test is only an N=1 proof-of-principle; it cannot establish generalizable cross-platform accuracy.
- MinION base quality scores are ignored and a 10% error model is used, so the approach depends on Mykrobe's error tolerance rather than per-base calibration.
- Cross-platform concordance does not validate resistance calls against phenotype — S. aureus panel sensitivity/specificity (99.1%/99.6%) came from separate phenotyped validation sets.

## Evidence

- [discussion] Our N=1 test of the Oxford Nanopore MinION machine offers only proof-of-principle, but unlike one recent report49, we were able to get fully concordant results: "Our N=1 test of the Oxford Nanopore MinION machine offers only proof-of-principle, but unlike one recent report49, we were able to get fully concordant results"
- [other] Mykrobe predictor gave identical correct calls from both MiSeq and MinION data: [evidence span withheld — anchor status: not_found]
- [discussion] Using an Illumina MiSeq 16.5 h run, our proposed sequencing workflow would provide a full set of predictions for all drugs at 36 h: "Using an Illumina MiSeq 16.5 h run, our proposed sequencing workflow would provide a full set of predictions for all drugs at 36 h"
- [abstract] software package (‘Mykrobe predictor') that takes raw sequence data as input: "We implement this method for Staphylococcus aureus and Mycobacterium tuberculosis in a software package (‘Mykrobe predictor') that takes raw sequence data as input"
- [discussion] The Mykrobe predictor software is freely available (opensource) at www.github.com/iqbal-lab/Mykrobe-predictor: "The Mykrobe predictor software is freely available (opensource) at www.github.com/iqbal-lab/Mykrobe-predictor for non-commercial academic and research use only"
