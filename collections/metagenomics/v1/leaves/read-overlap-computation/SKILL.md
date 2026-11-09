---
name: read-overlap-computation
description: Use when a short-read metagenome assembly or clustering pipeline needs
  pairwise read overlaps with identity and overlap-length scores, e.g. to build overlap
  graphs for strain-aware (haplotype-aware) OLC assembly.
when_to_use_negative:
- Input reads are long third-generation reads with high error rate — the paper notes
  third-generation error rates introduce unresolved issues and the method targets
  NGS short reads
- You need full-length strain-specific contigs spanning whole genomes — the paper
  states short Illumina read length constrains contig length
- A de Bruijn graph assembler (SPAdes, MEGAHIT, IDBA-UD, Minia3) is already required,
  since this skill implements the OLC overlap step of a strain-aware assembler rather
  than generic metagenome assembly
edam_operation: http://edamontology.org/operation_3230
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: minimap2
  role: performs all-vs-all read-to-read approximate overlap alignment, producing
    the CIGAR strings from which identity and overlap length are parsed
  repo: https://github.com/lh3/minimap2
- name: StrainXpress
  role: wrapper pipeline that runs the overlap computation, applies the 30 bp / 0.9
    identity filters, and scores overlaps with D = 0.9*i + 0.1*o/((r1+r2)/2)
  repo: https://github.com/HaploKit/StrainXpress
- name: HaploConduct
  role: upstream overlap-graph ploidy-aware assembly strategy whose FM-index-based
    overlap detection is replaced by Minimap2 in StrainXpress local assembly
  repo: https://github.com/HaploConduct/HaploConduct
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:8792ee54e1589d97a35552d891c411b2f37ca284d4d9d9867b4b256ffd544f81
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - HaploConduct
  - Minimap2
  - Pandas
  - Python
  - StrainXpress
  - minimap2
  edam_operation: http://edamontology.org/operation_3230
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# read-overlap-computation

## Summary

Compute suffix-prefix read-to-read overlaps and weighted overlap scores as in StrainXpress, using Minimap2 all-vs-all alignment of interleaved NGS FASTQ reads and scoring each overlap from its CIGAR string as D = 0.9*i + 0.1*o/((r1+r2)/2).

## When to use

Use when a short-read metagenome assembly or clustering pipeline needs pairwise read overlaps with identity and overlap-length scores, e.g. to build overlap graphs for strain-aware (haplotype-aware) OLC assembly.

## When NOT to use

- Input reads are long third-generation reads with high error rate — the paper notes third-generation error rates introduce unresolved issues and the method targets NGS short reads
- You need full-length strain-specific contigs spanning whole genomes — the paper states short Illumina read length constrains contig length
- A de Bruijn graph assembler (SPAdes, MEGAHIT, IDBA-UD, Minia3) is already required, since this skill implements the OLC overlap step of a strain-aware assembler rather than generic metagenome assembly

## Inputs

- Interleaved FASTQ file of quality-controlled NGS (Illumina) reads
- Minimap2 all-vs-all overlap alignments in PAF format with CIGAR strings (cg tag)

## Outputs

- overlap_scores.tsv: TSV with columns for the two read identifiers, overlap length, identity, and overlap score D

## How to apply

Load quality-controlled reads as interleaved FASTQ and run Minimap2 in all-vs-all overlap mode so every read is aligned against every other read, retaining the CIGAR string for each alignment. Parse each CIGAR to extract the alignment identity (i) and overlap length (o) between the two reads; score the overlap as D = 0.9*i + 0.1*o/((r1+r2)/2), where r1 and r2 are the read lengths. Filter out overlaps shorter than 30 bp or with identity below 0.9. Write surviving read pairs to a TSV with columns for the two read identifiers, overlap length, identity, and score D. The rationale is that scoring directly from the Minimap2 CIGAR avoids assessing overlap quality across the full alignment, and using Minimap2 (instead of FM-index-based overlap detection from the HaploConduct/SAVAGE template) keeps cluster sizes small while making local assembly about 8 times faster.

## Related tools

- **minimap2** (performs all-vs-all read-to-read approximate overlap alignment, producing the CIGAR strings from which identity and overlap length are parsed) — https://github.com/lh3/minimap2
- **StrainXpress** (wrapper pipeline that runs the overlap computation, applies the 30 bp / 0.9 identity filters, and scores overlaps with D = 0.9*i + 0.1*o/((r1+r2)/2)) — https://github.com/HaploKit/StrainXpress
- **HaploConduct** (upstream overlap-graph ploidy-aware assembly strategy whose FM-index-based overlap detection is replaced by Minimap2 in StrainXpress local assembly) — https://github.com/HaploConduct/HaploConduct

## Examples

```
python ../scripts/strainxpress.py -fq all_reads.fq
```

## Evaluation signals

- overlap_scores.tsv exists and is well-formed TSV with columns for two read identifiers, overlap length, identity, and score D
- Every row in overlap_scores.tsv has overlap length >= 30 bp and identity >= 0.9
- Every overlap score D falls in the range [0, 1], consistent with the weighted combination D = 0.9*i + 0.1*o/((r1+r2)/2)
- Each row contains a valid read pair drawn from the input interleaved FASTQ
- Downstream per-cluster overlap graphs in local assembly reproduce strain-aware haplotigs, reported as 8 times faster than the original HaploConduct template

## Limitations

- Short length of Illumina-type reads constrains contig length, preventing computation of strain-specific contigs that span entire genomes
- High error rate of third-generation sequencing reads introduces issues that remain unresolved by this overlap-based approach
- Minimap2 index parameters such as -k, -w, -H and -I cannot be changed once an index is built, so different data types require separate indexes

## Evidence

- [results] weighted overlap score from CIGAR: [evidence span withheld — anchor status: not_found]
- [results] minimap2 replaces FM-index in local assembly: [evidence span withheld — anchor status: not_found]
- [results] local assembly speedup: [evidence span withheld — anchor status: not_found]
- [readme] overlaps between long reads use case: [evidence span withheld — anchor status: external_doc]
- [readme] cigar output option: "You can ask minimap2 to generate CIGAR at the `cg` tag of PAF with:
sh
minimap2 -c ref.fa query.fq > alignment.paf"
- [readme] input format interleaved FASTQ: "The input file must be interleaved FASTQ and format like below:"
