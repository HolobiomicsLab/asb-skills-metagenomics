---
name: linked-read-fragment-parameter-tabulation
description: Use when multiple high-depth 10x Chromium libraries must be compared
  at a controlled read coverage per fragment (CR), requiring fragment inference and
  per-fragment subsampling to equalize parameters.
when_to_use_negative:
- Input is already low-coverage Linked-Read data whose realized per-fragment coverage
  matches the target CR, so subsampling would discard information without changing
  the comparison
- The goal is de novo assembly output evaluation rather than preparing matched Linked-Read
  input sets (see Evaluate_diploid_assembly for that task)
- Data are not barcode-linked short reads (e.g. PacBio or ONT long reads), since fragment
  inference depends on barcode-proximal read pairs
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Long Ranger
  role: Aligns raw 10x paired-end reads to hg38 and performs barcode-aware PCR duplicate
    removal before fragment inference
- name: LRTK-SIM
  role: Simulates 10x Linked-Read libraries under matched CF and CR parameters so
    real subsampled sets can be generated with the same parameter definitions (CR
    = coverage of short reads per fragment)
  repo: github.com/zhanglu295/LRTK-SIM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giz141
    title: Assessment of human diploid genome assembly with 10x Linked-Reads data
  source_content_hash: sha256:1e56b0fc7009229d2e14032c83eef54401e1c7a87d4011dd7578b24390492925
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - 10x Linked-Read sequencing
  - LRTK-SIM
  - Long Ranger
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# linked-read-fragment-parameter-tabulation

## Summary

Reconstructs original input DNA fragments from high-coverage 10x Linked-Read data and subsamples reads per fragment to a target read coverage per fragment (CR), producing matched Linked-Read sets (Rid) across libraries for fair assembly-parameters comparison.

## When to use

Use when multiple high-depth 10x Chromium libraries must be compared at a controlled read coverage per fragment (CR), requiring fragment inference and per-fragment subsampling to equalize parameters.

## When NOT to use

- Input is already low-coverage Linked-Read data whose realized per-fragment coverage matches the target CR, so subsampling would discard information without changing the comparison
- The goal is de novo assembly output evaluation rather than preparing matched Linked-Read input sets (see Evaluate_diploid_assembly for that task)
- Data are not barcode-linked short reads (e.g. PacBio or ONT long reads), since fragment inference depends on barcode-proximal read pairs

## Inputs

- Raw 10x Genomics Chromium paired-end FASTQ reads from high-coverage libraries (e.g. NA12878 / NA24385)
- hg38 human reference genome sequence
- Target per-fragment short-read coverage value (CR)

## Outputs

- Gzipped R1/R2 FASTQ pairs of subsampled reads (e.g. X_S1_L001_R1_001.fastq.gz and X_S1_L001_R2_001.fastq.gz)
- Per-partition table of fragment count, fragment length, and realized coverage

## How to apply

Align raw 10x paired-end reads to the hg38 reference with the Long Ranger pipeline and remove PCR duplicates in a barcode-aware manner using Long Ranger's duplicate marking. Infer input DNA fragments by grouping barcode-proximal read pairs: terminate fragment extension at gaps >50 kb, and retain only fragments supported by >=2 read pairs and >=2 kb in length. Discard partitions containing fewer than 3 inferred fragments, then randomly subsample read pairs within each retained fragment until the target short-read coverage per fragment (CR) is met. Write the result as gzipped R1/R2 FASTQ pairs and tabulate per-partition fragment count, length, and realized coverage; the CR parameter mirrors the 'CR, covergae of short reads for each fragment' control in LRTK-SIM, so simulated and real libraries can be matched on the same axis.

## Related tools

- **Long Ranger** (Aligns raw 10x paired-end reads to hg38 and performs barcode-aware PCR duplicate removal before fragment inference)
- **LRTK-SIM** (Simulates 10x Linked-Read libraries under matched CF and CR parameters so real subsampled sets can be generated with the same parameter definitions (CR = coverage of short reads per fragment)) — github.com/zhanglu295/LRTK-SIM

## Examples

```
python LRTK-SIM.py ./diploid_config
```

## Evaluation signals

- The output gzipped FASTQ pair exists and both R1 and R2 files are well-formed (readable, matching read counts)
- Realized per-fragment read coverage qualitatively matches the configured CR target (the paper reports no numeric tolerance)
- Every retained partition contains at least 3 inferred fragments
- All retained fragments are supported by >=2 read pairs and span >=2 kb, with extension terminated at gaps >50 kb
- Simulated output validated through 10x official pipelines: the simulated data have been tested and accepted by Long Ranger and Supernova

## Limitations

- The paper reports no numeric tolerance for matching realized CR to the target CR, so acceptance is a qualitative check
- Only SNVs are supported when generating diploid reference sequences via gen_fasta.py for simulation
- Generally only ~20-40% of the original long fragment sequence can be captured by the 10x Chromium system, limiting how faithfully fragments are observed

## Evidence

- [methods] Remove PCR duplicates in a barcode-aware manner using Long Ranger's duplicate marking.: [evidence span withheld — anchor status: not_found]
- [abstract] The optimal physical coverage was between 332× and 823× and assembly quality worsened if it increased to >1,000× for a given C.: "The optimal physical coverage was between 332× and 823× and assembly quality worsened if it increased to >1,000× for a given C."
- [readme] Two gzipped fastq files are generated and started by X_S1_L001_R1_001.fastq.gz and X_S1_L001_R2_001.fastq.gz to represent forward and reverse reads.: "Two gzipped fastq files are generated and started by X_S1_L001_R1_001.fastq.gz and X_S1_L001_R2_001.fastq.gz to represent forward and reverse reads."
- [readme] CR, covergae of short reads for each fragment: "`CR`, covergae of short reads for each fragment"
