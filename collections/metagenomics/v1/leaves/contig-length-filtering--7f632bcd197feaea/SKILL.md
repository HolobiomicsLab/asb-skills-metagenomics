---
name: contig-length-filtering
description: Use when preparing metagenomic or isolate assembly contigs/scaffolds
  (e.g. SPAdes output) as input for PlasFlow or other kmer-signature classifiers that
  perform poorly on short sequences.
when_to_use_negative:
- Input sequences are already short reads or already length-filtered below 1 kb targets
  — filtering would discard most data rather than improve classification
- The downstream tool does not depend on kmer coverage of contigs (e.g. read-based
  or assembly-graph methods such as Recycler or PlasmidSpades) — the length filter
  is a PlasFlow-specific preprocessing recommendation
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: PlasFlow
  role: Downstream consumer of the length-filtered FASTA; classification accuracy
    on contigs >1 kb is the reason for filtering
  repo: https://github.com/smaegol/PlasFlow
- name: filter_sequences_by_length.pl
  role: Perl script bundled with PlasFlow that performs the length filtering using
    Bioperl
  repo: https://github.com/smaegol/PlasFlow
- name: SPAdes
  role: Produces the scaffold FASTA (e.g. 240 scaffolds for Aeromonas sp. O23A) that
    is filtered
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:be2e701ec03bfa349a463eed4808d12f298431c716c32ad70f2501109f097ffd
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# contig-length-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filter an assembly (FASTA) by sequence length, retaining only contigs longer than a 1000 bp threshold, using the PlasFlow-provided Perl script filter_sequences_by_length.pl. This is a prerequisite step because kmer-based plasmid classifiers such as PlasFlow produce unreliable predictions on short contigs.

## When to use

Use when preparing metagenomic or isolate assembly contigs/scaffolds (e.g. SPAdes output) as input for PlasFlow or other kmer-signature classifiers that perform poorly on short sequences.

## When NOT to use

- Input sequences are already short reads or already length-filtered below 1 kb targets — filtering would discard most data rather than improve classification
- The downstream tool does not depend on kmer coverage of contigs (e.g. read-based or assembly-graph methods such as Recycler or PlasmidSpades) — the length filter is a PlasFlow-specific preprocessing recommendation

## Inputs

- Assembly FASTA file of contigs or scaffolds (e.g. SPAdes scaffolds.fasta)
- Sequence length threshold in bp (study used 1000 bp)

## Outputs

- Filtered FASTA containing only sequences longer than the threshold (e.g. 36 sequences >1 kb from 240 SPAdes scaffolds)
- Input for PlasFlow.py classification (--input)

## How to apply

Run filter_sequences_by_length.pl with -input pointing to the assembly FASTA (e.g. 240 SPAdes scaffolds), -output to a filtered FASTA, and -thresh set to the sequence length threshold in bp (1000 in the published workflow). In the Aeromonas sp. O23A case this reduced 240 scaffolds to 36 sequences >1 kb, which were then classified by PlasFlow.py with --threshold 0.7. The rationale, stated in the README, is that kmer-based methods cannot obtain proper kmer coverage on short sequences, so results there are unreliable, and filtering also speeds up PlasFlow and prevents excessive RAM usage. The script requires Perl 5 with Bioperl and Getopt modules. Downstream, expect the filtered FASTA to contain only sequences strictly longer than the threshold; verify by checking sequence lengths with any FASTA stats tool.

## Related tools

- **PlasFlow** (Downstream consumer of the length-filtered FASTA; classification accuracy on contigs >1 kb is the reason for filtering) — https://github.com/smaegol/PlasFlow
- **filter_sequences_by_length.pl** (Perl script bundled with PlasFlow that performs the length filtering using Bioperl) — https://github.com/smaegol/PlasFlow
- **SPAdes** (Produces the scaffold FASTA (e.g. 240 scaffolds for Aeromonas sp. O23A) that is filtered)

## Examples

```
filter_sequences_by_length.pl -input input_dataset.fasta -output filtered_output.fasta -thresh 1000
```

## Evaluation signals

- All sequences in the filtered output FASTA are strictly longer than the threshold (1000 bp in the study)
- Sequence count drops as expected: 240 SPAdes scaffolds reduced to 36 sequences >1 kb in the Aeromonas sp. O23A workflow
- Filtered FASTA is valid FASTA parseable by Bioperl and accepted by PlasFlow.py without errors
- Downstream PlasFlow predictions come only from retained contigs; below-threshold-length contigs are absent from the prediction TSV

## Limitations

- Results for short sequences are unreliable for kmer-based methods, so the filter trades sensitivity on small plasmids (some known plasmids are only 4-60 kb but their contigs may fall near the cutoff) for prediction reliability
- Filtering large assemblies without pre-filtering can cause too high RAM usage in PlasFlow; the README notes this as a motivation for the step
- The Perl script requires Bioperl and Getopt modules, which must be installed (conda -c bioconda perl-bioperl perl-getopt-long or cpan)

## Evidence
