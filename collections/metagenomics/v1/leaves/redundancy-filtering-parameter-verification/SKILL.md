---
name: redundancy-filtering-parameter-verification
description: Use when a downloaded or rebuilt Plass protein catalog (SRC or MERC FASTA)
  must be checked against published sequence counts, source-sample counts, and the
  90%/90% redundancy-reduction parameters.
when_to_use_negative:
- Input is raw metagenomic/metatranscriptomic FASTQ reads needing assembly itself
  — use 'plass assemble' to build the catalog first rather than verifying an existing
  one
- The catalog under inspection was clustered at different thresholds (not 90% identity
  / 90% coverage), so published SRC/MERC size figures are not valid comparison targets
- The goal is nucleotide contig assembly — that is PenguiN's scope, not protein catalog
  verification
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3174
tools:
- name: Plass
  role: Source of the SRC/MERC reference protein catalogs; can alternatively re-assemble
    raw paired-end FASTQ reads via 'plass assemble' to reproduce catalog contents
  repo: https://github.com/soedinglab/plass
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-019-0437-4
    title: Protein-level assembly increases protein sequence recovery from metagenomic
      samples manyfold
  source_content_hash: sha256:ad3f984d9b44596a98274f38e01f1c5692f325d5b5436fb1b90b6217b899b371
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/plass
  tools:
  - Plass
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# redundancy-filtering-parameter-verification

## Summary

Verifies that large de novo protein reference catalogs (SRC from 640 soil metagenomes, MERC from TARA ocean metatranscriptomes) match their reported redundancy-filtered sizes (2 billion and 292 million sequences) and the stated 90% sequence identity / 90% coverage clustering parameters.

## When to use

Use when a downloaded or rebuilt Plass protein catalog (SRC or MERC FASTA) must be checked against published sequence counts, source-sample counts, and the 90%/90% redundancy-reduction parameters.

## When NOT to use

- Input is raw metagenomic/metatranscriptomic FASTQ reads needing assembly itself — use 'plass assemble' to build the catalog first rather than verifying an existing one
- The catalog under inspection was clustered at different thresholds (not 90% identity / 90% coverage), so published SRC/MERC size figures are not valid comparison targets
- The goal is nucleotide contig assembly — that is PenguiN's scope, not protein catalog verification

## Inputs

- SRC catalog FASTA file (single-file protein catalog from Plass current-release server, SRA identifiers in headers)
- MERC catalog FASTA file (single-file protein catalog assembled from TARA ocean metatranscriptomes)
- Plass release metadata documenting the redundancy-reduction parameters

## Outputs

- catalog_size_verification.tsv (tab-delimited; rows for SRC and MERC with sequence counts and parameter confirmation)
- Per-catalog sequence record counts and SRA-derived sample provenance lists

## How to apply

Download the SRC and MERC catalog FASTA files from the Plass current-release server (http://wwwuser.gwdg.de/~compbiol/plass/current_release/), each provided as a single FASTA file whose headers contain Sequence Read Archive (SRA) identifiers. Stream-parse each FASTA, counting sequence records and extracting SRA identifiers from header lines to confirm sample provenance (640 soil samples for SRC; 775 marine eukaryotic metatranscriptomes for MERC). Confirm from the release metadata that both catalogs were redundancy-reduced to 90% sequence identity at 90% coverage, as stated in the paper and README. Optionally re-assemble raw paired-end FASTQ reads with 'plass assemble' instead of downloading prebuilt catalogs. Write the comparison to a tab-delimited catalog_size_verification.tsv with rows for SRC and MERC and judge correctness by agreement with the reported 2 billion (SRC) and 292 million (MERC) sequence totals.

## Related tools

- **Plass** (Source of the SRC/MERC reference protein catalogs; can alternatively re-assemble raw paired-end FASTQ reads via 'plass assemble' to reproduce catalog contents) — https://github.com/soedinglab/plass

## Examples

```
plass assemble reads_1.fastq.gz reads_2.fastq.gz assembly.fas tmp
```

## Evaluation signals

- catalog_size_verification.tsv exists, is tab-delimited, and contains rows for both SRC and MERC
- Counted SRC sequence total is consistent with the reported 2 billion sequences from 640 soil samples
- Counted MERC sequence total is consistent with the reported 292 million sequences from 775 marine eukaryotic metatranscriptomes
- Release metadata confirms both catalogs were redundancy-reduced at 90% sequence identity and 90% coverage
- Downloaded FASTA files parse cleanly and header SRA identifiers resolve to plausible sample provenance

## Limitations

- Published figures are approximate reported sizes (2 billion / 292 million); exact recomputed counts may differ modestly, so agreement is judged as consistency rather than exact equality
- Re-assembling from raw paired-end reads with 'plass assemble' to reproduce the catalogs is computationally enormous at the scale of 640 soil metagenomes or 775 TARA metatranscriptomes
- Sample provenance relies on SRA identifiers embedded in FASTA headers; malformed or missing headers weaken the provenance check

## Evidence

- [abstract] catalog sizes and parameters: "We assembled two redundancy-filtered reference protein catalogs, 2 billion sequences from 640 soil samples (soil reference protein catalog) and 292 million sequences from 775 marine eukaryotic"
- [readme] README release provenance and thresholds: "SRC was created by assembling 640 soil metagenome samples. MERC was assembled from the the metatranscriptomics datasets created by the TARA ocean expedition."
- [readme] README redundancy parameters: "Both catalogues were redundancy reduced to 90% sequence identity at 90% coverage."
- [readme] README FASTA format and headers: "Each catalog is a single FASTA file containing the sequences, the header identifiers contain the Sequence Read Archive (SRA) identifiers."
- [methods] assembly invocation: "plass assemble examples/reads_1.fastq.gz examples/reads_2.fastq.gz assembly.fas tmp"
