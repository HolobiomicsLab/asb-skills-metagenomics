---
name: probability-threshold-classification-of-scaffolds
description: Use when a metagenomic or genome assembly in fasta form must be partitioned
  into plasmid-derived vs chromosomal scaffolds based on kmer genome signatures, with
  unreliable low-confidence calls filtered out.
when_to_use_negative:
- Input consists of raw short reads rather than assembled contigs — PlasFlow requires
  an assembly because kmer signatures of short sequences are unreliable
- Sequences shorter than 1000 bp are the target — results for short sequences are
  explicitly unreliable in kmer-based methods like PlasFlow
- The goal is replicon reconstruction from paired reads and an assembly graph — tools
  like PlasmidSpades or Recycler are required for that, not signature-based classification
edam_operation: http://edamontology.org/operation_3472
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: PlasFlow
  role: Neural-network kmer-signature classifier that labels filtered contigs as plasmid,
    chromosome, or phylum-level class and bins sequences into output fasta files
  repo: https://github.com/smaegol/PlasFlow
- name: filter_sequences_by_length.pl
  role: Perl/Bioperl script that removes contigs below the 1000 bp length threshold
    before PlasFlow classification
  repo: https://github.com/smaegol/PlasFlow
- name: SPAdes
  role: Assembler used upstream to produce the scaffold fasta inputs (e.g., SPAdes
    3.9.1 assembly of C. freundii CAV1321) for PlasFlow analysis
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:e5fd7a248c7d36e85a95a3383f302d485943fc04c4b1c7a62bfe4d617301fa1b
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
  - SPAdes
  - filter_sequences_by_length.pl
  edam_operation: http://edamontology.org/operation_3472
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# probability-threshold-classification-of-scaffolds

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classify metagenomic assembly contigs/scaffolds as plasmid, chromosomal, or unclassified using PlasFlow's kmer-signature neural network, applying a 0.7 probability threshold to exclude uncertain predictions before binning sequences into fasta files.

## When to use

Use when a metagenomic or genome assembly in fasta form must be partitioned into plasmid-derived vs chromosomal scaffolds based on kmer genome signatures, with unreliable low-confidence calls filtered out.

## When NOT to use

- Input consists of raw short reads rather than assembled contigs — PlasFlow requires an assembly because kmer signatures of short sequences are unreliable
- Sequences shorter than 1000 bp are the target — results for short sequences are explicitly unreliable in kmer-based methods like PlasFlow
- The goal is replicon reconstruction from paired reads and an assembly graph — tools like PlasmidSpades or Recycler are required for that, not signature-based classification

## Inputs

- Assembly contigs/scaffolds in fasta format (e.g., SPAdes metagenome assemblies), filtered to sequences > 1000 bp
- PlasFlow trained neural network model files (from smaegol/PlasFlow_models)

## Outputs

- plasflow_predictions.tsv — tabular file with contig_id, label, and per-class probability columns
- Three fasta files binning input sequences into plasmids, chromosomes, and unclassified

## How to apply

First filter the assembly with PlasFlow's filter_sequences_by_length.pl, retaining only contigs longer than 1000 bp, because kmer-based methods perform poorly on short sequences and short contigs inflate RAM usage. Then run PlasFlow.py with --input <filtered.fasta>, --output plasflow_predictions.tsv, and --threshold 0.7; the tool computes kmer-frequency signatures in batches (tune --batch_size down if memory issues arise) and a neural network assigns each contig to one of 26 chromosome/plasmid phylum-level classes. Contigs whose assignment probability falls below the 0.7 cutoff are labeled unclassified. PlasFlow emits fasta files binning the input into plasmids, chromosomes, and unclassified; summarize per-sample plasmid fractions (e.g., 25.02–27.05% of sequences and 14.94–20.66% of cumulative scaffold length in the KOW/ZS mine metagenomes). The probability filtering is deliberate: excluding low-probability classifications raises overall accuracy and lowers the false-prediction rate.

## Related tools

- **PlasFlow** (Neural-network kmer-signature classifier that labels filtered contigs as plasmid, chromosome, or phylum-level class and bins sequences into output fasta files) — https://github.com/smaegol/PlasFlow
- **filter_sequences_by_length.pl** (Perl/Bioperl script that removes contigs below the 1000 bp length threshold before PlasFlow classification) — https://github.com/smaegol/PlasFlow
- **SPAdes** (Assembler used upstream to produce the scaffold fasta inputs (e.g., SPAdes 3.9.1 assembly of C. freundii CAV1321) for PlasFlow analysis)

## Examples

```
filter_sequences_by_length.pl -input input_dataset.fasta -output filtered_output.fasta -thresh 1000 && PlasFlow.py --input filtered_output.fasta --output plasflow_predictions.tsv --threshold 0.7
```

## Evaluation signals

- plasflow_predictions.tsv exists and is in tabular format with contig_id, label, and per-class probability columns
- Per-sample plasmid fractions (sequences and cumulative scaffold length) fall in the ranges reported in the manuscript (25.02–27.05% of sequences; 14.94–20.66% of cumulative length for the KOW/ZS samples)
- All contigs with assignment probability below 0.7 are labeled unclassified rather than forced into a plasmid or chromosome class
- Input fasta counts equal the sum of sequences in the plasmid, chromosome, and unclassified output fasta files (no sequences lost in binning)
- No contig shorter than the 1000 bp length threshold appears in the filtered input or the predictions file

## Limitations

- PlasFlow performs poorly on short sequences (<1000 bp) because proper kmer coverage cannot be obtained; pre-filtering is strongly recommended
- The project is no longer maintained ('NOT MAINTAINED', version 1.1, TensorFlow 0.10.0 / Python 3.5 dependencies), so modern environments may need workarounds
- If memory issues arise on large datasets, --batch_size must be reduced; alternatively use smaller batch numbers
- PlasFlow does not rely on assembly graphs or paired reads, so unlike PlasmidSpades or Recycler it cannot use those signals; model may need updating with newly published genomic data to cover novel taxa

## Evidence
