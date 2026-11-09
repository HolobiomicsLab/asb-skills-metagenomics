---
name: genome-annotation-parsing-with-biopython
description: Use when the input is a complete genome annotation (GenBank .gbff/.gbk
  plus FASTA, e.g. Deinococcus radiodurans R1) and the goal is to extract all annotated
  CDS/ORF lengths or adjacent-gene features for fitting a gene-length distribution
  model.
when_to_use_negative:
- Input is metagenomic read data rather than a finished genome annotation — this skill
  derives training statistics from curated annotation, it does not predict genes (use
  Glimmer-MG itself)
- You only need gene predictions and have no reference genome annotation to parse
  — the parsing step adds nothing for a pure prediction run
- The annotation is in formats Biopython SeqIO cannot parse (e.g. raw GFF3 without
  conversion) — convert first rather than hand-rolling a parser
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Biopython
  role: Parses GenBank/FASTA genome annotation records to extract annotated coding
    ORF lengths and adjacent-gene features for length-distribution modeling
- name: Glimmer-MG
  role: Defines the blended KDE/Gamma gene-length model and adjacent-gene feature
    statistics that the extracted ORF lengths are used to reproduce and validate
  repo: https://github.com/davek44/Glimmer-MG
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:e45b7ab8ffb4b78826c1fa8f1234c6331229a2763b504ccbf2fac6f176ba77ec
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/davek44/Glimmer-MG
  repo_url: https://github.com/davek44/Glimmer-MG
  tools:
  - Biopython
  - Glimmer-MG
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# genome-annotation-parsing-with-biopython

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parses GenBank/FASTA genome annotation records with Biopython to extract annotated coding ORFs and derived per-gene features (lengths, adjacent-gene orientations, intergenic distances). It is the data-preparation layer used to fit ORF length distributions such as those Glimmer-MG models for metagenomic gene prediction.

## When to use

Use when the input is a complete genome annotation (GenBank .gbff/.gbk plus FASTA, e.g. Deinococcus radiodurans R1) and the goal is to extract all annotated CDS/ORF lengths or adjacent-gene features for fitting a gene-length distribution model.

## When NOT to use

- Input is metagenomic read data rather than a finished genome annotation — this skill derives training statistics from curated annotation, it does not predict genes (use Glimmer-MG itself)
- You only need gene predictions and have no reference genome annotation to parse — the parsing step adds nothing for a pure prediction run
- The annotation is in formats Biopython SeqIO cannot parse (e.g. raw GFF3 without conversion) — convert first rather than hand-rolling a parser

## Inputs

- Complete genome annotation in GenBank format (.gbff/.gbk), e.g. Deinococcus radiodurans R1
- Genome sequence in FASTA format for coordinate/length validation
- Optional: translated CDS FASTA from the same annotation

## Outputs

- Vector of annotated coding ORF nucleotide lengths extracted from GenBank CDS features
- Fitted Gamma distribution parameters (MLE via scipy.stats.gamma.fit)
- Kernel-smoothed histogram (Gaussian KDE) of ORF lengths
- Blended ORF length-distribution curve (KDE first quartile, Gamma last quartile, linear interpolation between)
- Adjacent-gene orientation category frequencies and intergenic distance distributions
- Matplotlib figure reproducing Figure 1 (e.g. figure1_length_distributions.png)

## How to apply

Load the genome annotation with Bio.SeqIO.parse in 'genbank' (or 'fasta') format and extract the lengths of all annotated coding ORFs by iterating over CDS features, converting coordinates to nucleotide lengths. Fit a Gamma distribution to the ORF lengths by maximum likelihood with scipy.stats.gamma.fit, and compute a kernel-smoothed histogram (scipy.stats.gaussian_kde) to capture the empirical short/medium-length ORF distribution. Construct Glimmer-MG's blended model: kernel-smoothed histogram for the first length quartile, Gamma fit for the last quartile, and linear interpolation between the first and third quartiles — the blend is used because KDE is more accurate for short and medium ORFs while Gamma handles very long ORFs. In the same pass, compute adjacent-gene orientation category frequencies and intergenic distance distributions from feature coordinates. Overlay the Gamma fit, KDE, and blended curves with Matplotlib to reproduce the D. radiodurans R1 Figure 1 length-distribution curves.

## Related tools

- **Biopython** (Parses GenBank/FASTA genome annotation records to extract annotated coding ORF lengths and adjacent-gene features for length-distribution modeling)
- **Glimmer-MG** (Defines the blended KDE/Gamma gene-length model and adjacent-gene feature statistics that the extracted ORF lengths are used to reproduce and validate) — https://github.com/davek44/Glimmer-MG

## Examples

```
from Bio import SeqIO; orf_lens=[len(f) for rec in SeqIO.parse('dradiodurans.gbff','genbank') for f in rec.features if f.type=='CDS']
```

## Evaluation signals

- Extracted ORF lengths match CDS coordinate spans in the GenBank record (count and total length consistent with the annotation)
- Blended curve follows the empirical KDE in the low-length (first-quartile) region and the Gamma MLE curve in the high-length (last-quartile) region
- Overlay of Gamma fit, kernel-smoothed histogram, and blended curve is visually consistent with published Figure 1 for D. radiodurans R1
- Adjacent-gene orientation frequencies partition all annotated gene pairs into valid orientation categories with counts summing to the expected number of adjacent pairs

## Limitations

- The KDE component is 'useless for very long ORFs', so the blend relies on the Gamma fit only in the high-length tail; a poor Gamma MLE fit corrupts that tail
- Predicting genes on error-prone sequences, particularly those with indels, remains difficult for the downstream model these features train
- Highly diverged organisms where optimal model parameters are unclear challenge the assumption that a single reference genome's length statistics transfer

## Evidence

- [other] workflow grounding for Biopython extraction of ORF lengths: [evidence span withheld — anchor status: not_found]
- [other] blend rationale for KDE vs Gamma: [evidence span withheld — anchor status: not_found]
