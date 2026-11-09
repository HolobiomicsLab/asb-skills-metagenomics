---
name: sequence-length-filtering-of-assembly-contigs
description: Use when a metagenomic assembly (contigs/scaffolds in FASTA, e.g. KOW/ZS
  mine assemblies from MG-RAST mgp80771) is about to be classified by kmer-signature
  tools such as PlasFlow and contains many short contigs.
when_to_use_negative:
- Input is raw sequencing reads rather than assembled contigs (use read trimming tools
  like sickle instead)
- Short sequences themselves are the classification target — the PlasFlow authors
  state results for short sequences are unreliable for kmer-based methods
- The downstream tool requires an assembly graph and paired reads (e.g. PlasmidSpades
  or Recycler), which does not use contig FASTA filtered this way
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: PlasFlow
  role: Provides the filter_sequences_by_length.pl Perl script used to remove contigs
    <= 1000 bp prior to neural-network plasmid classification
  repo: https://github.com/smaegol/PlasFlow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:9f80ad8ecd3997cf84dc8b6434abb653b9694bb7cd3b7cad983de7a9661f92cb
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# sequence-length-filtering-of-assembly-contigs

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pre-classification filtering of metagenomic assembly contigs by length (retain only sequences longer than 1000 bp) before PlasFlow plasmid/chromosome classification, because kmer-based classifiers perform poorly on short contigs.

## When to use

Use when a metagenomic assembly (contigs/scaffolds in FASTA, e.g. KOW/ZS mine assemblies from MG-RAST mgp80771) is about to be classified by kmer-signature tools such as PlasFlow and contains many short contigs.

## When NOT to use

- Input is raw sequencing reads rather than assembled contigs (use read trimming tools like sickle instead)
- Short sequences themselves are the classification target — the PlasFlow authors state results for short sequences are unreliable for kmer-based methods
- The downstream tool requires an assembly graph and paired reads (e.g. PlasmidSpades or Recycler), which does not use contig FASTA filtered this way

## Inputs

- Metagenomic assembly contigs/scaffolds in FASTA format (e.g. KOW/ZS microbial mat assemblies from MG-RAST project mgp80771)
- Length threshold (1000 bp per the PlasFlow workflow)

## Outputs

- Filtered FASTA file containing only contigs longer than 1000 bp, suitable as PlasFlow --input

## How to apply

Run the PlasFlow-provided Perl script filter_sequences_by_length.pl on the assembly FASTA with a length threshold of 1000 bp, producing a filtered FASTA that retains only contigs longer than 1 kb. The rationale, per the PlasFlow authors, is that kmer-frequency signature methods cannot obtain proper kmer coverage from short sequences, so predictions on them are unreliable; filtering also speeds up PlasFlow and reduces RAM use. Feed the filtered FASTA directly into PlasFlow.py (e.g. --threshold 0.7) rather than the full assembly. Tune the --batch_size option of PlasFlow if memory issues remain after filtering large assemblies.

## Related tools

- **PlasFlow** (Provides the filter_sequences_by_length.pl Perl script used to remove contigs <= 1000 bp prior to neural-network plasmid classification) — https://github.com/smaegol/PlasFlow

## Examples

```
filter_sequences_by_length.pl -input KOW_assembly.fasta -output KOW_filtered.fasta -thresh 1000
```

## Evaluation signals

- The filtered output FASTA contains no sequence shorter than or equal to 1000 bp
- The output retains FASTA format with contig headers identical to the input, only with short entries removed
- The filtered file is the one passed to PlasFlow.py (--input filtered_output.fasta), not the original assembly
- Short-sequence count removed matches the expected distribution of short contigs in the metagenomic assembly (assemblies usually contain large numbers of short contigs)
- Downstream plasflow_predictions.tsv is produced without memory exhaustion; if RAM issues arise, adjust --batch_size

## Limitations

- Filtering discards all contigs at or below the 1000 bp threshold, so plasmid-derived short contigs are excluded from any plasmidome fraction estimates
- filter_sequences_by_length.pl requires Perl 5 with Bioperl and Getopt modules installed
- Filtering alone does not resolve memory problems on very large datasets; the README notes smaller --batch_size values in PlasFlow may still be needed

## Evidence

- [other] Filter each assembly with the provided Perl script filter_sequences_by_length.pl, using a length threshold of 1000 bp to retain only contigs longer than 1 kb.: [evidence span withheld — anchor status: not_found]
