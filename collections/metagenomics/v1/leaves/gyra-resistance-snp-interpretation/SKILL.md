---
name: gyra-resistance-snp-interpretation
description: Use when raw FASTQ reads (Illumina MiSeq or ONT MinION 2D) from the same
  bacterial isolate must yield comparable drug-resistance calls, and a single known
  SNP (gyrA S84L) is expected as the resistance determinant.
when_to_use_negative:
- The isolate has multiple unknown resistance mechanisms — Mykrobe's panel is trained
  on known mutations, so uncharacterized mechanisms (e.g. pyrazinamide in M. tuberculosis)
  will not be detected
- The input is assembled/variant-called VCF data rather than raw sequence data — Mykrobe
  predictor takes raw sequence data as input
- The analysis goal is per-drug sensitivity estimation for rare resistant phenotypes
  — the article notes false-negative rates were only estimated where >10 resistant
  samples existed in the validation set
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: Mykrobe predictor
  role: De Bruijn graph genotype caller that takes raw MiSeq/MinION reads and outputs
    species identification plus the resistance profile used to detect gyrA S84L ciprofloxacin
    resistance
  repo: https://github.com/Mykrobe-tools/mykrobe
- name: Oxford Nanopore MinION
  role: Single-molecule sequencing platform whose 2D reads are tested for concordance
    against the Illumina reference profile
- name: Illumina MiSeq
  role: Generates the reference (368x depth) resistance profile for the same isolate
    against which MinION calls are compared
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:20f3e851abcf31d309281dfd2681cf0c708bcf03806d14af59ae3fa1f144e8c8
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
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# gyra-resistance-snp-interpretation

## Summary

Interprets a single gyrA S84L SNP as the ciprofloxacin resistance determinant in a multi-drug-resistant S. aureus isolate, using Mykrobe predictor's de Bruijn graph genotype caller on both Illumina MiSeq and ONT MinION raw reads, and validating platform concordance (12/12 identical calls).

## When to use

Use when raw FASTQ reads (Illumina MiSeq or ONT MinION 2D) from the same bacterial isolate must yield comparable drug-resistance calls, and a single known SNP (gyrA S84L) is expected as the resistance determinant.

## When NOT to use

- The isolate has multiple unknown resistance mechanisms — Mykrobe's panel is trained on known mutations, so uncharacterized mechanisms (e.g. pyrazinamide in M. tuberculosis) will not be detected
- The input is assembled/variant-called VCF data rather than raw sequence data — Mykrobe predictor takes raw sequence data as input
- The analysis goal is per-drug sensitivity estimation for rare resistant phenotypes — the article notes false-negative rates were only estimated where >10 resistant samples existed in the validation set

## Inputs

- Raw ONT MinION 2D FASTQ reads (39x mean depth, 10% error model, base quality scores ignored)
- Raw Illumina MiSeq FASTQ reads (368x mean depth)
- Mykrobe predictor S. aureus resistance panel spanning a 12-antibiotic panel (penicillin, methicillin, gentamicin, trimethoprim, erythromycin, ciprofloxacin, clindamycin, fusidic acid, ...)

## Outputs

- Mykrobe predictor resistance profile per platform (JSON output) across the 12-antibiotic panel
- MinION vs MiSeq antibiotic-by-antibiotic concordance table (12/12 agreement expected)
- Confirmed gyrA S84L call as the ciprofloxacin resistance determinant in both platform outputs

## How to apply

Obtain raw reads for both platforms from the ENA/NCBI SRA accessions listed in Supplementary Data. Run Mykrobe predictor on the ONT MinION 2D reads (39x mean depth, 10% error model, base quality scores ignored) and on the Illumina MiSeq reads (368x mean depth) to predict the resistance profile across the 12-antibiotic panel. Compare MinION vs MiSeq calls antibiotic-by-antibiotic in R and tabulate concordance. The decision point is that gyrA S84L must be called as the ciprofloxacin resistance determinant in BOTH platform outputs, with no additional (false-positive) resistance SNPs reported by either platform.

## Related tools

- **Mykrobe predictor** (De Bruijn graph genotype caller that takes raw MiSeq/MinION reads and outputs species identification plus the resistance profile used to detect gyrA S84L ciprofloxacin resistance) — https://github.com/Mykrobe-tools/mykrobe
- **Oxford Nanopore MinION** (Single-molecule sequencing platform whose 2D reads are tested for concordance against the Illumina reference profile)
- **Illumina MiSeq** (Generates the reference (368x depth) resistance profile for the same isolate against which MinION calls are compared)

## Examples

```
mykrobe predict --sample MDR_SA_miseq --species saureus --output out.json --format json --seq reads.fq.gz
```

## Evaluation signals

- Concordance table shows 12/12 identical resistance calls between MinION and MiSeq (matching the paper's fully concordant N=1 result)
- gyrA S84L is reported as the ciprofloxacin resistance determinant in BOTH platform outputs
- No platform yields a false-positive resistance SNP beyond the expected gyrA S84L call
- Expected call pattern reproduced: resistant to penicillin, methicillin, gentamicin, trimethoprim, erythromycin, ciprofloxacin and clindamycin; susceptible to fusidic acid
- Mykrobe JSON output parses cleanly (e.g. via mykrobe-parser, which only works with JSON files) with no '[antimicrobial]_prediction' columns set to 'failed'

## Limitations

- The MinION result is only an N=1 proof-of-principle, not a validated clinical workflow
- Base quality scores are ignored for MinION reads under the 10% error model; very low-depth or low-quality samples may not reach the ≥5-read SNP support threshold used elsewhere in the paper
- Mykrobe predictor is limited to species with well-characterized resistance panels; for species where resistance mechanisms are poorly characterized, a large training set would be needed

## Evidence

- [abstract] We implement this method for Staphylococcus aureus and Mycobacterium tuberculosis in a software package (‘Mykrobe predictor') that takes raw sequence data as input: "We implement this method for Staphylococcus aureus and Mycobacterium tuberculosis in a software package (‘Mykrobe predictor') that takes raw sequence data as input"
- [discussion] Fully concordant MinION result: "Our N=1 test of the Oxford Nanopore MinION machine offers only proof-of-principle, but unlike one recent report49, we were able to get fully concordant results"
- [other] Workflow validation step for gyrA S84L: [evidence span withheld — anchor status: not_found]
- [methods] Mykrobe supports S. aureus resistance prediction: "[Mykrobe](https://github.com/Mykrobe-tools/mykrobe) is an open-sourced program the can predict antimicrobial resistance from *Staphylococcus aureus* and *Mycobacterium tuberculosis*."
