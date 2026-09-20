---
name: genome-assembly-n50-calculation
description: Use when a released genome assembly (contig/scaffold FASTA files) needs
  its reported statistics — contig N50, scaffold N50, and coverage against a reference
  genome — independently reproduced and validated.
when_to_use_negative:
- Input is raw sequencing reads rather than an assembled FASTA — assembly must first
  be produced (e.g. with SOAPdenovo2's pregraph/contig/map/scaff steps)
- No reference genome is available, since the coverage metric (93.91% for YH v2) requires
  aligning scaffolds to a reference such as hg19
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: SOAPdenovo2
  role: Assembler whose released YH genome v2 contig and scaffold FASTA outputs are
    the statistics being verified
  repo: https://github.com/aquaskyline/SOAPdenovo2
- name: SOAPdenovo
  role: Predecessor assembler whose first-version YH statistics (81.16% coverage)
    provide the comparison baseline
- name: GapCloser
  role: SOAPdenovo2 gap-closing module that contributes to the scaffold continuity
    being measured by scaffold N50
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/2047-217x-1-18
    title: 'SOAPdenovo2: an empirically improved memory-efficient short-read <i>de
      novo</i> assembler'
  source_content_hash: sha256:4b8e34698c0d3193b0f9ac3660a5922c761f70ceecf7466ecde42d4eec01b5b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/SOAPdenovo2
  tools:
  - GapCloser
  - SOAPdenovo
  - SOAPdenovo2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# genome-assembly-n50-calculation

## Summary

Recompute contig N50, scaffold N50, and reference-coverage metrics from a de novo genome assembly (FASTA) to independently verify published assembly statistics, as done for the YH genome v2 assembly produced by SOAPdenovo2.

## When to use

Use when a released genome assembly (contig/scaffold FASTA files) needs its reported statistics — contig N50, scaffold N50, and coverage against a reference genome — independently reproduced and validated.

## When NOT to use

- Input is raw sequencing reads rather than an assembled FASTA — assembly must first be produced (e.g. with SOAPdenovo2's pregraph/contig/map/scaff steps)
- No reference genome is available, since the coverage metric (93.91% for YH v2) requires aligning scaffolds to a reference such as hg19

## Inputs

- Contig FASTA file from the released assembly (e.g. SOAPdenovo2 *.contig output)
- Scaffold FASTA file from the released assembly (e.g. SOAPdenovo2 *.scafSeq output)
- Reference genome FASTA to compute coverage against (e.g. NCBI human reference genome hg19)

## Outputs

- Recomputed contig N50 (bp)
- Recomputed scaffold N50 (bp)
- Reference coverage as percentage of hg19 bases covered by aligned assembly sequences
- Metrics table (yh2_assembly_metrics.tsv) comparing recomputed vs reported values

## How to apply

Download the released contig and scaffold FASTA files (e.g. from the project FTP deposit ftp://public.genomics.org.cn/BGI/SOAPdenovo2). Sort contig lengths in descending order and find the length at which cumulative length reaches half of the total assembly length to compute the contig N50; repeat the identical computation on scaffold sequences. To obtain reference coverage, align the scaffold sequences to the reference genome (for YH v2, the NCBI human reference genome hg19) with a whole-genome sequence aligner and compute the percentage of reference bases covered by aligned assembly sequences. Compare the three recomputed values against the published claims — for YH v2: contig N50 ~20.9 kbp, scaffold N50 ~22 Mbp (3-fold and 50-fold longer than the first published version), and coverage 93.91% (up from 81.16%) — writing all metrics to a TSV for validation.

## Related tools

- **SOAPdenovo2** (Assembler whose released YH genome v2 contig and scaffold FASTA outputs are the statistics being verified) — https://github.com/aquaskyline/SOAPdenovo2
- **SOAPdenovo** (Predecessor assembler whose first-version YH statistics (81.16% coverage) provide the comparison baseline)
- **GapCloser** (SOAPdenovo2 gap-closing module that contributes to the scaffold continuity being measured by scaffold N50)

## Examples

```
python compute_n50.py --contig YHv2.contig --scaffold YHv2.scafSeq --reference hg19.fa --out yh2_assembly_metrics.tsv
```

## Evaluation signals

- Recomputed contig N50 is ~20.9 kbp, matching the value reported for the YH genome v2 assembly
- Recomputed scaffold N50 is ~22 Mbp, matching the reported value (3-fold and 50-fold longer than the first published version for contig and scaffold N50 respectively)
- Reference coverage against hg19 is ~93.91%, up from 81.16% in the first version
- N50 computation invariant: cumulative length at the N50 contig equals half of the total assembly length
- All three recomputed metrics are written to yh2_assembly_metrics.tsv for side-by-side comparison with reported values

## Limitations

- N50 is length-based only — it does not capture structural or substitution errors, which required separate evaluation (e.g. alignment to 134 Mb of published human SD sequences)
- Coverage metric depends on the chosen reference (hg19) and the whole-genome aligner used; values are not portable across references
- Published statistics assume the exact deposited FASTA; re-assembling from reads may yield different N50 values

## Evidence

- [abstract] YH genome contig and scaffold N50 were ~20.9 kbp and ~22 Mbp, 3-fold and 50-fold longer than the first published version: "the contig and scaffold N50 of the YH genome were ~20.9 kbp and ~22 Mbp, respectively, which is 3-fold and 50-fold longer than the first published version."
- [abstract] YH genome coverage increased from 81.16% to 93.91%: "The genome coverage increased from 81.16% to 93.91%, and memory consumption was ~2/3 lower during the point of largest memory consumption."
- [results] YH deposit FTP location: "are available from our FTP server ftp://public.genomics.org.cn/BGI/SOAPdenovo2"
- [results] Coverage computed by aligning to hg19: "by aligning them to the NCBI human reference genome hg19"
- [readme] Output FASTA files that N50 is computed on: "a. *.contig
  contig sequences without using mate pair information.
b. *.scafSeq
  scaffold sequences (final contig sequences can be extracted by breaking down scaffold sequences at gap regions)."
