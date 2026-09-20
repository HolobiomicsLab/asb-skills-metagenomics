---
name: de-bruijn-graph-genotype-calling
description: Use when raw FASTQ reads (Illumina MiSeq or high-error-rate ONT MinION)
  from S. aureus or M. tuberculosis isolates must be converted directly into an antibiotic-resistance
  call set without assembly or SNP-calling pipelines.
when_to_use_negative:
- The isolate belongs to a species with poorly characterized resistance mechanisms
  and no large training set with WGS + phenotype data exists
- The goal is de novo resistance-mechanism discovery rather than genotyping known
  mutations — Mykrobe only calls variants in its curated panel
- M. tuberculosis resistance to pyrazinamide or novel/unknown mechanisms is the target
  — known mutations are poorly predictive and the panel excludes them
edam_operation: http://edamontology.org/operation_3200
edam_topics:
- http://edamontology.org/topic_3301
- http://edamontology.org/topic_0625
tools:
- name: Mykrobe predictor
  role: De Bruijn graph genotype caller that takes raw sequence data as input and
    predicts species and resistance profile
  repo: https://github.com/Mykrobe-tools/mykrobe
- name: mykrobe-parser
  role: R script parsing Mykrobe JSON output for M. tuberculosis into tidy per-genetic-region
    CSV reports
  repo: https://github.com/phac-nml/mykrobe-parser
- name: Oxford Nanopore MinION
  role: Single-molecule sequencer providing the high-error-rate reads the caller must
    tolerate (N=1 proof-of-principle, fully concordant with MiSeq)
- name: Illumina MiSeq
  role: Reference short-read platform producing the comparison resistance profile
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:141abbaa159cab5517293aaa0d8327a91935369c1d1e670a0c3a08858541cdad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mykrobe-tools/mykrobe
  tools:
  - Illumina MiSeq
  - Mykrobe predictor
  - Oxford Nanopore MinION
  - mykrobe-parser
  edam_operation: http://edamontology.org/operation_3200
  edam_topics:
  - http://edamontology.org/topic_3301
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# de-bruijn-graph-genotype-calling

## Summary

Predict species identity and antibiotic-resistance profiles from raw sequence data by genotyping known resistance determinants against a de Bruijn graph representation of bacterial diversity, implemented in Mykrobe predictor and demonstrated on both Illumina MiSeq and Oxford Nanopore MinION reads.

## When to use

Use when raw FASTQ reads (Illumina MiSeq or high-error-rate ONT MinION) from S. aureus or M. tuberculosis isolates must be converted directly into an antibiotic-resistance call set without assembly or SNP-calling pipelines.

## When NOT to use

- The isolate belongs to a species with poorly characterized resistance mechanisms and no large training set with WGS + phenotype data exists
- The goal is de novo resistance-mechanism discovery rather than genotyping known mutations — Mykrobe only calls variants in its curated panel
- M. tuberculosis resistance to pyrazinamide or novel/unknown mechanisms is the target — known mutations are poorly predictive and the panel excludes them

## Inputs

- Raw sequence reads (FASTQ) from Illumina MiSeq, e.g. 368x mean depth
- Raw ONT MinION 2D reads (FASTQ), e.g. 39x mean depth, prepared with SQK-MAP005 kit
- Mykrobe resistance panel (201901 or 202010 for M. tuberculosis)
- Species of interest (S. aureus, M. tuberculosis, S. sonnei, S. typhi, S. Paratyphi B)

## Outputs

- Species call and resistance profile across the antibiotic panel (S. aureus: 12 antibiotics)
- Mykrobe JSON results file (--format json) with resistance calls and variant/genetic-region detail
- Mykrobe Parser outputs: output-report.csv (per-antimicrobial predictions) and output-jsondata.csv (phylo_group, species, lineage, depth, % coverage)

## How to apply

Run Mykrobe predictor (a de Bruijn graph genotype caller) directly on raw reads, taking FASTQ as input and emitting JSON resistance predictions across the 12-antibiotic S. aureus panel. For ONT MinION data, run with the nanopore error model (10% error, base quality scores ignored); for Illumina MiSeq data use the standard model — the paper showed identical calls at 39x mean MinION depth vs 368x MiSeq depth. Compare predictions antibiotic-by-antibiotic across platforms in R and check key determinants such as gyrA S84L for ciprofloxacin resistance. For M. tuberculosis outputs, optionally parse the JSON with Mykrobe Parser (requires --format json, --panel 201901 or 202010) to split mutations into individual genetic regions in tidy CSV format.

## Related tools

- **Mykrobe predictor** (De Bruijn graph genotype caller that takes raw sequence data as input and predicts species and resistance profile) — https://github.com/Mykrobe-tools/mykrobe
- **mykrobe-parser** (R script parsing Mykrobe JSON output for M. tuberculosis into tidy per-genetic-region CSV reports) — https://github.com/phac-nml/mykrobe-parser
- **Oxford Nanopore MinION** (Single-molecule sequencer providing the high-error-rate reads the caller must tolerate (N=1 proof-of-principle, fully concordant with MiSeq))
- **Illumina MiSeq** (Reference short-read platform producing the comparison resistance profile)

## Examples

```
mykrobe predict --sample my_sample_name --species tb --output out.json --format json --seq reads.fq.gz
```

## Evaluation signals

- For S. aureus, per-antibiotic sensitivity/specificity should be in the ~99.1%/99.6% range on an independent validation set (n=470)
- On the MinION vs MiSeq concordance test, expect 12/12 identical resistance calls for the MDR S. aureus isolate (penicillin/methicillin/gentamicin/trimethoprim/erythromycin/ciprofloxacin/clindamycin resistant; fusidic acid susceptible)
- gyrA S84L is called as the ciprofloxacin resistance determinant in both platform outputs, with no additional false-positive resistance SNPs
- Mykrobe JSON output parses without error in Mykrobe Parser (v0.7.0-compatible, 201901/202010 panel); gene-prediction failures should surface as '[antimicrobial]_prediction' = 'failed' with genetic regions 'NA'
- Results generated as a clinician-friendly report within 3 minutes on a laptop

## Limitations

- The MinION test is only an N=1 proof-of-principle, though fully concordant with MiSeq
- M. tuberculosis sensitivity is lower (82.6%) than traditional DST and completely excludes pyrazinamide since known mutations are poorly predictive
- The current implementation models only major/minor clones; a more general mixture model is needed for direct-from-sputum M. tuberculosis analysis

## Evidence

- [abstract] de Bruijn graph representation for raw-read genotyping: "Here we show how de Bruijn graph representation of bacterial diversity can be used to identify species and resistance profiles of clinical isolates"
- [abstract] Mykrobe predictor takes raw sequence data as input: "We implement this method for Staphylococcus aureus and Mycobacterium tuberculosis in a software package (‘Mykrobe predictor') that takes raw sequence data as input"
- [discussion] Nanopore concordance with Illumina: "Our N=1 test of the Oxford Nanopore MinION machine offers only proof-of-principle, but unlike one recent report49, we were able to get fully concordant results"
- [abstract] TB resistance prediction performance: "For M. tuberculosis, our method predicts resistance with sensitivity/specificity of 82.6%/98.5% (independent validation set, n=1,609)"
- [readme] Mykrobe predictor CLI invocation: [evidence span withheld — anchor status: not_found]
- [readme] Mykrobe Parser panel and format requirements: "Mykrobe Parser is compatible with [Mykrobe v0.7.0](https://anaconda.org/bioconda/mykrobe/files?version=0.7.0) and the "201901" / "202010" panels."
