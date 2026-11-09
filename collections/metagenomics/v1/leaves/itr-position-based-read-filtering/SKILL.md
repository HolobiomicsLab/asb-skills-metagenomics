---
name: itr-position-based-read-filtering
description: Use when you have read-level inverted terminal repeat (ITR) calls with
  LCoord/RCoord positions and paired assemblies, and need to keep only reads where
  repeat positions imply an IS within a plausible size range.
when_to_use_negative:
- The input contains no read-level ITR calls (e.g. raw FASTQ.GZ that has not been
  through the pal-MEM maximal exact matching step) — map-and-filter would have no
  repeat positions to test.
- The target ISs lack inverted terminal repeats, since Palidis is only equipped with
  discovering ISs with ITRs.
- You expect ISs whose ITRs are separated by more than the maximum IS length (e.g.
  regions lengthened by nested insertion events beyond 3000 bp), which this filter
  will discard by design.
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Bowtie2
  role: Maps the pal-MEM repeat-containing reads against their associated assemblies
    to obtain mapped read alignments for filtering
- name: Palidis
  role: Provides the getCandidateITRs Python filter step that applies the minimum-to-maximum
    IS length proximity filter to repeat positions
  repo: https://github.com/blue-moon22/palidis4
- name: pal-MEM
  role: Upstream tool producing the repeat-containing reads with LCoord/RCoord ITR
    coordinates that are then filtered
  repo: https://github.com/blue-moon22/pal-MEM
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:84f5a0ac5586f521f099105a276f58dedd9b64fc14e4ba61cc26a44aa3272719
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/palidis4
  tools:
  - Bowtie2
  - Palidis
  - pal-MEM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# ITR position-based read filtering

## Summary

A filtering step in the Palidis pipeline that maps repeat-containing reads (output of pal-MEM) back to their associated assemblies with Bowtie2, then retains only mapped reads whose ITR repeat coordinates fall within a user-defined minimum-to-maximum insertion sequence (IS) length interval (500–3000 bp for Palidis candidates).

## When to use

Use when you have read-level inverted terminal repeat (ITR) calls with LCoord/RCoord positions and paired assemblies, and need to keep only reads where repeat positions imply an IS within a plausible size range.

## When NOT to use

- The input contains no read-level ITR calls (e.g. raw FASTQ.GZ that has not been through the pal-MEM maximal exact matching step) — map-and-filter would have no repeat positions to test.
- The target ISs lack inverted terminal repeats, since Palidis is only equipped with discovering ISs with ITRs.
- You expect ISs whose ITRs are separated by more than the maximum IS length (e.g. regions lengthened by nested insertion events beyond 3000 bp), which this filter will discard by design.

## Inputs

- Repeat-containing reads and their discordant paired mates output by pal-MEM (e.g. _IR_1.fasta, _IR_2.fasta, _discord_non_IR fasta files, or FASTQ.GZ equivalents)
- Assembly FASTA files associated with each metagenomic sample (Bowtie2 indices built via buildDB)
- User-defined minimum and maximum IS length parameters (e.g. 500 bp minimum, 3000 bp maximum)
- pal-MEM _IR.tab file with LCoord/RCoord repeat coordinates per read

## Outputs

- candidate_IS_reads.fastq containing mapped, proximity-filtered reads with candidate ITRs
- Candidate ITR read set ready for pooling and CD-HIT-EST clustering (clusterReads step)

## How to apply

First take the repeat-containing reads and their paired mates produced by the pal-MEM maximal exact matching step. Align these reads to their associated assemblies using Bowtie2 in paired-end or single-end short-read mode to obtain mapped read alignments. Then run Palidis' Python filter script (the getCandidateITRs stage) on the alignments, parsing each read's repeat positions (LCoord/RCoord) and retaining only mapped reads whose repeat positions lie between the minimum and maximum IS length — the 500–3000 bp proximity range used for Palidis candidates. Retained reads are written to candidate_IS_reads.fastq for downstream pooling and CD-HIT-EST clustering of IS candidates. The rationale is that reads whose ITR positions are too close or too far apart on the assembly do not plausibly flank a single insertion sequence.

## Related tools

- **Bowtie2** (Maps the pal-MEM repeat-containing reads against their associated assemblies to obtain mapped read alignments for filtering)
- **Palidis** (Provides the getCandidateITRs Python filter step that applies the minimum-to-maximum IS length proximity filter to repeat positions) — https://github.com/blue-moon22/palidis4
- **pal-MEM** (Upstream tool producing the repeat-containing reads with LCoord/RCoord ITR coordinates that are then filtered) — https://github.com/blue-moon22/pal-MEM

## Evaluation signals

- Every retained read in candidate_IS_reads.fastq maps to its associated assembly and its repeat positions (LCoord/RCoord distance) lie within the configured 500–3000 bp interval.
- Reads whose repeat positions fall outside the minimum-to-maximum IS length are absent from the output.
- The output preserves read pairs: reads and their discordant non-IR mates from pal-MEM are carried through consistently.
- Downstream CD-HIT-EST clustering of the filtered candidates succeeds, indicating the FASTQ is well-formed and non-degenerate.
- Setting the maximum IS length to a much larger value recovers additional candidates corresponding to nested/lengthened insertions, confirming the filter behaves as a strict distance cutoff.

## Limitations

- ISs flanked by ITRs beyond the 500–3000 bp proximity range, such as regions lengthened by nested insertion events, are missed by this filter.
- The default maximum IS length may need to be increased in future ISC iterations to capture nested or lengthened insertions.
- The filter depends on reads mapping to an associated assembly; reads from loci absent from assemblies cannot be filtered by repeat position.

## Evidence

- [readme] Map reads against assemblies using Bowtie2: "Map reads against assemblies using Bowtie2 [`filterContigs` `buildDB` `mapreads`]"
- [readme] Get candidate ITRs by distance filters: "Get candidate ITRs by distance filters [`getCandidateITRs`]"
- [discussion] ISs beyond proximity range missed: [evidence span withheld — anchor status: not_found]
- [full_text] Bowtie2 mapping and Python position filter description: [evidence span withheld — anchor status: not_found]
