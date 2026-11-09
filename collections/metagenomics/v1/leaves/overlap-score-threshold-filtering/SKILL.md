---
name: overlap-score-threshold-filtering
description: Use when quality-controlled NGS reads must be linked by suffix-prefix
  overlaps for OLC assembly, and each Minimap2-derived overlap needs a weighted identity/length
  score plus a length and identity cutoff.
when_to_use_negative:
- When reads are long, high-error-rate third-generation reads (e.g. PacBio CLR or
  noisy ONT) — the paper notes the high error rate of third-generation reads still
  introduces unresolved issues
- When the goal is generic read mapping to a reference rather than constructing read-overlap
  edges for strain-aware de novo assembly
- When overlaps are already scored by a downstream tool with its own scoring scheme
  (e.g. SAVAGE's statistically calibrated overlap grouping in HaploConduct)
edam_operation: http://edamontology.org/operation_0292
edam_topics: []
tools:
- name: Minimap2
  role: Performs all-vs-all suffix-prefix overlap detection between reads and emits
    the CIGAR strings from which identity and overlap length are parsed
  repo: https://github.com/lh3/minimap2
- name: StrainXpress
  role: Implements the weighted overlap scoring formula and the 30 bp / 0.9 identity
    filtering thresholds as part of its read clustering preprocessing
  repo: https://github.com/HaploKit/StrainXpress
- name: Pandas
  role: Used within the StrainXpress pipeline to assemble and write the filtered overlap
    score TSV
  repo: https://github.com/kangxiongbin/StrainXpress
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:f6d2a0e3bedea52a04b18209bac559560525cfa80d00817b5837006607e82d9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - Minimap2
  - Pandas
  - Python
  - StrainXpress
  edam_operation: http://edamontology.org/operation_0292
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# overlap-score-threshold-filtering

## Summary

Computes StrainXpress overlap scores D = 0.9*i + 0.1*o/((r1+r2)/2) from Minimap2 CIGAR strings for suffix-prefix read overlaps, then filters overlaps by length (>= 30 bp) and identity (>= 0.9). This converts raw all-vs-all alignments into a ranked, filtered edge set for overlap-graph-based strain-aware metagenome assembly.

## When to use

Use when quality-controlled NGS reads must be linked by suffix-prefix overlaps for OLC assembly, and each Minimap2-derived overlap needs a weighted identity/length score plus a length and identity cutoff.

## When NOT to use

- When reads are long, high-error-rate third-generation reads (e.g. PacBio CLR or noisy ONT) — the paper notes the high error rate of third-generation reads still introduces unresolved issues
- When the goal is generic read mapping to a reference rather than constructing read-overlap edges for strain-aware de novo assembly
- When overlaps are already scored by a downstream tool with its own scoring scheme (e.g. SAVAGE's statistically calibrated overlap grouping in HaploConduct)

## Inputs

- Interleaved FASTQ file of quality-controlled NGS short reads (e.g. all_reads.fq in @S0R0/1, @S0R0/2 layout)
- Minimap2 all-vs-all overlap alignments in PAF format with CIGAR strings at the 'cg' tag

## Outputs

- overlap_scores.tsv containing one row per surviving read pair with the two read identifiers, overlap length (>= 30 bp), identity (>= 0.9), and weighted overlap score D in [0, 1]

## How to apply

Run Minimap2 in all-vs-all overlap mode over the interleaved FASTQ read sample, requesting CIGAR output (the 'cg' tag of PAF) for every read pair alignment. Parse each CIGAR to extract the alignment identity (i) and overlap length (o), and record the two read lengths r1 and r2. Score each overlap with StrainXpress's formula D = 0.9*i + 0.1*o/((r1+r2)/2), which weights identity heavily while normalizing overlap length by the mean read length, avoiding a full-length re-examination of each overlap. Discard overlaps shorter than 30 bp or with identity below 0.9, then write surviving read pairs to a TSV with read identifiers, overlap length, identity, and score D for downstream overlap-graph construction.

## Related tools

- **Minimap2** (Performs all-vs-all suffix-prefix overlap detection between reads and emits the CIGAR strings from which identity and overlap length are parsed) — https://github.com/lh3/minimap2
- **StrainXpress** (Implements the weighted overlap scoring formula and the 30 bp / 0.9 identity filtering thresholds as part of its read clustering preprocessing) — https://github.com/HaploKit/StrainXpress
- **Pandas** (Used within the StrainXpress pipeline to assemble and write the filtered overlap score TSV) — https://github.com/kangxiongbin/StrainXpress

## Examples

```
minimap2 -c reads.fq reads.fq > overlaps.paf; python ../scripts/strainxpress.py -fq all_reads.fq
```

## Evaluation signals

- Every row of overlap_scores.tsv contains a read pair, an overlap length >= 30, an identity >= 0.9, and a score D within [0, 1]
- No surviving overlap has identity below 0.9 or length below 30 bp (cutoff invariants hold for the full file)
- For each row, D recomputed as 0.9*i + 0.1*o/((r1+r2)/2) from the CIGAR-derived identity and overlap length matches the stored score
- The output file exists and is well-formed TSV with the expected columns (read id 1, read id 2, overlap length, identity, score)
- The number of rows is consistent with the number of Minimap2 PAF alignments passing the two thresholds

## Limitations

- Short length of Illumina-type reads constrains contig length, preventing full-length strain-specific contigs despite accurate overlaps
- The scoring avoids assessing overlap quality across the full overlap length, relying on the CIGAR-derived summary statistics
- Requires linux-based systems and python3; the StrainXpress pipeline expects interleaved FASTQ input in the specific /1 and /2 mate layout

## Evidence

- [full_text] StrainXpress computes an overlap score for each Minimap2-identified suffix-prefix read overlap directly from the CIGAR string output by Minimap2: [evidence span withheld — anchor status: not_found]
- [readme] StrainXpress is a de novo assembly method which base on overlap-layout-consensus (OLC) paradigm: "StrainXpress is a de novo assembly method which base on overlap-layout-consensus (OLC) paradigm and can fast and accurately assemble high complexity metagenome sequencing data at strain resolution."
- [readme] You can ask minimap2 to generate CIGAR at the cg tag of PAF with: minimap2 -c ref.fa query.fq: [evidence span withheld — anchor status: external_doc]
- [discussion] patterns of co-occurring mutations can be highlighted optimally with overlap graphs: "patterns of co-occurring mutations—as the decisive characteristic sequential phenomenon of strains—can be highlighted optimally with overlap graphs"
