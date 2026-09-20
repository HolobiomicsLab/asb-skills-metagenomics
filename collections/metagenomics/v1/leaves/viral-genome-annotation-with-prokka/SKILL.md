---
name: viral-genome-annotation-with-prokka
description: Use when raw (unannotated) viral genome files must be prepared as input
  for HUBDesign's clustering phase, which takes annotated genomes plus a GenomeInfo.tab
  mapping annotated-genome paths to NCBI taxon-ids.
when_to_use_negative:
- Genomes are already annotated (e.g. RefSeq GFF available) — HUBDesign takes annotated
  genomes directly and re-annotation is unnecessary
- Target organisms have unknown or unannotatable genomes — the paper notes probes
  cannot be designed for such targets, only for close relatives with annotated genomes
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_0080
tools:
- name: PROKKA
  role: 'Suggested annotation tool: runs on raw genome FASTA files with kingdom set
    to viruses to produce the annotated genomes HUBDesign requires'
  repo: github.com/zacherydickson/HUBDesign
- name: HUBDesign
  role: Downstream consumer of the PROKKA-annotated genomes via bin/HUBDesign GenomeInfo.tab,
    which collapses gene clusters and selects tiled probes
  repo: github.com/zacherydickson/HUBDesign
- name: Perl 5.10.0+
  role: Runtime dependency of the HUBDesign pipeline that consumes the annotated genomes
- name: BioPerl 1.7.10+
  role: Runtime dependency of the HUBDesign pipeline that parses the PROKKA annotation
    files
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100069
    title: Probe design for simultaneous, targeted capture of diverse metagenomic
      targets
  source_content_hash: sha256:30d16e17f0cfde1a52babee36c39b740043ae9b64e5488872cd8d05392723c8c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/zacherydickson/HUBDesign
  tools:
  - BioPerl
  - BioPerl 1.7.10+
  - HUBDesign
  - MAFFT
  - PROKKA
  - Perl
  - Perl 5.10.0+
  - SA_BOND
  - sdust
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# viral-genome-annotation-with-prokka

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate raw viral genome FASTA files with PROKKA (kingdom set to viruses) to produce the GFF annotations and taxon-id mapping that HUBDesign requires as input for probe design.

## When to use

Use when raw (unannotated) viral genome files must be prepared as input for HUBDesign's clustering phase, which takes annotated genomes plus a GenomeInfo.tab mapping annotated-genome paths to NCBI taxon-ids.

## When NOT to use

- Genomes are already annotated (e.g. RefSeq GFF available) — HUBDesign takes annotated genomes directly and re-annotation is unnecessary
- Target organisms have unknown or unannotatable genomes — the paper notes probes cannot be designed for such targets, only for close relatives with annotated genomes

## Inputs

- Raw viral genome FASTA files (e.g. 56 RefSeq coronavirus genome files)
- NCBI taxon-ids for each genome (used to name annotated outputs and build GenomeInfo.tab)
- PROKKA installed with kingdom option supporting viruses

## Outputs

- PROKKA-annotated genome files (GFF and associated annotation outputs), one per input genome, named by taxon-id
- A tab-delimited GenomeInfo.tab file mapping each annotated genome path to its NCBI taxon-id

## How to apply

Run PROKKA on each raw genome FASTA with the kingdom parameter set to viruses, as done for the 56 RefSeq coronavirus genomes in the HUBDesign test directory. Name each annotated output file by the genome's NCBI taxon-id so that a tab-delimited GenomeInfo.tab file (columns: path-to-annotated-genome, taxon-id) can be generated, e.g. by printing the path and basename of each file with awk. These annotated genomes feed HUBDesign's Clustering.pl step, which collapses gene sequences at most 15% divergent (r2t-divergence) into representative sequences via MAFFT; annotation quality therefore directly determines which candidate probes SA_BOND can later identify per pseudo-genome. Verify that each genome produces a valid GFF annotation before constructing the guide tree from NCBI taxonomy lineages.

## Related tools

- **PROKKA** (Suggested annotation tool: runs on raw genome FASTA files with kingdom set to viruses to produce the annotated genomes HUBDesign requires) — github.com/zacherydickson/HUBDesign
- **HUBDesign** (Downstream consumer of the PROKKA-annotated genomes via bin/HUBDesign GenomeInfo.tab, which collapses gene clusters and selects tiled probes) — github.com/zacherydickson/HUBDesign
- **Perl 5.10.0+** (Runtime dependency of the HUBDesign pipeline that consumes the annotated genomes)
- **BioPerl 1.7.10+** (Runtime dependency of the HUBDesign pipeline that parses the PROKKA annotation files)

## Examples

```
prokka --kingdom Viruses --outdir annotated genome.fasta && awk '{print $1"\t"$2}' annotated/*.gff.list > GenomeInfo.tab
```

## Evaluation signals

- Each raw FASTA yields a valid PROKKA annotation directory containing a parseable GFF with the kingdom set to viruses
- Annotated files are named by NCBI taxon-id so the GenomeInfo.tab correctly pairs each path with its taxon-id
- HUBDesign's clustering phase accepts the annotated GFFs without format errors and produces RepSeq.fna cluster sequences
- Downstream run on the 56 annotated RefSeq coronavirus genomes reproduces the reference probe set (13,500 probes of 75 nt, 25.0% nucleotide coverage, 4.72x depth, efficiency 1.87)

## Limitations

- PROKKA annotation quality bounds HUBDesign performance: probes cannot be designed for targets with unknown or unannotatable genomes, only for close relatives
- The published coronavirus/sepsis probe sets were produced with earlier HUBDesign versions; the current version constructs a new cluster-assignment tree, so results may differ from the validated outputs
- Mispredicted genes or missed genes from annotation propagate through clustering (15% divergence cutoff) and may reduce probe coverage or specificity

## Evidence
