---
name: python-script-based-alignment-filtering
description: Use when you have FASTA/FASTQ reads already flagged as containing inverted
  terminal repeats (with LCoord/RCoord ITR positions from pal-MEM) and need to keep
  only those whose flanking genomic context maps to assemblies within the expected
  IS length range (e.g. 500-3000 bp).
when_to_use_negative:
- Input reads have not yet been screened for inverted repeats by pal-MEM (no LCoord/RCoord
  ITR coordinates available in read names)
- Target insertion sequences lack ITRs or have ITRs shorter than the tool's minimum
  (e.g. ITRs as short as ten nucleotides are missed at default settings)
- Regions whose ITRs are separated beyond the 500-3000 bp proximity range, e.g. lengthened
  by nested insertion events — these will be incorrectly filtered out
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3697
tools:
- name: Palidis
  role: Pipeline whose getCandidateITRs step applies the Python distance-filter script
    to Bowtie2 alignments to produce candidate ITR reads
  repo: https://github.com/blue-moon22/palidis4
- name: Bowtie2
  role: Maps the pal-MEM repeat-containing reads (paired/single-end short-read mode)
    against their associated assemblies to produce mapped read alignments
- name: pal-MEM
  role: Upstream step that identifies the inverted terminal repeats and their LCoord/RCoord
    positions in metagenomic read files
  repo: https://github.com/blue-moon22/pal-MEM
- name: CD-HIT-EST
  role: Downstream step that clusters the retained candidate_IS_reads.fastq sequences
    into non-redundant IS candidates
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:1d6c74c8fab9495cfecb2908a2c445289ba517415e8af7d98754b0575a341ddd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/palidis4
  tools:
  - Bowtie2
  - CD-HIT-EST
  - Palidis
  - Python
  - pal-MEM
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# python-script-based-alignment-filtering

## Summary

Palidis Step 2-style proximity filter: map repeat-containing reads (output of pal-MEM) to their associated assemblies with Bowtie2, then apply a Python filter script that retains only mapped reads whose ITR repeat positions fall within the user-defined minimum-to-maximum insertion sequence (IS) length interval (default 500-3000 bp), producing candidate ITR reads for downstream clustering.

## When to use

Use when you have FASTA/FASTQ reads already flagged as containing inverted terminal repeats (with LCoord/RCoord ITR positions from pal-MEM) and need to keep only those whose flanking genomic context maps to assemblies within the expected IS length range (e.g. 500-3000 bp).

## When NOT to use

- Input reads have not yet been screened for inverted repeats by pal-MEM (no LCoord/RCoord ITR coordinates available in read names)
- Target insertion sequences lack ITRs or have ITRs shorter than the tool's minimum (e.g. ITRs as short as ten nucleotides are missed at default settings)
- Regions whose ITRs are separated beyond the 500-3000 bp proximity range, e.g. lengthened by nested insertion events — these will be incorrectly filtered out

## Inputs

- Repeat-containing reads from the pal-MEM step (FASTA files with suffix _IR.fasta/_IR_1.fasta/_IR_2.fasta and _discord_non_IR*.fasta), with LCoord/RCoord ITR coordinates encoded in read names
- Associated metagenomic sample assemblies (FASTA) used as Bowtie2 reference
- User-defined minimum and maximum IS length thresholds (e.g. 500 and 3000 bp)

## Outputs

- candidate_IS_reads.fastq: retained mapped reads whose repeat positions fall within the min-to-max IS length interval
- Candidate ITR reads suitable for pooling and CD-HIT-EST clustering into non-redundant IS candidates

## How to apply

First, take the repeat-containing read pairs output by the pal-MEM step of Palidis (the _IR.fasta files and their paired non-IR mates) and align them to their associated sample assemblies using Bowtie2 in paired-end (or single-end) short-read mode. Then run the repository's Python filter script (Palidis' getCandidateITRs step) on the resulting alignments: for each mapped read, parse the LCoord/RCoord ITR coordinates recorded in the read name and retain the read only if the repeat positions lie between the user-defined minimum and maximum IS lengths — the 500-3000 bp proximity range used for Palidis candidates. Reads passing this distance filter are written to candidate_IS_reads.fastq, which serves as the pooled input for CD-HIT-EST clustering of insertion sequence candidates. The rationale is that a genuine IS places its ITRs within a bounded genomic span; reads whose repeat separation falls outside this range (e.g. lengthened by nested insertion events) are discarded as unlikely ITR pairs.

## Related tools

- **Palidis** (Pipeline whose getCandidateITRs step applies the Python distance-filter script to Bowtie2 alignments to produce candidate ITR reads) — https://github.com/blue-moon22/palidis4
- **Bowtie2** (Maps the pal-MEM repeat-containing reads (paired/single-end short-read mode) against their associated assemblies to produce mapped read alignments)
- **pal-MEM** (Upstream step that identifies the inverted terminal repeats and their LCoord/RCoord positions in metagenomic read files) — https://github.com/blue-moon22/pal-MEM
- **CD-HIT-EST** (Downstream step that clusters the retained candidate_IS_reads.fastq sequences into non-redundant IS candidates)

## Evaluation signals

- Every read in candidate_IS_reads.fastq is a mapped (aligned) read and its parsed LCoord/RCoord repeat separation lies within the configured min-max IS length interval (e.g. 500-3000 bp)
- Discordant/read pairs whose mates failed the proximity filter are absent from the candidate output but present in the pal-MEM _IR inputs, confirming filtering occurred
- Bowtie2 alignment completed with a sensible mapping rate for short reads against the associated assemblies (no empty SAM/BAM output)
- Downstream CD-HIT-EST clustering runs on the candidate file and yields clusters that survive the subsequent cluster-concordance (getITRs) step
- Output is in FASTQ format ready for pooling across samples, per the Palidis workflow

## Limitations

- Insertion sequences flanked by ITRs beyond the 500-3000 bp proximity range (e.g. regions lengthened by nested insertion events) are missed by the distance filter
- The filter inherits Palidis' dependence on ITR-containing reads: ISs lacking ITRs, or with ITRs shorter than the configured minimum (default settings miss ITRs as short as ten nucleotides), are not captured
- Tightening the thresholds (lower minimum ITR length, smaller k-mer length upstream) increases computational cost, especially with complex mixed microbial genomes

## Evidence

- [methods] Bowtie2 mapping + Python proximity filter (task card): [evidence span withheld — anchor status: not_found]
- [readme] palidis4 pipeline steps: "3. Map reads against assemblies using Bowtie2 [`filterContigs` `buildDB` `mapreads`]
4. Get candidate ITRs by distance filters [`getCandidateITRs`]"
- [discussion] Proximity range limitation: [evidence span withheld — anchor status: not_found]
- [discussion] ITR length floor: "the catalogue currently contains ISs with ITRs that are 25 or greater nucleotides in length as generated by Palidis, although ITRs can be as short as ten nucleotides in length"
- [readme] ITR coordinates in read names: [evidence span withheld — anchor status: external_doc]
