---
name: cross-tool-prediction-comparison
description: Use when two plasmid-prediction tools have been run on identical contig/fragment
  sets and you must quantify error rates, agreement, and plasmid:chromosome separation
  ratios to decide which classifier to trust.
when_to_use_negative:
- Inputs are raw sequencing reads rather than assembled contigs/fragments — kmer-based
  classifiers are unreliable on short sequences
- Only one classifier's output exists and there is no ground truth or second tool
  to compare against
- The goal is replicon typing of known plasmids in isolate assemblies (use PlasmidFinder)
  rather than metagenomic plasmid/chromosome separation
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: PlasFlow
  role: Neural-network-based plasmid classifier providing reference predictions with
    --threshold 0.7 probability filtering
  repo: https://github.com/smaegol/PlasFlow
- name: cBar
  role: Self-organizing-map genomic-signature classifier whose chromosomal/plasmid
    calls are compared against PlasFlow
- name: filter_sequences_by_length.pl
  role: Perl script filtering assemblies to contigs longer than 1000 bp before classification,
    since kmer-based methods fail on short sequences
  repo: https://github.com/smaegol/PlasFlow
- name: SPAdes
  role: Assembler used to produce the Zhang and Szczepanowski plasmidome assemblies
    that serve as comparison inputs
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:a11d661e139cc68749a2c57d96ed7e5db82167143610bc0d0c6f4df564ff7912
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
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# cross-tool-prediction-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark plasmid-classification tools (PlasFlow vs cBar) against each other and ground truth by computing false-positive/false-negative rates, prediction concordance, and plasmid-to-chromosome ratios on the same fragment sets and plasmidome assemblies.

## When to use

Use when two plasmid-prediction tools have been run on identical contig/fragment sets and you must quantify error rates, agreement, and plasmid:chromosome separation ratios to decide which classifier to trust.

## When NOT to use

- Inputs are raw sequencing reads rather than assembled contigs/fragments — kmer-based classifiers are unreliable on short sequences
- Only one classifier's output exists and there is no ground truth or second tool to compare against
- The goal is replicon typing of known plasmids in isolate assemblies (use PlasmidFinder) rather than metagenomic plasmid/chromosome separation

## Inputs

- Chromosome and plasmid fragment validation datasets downloaded from RefSeq with known fragment origin
- Reference plasmidome assemblies: Brown Kav (MG-RAST 4460391.3), Szczepanowski (FTP, SPAdes), Sentchilo (MG-RAST 4474000.3), Zhang (SRA SRP007256.1, SPAdes)
- Contigs filtered to >1000 bp (FASTA)
- Per-tool prediction tables (PlasFlow TSV predictions; cBar chromosomal/plasmid calls)

## Outputs

- False-positive and false-negative rates per tool on the validation dataset (PlasFlow: 2.21% FP / 3.00% FN; cBar: 6.46% FP / 15.14% FN)
- Inter-tool concordance fraction (~79% of predictions consistent without probability filtering)
- Plasmid:chromosome contig ratios per dataset and tool (PlasFlow: 15.73, 13.55, 4.72, 2.83; cBar: 1.81, 1.72, 0.69, 1.40)
- Fraction of contigs predicted by both tools and fraction of shared predictions labeled plasmid (~44% overlap, ~85% shared plasmid)

## How to apply

Run both classifiers on identical inputs: cBar (genomic-signature SOM-based) and PlasFlow.py with --threshold 0.7 (default probability cutoff labeling low-confidence contigs unclassified). On labeled validation fragments (RefSeq chromosome/plasmid fragments of varying lengths), score each tool's calls as true/false positive or true/false negative against known fragment origin to obtain FPR/FNR. Quantify concordance as the fraction of fragments assigned the same class by both tools, both unfiltered (expect ~79% agreement) and probability-filtered. On the four reference plasmidome assemblies (Brown Kav MG-RAST 4460391.3, Szczepanowski, Sentchilo MG-RAST 4474000.3, Zhang SRA SRP007256.1, all SPAdes-assembled and filtered to contigs >1000 bp with filter_sequences_by_length.pl), compute plasmid:chromosome contig ratios per tool and the fraction of contigs predicted by both (expect ~44% overlap with ~85% of shared contigs labeled plasmid).

## Related tools

- **PlasFlow** (Neural-network-based plasmid classifier providing reference predictions with --threshold 0.7 probability filtering) — https://github.com/smaegol/PlasFlow
- **cBar** (Self-organizing-map genomic-signature classifier whose chromosomal/plasmid calls are compared against PlasFlow)
- **filter_sequences_by_length.pl** (Perl script filtering assemblies to contigs longer than 1000 bp before classification, since kmer-based methods fail on short sequences) — https://github.com/smaegol/PlasFlow
- **SPAdes** (Assembler used to produce the Zhang and Szczepanowski plasmidome assemblies that serve as comparison inputs)

## Examples

```
filter_sequences_by_length.pl -input assembly.fasta -output filtered.fasta -thresh 1000 && PlasFlow.py --input filtered.fasta --output predictions.tsv --threshold 0.7
```

## Evaluation signals

- PlasFlow validation false-positive rate ≈2.21% and false-negative rate ≈3.00%
- cBar validation false-positive rate ≈6.46% and false-negative rate ≈15.14%
- Concordance between the two tools ≈79% of predictions consistent (unfiltered PlasFlow)
- PlasFlow plasmid:chromosome ratios on reference plasmidomes match 15.73 (Brown Kav), 13.55 (Szczepanowski), 4.72 (Sentchilo), 2.83 (Zhang), all exceeding cBar's 1.81, 1.72, 0.69, 1.40
- Shared-prediction statistics reproduce approximately 44% overlap with 85% of shared contigs classified as plasmid

## Limitations

- PlasFlow and other kmer-based classifiers perform unreliably on sequences shorter than 1000 bp, so contig length filtering is mandatory before comparison
- With --threshold 0.7, contigs below the probability cutoff are labeled unclassified; concordance differs depending on whether probability filtering is applied
- Conclusions may not transfer to novel taxa: the models must be regularly updated with newly published genomic data
- PlasFlow is no longer maintained (version 1.1, 2018); the README warns to use it at your own risk

## Evidence
