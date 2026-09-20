---
name: minimap2-reference-alignment
description: Use when assembler-generated contigs must be mapped to known reference
  replicons (multi-FASTA passed via --reference) to compute reference-based assembly
  quality metrics like LSA.
when_to_use_negative:
- No reference replicon sequences are available — reference-free/global contig metrics
  should be used instead
- Input is raw reads rather than assembler-generated contigs — this skill aligns FS
  contigs only
- Contigs shorter than the --minLength threshold (default 1000 bp) — they are excluded
  before alignment
edam_operation: http://edamontology.org/operation_0526
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: minimap2
  role: Aligner used (v2.22) to index reference replicons and map FS contigs, producing
    reference-dependent PAF alignments
- name: BBTools
  role: Pre-alignment filter (v38.44) that removes contigs below the --minLength threshold
    to define the FS set
- name: LMAS
  role: Nextflow workflow orchestrating the filtering, minimap2 alignment, and downstream
    LSA metric computation
  repo: https://github.com/B-UMMI/LMAS
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giac122
    title: 'LMAS: evaluating metagenomic short <i>de novo</i> assembly methods through
      defined communities'
  source_content_hash: sha256:f96f72207c364e93d149a93a98e0ad74d8fc5064d08eaf2494e42b223e7c7231
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/LMAS
  tools:
  - BBTools
  - LMAS
  - minimap2
  edam_operation: http://edamontology.org/operation_0526
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# minimap2-reference-alignment

## Summary

Align BBTools-filtered (FS) assembler contigs against reference replicons with minimap2 (v2.22) to produce per-sample PAF alignments that underpin LMAS reference-dependent metrics such as the LSA (longest single alignment) fraction.

## When to use

Use when assembler-generated contigs must be mapped to known reference replicons (multi-FASTA passed via --reference) to compute reference-based assembly quality metrics like LSA.

## When NOT to use

- No reference replicon sequences are available — reference-free/global contig metrics should be used instead
- Input is raw reads rather than assembler-generated contigs — this skill aligns FS contigs only
- Contigs shorter than the --minLength threshold (default 1000 bp) — they are excluded before alignment

## Inputs

- Contig FASTA files produced by each assembler in results/assembly/<assembler>/
- Single multi-FASTA of reference replicons passed via the --reference parameter
- --minLength parameter setting (default: 1000 bp)

## Outputs

- One minimap2 alignment file (PAF) per sample mapping FS contigs to reference replicons
- lsa_metrics.csv with one LSA entry per assembler-reference pair

## How to apply

First apply the BBTools (v38.44) length filter defined by the --minLength parameter (default 1000 bp) to keep only the filtered set (FS) contigs from results/assembly/<assembler>/. Index the single multi-FASTA of reference replicons and align the FS contigs against it with minimap2 (v2.22), writing one alignment file (PAF) per sample. Downstream, for each assembler-reference pair, extract all single contiguous alignments, identify the longest, and normalize by the reference replicon length to get the LSA fraction bounded in (0, 1]. Validate that a valid alignment file exists for every sample and that all retained contigs meet the --minLength threshold.

## Related tools

- **minimap2** (Aligner used (v2.22) to index reference replicons and map FS contigs, producing reference-dependent PAF alignments)
- **BBTools** (Pre-alignment filter (v38.44) that removes contigs below the --minLength threshold to define the FS set)
- **LMAS** (Nextflow workflow orchestrating the filtering, minimap2 alignment, and downstream LSA metric computation) — https://github.com/B-UMMI/LMAS

## Examples

```
minimap2 -x asm5 reference.fasta results/assembly/megahit/contigs.fasta > results/alignment/megahit/sample1.paf
```

## Evaluation signals

- A valid alignment file exists for every sample-assembler combination
- Every contig appearing in the alignment meets the --minLength threshold (default 1000 bp)
- lsa_metrics.csv contains exactly one LSA entry per assembler-reference pair
- All LSA values fall within (0, 1], consistent with their definition as a fraction of the reference replicon length
- Alignment records are in PAF format from minimap2 runs on FS contigs

## Limitations

- LSA and related reference-based metrics depend on the quality/completeness of the supplied reference replicons
- Contigs below --minLength (default 1000 bp) are excluded, so short-contig signal is lost before alignment
- The --mapped_reads_threshold parameter controls the minimum percentage of a read aligning to a contig to be considered mapped

## Evidence

- [methods] Contigs exceeding the user-defined minimum sequence length are filtered using BBTools (version 38.44) to obtain the filtered set (FS): [evidence span withheld — anchor status: not_found]
- [methods] Index the reference replicons and align the FS contigs against them using minimap2 (v2.22) to obtain reference-dependent mappings: [evidence span withheld — anchor status: not_found]
- [methods] LMAS computes the LSA metric per reference as the fraction of the longest single alignment between a contig and the reference: [evidence span withheld — anchor status: not_found]
- [intro] MetaQUAST [11], a modification of QUAST, extends the original software by performing assembly evaluation based on aligning contigs to a reference: "MetaQUAST [11], a modification of QUAST, extends the original software by performing assembly evaluation based on aligning contigs to a reference"
