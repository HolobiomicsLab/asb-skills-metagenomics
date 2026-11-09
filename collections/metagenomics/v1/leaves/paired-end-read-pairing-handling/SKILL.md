---
name: paired-end-read-pairing-handling
description: Use when classifying paired-end FASTQ read pairs (e.g. simulated 2x250
  nt Illumina reads) with Kaiju, or when benchmarking throughput where read-pair ordering
  and naming conventions must be preserved.
when_to_use_negative:
- Input is single-end reads or already-translated protein sequences — then only -i
  (or additionally -p for protein input) is needed and pairing logic is irrelevant
- Read pairs are unordered or names mismatched (e.g. after subsetting only one mate)
  — Kaiju will error because mate names are compared after stripping suffixes after
  '/' or spaces
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Kaiju
  role: Classifies paired-end reads jointly via -i/-j arguments, matching mate names
    after stripping suffixes after '/' or spaces
  repo: https://github.com/bioinformatics-centre/kaiju
- name: ART
  role: Simulates the paired-end (e.g. 2x250 nt) benchmark reads whose pairing must
    be preserved during classification
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:f43264e22e314a405df4ed79a52cefe5019c6e5c68a8788fde2ed6bd4a458a48
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinformatics-centre/kaiju
  tools:
  - ART
  - Clark
  - Kaiju
  - Kraken
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# paired-end-read-pairing-handling

## Summary

Correctly supplies and validates paired-end FASTQ read pairs to Kaiju's taxonomic classifier, using the -i/-j input arguments and ensuring read-name consistency so both mates of a pair are classified jointly. Paired-end length strongly affects runtime, so pairing handling is integral to the runtime/memory benchmark.

## When to use

Use when classifying paired-end FASTQ read pairs (e.g. simulated 2x250 nt Illumina reads) with Kaiju, or when benchmarking throughput where read-pair ordering and naming conventions must be preserved.

## When NOT to use

- Input is single-end reads or already-translated protein sequences — then only -i (or additionally -p for protein input) is needed and pairing logic is irrelevant
- Read pairs are unordered or names mismatched (e.g. after subsetting only one mate) — Kaiju will error because mate names are compared after stripping suffixes after '/' or spaces

## Inputs

- Forward-mate FASTQ (or FASTA, possibly gzip-compressed) file for -i
- Reverse-mate FASTQ (or FASTA) file for -j with reads in the same order and matching names
- Kaiju index file (.fmi, BWT + FM-index) and NCBI taxonomy files nodes.dmp and names.dmp

## Outputs

- Tab-separated Kaiju output file (one line per read pair: C/U status, read name, NCBI taxon identifier; optional verbose columns with -v)
- Throughput measurements in reads per second per read type for the runtime benchmark

## How to apply

Supply the first mate file with -i and the second mate file with -j; Kaiju compares read names between the two files after stripping all characters after a '/' or space, and issues an error if names are not identical, so input pairs must be in the same order. Read pairs may be in FASTQ or FASTA, optionally gzip-compressed. Kaiju translates each read into six reading frames and classifies the pair jointly, printing one output line per read pair when run e.g. as 'kaiju -z 25 -t nodes.dmp -f kaiju_db.fmi -i R1.fastq -j R2.fastq -o kaiju.out'. In the benchmark, paired-end 250 nt reads are the slowest configuration (Kaiju MEM ~97k reads/s vs 173k for single-end 100 nt), so expect roughly halved throughput relative to short single-end reads. When simulating benchmark reads with ART, keep mates synchronized and preserve naming so Kaiju's name check passes.

## Related tools

- **Kaiju** (Classifies paired-end reads jointly via -i/-j arguments, matching mate names after stripping suffixes after '/' or spaces) — https://github.com/bioinformatics-centre/kaiju
- **ART** (Simulates the paired-end (e.g. 2x250 nt) benchmark reads whose pairing must be preserved during classification)

## Examples

```
kaiju -z 25 -t nodes.dmp -f kaiju_db.fmi -i sample_R1.fastq -j sample_R2.fastq -o kaiju.out
```

## Evaluation signals

- Kaiju completes without a read-name mismatch error, confirming both mate files are in the same order with identical names after suffix stripping
- Output file contains exactly one line per read pair (27.24M lines for the benchmark dataset), not one per individual mate
- Every output line starts with C (classified) or U (unclassified) followed by the read name and a valid NCBI taxon identifier present in nodes.dmp
- Benchmarked paired-end 250 nt throughput falls near the reported ranges (Kaiju MEM ~97k r.p.s., Kraken ~24k, Clark ~19k) and peak memory near 5.6 GB for Kaiju

## Limitations

- Kaiju's protein-level approach cannot classify reads from non-protein-coding genomic regions, which affects paired-end assignments regardless of pairing correctness
- Throughput drops substantially for paired-end 250 nt reads (e.g. MEM mode ~97k r.p.s. vs 173k r.p.s. for single-end 100 nt), so runtime budgeting must account for read length and pairing
- Allowing more mismatches (Greedy mode -e) increases runtime, compounding the cost on long paired-end reads

## Evidence

- [readme] For paired-end reads use -i firstfile.fastq and -j secondfile.fastq.: "For paired-end reads use `-i firstfile.fastq` and `-j secondfile.fastq`."
- [readme] The reads must be in the same order in both files.: "The reads must be in the same order in both files. Kaiju will strip suffixes from the read names by deleting all characters after a `/` or space."
- [results] paired-end 250 nt runs slowest (MEM 97k, Kraken 24k, Clark 19k r.p.s.); peak memory use is 5.6 GB for Kaiju: [evidence span withheld — anchor status: not_found]
- [discussion] Kaiju translates each read into the six possible reading frames, which are then split at stop codons into amino acid fragments.: "Kaiju translates each read into the six possible reading frames, which are then split at stop codons into amino acid fragments."
