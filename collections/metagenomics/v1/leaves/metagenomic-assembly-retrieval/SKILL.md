---
name: metagenomic-assembly-retrieval
description: Use when reproducing plasmidome classification benchmarks that require
  downloading the four reference plasmidome assemblies and filtering contigs to >1000
  bp before running kmer-based plasmid/chromosome classifiers.
when_to_use_negative:
- When input contigs are already length-filtered or all exceed 1000 bp, re-filtering
  adds no value
- When the analysis targets plasmid recovery from assembly graphs with paired reads
  (e.g. Recycler or PlasmidSpades), which require an assembly graph rather than retrieved
  reference assemblies
- When analyzing isolate genome assemblies rather than reference plasmidome datasets
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: PlasFlow
  role: Downstream kmer-based plasmid classifier applied to the retrieved and filtered
    plasmidome assemblies with --threshold 0.7
  repo: https://github.com/smaegol/PlasFlow
- name: cBar
  role: Comparison classifier run on the same filtered assemblies to obtain baseline
    plasmid/chromosome calls
- name: SPAdes
  role: Assembler used to produce the Zhang and Szczepanowski reference plasmidome
    assemblies
- name: filter_sequences_by_length.pl
  role: Perl script (provided with PlasFlow) used to keep only contigs longer than
    1000 bp
  repo: https://github.com/smaegol/PlasFlow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:ce2773f5718ef12494b688a1ea15d78d14ecd31924f291c2a92d530e959cb750
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
  edam_operation: http://edamontology.org/operation_0252
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

# metagenomic-assembly-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieves and prepares reference plasmidome metagenomic assemblies (Brown Kav, Sentchilo, Zhang, Szczepanowski) so that kmer-based plasmid classifiers such as PlasFlow and cBar can be applied to comparable inputs.

## When to use

Use when reproducing plasmidome classification benchmarks that require downloading the four reference plasmidome assemblies and filtering contigs to >1000 bp before running kmer-based plasmid/chromosome classifiers.

## When NOT to use

- When input contigs are already length-filtered or all exceed 1000 bp, re-filtering adds no value
- When the analysis targets plasmid recovery from assembly graphs with paired reads (e.g. Recycler or PlasmidSpades), which require an assembly graph rather than retrieved reference assemblies
- When analyzing isolate genome assemblies rather than reference plasmidome datasets

## Inputs

- MG-RAST assembly Brown Kav (accession 4460391.3)
- MG-RAST assembly Sentchilo (accession 4474000.3)
- SRA study SRP007256.1 (Zhang), assembled with SPAdes
- Szczepanowski plasmidome assembly obtained by FTP download, assembled with SPAdes
- FASTA-format metagenomic assemblies

## Outputs

- Filtered FASTA files containing only contigs longer than 1000 bp for each of the four plasmidome assemblies
- Length-filtered assemblies ready for PlasFlow and cBar classification runs

## How to apply

Retrieve each reference plasmidome assembly from its stated source: Brown Kav (MG-RAST 4460391.3), Sentchilo (MG-RAST 4474000.3), Zhang (SRA SRP007256.1, assembled with SPAdes), and Szczepanowski (FTP download, assembled with SPAdes). Filter every assembly with filter_sequences_by_length.pl leaving only contigs longer than 1000 bp, because kmer-based classifiers are unreliable on shorter sequences. Provide the filtered FASTA files to downstream classifiers (e.g. PlasFlow.py with --threshold 0.7) so results are comparable across tools. Expect downstream plasmid:chromosome contig ratios of approximately 15.73, 4.72, 13.55, and 2.83 for PlasFlow and 1.81, 1.72, 0.69, and 1.40 for cBar across the four datasets. Validation: accept the retrieval/preparation step if the reproduced ratios match these reported values and the shared-prediction statistics reproduce roughly 44% tool overlap with ~85% of shared contigs labeled plasmid.

## Related tools

- **PlasFlow** (Downstream kmer-based plasmid classifier applied to the retrieved and filtered plasmidome assemblies with --threshold 0.7) — https://github.com/smaegol/PlasFlow
- **cBar** (Comparison classifier run on the same filtered assemblies to obtain baseline plasmid/chromosome calls)
- **SPAdes** (Assembler used to produce the Zhang and Szczepanowski reference plasmidome assemblies)
- **filter_sequences_by_length.pl** (Perl script (provided with PlasFlow) used to keep only contigs longer than 1000 bp) — https://github.com/smaegol/PlasFlow

## Examples

```
filter_sequences_by_length.pl -input input_dataset.fasta -output filtered_output.fasta -thresh 1000
```

## Evaluation signals

- PlasFlow plasmid:chromosome contig ratios on the four filtered datasets match 15.73 (Brown Kav), 4.72 (Sentchilo), 13.55 (Zhang), and 2.83 (Szczepanowski)
- cBar ratios match approximately 1.81, 1.72, 0.69, and 1.40 on the same datasets
- Fraction of contigs predicted by both tools reproduces roughly 44% overlap, with about 85% of shared contigs labeled plasmid
- Filtered FASTA files contain no contigs shorter than or equal to 1000 bp
- PlasFlow prediction tables assign every retained contig a probability, labeling those below threshold 0.7 as unclassified

## Limitations

- Kmer-based classifiers including PlasFlow do not perform well on short sequences because proper kmer coverage is hard to obtain, hence the 1000 bp cutoff
- Length filtering before PlasFlow is needed to prevent excessive RAM usage on metagenomic assemblies with many short contigs
- Trained model weights and supplementary tables referenced in the article are distributed in separate repositories/files and not inline
- PlasFlow is no longer maintained (NOT MAINTAINED per README); use at your own risk

## Evidence

- [methods] Retrieve the four reference plasmidome assemblies: Brown Kav (MG-RAST 4460391.3), Sentchilo (MG-RAST 4474000.3), Zhang (SRA SRP007256.1, assembled with SPAdes), and Szczepanowski (FTP download, assembled with SPAdes).: [evidence span withheld — anchor status: not_found]
- [results] On the reference plasmidome datasets, PlasFlow obtained plasmid-to-chromosome ratios of 15.73 (Brown Kav), 13.55 (Szczepanowski), 4.72 (Sentchilo), and 2.83 (Zhang): [evidence span withheld — anchor status: not_found]
