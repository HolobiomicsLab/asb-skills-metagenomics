---
name: fasta-sequence-binning-and-export
description: Use when a PlasFlow predictions TSV assigns contigs to plasmid/chromosome/unclassified
  classes and you need per-class FASTA bins plus sequence-count and length-fraction
  summaries per sample.
when_to_use_negative:
- Input is raw short reads rather than assembled contigs — kmer-based PlasFlow results
  on sequences shorter than 1000 bp are unreliable
- You need paired-read or assembly-graph-based plasmid recovery (e.g. circular plasmid
  reconstruction), where PlasmidSpades or Recycler are the appropriate tools
- Input sequences are already binned or a feature table exists — reclassification
  is unnecessary
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: PlasFlow
  role: Neural-network classifier that assigns each contig to one of 26 chromosome/plasmid
    phylum-level classes and produces the predictions TSV and binned FASTA files
  repo: https://github.com/smaegol/PlasFlow
- name: filter_sequences_by_length.pl
  role: Perl script (Bioperl-based) that retains only contigs longer than the 1000
    bp threshold before classification and binning
  repo: https://github.com/smaegol/PlasFlow
- name: SPAdes
  role: Assembler producing the contig/scaffold FASTA files that serve as upstream
    input for classification and binning
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:fe9484a194185712a9ff826e61fdb30d9c7d26d7923820d7e68f64545ec59672
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
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# fasta-sequence-binning-and-export

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Export PlasFlow-classified metagenomic contigs into plasmid, chromosome, and unclassified FASTA bins and summarize the fraction of sequences and cumulative scaffold length assigned to each bin, as done for the KOW and ZS heavy-metal mine metagenomes.

## When to use

Use when a PlasFlow predictions TSV assigns contigs to plasmid/chromosome/unclassified classes and you need per-class FASTA bins plus sequence-count and length-fraction summaries per sample.

## When NOT to use

- Input is raw short reads rather than assembled contigs — kmer-based PlasFlow results on sequences shorter than 1000 bp are unreliable
- You need paired-read or assembly-graph-based plasmid recovery (e.g. circular plasmid reconstruction), where PlasmidSpades or Recycler are the appropriate tools
- Input sequences are already binned or a feature table exists — reclassification is unnecessary

## Inputs

- PlasFlow prediction TSV (plasflow_predictions.tsv) with contig_id, label, and per-class probability columns
- Length-filtered assembly FASTA (contigs >1000 bp) used as PlasFlow input
- Per-sample contig-to-sample mapping (e.g. Supplemental Table S3 accessions from MG-RAST project mgp80771)

## Outputs

- plasmids.fasta — input sequences binned to plasmids
- chromosomes.fasta — input sequences binned to chromosomes
- unclassified.fasta — sequences below the 0.7 probability threshold
- Per-sample summary table of plasmid fractions by sequence count and cumulative scaffold length

## How to apply

After running PlasFlow.py on length-filtered contigs (>1000 bp) with --threshold 0.7, parse the tabular output containing contig_id, label, and per-class probability columns. Partition the input filtered FASTA into three files (plasmids, chromosomes, unclassified), writing each contig to the file matching its label; contigs with assignment probability below 0.7 fall into the unclassified bin. For each sample, compute the percentage of sequences and the percentage of cumulative scaffold length assigned to the plasmid bin. Rationale: probability thresholding excludes uncertain predictions, so the exported bins reflect only high-confidence plasmid/chromosome calls. Validate against reported ranges — for the six KOW/ZS samples, plasmid-origin fractions of 25.02–27.05% of sequences and 14.94–20.66% of cumulative scaffold length are expected.

## Related tools

- **PlasFlow** (Neural-network classifier that assigns each contig to one of 26 chromosome/plasmid phylum-level classes and produces the predictions TSV and binned FASTA files) — https://github.com/smaegol/PlasFlow
- **filter_sequences_by_length.pl** (Perl script (Bioperl-based) that retains only contigs longer than the 1000 bp threshold before classification and binning) — https://github.com/smaegol/PlasFlow
- **SPAdes** (Assembler producing the contig/scaffold FASTA files that serve as upstream input for classification and binning)

## Examples

```
PlasFlow.py --input filtered.fasta --output plasflow_predictions.tsv --threshold 0.7 && filter_sequences_by_length.pl -input input_dataset.fasta -output filtered_output.fasta -thresh 1000
```

## Evaluation signals

- plasflow_predictions.tsv exists in tabular format with contig_id, label, and per-class probability columns
- Every contig from the filtered input FASTA appears in exactly one of the three output FASTA files (plasmids, chromosomes, unclassified) — no duplicates or losses
- All sequences in the unclassified bin had assignment probability below the 0.7 threshold
- Computed plasmid fractions per sample fall within the reported ranges (25.02–27.05% of sequences; 14.94–20.66% of cumulative scaffold length for KOW/ZS samples)
- Summed scaffold lengths across the three bins equal the total length of the filtered input FASTA

## Limitations

- PlasFlow results on short sequences (<1000 bp) are unreliable because kmer coverage is inadequate; filtering short contigs is required before binning
- Below the 0.7 probability threshold, contigs cannot be confidently assigned and are exported as unclassified, reducing the classified fraction
- The project is no longer maintained ('NOT MAINTAINED'); for large datasets the README recommends smaller --batch_size values if problems occur
- Reported plasmid fraction ranges are sample-specific (KOW/ZS mine metagenomes) and may not generalize to other environments

## Evidence

- [supplementary] Validation against reported KOW/ZS plasmid fractions: [evidence span withheld — anchor status: not_found]
