---
name: plasmid-chromosome-ratio-calculation
description: Use when you need to quantify and compare plasmid content across metagenomic/plasmidome
  assemblies by computing plasmid:chromosome contig ratios from kmer-signature-based
  classifiers such as PlasFlow and cBar.
when_to_use_negative:
- Input contains large numbers of contigs shorter than 1000 bp and you cannot filter
  — kmer-based classification results on short sequences are unreliable
- You require recovery of circular plasmids from assembly graphs with paired reads
  — use PlasmidSpades or Recycler instead, which use the assembly graph rather than
  genomic signatures
- You need replicate/plasmid replicon typing of known plasmid families — PlasmidFinder
  is a replicon-identification web tool, not a contig-level plasmid/chromosome classifier
tools:
- name: PlasFlow
  role: Neural-network classifier run with --threshold 0.7 on filtered assemblies
    to produce plasmid/chromosome/unclassified bins used for ratio computation
  repo: https://github.com/smaegol/PlasFlow
- name: cBar
  role: SOM-based comparator run on the same filtered assemblies to benchmark plasmid/chromosome
    ratios against PlasFlow
- name: filter_sequences_by_length.pl
  role: Perl script (Bioperl-based) that pre-filters assemblies to contigs longer
    than 1000 bp before classification
  repo: https://github.com/smaegol/PlasFlow
- name: SPAdes
  role: Assembler used to generate the Zhang and Szczepanowski plasmidome assemblies
    that serve as inputs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:1b7465ae876e706e6ed99bf9977249d9fd5eb2b4a75570185faa546aa7628c67
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# plasmid-chromosome-ratio-calculation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies plasmid enrichment in metagenomic assemblies by classifying filtered contigs with PlasFlow (and optionally cBar) into plasmid, chromosome, and unclassified bins, then computing plasmid-to-chromosome contig ratios as a comparative metric across plasmidome datasets.

## When to use

Use when you need to quantify and compare plasmid content across metagenomic/plasmidome assemblies by computing plasmid:chromosome contig ratios from kmer-signature-based classifiers such as PlasFlow and cBar.

## When NOT to use

- Input contains large numbers of contigs shorter than 1000 bp and you cannot filter — kmer-based classification results on short sequences are unreliable
- You require recovery of circular plasmids from assembly graphs with paired reads — use PlasmidSpades or Recycler instead, which use the assembly graph rather than genomic signatures
- You need replicate/plasmid replicon typing of known plasmid families — PlasmidFinder is a replicon-identification web tool, not a contig-level plasmid/chromosome classifier

## Inputs

- Metagenomic plasmidome assembly FASTA files (e.g. MG-RAST 4460391.3, MG-RAST 4474000.3, SRA SRP007256.1)
- Filtered assemblies with contigs > 1000 bp (filter_sequences_by_length.pl output)
- cBar plasmid/chromosome calls on the same filtered assemblies (optional comparison)

## Outputs

- PlasFlow prediction table (per-contig TSV with probabilities and labels)
- Fasta bins of input sequences classified as plasmids, chromosomes, and unclassified
- Plasmid-to-chromosome contig ratio per dataset and tool
- Shared-prediction statistics (fraction of contigs predicted by both tools; fraction of shared predictions labeled plasmid)

## How to apply

Retrieve the target plasmidome assemblies (e.g. Brown Kav MG-RAST 4460391.3, Sentchilo MG-RAST 4474000.3, Zhang SRP007256.1, Szczepanowski FTP download), then filter each assembly with filter_sequences_by_length.pl leaving only contigs longer than 1000 bp, since kmer-based classifiers are unreliable on shorter sequences. Run PlasFlow.py on each filtered assembly with --threshold 0.7 (the default probability cutoff below which contigs are labeled unclassified) to obtain per-contig predictions plus plasmid/chromosome/unclassified fasta bins. Optionally run cBar on the same filtered assemblies for comparison. Compute the plasmid:chromosome contig ratio per dataset and tool, and additionally calculate the fraction of contigs predicted by both tools and the fraction of shared predictions labeled plasmid.

## Related tools

- **PlasFlow** (Neural-network classifier run with --threshold 0.7 on filtered assemblies to produce plasmid/chromosome/unclassified bins used for ratio computation) — https://github.com/smaegol/PlasFlow
- **cBar** (SOM-based comparator run on the same filtered assemblies to benchmark plasmid/chromosome ratios against PlasFlow)
- **filter_sequences_by_length.pl** (Perl script (Bioperl-based) that pre-filters assemblies to contigs longer than 1000 bp before classification) — https://github.com/smaegol/PlasFlow
- **SPAdes** (Assembler used to generate the Zhang and Szczepanowski plasmidome assemblies that serve as inputs)

## Examples

```
filter_sequences_by_length.pl -input input_dataset.fasta -output filtered_output.fasta -thresh 1000; PlasFlow.py --input filtered_output.fasta --output test.plasflow_predictions.tsv --threshold 0.7
```

## Evaluation signals

- Reproduced PlasFlow plasmid:chromosome ratios match the reported values: 15.73 (Brown Kav), 4.72 (Sentchilo), 13.55 (Szczepanowski), 2.83 (Zhang)
- Reproduced cBar ratios match the reported values: 1.81, 1.72, 0.69, 1.40 for the same four datasets
- Shared-prediction statistics reproduce approximately 44% overlap between the two tools, with ~85% of shared contigs classified as plasmid
- Every input contig appears in exactly one of the three PlasFlow output bins (plasmid, chromosome, unclassified)
- No contig shorter than 1000 bp is present in the input to PlasFlow (verify the filter step ran)

## Limitations

- Kmer-based classifiers such as PlasFlow perform poorly on short sequences because kmer coverage is insufficient; results for contigs shorter than 1000 bp are unreliable
- Contigs below the --threshold 0.7 probability cutoff are labeled unclassified and excluded from the ratio, so ratios depend on the chosen cutoff
- PlasFlow 1.1 is no longer maintained and requires a legacy environment (Python 3.5, TensorFlow 0.10.0rc0); for very large datasets, smaller --batch_size values may be needed

## Evidence
