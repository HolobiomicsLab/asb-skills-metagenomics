---
name: bacterial-isolate-genomics-read-processing
description: Use when raw paired-end Illumina FASTQ reads from bacterial isolate genomes
  (e.g., CDC & FDA Antibiotic Resistance Isolate Bank isolates) need quality trimming,
  adapter removal, contaminant read removal, and merging of overlapping mates before
  targeted assembly or annotation.
when_to_use_negative:
- Input is already assembled contigs or a genome rather than raw short reads — the
  trimming/merging step targets unassembled Illumina FASTQ data
- Reads are long-read (e.g., PacBio/ONT) or non-Illumina; the workflow expects short
  paired-end genomic DNA reads (e.g., 90–226 bp in the article's datasets)
- Only protein-level Cas HMM screening is needed on pre-cleaned reads and no trimming/merging/contaminant
  removal is required — the --trim/--clean step is optional
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0622
tools:
- name: bbduk
  role: 'Implements the trimming and cleaning options: quality trimming, adapter removal,
    and removal of contaminating/reference-matching sequences on the short DNA sequencing
    reads'
  repo: https://jgi.doe.gov/data-and-tools/bbtools/
- name: bbmerge
  role: Merges overlapping paired-end reads during the --trim/--clean preprocessing
    step
  repo: https://jgi.doe.gov/data-and-tools/bbtools/
- name: CasCollect
  role: Pipeline wrapper exposing the optional read filtering step (--trim/--clean)
    ahead of seed generation, expansion, assembly, and annotation
  repo: https://github.com/sandialabs/CasCollect
- name: Seqtk
  role: Listed as a CasCollect dependency for read processing tasks (e.g., read stripping)
    within the pipeline's tooling
  repo: https://github.com/lh3/seqtk
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:df52ee044d1aa2579adeb1363c33db0cc9bb6f1e9f18beec6ed12f1f02b20912
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CRISPRCasFinder
  - CasCollect
  - HMMER
  - Seqtk
  - bbduk
  - bbmerge
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# bacterial-isolate-genomics-read-processing

## Summary

Quality trim, adapter removal, contaminant filtering, and overlap merging of raw paired-end Illumina reads from bacterial isolates (e.g., the CDC & FDA Antibiotic Resistance Isolate Bank P. aeruginosa panel) to prepare clean inputs for targeted CRISPR/Cas assembly. Implemented via the BBTools programs bbduk and bbmerge, exposed as CasCollect's optional --trim/--clean read-filtering step.

## When to use

Use when raw paired-end Illumina FASTQ reads from bacterial isolate genomes (e.g., CDC & FDA Antibiotic Resistance Isolate Bank isolates) need quality trimming, adapter removal, contaminant read removal, and merging of overlapping mates before targeted assembly or annotation.

## When NOT to use

- Input is already assembled contigs or a genome rather than raw short reads — the trimming/merging step targets unassembled Illumina FASTQ data
- Reads are long-read (e.g., PacBio/ONT) or non-Illumina; the workflow expects short paired-end genomic DNA reads (e.g., 90–226 bp in the article's datasets)
- Only protein-level Cas HMM screening is needed on pre-cleaned reads and no trimming/merging/contaminant removal is required — the --trim/--clean step is optional

## Inputs

- Raw paired-end Illumina FASTQ sequencing reads from bacterial isolate genomes (e.g., 66 P. aeruginosa clinical isolates from the CDC & FDA Antibiotic Resistance Isolate Bank)
- Optional contaminant reference FASTA (e.g., human genome via -ref; default: human genome)
- Optional user-defined CRISPR repeat FASTA for downstream DNA-mode seeding (e.g., Pseudomonas_aeruginosa_DK2.fas)

## Outputs

- Quality-trimmed, adapter-free, overlap-merged paired-end reads ready for seed generation
- Contaminant-depleted read sets when --clean with -ref is used
- Downstream CRISPR array and cas operon contigs annotated by CRISPRCasFinder per isolate

## How to apply

Run the trimming/cleaning step first in the CasCollect pipeline: pass --trim to quality-trim reads, remove adapters, and merge overlapping reads using bbduk and bbmerge from BBTools. If contaminating sequences must be removed, use --clean instead, supplying a user-defined contaminant reference via -ref file.fasta (default is the human genome). Rationale: cleaning the raw reads before seed generation, read subset expansion (e.g., -cycle 2 with 95% VSEARCH identity), SPAdes assembly, and CRISPRCasFinder annotation reduces spurious matches and improves targeted-assembly quality. CasCollect checks for required software (BBTools, Seqtk, FragGeneScan, HMMER, VSEARCH, SPAdes, CRISPRCasFinder) in the user's path before running; the Check.py script can report and auto-download missing pieces. In the P. aeruginosa case study, this preprocessing enabled cas operon and CRISPR array detection from unassembled reads that closely matched the complete assembly pipeline.

## Related tools

- **bbduk** (Implements the trimming and cleaning options: quality trimming, adapter removal, and removal of contaminating/reference-matching sequences on the short DNA sequencing reads) — https://jgi.doe.gov/data-and-tools/bbtools/
- **bbmerge** (Merges overlapping paired-end reads during the --trim/--clean preprocessing step) — https://jgi.doe.gov/data-and-tools/bbtools/
- **CasCollect** (Pipeline wrapper exposing the optional read filtering step (--trim/--clean) ahead of seed generation, expansion, assembly, and annotation) — https://github.com/sandialabs/CasCollect
- **Seqtk** (Listed as a CasCollect dependency for read processing tasks (e.g., read stripping) within the pipeline's tooling) — https://github.com/lh3/seqtk

## Examples

```
python CasCollect.py --trim -nucl -query Pseudomonas_aeruginosa_DK2.fas -cycle 2 -out out/ isolate_R1.fastq isolate_R2.fastq
```

## Evaluation signals

- Output reads no longer contain adapter sequences and low-quality ends have been trimmed (compare before/after with FastQC-style metrics)
- Overlapping mate pairs were merged by bbmerge (merged-read fraction reported)
- With --clean, reads matching the -ref contaminant reference (default: human genome) are absent from the output
- CasCollect's pre-run software check (or Check.py) reports all seven dependencies (BBTools, Seqtk, FragGeneScan, HMMER, VSEARCH, SPAdes, CRISPRCasFinder) present in the user's path
- Downstream validation: expanded read subsets assemble with SPAdes and CRISPRCasFinder identifies CRISPR arrays and cas operons across the 66-isolate P. aeruginosa panel, matching the complete-assembly benchmark with only minor single-strain differences

## Limitations

- Read filtering is an optional step of the five-step CasCollect pipeline; skipping --trim/--clean leaves raw reads unprocessed and may degrade seed matching
- --clean uses a default human genome contaminant reference; other contaminants require an explicit -ref FASTA
- The article provides no software version numbers or accession IDs for tools or datasets, complicating reproducibility
- In the 66-isolate P. aeruginosa case study, results differed from complete assembly for minor single-strain details (e.g., strain AR110 spacer count and strain AR356 cas operon)

## Evidence

- [methods] --trim              runs quality trim, remove adapters, merge overlapping reads: "--trim              runs quality trim, remove adapters, merge overlapping reads"
- [results] The trimming and cleaning options are implemented by bbduk on the short DNA sequencing reads: "The trimming and cleaning options are implemented by bbduk on the short DNA sequencing reads"
- [results] The CasCollect pipeline was created from five distinct and optional steps: (i) read filtering, (ii) seed generation, (iii) read subset expansion, (iv) assembly and (v) annotation: "The CasCollect pipeline was created from five distinct and optional steps: (i) read filtering, (ii) seed generation, (iii) read subset expansion, (iv) assembly and (v) annotation"
- [readme] CasCollect is a pipeline for the detection of Cas genes and CRISPR arrays from unassembled raw illumina sequencing reads.: "CasCollect is a pipeline for the detection of Cas genes and CRISPR arrays from unassembled raw illumina sequencing reads."
- [results] executing the two BBTools programs (https://jgi.doe.gov/data-and-tools/bbtools/) bbduk and bbmerge: "executing the two BBTools programs (https://jgi.doe.gov/data-and-tools/bbtools/) bbduk and bbmerge"
