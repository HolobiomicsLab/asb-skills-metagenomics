---
name: short-read-quality-trimming
description: Use when raw Illumina paired-end FASTQ reads must be quality-trimmed
  and adapter-stripped to remove low-quality sequence and adapters before targeted
  assembly or seed-read searching.
when_to_use_negative:
- Input is already trimmed, cleaned, or assembled contigs rather than raw short reads
  — re-running bbduk trimming adds runtime with little benefit
- Long-read or single-end-only data where bbmerge overlap merging of paired reads
  is not applicable
- A complete de novo assembly is the goal and trimming is handled by the assembler's
  own filtering options
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: bbduk
  role: Implements the trimming and cleaning options on short DNA sequencing reads
    (quality trim, adapter removal, contaminant removal)
  repo: https://jgi.doe.gov/data-and-tools/bbtools/
- name: BBTools
  role: Tool suite (bbduk and bbmerge) invoked by CasCollect's --trim/--clean flags
  repo: https://jgi.doe.gov/data-and-tools/bbtools/
- name: Seqtk
  role: Lightweight FASTQ/FASTA processing tool listed as a CasCollect dependency
    for read manipulation
  repo: https://github.com/lh3/seqtk
- name: CasCollect
  role: Pipeline wrapper whose --trim/--clean flags trigger the read filtering step
    before seed generation
  repo: https://github.com/sandialabs/CasCollect
provenance:
  source_task_ids:
  - task_005
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:7d505c4b9c859c9a4dc6978c7d5a7f1701b18d8d2313251acd47c6dc46d61e0d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BBTools
  - CRISPRCasFinder
  - CasCollect
  - HMMER
  - PRICE
  - Seqtk
  - bbduk
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# short-read-quality-trimming

## Summary

The CasCollect read filtering step: quality trimming, adapter removal, optional contaminant-read removal and merging of overlapping paired-end reads using BBTools (bbduk/bbmerge), run with the --trim or --clean flag before seed generation.

## When to use

Use when raw Illumina paired-end FASTQ reads must be quality-trimmed and adapter-stripped to remove low-quality sequence and adapters before targeted assembly or seed-read searching.

## When NOT to use

- Input is already trimmed, cleaned, or assembled contigs rather than raw short reads — re-running bbduk trimming adds runtime with little benefit
- Long-read or single-end-only data where bbmerge overlap merging of paired reads is not applicable
- A complete de novo assembly is the goal and trimming is handled by the assembler's own filtering options

## Inputs

- Raw Illumina paired-end FASTQ sequencing read files (forward/reverse), e.g. isolate genomes or metagenomes
- Optional user-defined contaminant reference FASTA via -ref file.fasta (default: human genome)

## Outputs

- Quality-trimmed, adapter-removed read set, with overlapping read pairs merged, fed into the seed-generation step
- Optionally, a read set depleted of contaminant-matching reads (when --clean/-ref is used)

## How to apply

Enable the filtering step of CasCollect with --trim, which runs quality trim, removes adapters, and merges overlapping reads; use --clean instead when contaminating reads (default: human genome, or a user-defined -ref file.fasta) should also be removed. The trimming and cleaning options are implemented by bbduk, with read merging performed by bbmerge from BBTools. Run this step before seed generation (HMMER/vsearch) because clean, merged reads improve both protein seed matching via FragGeneScan/HMMER against 120 Cas protein HMMs and DNA-mode vsearch matching against CRISPR repeat queries. Treat this step as optional — the CasCollect pipeline's five steps (read filtering, seed generation, read subset expansion, assembly, annotation) are selectable. Check that required tools are in the user's path beforehand; the Check.py script can report and auto-download missing software.

## Related tools

- **bbduk** (Implements the trimming and cleaning options on short DNA sequencing reads (quality trim, adapter removal, contaminant removal)) — https://jgi.doe.gov/data-and-tools/bbtools/
- **BBTools** (Tool suite (bbduk and bbmerge) invoked by CasCollect's --trim/--clean flags) — https://jgi.doe.gov/data-and-tools/bbtools/
- **Seqtk** (Lightweight FASTQ/FASTA processing tool listed as a CasCollect dependency for read manipulation) — https://github.com/lh3/seqtk
- **CasCollect** (Pipeline wrapper whose --trim/--clean flags trigger the read filtering step before seed generation) — https://github.com/sandialabs/CasCollect

## Examples

```
python CasCollect.py --trim -fwd reads_R1.fq -rev reads_R2.fq -out out_dir
```

## Evaluation signals

- CasCollect's pre-run check (or Check.py) reports BBTools and all other required software present in the user's path
- Downstream seed generation proceeds: CasCollect proceeds from read filtering to seed generation without failing on empty or malformed reads
- Adapters and low-quality tails are removed while reads remain in FASTQ paired structure, with overlapping pairs merged by bbmerge
- With --clean, contaminant-matching reads (default human genome or user -ref FASTA) are depleted from the read set
- Pipeline runtime stays consistent with the benchmarked behavior (e.g. isolate runs of ~20–50 min total with trimming enabled)

## Limitations

- Contaminant removal defaults to the human genome; users must supply a custom -ref file.fasta for other contaminants, and off-target removal risks discarding biologically relevant reads
- The filtering step is optional and its outcome is hardware/tool-path dependent — CasCollect aborts or warns if BBTools, Seqtk, FragGeneScan, HMMER, VSEARCH, SPAdes, or CRISPRCasFinder are missing from the path
- No software version numbers are given for bbduk/bbmerge in the article, so exact trimming parameter behavior may vary across BBTools releases

## Evidence

- [results] Trimming removes adapters from reads, while the cleaning option removes adapters and includes the removal of defined 'contaminating' sequences: [evidence span withheld — anchor status: not_found]
- [results] bbduk implements trimming/cleaning: "The trimming and cleaning options are implemented by bbduk on the short DNA sequencing reads"
- [methods] --trim flag definition: "--trim              runs quality trim, remove adapters, merge overlapping"
- [results] BBTools suite used by CasCollect: "executing the two BBTools programs (https://jgi.doe.gov/data-and-tools/bbtools/) bbduk and bbmerge"
- [results] Pipeline step structure: "The CasCollect pipeline was created from five distinct and optional steps: (i) read filtering, (ii) seed generation"
