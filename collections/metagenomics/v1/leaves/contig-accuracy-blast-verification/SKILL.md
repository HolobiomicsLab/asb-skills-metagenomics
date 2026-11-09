---
name: contig-accuracy-blast-verification
description: Use when you have assembled or contig-extension output (e.g. ContigExtender
  output/contigs.fasta from viral metagenomics seed contigs) and need to verify each
  contig matches its reference genome before comparing tools or reporting lengths.
when_to_use_negative:
- Input is raw sequencing reads (FASTQ) rather than assembled/extended contigs — preprocessing
  or extension, not accuracy verification, is the appropriate step
- No reference genome or reference nucleotide database is available for the target
  organism, since accuracy is defined as similarity to the NCBI viral reference
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0797
tools:
- name: ContigExtender
  role: Produces the extended contigs (r-OLC strategy) whose accuracy is verified
    by BLAST
  repo: https://github.com/dengzac/contig-extender
- name: Bowtie2
  role: Performs the read-to-contig alignments underlying extension before BLAST verification
- name: PRINSEQ
  role: DUST complexity filtering of reads (complex-threshold 15) prior to extension,
    reducing false candidate reads
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1186/s12859-021-04038-2
    title: 'ContigExtender: a new approach to improving de novo sequence assembly
      for viral metagenomics data'
  source_content_hash: sha256:2ebb8b9efd70876ec0c9495cf170cc401ddbf4cbd364e9452bfcc673f49b70a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dengzac/contig-extender
  tools:
  - Bowtie2
  - ContigExtender
  - PRICE
  - PRINSEQ
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# contig-accuracy-blast-verification

## Summary

Verify the accuracy of extended/assembly contigs by nucleotide BLAST against NCBI viral reference nucleotide sequences, confirming high per-contig similarity (e.g. >98% single-segment alignment). This is the accuracy check the ContigExtender workflow relies on to distinguish correct extensions from false ones.

## When to use

Use when you have assembled or contig-extension output (e.g. ContigExtender output/contigs.fasta from viral metagenomics seed contigs) and need to verify each contig matches its reference genome before comparing tools or reporting lengths.

## When NOT to use

- Input is raw sequencing reads (FASTQ) rather than assembled/extended contigs — preprocessing or extension, not accuracy verification, is the appropriate step
- No reference genome or reference nucleotide database is available for the target organism, since accuracy is defined as similarity to the NCBI viral reference

## Inputs

- Extended contigs in FASTA format (e.g. output/contigs.fasta from ContigExtender)
- Reference viral genome FASTA (e.g. BKV.fasta or NCBI viral reference nucleotide sequences)

## Outputs

- Per-contig BLAST alignment report with identity and single-segment alignment coverage vs the reference genome
- Per-contig length-gain table (contigs extended by >=200 bp recorded for accuracy verification)

## How to apply

After running contig extension (e.g. ContigExtender via the Dockerized extender_wrapper with default min-overlap-length 15, min-branch-score 5, extend-tolerance 2.5), collect contigs from output/contigs.fasta, which are sorted by length and named in input order. Perform nucleotide BLAST of each extended contig against its reference viral genome (e.g. NCBI viral reference nucleotide sequences or the provided BKV.fasta). Judge success by alignment coverage and identity: in the NIBSC sample 26 benchmark, all but two of 26 final contigs (1.7-10 kb, largest extension 5.8 kb) showed nearly perfect alignments with greater than 98% single-segment coverage. For simulated BASV benchmarks (100 bp/10x, 250 bp/20x, error 0.01 from pIRS), expected contig lengths of 6.7 kb and 11.9 kb should coincide with high similarity to the NCBI reference. Flag any contig with fragmented or low-identity multi-segment alignments as a possible false extension.

## Related tools

- **ContigExtender** (Produces the extended contigs (r-OLC strategy) whose accuracy is verified by BLAST) — https://github.com/dengzac/contig-extender
- **Bowtie2** (Performs the read-to-contig alignments underlying extension before BLAST verification)
- **PRINSEQ** (DUST complexity filtering of reads (complex-threshold 15) prior to extension, reducing false candidate reads)

## Examples

```
./dist/extender_wrapper --coverage 50 examples/BKV_seed_1000_867.fa examples/BKV_250_50_0.01_0_.fastq && blastn -query output/contigs.fasta -subject examples/BKV.fasta
```

## Evaluation signals

- Extended contigs show high similarity to the NCBI viral reference nucleotide sequences via nucleotide BLAST
- At least 24 of 26 extended contigs from the NIBSC dataset achieve >98% single-segment alignment to their reference
- Reproduced contig lengths match Table 2 values: 6.7 kb at 100 bp/10x/0.01 and 11.9 kb at 250 bp/20x/0.01 from 1 kb seeds
- Output contigs in output/contigs.fasta are sorted by length and named in input order (e.g. >reference_1 for input >reference)
- Contigs selected for reporting were extended by >=200 bp with final lengths in the 1.7-10 kb range

## Limitations

- Accuracy is measured only against known reference genomes, so genuinely novel viruses lacking close references cannot be fully validated
- Higher --extend-tolerance values (up to 3.5) force extension but risk false extensions, which BLAST verification must catch
- Accuracy was not uniform: two of 26 NIBSC extended contigs did not achieve nearly perfect (>98%) single-segment alignments

## Evidence

- [intro] contig accuracy, which is measured by similarity to NCBI viral reference nucleotide sequences using nucleotide BLAST: "contig accuracy, which is measured by similarity to NCBI viral reference nucleotide sequences using nucleotide BLAST"
- [results] Of the 26 final contigs from the NIBSC dataset, all but two have nearly perfect alignments to reference genomes with greater than 98% single segment alignment.: [evidence span withheld — anchor status: not_found]
- [readme] To verify the accuracy of the extended contig, the reference genome is provided in BKV.fasta.: "To verify the accuracy of the extended contig, the reference genome is provided in BKV.fasta."
- [intro] ContigExtender is effective at extending seed contigs while maintaining high levels of contig accuracy.: "ContigExtender is effective at extending seed contigs while maintaining high levels of contig accuracy."
