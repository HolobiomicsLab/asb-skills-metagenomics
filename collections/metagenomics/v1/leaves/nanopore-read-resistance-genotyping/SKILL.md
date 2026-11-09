---
name: nanopore-read-resistance-genotyping
description: Use when you have raw single-molecule nanopore (MinION) reads of a bacterial
  isolate and need antibiotic resistance predictions without prior error correction,
  ideally benchmarked against a second sequencing platform.
when_to_use_negative:
- 'The aim is M. tuberculosis resistance prediction from direct sputum mixtures: the
  article notes the current implementation models only major/minor clones and a more
  general mixture model would be needed'
- 'You need validated per-drug sensitivity for nanopore data: the MinION test is only
  an N=1 proof-of-principle with a single isolate'
- Input is not raw sequence data but already-called variant lists or phenotype tables;
  Mykrobe predictor requires raw reads as input
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_3299
tools:
- name: Mykrobe predictor
  role: De Bruijn graph genotype caller run on raw MinION and MiSeq reads to predict
    the resistance profile
  repo: https://github.com/Mykrobe-tools/mykrobe
- name: Oxford Nanopore MinION
  role: Single-molecule sequencing platform providing the high-error-rate reads being
    genotyped (library prepared with SQK-MAP005)
- name: Illumina MiSeq
  role: Second-platform sequencing of the same isolate providing the reference resistance
    profile for concordance comparison
- name: mykrobe-parser
  role: R script that parses Mykrobe JSON output into tidy tabular reports for downstream
    comparison
  repo: https://github.com/phac-nml/mykrobe-parser
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:4873147015923147812067b2a7a3c65668577b7744fc562028b219be040a5ccd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mykrobe-tools/mykrobe
  tools:
  - Illumina MiSeq
  - Mykrobe predictor
  - Oxford Nanopore MinION
  - mykrobe-parser
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# nanopore-read-resistance-genotyping

## Summary

Runs Mykrobe predictor's de Bruijn graph genotyper directly on high-error-rate Oxford Nanopore MinION reads (e.g. a 39x-depth MDR S. aureus isolate, 10% error model, base qualities ignored) to call a resistance profile that can be validated against Illumina MiSeq calls from the same isolate.

## When to use

Use when you have raw single-molecule nanopore (MinION) reads of a bacterial isolate and need antibiotic resistance predictions without prior error correction, ideally benchmarked against a second sequencing platform.

## When NOT to use

- The aim is M. tuberculosis resistance prediction from direct sputum mixtures: the article notes the current implementation models only major/minor clones and a more general mixture model would be needed
- You need validated per-drug sensitivity for nanopore data: the MinION test is only an N=1 proof-of-principle with a single isolate
- Input is not raw sequence data but already-called variant lists or phenotype tables; Mykrobe predictor requires raw reads as input

## Inputs

- Raw Oxford Nanopore MinION 2D FASTQ reads (e.g. 39x mean depth) of an S. aureus isolate
- Illumina MiSeq FASTQ reads (e.g. 368x mean depth) of the same isolate for concordance comparison
- Mykrobe resistance panel/k-mer database for S. aureus (12-antibiotic panel)
- Sequencing accessions from Supplementary Data 1-10 (ENA/NCBI SRA)

## Outputs

- Mykrobe predictor JSON resistance report (species call, per-drug resistant/susceptible calls, causative variants such as gyrA S84L)
- Antibiotic-by-antibiotic MinION vs MiSeq concordance table (expected 12/12 agreement in the paper's proof-of-principle)

## How to apply

Obtain raw ONT MinION 2D reads (the paper's N=1 MDR S. aureus isolate had 39x mean depth) from ENA/SRA accessions listed in Supplementary Data 1-10, plus the Illumina MiSeq reads (368x mean depth) of the same isolate. Run Mykrobe predictor on the MinION reads under a 10% error model with base quality scores ignored, predicting across the 12-antibiotic S. aureus panel; run it identically on the MiSeq reads to generate the reference profile. Compare calls antibiotic-by-antibiotic (e.g. tabulate concordance in R) and inspect variant-level output to confirm the causative SNP (here gyrA S84L for ciprofloxacin) is present on both platforms and that no false-positive resistance SNPs are reported. Mykrobe outputs JSON (--format ), which can be parsed downstream with tools such as the phac-nml mykrobe-parser R script. Rationale: the de Bruijn graph approach tolerates high per-read error because genotype likelihood is aggregated over many k-mers, so no basecall error correction is required.

## Related tools

- **Mykrobe predictor** (De Bruijn graph genotype caller run on raw MinION and MiSeq reads to predict the resistance profile) — https://github.com/Mykrobe-tools/mykrobe
- **Oxford Nanopore MinION** (Single-molecule sequencing platform providing the high-error-rate reads being genotyped (library prepared with SQK-MAP005))
- **Illumina MiSeq** (Second-platform sequencing of the same isolate providing the reference resistance profile for concordance comparison)
- **mykrobe-parser** (R script that parses Mykrobe JSON output into tidy tabular reports for downstream comparison) — https://github.com/phac-nml/mykrobe-parser

## Examples

```
mykrobe predict --sample mdr_sa_minion --species staph --output minion. --format  --seq minion_reads.fq.gz
```

## Evaluation signals

- Concordance table shows 12/12 identical drug calls between MinION and MiSeq for the isolate (paper reports fully concordant results)
- gyrA S84L is called as the ciprofloxacin resistance determinant in both platform outputs
- No platform output reports a false-positive resistance SNP beyond known determinants
- Resistance/susceptibility pattern matches expectation: resistant to penicillin, methicillin, gentamicin, trimethoprim, erythromycin, ciprofloxacin, clindamycin; susceptible to fusidic acid
- If using mykrobe-parser, JSON inputs were produced with --format  and a compatible panel (201901/202010); failed gene predictions appear as 'failed' with regions 'NA'

## Limitations

- The MinION result is an N=1 proof-of-principle; platform concordance is not validated at scale
- Base quality scores are ignored and a 10% error model is assumed; very low-depth nanopore runs may not genotyped reliably
- Downstream mykrobe-parser is M. tuberculosis specific and removes samples detected as non-tuberculous mycobacteria

## Evidence

- [discussion] N=1 MinION concordance: "Our N=1 test of the Oxford Nanopore MinION machine offers only proof-of-principle, but unlike one recent report49, we were able to get fully concordant results"
- [abstract] Raw input requirement: "We implement this method for Staphylococcus aureus and Mycobacterium tuberculosis in a software package (‘Mykrobe predictor') that takes raw sequence data as input"
- [discussion] MiSeq turnaround: "Using an Illumina MiSeq 16.5 h run, our proposed sequencing workflow would provide a full set of predictions for all drugs at 36 h"
- [readme] Parser input format: [evidence span withheld — anchor status: not_found]
- [readme] Parser QC behaviour: "Samples that Mykrobe detects non-tuberculous mycobacteria in are removed for quality control purposes."
- [abstract] Feasibility finding: "We give evidence that minor alleles improve detection of extremely drug-resistant strains, and demonstrate feasibility of the use of emerging single-molecule nanopore sequencing techniques"
