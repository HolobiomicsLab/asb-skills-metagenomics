---
name: viral-reference-database-preparation
description: Use when installing or rebuilding the VirMAP pipeline's reference databases
  before processing metagenomic samples, or when Genbank gbvrl/gbphage content has
  grown and the prebuilt viral nucleotide/protein databases must be regenerated.
when_to_use_negative:
- When the VirMAP prebuilt databases already exist and match the desired Genbank snapshot
  — rebuilding wastes significant SSD/RAM provisioning time
- When the analysis targets non-viral taxonomy only, since these databases span only
  the viral superkingdom (gbvrl/gbphg divisions)
- When working with a custom non-Genbank reference set not supported by the virmapInstall.pl
  pipeline
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: VirMAP
  role: Perl-implemented pipeline whose installer (virmapInstall.pl) builds and manages
    the prebuilt reference databases
  repo: https://github.com/cmmr/VirMAP
- name: DIAMOND
  role: Provides the .dmnd format used for virDmnd.dmnd and gbBlastx.dmnd translated-alignment
    databases
  repo: https://github.com/bbuchfink/diamond
- name: BBMap
  role: Nucleotide aligner consuming the virBbmap/gbBlastn database indexes built
    from gbvrl/gbphg
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:42171d048ef0a977968dfaafacb9f2ff4f2a7350d1f66b74b92651d44749d3aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cmmr/VirMAP
  tools:
  - BBMap
  - DIAMOND
  - MEGAHIT
  - Perl
  - Tadpole
  - VirMAP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# viral-reference-database-preparation

## Summary

Prepare the VirMAP reference databases (Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd) from the Genbank viral (gbvrl) and phage (gbphg) divisions so that metagenomic reads can be taxonomically classified by both nucleotide and translated alignments.

## When to use

Use when installing or rebuilding the VirMAP pipeline's reference databases before processing metagenomic samples, or when Genbank gbvrl/gbphage content has grown and the prebuilt viral nucleotide/protein databases must be regenerated.

## When NOT to use

- When the VirMAP prebuilt databases already exist and match the desired Genbank snapshot — rebuilding wastes significant SSD/RAM provisioning time
- When the analysis targets non-viral taxonomy only, since these databases span only the viral superkingdom (gbvrl/gbphg divisions)
- When working with a custom non-Genbank reference set not supported by the virmapInstall.pl pipeline

## Inputs

- Genbank viral (gbvrl) division sequences
- Genbank phage (gbphg) division sequences
- VirMAP installer script (virmapInstall.pl) and configuration files

## Outputs

- Taxonomy.virmap taxonomy database
- virDmnd.dmnd DIAMOND protein database
- virBbmap BBMap nucleotide reference index
- gbBlastn nucleotide alignment database
- gbBlastx.dmnd translated-alignment DIAMOND database

## How to apply

Install VirMAP via the virmapInstall.pl script from the cmmr/virmap repository, which builds the prebuilt database set — Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, and gbBlastx.dmnd — sourced from the Genbank viral (gbvrl) and phage (gbphg) divisions. The nucleotide side (gbBlastn, virBbmap) supports BBMap-based nucleotide alignment, while the protein side (virDmnd.dmnd, gbBlastx.dmnd) is consumed by DIAMOND for translated alignments, which is what lets VirMAP classify low-coverage and divergent viruses independent of genome coverage or read overlap. Provision a large SSD and at least 64 GB RAM per simultaneous VirMAP instance before installation, because the database scales with Genbank growth. After installation, verify that all five database files exist and that a test run (e.g., sample SRR9875293) with Virmap.pl --useMegahit --useBbnorm completes alignment against both the nucleotide and protein databases.

## Related tools

- **VirMAP** (Perl-implemented pipeline whose installer (virmapInstall.pl) builds and manages the prebuilt reference databases) — https://github.com/cmmr/VirMAP
- **DIAMOND** (Provides the .dmnd format used for virDmnd.dmnd and gbBlastx.dmnd translated-alignment databases) — https://github.com/bbuchfink/diamond
- **BBMap** (Nucleotide aligner consuming the virBbmap/gbBlastn database indexes built from gbvrl/gbphg)

## Examples

```
perl virmapInstall.pl
```

## Evaluation signals

- All five expected database artifacts exist on disk: Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd
- A test run on a known sample (e.g., SRR9875293) with --useMegahit --useBbnorm completes both nucleotide and translated alignment stages without database errors
- The installed databases report Genbank gbvrl and gbphg divisions as their source
- Reproduction of influenza runs on BioProject PRJEB7888 yields 509,039 recruited influenza reads, matching the published database-backed result

## Limitations

- As Genbank expands, the minimum SSD and RAM requirements will expand as well, so database size and provisioning must be re-checked at each rebuild
- No software version or Genbank snapshot/download date for the database build is specified in the article or README, complicating exact reproducibility
- Supplementary Note 4 describing the 16-division custom Genbank database is not provided in the main text

## Evidence

- [intro] clustered nucleotide and translated alignments to the Genbank viral and phage divisions (gbvrl and gbphage): "A putative list of viral genomes and protein pseudo-scaffolds are constructed from clustered nucleotide and translated alignments to the Genbank viral and phage divisions (gbvrl and gbphage)."
- [discussion] sourced from Genbank viral and phage divisions: "sourced from the Genbank viral (gbvrl) and phage (gbphg) divisions"
- [methods] resource requirements grow with Genbank: "As Genbank expands, the minimum SSD and RAM requirements will expand as well."
- [methods] VirMAP installer availability: "https://raw.githubusercontent.com/cmmr/virmap/master/virmapInstall.pl"
