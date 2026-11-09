---
name: short-read-to-assembly-mapping
description: Use when you have FASTA reads flagged by pal-MEM as containing inverted
  repeats and per-sample assemblies, and need to anchor those reads genomically and
  filter candidate ITRs by IS-length proximity before clustering.
when_to_use_negative:
- Input reads have not been pre-processed through convertToFasta and pal-MEM, so no
  ITR-containing reads or LCoord/RCoord coordinates exist to filter
- The goal is to discover ISs lacking ITRs — Palidis is only equipped for ISs with
  ITRs, so mapping-and-proximity filtering cannot recover them
- Suspected insertions are lengthened by nested insertion events extending beyond
  the 500–3000 bp proximity range, which this filter will incorrectly discard
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Bowtie2
  role: Aligns pal-MEM repeat-containing reads against their associated assemblies
    in paired/single-end short-read mode to produce the alignments the proximity filter
    runs on
- name: pal-MEM
  role: Upstream step producing the repeat-containing reads and LCoord/RCoord ITR
    coordinates that are mapped and filtered
  repo: https://github.com/blue-moon22/pal-MEM
- name: Palidis (palidis4)
  role: Pipeline orchestrating the mapping (mapreads) and distance-filter (getCandidateITRs)
    steps
  repo: https://github.com/blue-moon22/palidis4
- name: Python
  role: Filter script retains only mapped reads whose repeat positions lie within
    the minimum-to-maximum IS length interval
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:24ba4c22b32ceaa544d63c9d35d9ffa6ac14025c16989ce5c151057682332616
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/pal-MEM
  tools:
  - Bowtie2
  - Palidis
  - Palidis (palidis4)
  - Python
  - pal-MEM
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# short-read-to-assembly-mapping

## Summary

Maps repeat-containing metagenomic short reads (output of pal-MEM) back to their associated sample assemblies with Bowtie2, then applies a distance filter so only mapped reads whose ITR LCoord/RCoord positions fall within the user-defined minimum-to-maximum IS length interval (e.g. the 500–3000 bp proximity range) are retained as candidate ITR reads.

## When to use

Use when you have FASTA reads flagged by pal-MEM as containing inverted repeats and per-sample assemblies, and need to anchor those reads genomically and filter candidate ITRs by IS-length proximity before clustering.

## When NOT to use

- Input reads have not been pre-processed through convertToFasta and pal-MEM, so no ITR-containing reads or LCoord/RCoord coordinates exist to filter
- The goal is to discover ISs lacking ITRs — Palidis is only equipped for ISs with ITRs, so mapping-and-proximity filtering cannot recover them
- Suspected insertions are lengthened by nested insertion events extending beyond the 500–3000 bp proximity range, which this filter will incorrectly discard

## Inputs

- pal-MEM _IR fasta files (paired _IR_1.fasta/_IR_2.fasta or single _IR.fasta) containing reads with inverted repeats and LCoord/RCoord ITR coordinates in sequence headers
- Per-sample genome assemblies (FASTA) used as Bowtie2 reference
- User-defined minimum and maximum IS length thresholds (e.g. 500–3000 bp)

## Outputs

- Mapped read alignments (BAM/SAM) of repeat-containing reads against assemblies
- Filtered candidate ITR reads (candidate_IS_reads.fastq) passing the IS-length proximity filter
- IS candidate pool for downstream CD-HIT-EST clustering

## How to apply

After pal-MEM outputs reads containing inverted terminal repeats (_IR fasta files with LCoord/RCoord ITR coordinates in the read headers), align these repeat-containing reads to their associated sample assemblies using Bowtie2 in paired-end or single-end short-read mode. Run the repository's Python filter script (the getCandidateITRs step) on the resulting alignments, retaining only mapped reads whose repeat coordinates (LCoord/RCoord) span a distance within the user-defined minimum-to-maximum IS length interval — the paper uses a 500–3000 bp proximity range. Write the retained reads to candidate_IS_reads.fastq for downstream CD-HIT-EST clustering. The rationale is that a true IS has its two ITRs flanking an insert of plausible IS length (here 500–3000 bp); reads whose repeat positions fall outside this range are likely artifacts or nested/lengthened insertions and are discarded.

## Related tools

- **Bowtie2** (Aligns pal-MEM repeat-containing reads against their associated assemblies in paired/single-end short-read mode to produce the alignments the proximity filter runs on)
- **pal-MEM** (Upstream step producing the repeat-containing reads and LCoord/RCoord ITR coordinates that are mapped and filtered) — https://github.com/blue-moon22/pal-MEM
- **Palidis (palidis4)** (Pipeline orchestrating the mapping (mapreads) and distance-filter (getCandidateITRs) steps) — https://github.com/blue-moon22/palidis4
- **Python** (Filter script retains only mapped reads whose repeat positions lie within the minimum-to-maximum IS length interval)

## Examples

```
palidis --mapreads <sample_assembly.fasta> <palmem_IR_1.fasta> <palmem_IR_2.fasta> && palidis getCandidateITRs --min_IS 500 --max_IS 3000 --out candidate_IS_reads.fastq
```

## Evaluation signals

- Every read written to candidate_IS_reads.fastq is present in the Bowtie2 alignments (i.e. mapped) and has LCoord/RCoord-derived repeat span within the 500–3000 bp interval
- Reads with ITR coordinates outside the user-defined min–max IS length are absent from the candidate output
- The _IR.tab input headers parse correctly: LCoord and RCoord fields are extracted and their difference used as the distance metric
- Downstream CD-HIT-EST clustering receives non-empty candidate_IS_reads.fastq per sample, consistent with pal-MEM output volume

## Limitations

- ISs flanked by ITRs beyond the 500 to 3000 bp proximity range (e.g. regions lengthened by nested insertion events) are missed by Palidis
- Maximum IS length may need to be increased to capture nested/lengthened insertions
- Lowering the minimum ITR length threshold and k-mer length (pal-MEM default -k 15) makes the upstream step more computationally intensive on complex mixed microbial genomes

## Evidence

- [methods] In Palidis Step 2, repeat-containing reads are mapped with Bowtie2 against their associated assemblies: [evidence span withheld — anchor status: not_found]
- [readme] Map reads against assemblies using Bowtie2: "3. Map reads against assemblies using Bowtie2 [`filterContigs` `buildDB` `mapreads`] 4. Get candidate ITRs by distance filters [`getCandidateITRs`]"
- [discussion] ITRs beyond the proximity range are missed: [evidence span withheld — anchor status: not_found]
- [readme] pal-MEM output headers contain ITR coordinates: [evidence span withheld — anchor status: external_doc]
