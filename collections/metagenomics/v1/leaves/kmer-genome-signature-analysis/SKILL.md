---
name: kmer-genome-signature-analysis
description: Use when you have a metagenomic or plasmidome assembly (FASTA contigs)
  and need to separate plasmid from chromosomal sequences and obtain initial phylum-level
  taxonomic labels.
when_to_use_negative:
- Input is raw sequencing reads rather than assembled contigs — PlasFlow is designed
  for metagenomic assemblies
- Contigs are shorter than 1000 bp — results for short sequences are unreliable because
  kmer coverage is insufficient
- The analysis requires recovering circular plasmids from assembly graphs with paired
  reads — use PlasmidSpades or Recycler instead, as PlasFlow does not use assembly
  graphs
edam_operation: http://edamontology.org/operation_3472
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: PlasFlow
  role: Neural-network classifier using kmer genomic signatures to label contigs as
    plasmid/chromosome with phylum-level classes
  repo: https://github.com/smaegol/PlasFlow
- name: filter_sequences_by_length.pl
  role: Perl/Bioperl script that filters the assembly to contigs longer than 1000
    bp before classification
  repo: https://github.com/smaegol/PlasFlow
- name: cBar
  role: SOM-based kmer signature classifier used as a comparison baseline for plasmid
    identification
- name: SPAdes
  role: Assembler producing the scaffold FASTA inputs for classification
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:9e1e3745dafa4811df07b187c58658de9dba4d5945eff3ff398e0c402c5fddc3
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
  - cBar
  - filter_sequences_by_length.pl
  edam_operation: http://edamontology.org/operation_3472
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# kmer-genome-signature-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies metagenomic contigs as plasmid or chromosome using kmer-based genomic signatures fed to a neural network (PlasFlow), achieving up to 96% accuracy without prior knowledge of sample taxonomic or functional composition.

## When to use

Use when you have a metagenomic or plasmidome assembly (FASTA contigs) and need to separate plasmid from chromosomal sequences and obtain initial phylum-level taxonomic labels.

## When NOT to use

- Input is raw sequencing reads rather than assembled contigs — PlasFlow is designed for metagenomic assemblies
- Contigs are shorter than 1000 bp — results for short sequences are unreliable because kmer coverage is insufficient
- The analysis requires recovering circular plasmids from assembly graphs with paired reads — use PlasmidSpades or Recycler instead, as PlasFlow does not use assembly graphs

## Inputs

- Metagenomic/plasmidome assembly FASTA (e.g. SPAdes 3.9.1 scaffolds) filtered to contigs > 1000 bp
- Reference plasmidome assemblies: Brown Kav (MG-RAST 4460391.3), Sentchilo (MG-RAST 4474000.3), Zhang (SRA SRP007256.1), Szczepanowski (FTP)
- SPAdes assemblies of 42 plasmid-rich bacterial genomes with GenBank reference plasmids for benchmarking

## Outputs

- Tab-separated prediction file with contig_id, contig_name, contig_length, label and per-class probabilities
- FASTA files binning input sequences into plasmids, chromosomes and unclassified
- Plasmid:chromosome contig ratios and recall/precision statistics (e.g. 85.98% recall, 72.17% precision on the 42-genome benchmark)

## How to apply

First filter the assembly with filter_sequences_by_length.pl to keep only contigs longer than 1000 bp, since kmer-based classifiers are unreliable on shorter sequences and filtering also reduces RAM use. Then run PlasFlow.py with --threshold 0.7 (the default probability cutoff below which contigs are labeled unclassified) to classify contigs into 26 phylum-level chromosome/plasmid classes using models trained on 10 kb fragments from 9,565 bacterial chromosomes and plasmids. Collect the tab-separated predictions (contig_id, contig_name, contig_length, label, per-class probabilities) plus the binned plasmids/chromosomes/unclassified FASTA files. To compare against an alternative, run cBar on the same filtered assemblies and compute plasmid:chromosome contig ratios (expected PlasFlow values 15.73, 13.55, 4.72, 2.83 on the Brown Kav, Szczepanowski, Sentchilo and Zhang plasmidomes vs cBar ~1.81, 1.72, 0.69, 1.40).

## Related tools

- **PlasFlow** (Neural-network classifier using kmer genomic signatures to label contigs as plasmid/chromosome with phylum-level classes) — https://github.com/smaegol/PlasFlow
- **filter_sequences_by_length.pl** (Perl/Bioperl script that filters the assembly to contigs longer than 1000 bp before classification) — https://github.com/smaegol/PlasFlow
- **cBar** (SOM-based kmer signature classifier used as a comparison baseline for plasmid identification)
- **SPAdes** (Assembler producing the scaffold FASTA inputs for classification)

## Examples

```
filter_sequences_by_length.pl -input input_dataset.fasta -output filtered_output.fasta -thresh 1000; PlasFlow.py --input filtered_output.fasta --output test.plasflow_predictions.tsv --threshold 0.7
```

## Evaluation signals

- Plasmid:chromosome contig ratios on the four reference plasmidomes match 15.73 (Brown Kav), 13.55 (Szczepanowski), 4.72 (Sentchilo), 2.83 (Zhang) for PlasFlow vs ~1.81, 1.72, 0.69, 1.40 for cBar
- Recall on the 42-genome benchmark reaches ~85.98% with precision ~72.17%, exceeding cBar (76.82%/60.51%) and Recycler (12.0%/30.00%)
- Shared-prediction statistics with cBar reproduce ~44% overlap with ~85% of shared contigs labeled plasmid
- Classification accuracy on synthetic test fragments reaches up to 96%
- Contigs below the 0.7 probability threshold are correctly binned as unclassified rather than assigned

## Limitations

- Short sequences (<1000 bp) give unreliable predictions because proper kmer coverage is hard to obtain
- The project is NOT MAINTAINED and pinned to old dependencies (Python 3.5, TensorFlow 0.10.0, scikit-learn 0.18.1, R 3.25)
- The model's performance depends on training data coverage and 'can only be overcome by regularly updating the model with newly published genomic data'
- For large datasets, batch_size may need to be reduced if the new version still causes problems

## Evidence
