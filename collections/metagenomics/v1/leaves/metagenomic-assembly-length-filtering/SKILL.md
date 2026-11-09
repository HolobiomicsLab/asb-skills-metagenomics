---
name: metagenomic-assembly-length-filtering
description: Use when a metagenomic or isolate assembly (e.g., SPAdes 3.9.1 scaffolds)
  contains many short contigs that are about to be classified by a kmer-based method
  (PlasFlow) and results for short sequences would otherwise be unreliable.
when_to_use_negative:
- All input contigs already exceed the 1000 bp threshold, so filtering would be a
  no-op
- Input is raw FASTQ reads rather than an assembly — read-level quality/length trimming
  (e.g., with sickle) is a different operation
- Running graph-based plasmid tools (Recycler, PlasmidSpades) that require the full
  assembly graph and paired reads rather than filtered FASTA contigs
edam_operation: http://edamontology.org/operation_0253
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: filter_sequences_by_length.pl
  role: Perl script that removes contigs shorter than the -thresh length cutoff (1000
    bp) from an assembly FASTA before PlasFlow classification
  repo: https://github.com/smaegol/PlasFlow
- name: PlasFlow
  role: Downstream kmer-signature neural-network classifier whose accuracy and RAM
    usage benefit from the length-filtered input
  repo: https://github.com/smaegol/PlasFlow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:a10705d88ea68a35a0fd286dfa1e69f4db1f9bc35a97a82c9de92da3215120c1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/smaegol/PlasFlow
  repo_url: https://github.com/smaegol/PlasFlow
  tools:
  - PlasFlow
  - filter_sequences_by_length.pl
  edam_operation: http://edamontology.org/operation_0253
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# metagenomic-assembly-length-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pre-filter metagenomic assembly contigs by length (keeping only sequences longer than 1000 bp) before running kmer-signature-based classifiers such as PlasFlow, because kmer-based plasmid prediction is unreliable on short contigs and unfiltered assemblies waste RAM and runtime.

## When to use

Use when a metagenomic or isolate assembly (e.g., SPAdes 3.9.1 scaffolds) contains many short contigs that are about to be classified by a kmer-based method (PlasFlow) and results for short sequences would otherwise be unreliable.

## When NOT to use

- All input contigs already exceed the 1000 bp threshold, so filtering would be a no-op
- Input is raw FASTQ reads rather than an assembly — read-level quality/length trimming (e.g., with sickle) is a different operation
- Running graph-based plasmid tools (Recycler, PlasmidSpades) that require the full assembly graph and paired reads rather than filtered FASTA contigs

## Inputs

- Assembly FASTA file of contigs/scaffolds (e.g., SPAdes 3.9.1 assembly of Citrobacter freundii strain CAV1321, NCBI GCA_001022155.1)
- Length threshold parameter (1000 bp, passed via -thresh)

## Outputs

- Filtered FASTA file containing only sequences longer than the 1000 bp threshold
- Count of discarded short contigs (implicit from input vs. filtered FASTA sizes)

## How to apply

Run the Perl script `filter_sequences_by_length.pl` (bundled with PlasFlow, requires BioPerl and Getopt) with `-input` pointing to the assembly FASTA (e.g., SPAdes 3.9.1 scaffolds of the 42 plasmid-rich bacterial genomes), `-output` for the filtered FASTA, and `-thresh 1000` to retain only contigs longer than 1000 bp. The rationale, per the PlasFlow authors, is that kmer-signature methods cannot get proper kmer coverage from short sequences, so short-sequence results are unreliable; the length filter also improves results, speeds up PlasFlow, and prevents excessive RAM usage on large metagenomic assemblies. Apply the filter before every PlasFlow.py run and keep the filtered FASTA as the `--input` to classification, which is run afterwards with `--threshold 0.7`.

## Related tools

- **filter_sequences_by_length.pl** (Perl script that removes contigs shorter than the -thresh length cutoff (1000 bp) from an assembly FASTA before PlasFlow classification) — https://github.com/smaegol/PlasFlow
- **PlasFlow** (Downstream kmer-signature neural-network classifier whose accuracy and RAM usage benefit from the length-filtered input) — https://github.com/smaegol/PlasFlow

## Examples

```
filter_sequences_by_length.pl -input input_dataset.fasta -output filtered_output.fasta -thresh 1000
```

## Evaluation signals

- Every sequence in the filtered FASTA has length strictly greater than the -thresh value (1000 bp); a sequence-utils check should show zero violations
- Total contig count of the filtered FASTA is less than the input assembly, while all retained sequences also appear in the input FASTA
- Downstream PlasFlow.py run completes without RAM-related failures on the filtered set and yields predictions TSV plus plasmids/chromosomes/unclassified FASTA bins
- On the benchmark of 42 plasmid-rich bacterial genomes, the filtered-input workflow reproduces the reported PlasFlow plasmid recall of 85.98% and precision of 72.17%
- Using a different threshold (e.g., 500 bp) should visibly change accuracy on short-contig-rich assemblies if results for short sequences were unreliable

## Limitations

- Kmer-based methods do not perform well on short sequences because it is hard to get proper kmer coverage, so even after filtering, predictions on sequences near the 1000 bp cutoff remain less reliable
- The 1000 bp cutoff trades sensitivity for reliability: true plasmid contigs shorter than the threshold are excluded before classification and can never be recovered by PlasFlow
- On very large assemblies, PlasFlow may still need smaller --batch_size values even after length filtering (per the v1.1 README note); the README also warns the project is NOT MAINTAINED, so dependency fixes (e.g., TensorFlow 0.10.0, Python 3.5) may not be available

## Evidence
