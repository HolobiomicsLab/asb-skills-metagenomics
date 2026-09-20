---
name: metagenomic-is-candidate-screening
description: Use when paired-end short-read metagenomic reads have already been screened
  for inverted repeats by pal-MEM (LCoord/RCoord annotated FASTA) and sample assemblies
  exist, and you need proximity-filtered candidate ITR reads.
when_to_use_negative:
- You are looking for insertion sequences lacking inverted terminal repeats — Palidis
  is only equipped with discovering ISs with ITRs
- The repeat-containing reads have not yet been identified (no pal-MEM _IR output)
  — run the maximal exact matching step first
- Expected ISs may be longer than the proximity range, e.g. regions lengthened by
  nested insertion events outside 500–3000 bp
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: pal-MEM
  role: Upstream maximal exact matching step that identifies reads containing inverted
    terminal repeats (IRs) and annotates LCoord/RCoord coordinates before mapping
  repo: https://github.com/blue-moon22/pal-MEM
- name: Bowtie2
  role: Aligns repeat-containing reads (plus discordant pairs) against sample assemblies
    in paired/single-end short-read mode to provide genomic context for filtering
- name: Palidis (palidis4)
  role: Pipeline wrapper whose filterContigs/buildDB/mapreads and getCandidateITRs
    steps perform the mapping and IS-length proximity filtering
  repo: https://github.com/blue-moon22/palidis4
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:a2482de3934b58bff41368aefc1fa8673be51828873fe929c1cde7d10ad074e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/pal-MEM
  tools:
  - Bowtie2
  - Palidis
  - Palidis (palidis4)
  - pal-MEM
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metagenomic-is-candidate-screening

## Summary

A Palidis pipeline step that maps pal-MEM repeat-containing reads back to sample assemblies with Bowtie2 and retains reads whose inverted terminal repeat (ITR) coordinates fall within a user-defined minimum-to-maximum IS length interval (e.g. 500–3000 bp), yielding candidate insertion sequence (IS) reads for downstream clustering.

## When to use

Use when paired-end short-read metagenomic reads have already been screened for inverted repeats by pal-MEM (LCoord/RCoord annotated FASTA) and sample assemblies exist, and you need proximity-filtered candidate ITR reads.

## When NOT to use

- You are looking for insertion sequences lacking inverted terminal repeats — Palidis is only equipped with discovering ISs with ITRs
- The repeat-containing reads have not yet been identified (no pal-MEM _IR output) — run the maximal exact matching step first
- Expected ISs may be longer than the proximity range, e.g. regions lengthened by nested insertion events outside 500–3000 bp

## Inputs

- pal-MEM output FASTA files of inverted-repeat-containing reads (_IR.fasta / _IR_1.fasta, _IR_2.fasta) with LCoord/RCoord ITR coordinates in sequence headers
- discordant non-IR paired reads (_discord_non_IR.fasta) from pal-MEM
- Sample assembly contigs/sequences in FASTA format for Bowtie2 index building
- User-defined minimum and maximum IS length (proximity range, e.g. 500–3000 bp)

## Outputs

- Mapped read alignments (Bowtie2 SAM/BAM) of IR-containing reads against sample assemblies
- candidate_IS_reads.fastq — retained candidate ITR reads passing the proximity filter, for pooling and CD-HIT-EST clustering

## How to apply

First run pal-MEM on FASTQ.GZ/FASTA reads (converted via convertToFasta) to obtain _IR.fasta files containing reads with inverted repeats annotated with LCoord/RCoord. Build Bowtie2 indexes from each sample's assembly (buildDB step) and align the IR-containing reads plus their discordant pairs in paired- or single-end short-read mode (mapreads step). Then run the Palidis filter/getCandidateITRs step, which parses the alignments and retains only mapped reads whose repeat positions (LCoord to RCoord span) fit between the user-specified minimum and maximum IS length — the paper's default proximity range is 500–3000 bp. Retained reads are written to candidate_IS_reads.fastq for pooling and CD-HIT-EST clustering in subsequent Palidis steps. Rationale: mapping to the assembly confirms the repeat-bearing read has genomic context nearby, while the distance filter ensures the flanking regions are consistent with a complete insertion sequence rather than an unrelated repeat.

## Related tools

- **pal-MEM** (Upstream maximal exact matching step that identifies reads containing inverted terminal repeats (IRs) and annotates LCoord/RCoord coordinates before mapping) — https://github.com/blue-moon22/pal-MEM
- **Bowtie2** (Aligns repeat-containing reads (plus discordant pairs) against sample assemblies in paired/single-end short-read mode to provide genomic context for filtering)
- **Palidis (palidis4)** (Pipeline wrapper whose filterContigs/buildDB/mapreads and getCandidateITRs steps perform the mapping and IS-length proximity filtering) — https://github.com/blue-moon22/palidis4

## Examples

```
bowtie2 -x sample_assembly -1 reads_IR_1.fasta -2 reads_IR_2.fasta -S mapped.sam
```

## Evaluation signals

- Retained reads' ITR spans (RCoord − LCoord) all fall within the configured minimum-to-maximum IS length interval (e.g. 500–3000 bp)
- Every entry in candidate_IS_reads.fastq corresponds to a read that mapped to its sample assembly under Bowtie2
- Sequence headers of retained reads retain the pal-MEM LCoord/RCoord annotation for downstream clustering
- No unmapped reads survive the filter; only mapped candidates are pooled for CD-HIT-EST clustering
- Adjusting the maximum IS length upward should recover additional candidates for lengthened/nested insertions (a documented behavior)

## Limitations

- ISs flanked by ITRs beyond the 500–3000 bp proximity range, e.g. regions lengthened by nested insertion events, are missed by Palidis
- Palidis is only equipped with discovering ISs with ITRs; ISC contains only common DDE-type ISs with ITRs (25+ nucleotides at default settings), although ITRs can be as short as ten nucleotides
- Lower minimum ITR length thresholds and smaller k-mer lengths (pal-MEM defaults: -l 24, -m 100, -k 15) make the workflow more computationally intensive, especially for complex mixed microbial genomes

## Evidence

- [readme] Palidis pipeline steps 3–4: "3. Map reads against assemblies using Bowtie2 [`filterContigs` `buildDB` `mapreads`]
4. Get candidate ITRs by distance filters [`getCandidateITRs`]"
- [readme] pal-MEM IR output annotation: [evidence span withheld — anchor status: external_doc]
- [discussion] Proximity range miss limitation: [evidence span withheld — anchor status: not_found]
- [discussion] ITR-only scope: "Palidis is currently only equipped with discovering ISs with ITRs."
