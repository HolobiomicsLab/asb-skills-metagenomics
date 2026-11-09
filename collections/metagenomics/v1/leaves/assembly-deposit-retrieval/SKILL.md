---
name: assembly-deposit-retrieval
description: Use when a published genome assembly paper reports assembly statistics
  (N50, coverage) and you need to download the deposited FASTA files and independently
  reproduce those metrics.
when_to_use_negative:
- You need to run the de novo assembly itself from raw reads rather than evaluate
  an already-deposited assembly
- The deposit provides only read data (FASTQ/BAM) with no assembled contig/scaffold
  FASTA to measure
- The paper reports no assembly statistics to validate against, so there is no target
  to compare recomputed metrics to
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: SOAPdenovo2
  role: Assembler whose deposited YH genome v2 assembly output (contigs and scaffolds)
    is retrieved and evaluated; the deposit also distributes the tool itself
  repo: https://github.com/aquaskyline/SOAPdenovo2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/2047-217x-1-18
    title: 'SOAPdenovo2: an empirically improved memory-efficient short-read <i>de
      novo</i> assembler'
  source_content_hash: sha256:35056515090865999ca5f282c6aea8febb4812688f2950613c55fac3db78629b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/SOAPdenovo2
  tools:
  - SOAPdenovo2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# assembly-deposit-retrieval

## Summary

Retrieves a genome assembly deposit (e.g. the YH genome v2 contig and scaffold FASTA files from the SOAPdenovo2 FTP server), recomputes its assembly statistics (contig N50, scaffold N50, reference coverage), and verifies them against values reported in the accompanying publication.

## When to use

Use when a published genome assembly paper reports assembly statistics (N50, coverage) and you need to download the deposited FASTA files and independently reproduce those metrics.

## When NOT to use

- You need to run the de novo assembly itself from raw reads rather than evaluate an already-deposited assembly
- The deposit provides only read data (FASTQ/BAM) with no assembled contig/scaffold FASTA to measure
- The paper reports no assembly statistics to validate against, so there is no target to compare recomputed metrics to

## Inputs

- Assembly deposit URL (e.g. ftp://public.genomics.org.cn/BGI/SOAPdenovo2)
- Contig FASTA file from the deposited assembly
- Scaffold FASTA file from the deposited assembly
- Reference genome FASTA (NCBI human reference hg19)
- Reported target metrics: contig N50 ~20.9 kbp, scaffold N50 ~22 Mbp, coverage 93.91%

## Outputs

- yh2_assembly_metrics.tsv containing recomputed contig N50, scaffold N50, and reference coverage
- Alignment of assembly scaffolds to the reference genome

## How to apply

Download the released assembly contig and scaffold FASTA files from the project deposit (e.g. ftp://public.genomics.org.cn/BGI/SOAPdenovo2 for the YH genome v2). Compute contig N50 by sorting contig lengths in descending order and finding the length at which cumulative length reaches half the total assembly length; repeat identically on scaffold sequences. Align the scaffold sequences to a reference (here NCBI human reference hg19) with a whole-genome aligner and compute reference coverage as the percentage of reference bases covered by aligned assembly sequences. Compare recomputed metrics against the paper's reported values — for YH v2: contig N50 ~20.9 kbp, scaffold N50 ~22 Mbp, coverage 93.91% (up from 81.16% in v1) — and write the three metrics to a table such as yh2_assembly_metrics.tsv. Accept the result only when recomputed values match the reported ones within approximation.

## Related tools

- **SOAPdenovo2** (Assembler whose deposited YH genome v2 assembly output (contigs and scaffolds) is retrieved and evaluated; the deposit also distributes the tool itself) — https://github.com/aquaskyline/SOAPdenovo2

## Examples

```
wget -m ftp://public.genomics.org.cn/BGI/SOAPdenovo2 && python compute_n50.py --fasta *.contig --fasta *.scafSeq --ref hg19.fa -o yh2_assembly_metrics.tsv
```

## Evaluation signals

- Recomputed contig N50 is approximately 20.9 kbp, matching the reported value
- Recomputed scaffold N50 is approximately 22 Mbp, matching the reported value
- Recomputed reference coverage is approximately 93.91% of hg19 bases
- All three metrics are written to yh2_assembly_metrics.tsv
- Scaffold N50 is roughly 50-fold and contig N50 roughly 3-fold longer than the first published YH version, consistent with the paper's claims

## Limitations

- Values are approximate (~); exact recomputation may differ slightly due to aligner choice and gap handling when extracting contigs from scaffolds at gap regions
- Reference coverage depends on the chosen reference (hg19) and alignment parameters, so coverage is not an assembly-intrinsic statistic
- The deposit URL is an FTP server and may be unavailable; no changelog was found for the repository

## Evidence

- [abstract] Recomputing assembly metrics on the released YH version 2 assembly confirms the reported values: contig N50 ~20.9 kbp, scaffold N50 ~22 Mbp, and reference coverage of 93.91%: "the contig and scaffold N50 of the YH genome were ~20.9 kbp and ~22 Mbp, respectively, which is 3-fold and 50-fold longer than the first published version."
- [abstract] Coverage improvement from v1 to v2 confirms the reported coverage metric: "The genome coverage increased from 81.16% to 93.91%, and memory consumption was ~2/3 lower during the point of largest memory consumption."
- [results] Deposit location for the released assembly: "are available from our FTP server ftp://public.genomics.org.cn/BGI/SOAPdenovo2"
- [results] Reference-alignment step used to compute coverage: "by aligning them to the NCBI human reference genome hg19"
- [readme] Output FASTA structure of the deposit/assembly: "b. *.scafSeq
  scaffold sequences (final contig sequences can be extracted by breaking down scaffold sequences at gap regions)."
