---
name: iterative-seed-expansion-implementation
description: Use when raw Illumina FASTQ reads contain a small seed set (HMM- or repeat-derived)
  and neighboring reads must be recruited for targeted assembly without assembling
  the whole dataset.
when_to_use_negative:
- The dataset has already been fully assembled and annotated — conventional assembly/annotation
  pipelines are the alternative CasCollect was benchmarked against, and full assembly
  forecloses the runtime benefit (almost two orders of magnitude)
- Sequencing data are not short-read DNA sequencing reads (e.g. long-read or amino-acid
  inputs), since VSEARCH does not support amino acid sequences or local alignments
- The target is a whole-genome assembly rather than a targeted specialty gene region
  around seed matches
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: CasCollect
  role: Python wrapper that orchestrates the iterative cycles, manages matched/unmatched
    read sets, and writes expanded_reads.fastq with per-cycle recruitment counts
  repo: https://github.com/sandialabs/CasCollect
- name: VSEARCH
  role: Performs the per-cycle identity search of all reads against the current seed/subset
    set at the percent-match threshold set by -match (default 95%)
  repo: https://github.com/torognes/vsearch
- name: HMMER
  role: Upstream seed-generation step that supplies the HMM/query-derived seed reads
    that prime the expansion
- name: SPAdes
  role: Consumes the expanded read subset downstream for targeted assembly of CRISPR/Cas
    regions (metaSPAdes for metagenomes)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:449b4dcd046a751be29383d7f867824dcef2ff00c68c9fcae7c2c6685ef8112c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sandialabs/CasCollect
  tools:
  - CasCollect
  - HMMER
  - SPAdes
  - VSEARCH
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# iterative-seed-expansion-implementation

## Summary

Iteratively expands a small set of seed reads (e.g. cas-gene or CRISPR-repeat matches) into a larger read subset by repeated VSEARCH identity matching, so that only the CRISPR/Cas-relevant fraction of a raw FASTQ dataset feeds targeted assembly.

## When to use

Use when raw Illumina FASTQ reads contain a small seed set (HMM- or repeat-derived) and neighboring reads must be recruited for targeted assembly without assembling the whole dataset.

## When NOT to use

- The dataset has already been fully assembled and annotated — conventional assembly/annotation pipelines are the alternative CasCollect was benchmarked against, and full assembly forecloses the runtime benefit (almost two orders of magnitude)
- Sequencing data are not short-read DNA sequencing reads (e.g. long-read or amino-acid inputs), since VSEARCH does not support amino acid sequences or local alignments
- The target is a whole-genome assembly rather than a targeted specialty gene region around seed matches

## Inputs

- Seed read set in FASTA format (user-defined seed reads or reads derived from HMMER/VSEARCH seed generation)
- Raw paired-end Illumina sequencing reads in FASTQ format (possibly gzipped, accepted by VSEARCH)
- Configuration parameters: -match identity threshold (default 95) and -cycle expansion count (default 5)

## Outputs

- expanded_reads.fastq containing the final expanded read subset in FASTQ format
- Per-cycle read recruitment counts reported by CasCollect

## How to apply

Load the seed read set (a user-defined FASTA of seeds or HMM/query-derived seed reads from HMMER against 120 Cas protein profile HMMs or VSEARCH against CRISPR array repeats) together with the target FASTQ read file(s). Run VSEARCH searching all remaining reads against the current seed/subset set at the default 95% percent-match identity threshold (adjustable with -match). Add newly matched reads to the expanded subset and remove them from the search database so they are not re-searched, eliminating redundant searches. Repeat for the configured number of cycles (-cycle, default five), then write the final expanded subset to expanded_reads.fastq and report per-cycle recruitment counts before handing the subset to SPAdes/metaSPAdes assembly.

## Related tools

- **CasCollect** (Python wrapper that orchestrates the iterative cycles, manages matched/unmatched read sets, and writes expanded_reads.fastq with per-cycle recruitment counts) — https://github.com/sandialabs/CasCollect
- **VSEARCH** (Performs the per-cycle identity search of all reads against the current seed/subset set at the percent-match threshold set by -match (default 95%)) — https://github.com/torognes/vsearch
- **HMMER** (Upstream seed-generation step that supplies the HMM/query-derived seed reads that prime the expansion)
- **SPAdes** (Consumes the expanded read subset downstream for targeted assembly of CRISPR/Cas regions (metaSPAdes for metagenomes))

## Examples

```
python CasCollect.py -fq reads.fastq -match 95 -cycle 5
```

## Evaluation signals

- expanded_reads.fastq exists, is non-empty, and is valid FASTQ
- Subset size grows monotonically across the configured expansion cycles (default five)
- Default run used 95% identity (-match) and five cycles (-cycle) as specified
- Matched reads are removed from the database between cycles, so no read is re-searched — per-cycle newly recruited counts should not include reads from prior cycles
- Final expanded subset, when assembled with SPAdes/metaSPAdes and annotated, recovers the known cas genes and CRISPR arrays (as demonstrated on E. coli KLY, P. aeruginosa VA-134, S. pyogenes M39)

## Limitations

- The 95% identity default is 'moderately high' and may miss more divergent neighboring reads; it must be tuned via -match for distant homologs
- A fixed cycle count (default five) may under-expand for long CRISPR/Cas loci or over-expand into non-target regions if -cycle/-match are not adjusted
- Expansion only recruits reads similar to already-recruited seeds; it cannot recover loci with no seed representation, and VSEARCH alignment is global rather than local

## Evidence

- [results] default read subset expansion identity match of 95% (adjustable with -match): "The default identity match has been set to a moderately high value of 95%, however, this can be adjusted with the -match command."
- [results] default five expansion cycles (adjustable with -cycle): "The default number of these read subset expansion cycles is set at five and can be adjusted with the -cycle command."
- [results] matched reads removed from database between cycles: "This process eliminates redundant searches by removing reads from the database set that have been matched in the previous cycle"
- [results] pipeline step ordering including read subset expansion: "The CasCollect pipeline was created from five distinct and optional steps: (i) read filtering, (ii) seed generation, (iii) read subset expansion, (iv) assembly and (v) annotation"
- [results] VSEARCH used for expansion-stage identity matching: "DNA mode implements VSEARCH (22) for identity matching to a user-defined set of CRISPR array repeats"
