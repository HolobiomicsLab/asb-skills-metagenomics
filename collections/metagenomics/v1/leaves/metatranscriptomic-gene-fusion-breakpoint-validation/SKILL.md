---
name: metatranscriptomic-gene-fusion-breakpoint-validation
description: Use when DNA-derived HGT breakpoint candidates (e.g., from LEMON get_acc_bkp)
  exist for a metagenomic sample and a matched RNA-seq/metatranscriptome sample is
  available for independent validation.
when_to_use_negative:
- No matched metatranscriptome (RNA-seq) sample exists for the metagenomic sample
  whose breakpoints are being validated — DNA-only data cannot support fusion-point
  validation
- The goal is de novo HGT discovery from transcript data alone; this skill only validates
  existing LEMON breakpoint calls
- Breakpoints were called by a tool other than LEMON with unverified positional accuracy,
  since the 200 bp concordance criterion assumes refined exact breakpoint positions
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STAR-Fusion
  role: Detects gene fusion points in matched metatranscriptome samples; fusion points
    within 200 bp of LEMON HGT breakpoints validate the calls (run with default parameters)
- name: LEMON
  role: Produces the accurate HGT breakpoint calls (get_raw_bkp.py + get_acc_bkp)
    that are being validated against transcriptomic fusion evidence
  repo: https://github.com/lichen2018/LEMON
- name: BWA
  role: Aligns paired-end shotgun reads (bwa mem -M) against the reference set as
    the upstream step generating junction/split reads for LEMON breakpoints
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-3301-8
    title: 'LEMON: a method to construct the local strains at horizontal gene transfer
      sites in gut metagenomics'
  source_content_hash: sha256:dd31b1631f3e9255367bb35f408512a7d004ac71e771a43010c00eefaa4c9fce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen2018/LEMON
  tools:
  - BWA
  - Gurobipy
  - LEMON
  - LUMPY
  - STAR-Fusion
  - iRep
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# metatranscriptomic-gene-fusion-breakpoint-validation

## Summary

Validate LEMON's HGT breakpoint calls from gut shotgun metagenomics by running STAR-Fusion on matched HMP IBD metatranscriptome samples and checking whether gene fusion points fall within 200 bp upstream or downstream of the predicted breakpoints.

## When to use

Use when DNA-derived HGT breakpoint candidates (e.g., from LEMON get_acc_bkp) exist for a metagenomic sample and a matched RNA-seq/metatranscriptome sample is available for independent validation.

## When NOT to use

- No matched metatranscriptome (RNA-seq) sample exists for the metagenomic sample whose breakpoints are being validated — DNA-only data cannot support fusion-point validation
- The goal is de novo HGT discovery from transcript data alone; this skill only validates existing LEMON breakpoint calls
- Breakpoints were called by a tool other than LEMON with unverified positional accuracy, since the 200 bp concordance criterion assumes refined exact breakpoint positions

## Inputs

- Accurate HGT breakpoint file produced by LEMON getAccBkp (get_acc_bkp) from shotgun reads
- Matched metatranscriptome (RNA-seq) FASTQ files from the same HMP IBD sample
- Metagenomic reference set S (16,093 species, 1,246,881 scaffolds)

## Outputs

- STAR-Fusion gene fusion calls (default output tables) from the matched metatranscriptome sample
- Validation verdict per HGT breakpoint: confirmed if a STAR-Fusion fusion point is within 200 bp of the breakpoint

## How to apply

First obtain accurate HGT breakpoint positions from the shotgun data using LEMON's split-read-based refinement (get_acc_bkp, with log-likelihood quality scoring of split-read alignments). Then run STAR-Fusion with default parameters on the matched metatranscriptome sample from the same subject/sample pair. Accept the validation if gene fusion points reported by STAR-Fusion lie within 200 bp upstream or downstream of the HGT breakpoints called by LEMON. This cross-modality (DNA/RNA) agreement is the article's real-data evidence that the reconstructed strains and their breakpoint calls are accurate.

## Related tools

- **STAR-Fusion** (Detects gene fusion points in matched metatranscriptome samples; fusion points within 200 bp of LEMON HGT breakpoints validate the calls (run with default parameters))
- **LEMON** (Produces the accurate HGT breakpoint calls (get_raw_bkp.py + get_acc_bkp) that are being validated against transcriptomic fusion evidence) — https://github.com/lichen2018/LEMON
- **BWA** (Aligns paired-end shotgun reads (bwa mem -M) against the reference set as the upstream step generating junction/split reads for LEMON breakpoints)

## Examples

```
LEMON/getAccBkp/get_acc_bkp -r metagenomic_reference.fa -u unique_reads.bam -s split_reads.bam -b raw_breakpoints.txt -o accurate_breakpoints.txt -t 8  # then STAR-Fusion (default params) on matched RNA-seq; accept breakpoints with fusion points within 200 bp
```

## Evaluation signals

- A STAR-Fusion gene fusion point maps within 200 bp upstream or downstream of the LEMON-called HGT breakpoint pair
- The breakpoints being validated are the accurate ones from get_acc_bkp (refined by split reads), not raw DBSCAN clusters from get_raw_bkp.py
- STAR-Fusion is run with default parameters, matching the article's protocol
- Validation passes only in combination with the other real-data signals: reconstructed strains reproduce complex donor-segment insertions (e.g., NZ_DS990133.1 in sample F-5) and some reconstructed strains show lower iRep standard error than the original references

## Limitations

- Validation requires matched metatranscriptome data, which is only available for a subset of HMP IBD samples
- STAR-Fusion only detects fusions expressed at the transcript level; silent or low-expression HGT events may lack corroborating fusion points
- The 200 bp concordance window is a heuristic; fusions slightly outside the window are not automatically false positives

## Evidence

- [abstract] Gene fusion points detected in real data near HGT breakpoints validate the accuracy of LEMON.: "the gene fusion points detected in real data near HGT breakpoints validate the accuracy of LEMON"
- [methods] STAR-Fusion and other tools run with default parameters.: "Most third-party tools used in this article are set default parameters, including BWA, LUMPY [25], iRep [26], and STAR-Fusion [27]."
- [results] Validation criterion: fusion points within 200 bp of breakpoints.: [evidence span withheld — anchor status: not_found]
- [readme] LEMON get_acc_bkp usage from README.: "usage: LEMON/getAccBkp/get_acc_bkp [options]"
