---
name: metagenomic-reference-catalog-retrieval
description: Use when a study needs large, publicly available, redundancy-filtered
  reference protein catalogs (e.g. as search database input for HH-suite3/BFD or for
  catalog size/provenance audits) and the input is metagenomic or metatranscriptomic
  protein sequence space.
when_to_use_negative:
- The analysis requires nucleotide contigs or whole-genome assemblies rather than
  protein sequences — use PenguiN nucleotide assembly instead.
- A curated, annotated protein database (e.g. UniProt) suffices; these catalogs are
  redundancy-filtered at 90% but not functionally annotated.
- The metagenomic dataset is small or low-complexity where downloading a 2-billion-sequence
  reference catalog is unnecessary overhead.
edam_operation: http://edamontology.org/operation_2479
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: Plass
  role: Source of the prebuilt SRC and MERC catalogs; also used to re-assemble the
    catalogs from raw paired-end FASTQ reads via 'plass assemble'
  repo: https://github.com/soedinglab/plass
- name: HH-suite3
  role: 'Consumer of the catalogs: sequences from SRC and MERC are included in the
    BFD HH-suite3 database'
  repo: https://github.com/soedinglab/hh-suite
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-019-0437-4
    title: Protein-level assembly increases protein sequence recovery from metagenomic
      samples manyfold
  source_content_hash: sha256:657b862e14a9aa378d57c2b58416a9772bdd2f5af1bf3dbc01b8df9ff19016ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/plass
  tools:
  - HH-suite3
  - Plass
  edam_operation: http://edamontology.org/operation_2479
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# metagenomic-reference-catalog-retrieval

## Summary

Retrieves and verifies the Plass-built reference protein catalogs — the Soil Reference Catalog (SRC, 2 billion sequences from 640 soil metagenome samples) and the Marine Eukaryotic Reference Catalog (MERC, 292 million sequences from 775 TARA ocean metatranscriptomes) — as single FASTA files, confirming their 90%/90% redundancy-reduction provenance.

## When to use

Use when a study needs large, publicly available, redundancy-filtered reference protein catalogs (e.g. as search database input for HH-suite3/BFD or for catalog size/provenance audits) and the input is metagenomic or metatranscriptomic protein sequence space.

## When NOT to use

- The analysis requires nucleotide contigs or whole-genome assemblies rather than protein sequences — use PenguiN nucleotide assembly instead.
- A curated, annotated protein database (e.g. UniProt) suffices; these catalogs are redundancy-filtered at 90% but not functionally annotated.
- The metagenomic dataset is small or low-complexity where downloading a 2-billion-sequence reference catalog is unnecessary overhead.

## Inputs

- Plass current-release catalog FASTA files (SRC and MERC) from http://wwwuser.gwdg.de/~compbiol/plass/current_release/
- Release metadata documenting the 90% sequence identity / 90% coverage redundancy-reduction parameters
- Optional: raw paired-end metagenomic/metatranscriptomic FASTQ reads for re-assembly with plass

## Outputs

- catalog_size_verification.tsv — tab-delimited table with SRC and MERC rows containing sequence counts and sample counts
- Per-catalog SRA identifier lists extracted from FASTA headers confirming sample provenance
- Optionally: freshly assembled protein catalogs (assembly.fas) from plass assemble

## How to apply

Download the SRC and MERC catalog FASTA files from the Plass current-release server (http://wwwuser.gwdg.de/~compbiol/plass/current_release/); each catalog ships as a single FASTA file whose header identifiers embed the SRA identifiers of the source samples. Stream-parse each FASTA, counting sequence records and extracting SRA accessions from headers to confirm sample provenance (640 soil samples for SRC, 775 marine eukaryotic metatranscriptomes from the TARA ocean expedition for MERC). Confirm from release metadata that both catalogs were redundancy reduced to 90% sequence identity at 90% coverage. Optionally, instead of the prebuilt catalogs, re-assemble raw paired-end FASTQ reads with 'plass assemble' to reproduce the catalogs from scratch. Record the counted sequence totals (2 billion for SRC, 292 million for MERC) in a tab-delimited comparison file such as catalog_size_verification.tsv.

## Related tools

- **Plass** (Source of the prebuilt SRC and MERC catalogs; also used to re-assemble the catalogs from raw paired-end FASTQ reads via 'plass assemble') — https://github.com/soedinglab/plass
- **HH-suite3** (Consumer of the catalogs: sequences from SRC and MERC are included in the BFD HH-suite3 database) — https://github.com/soedinglab/hh-suite

## Examples

```
plass assemble reads_1.fastq.gz reads_2.fastq.gz assembly.fas tmp
```

## Evaluation signals

- catalog_size_verification.tsv exists and is tab-delimited with rows for SRC and MERC.
- SRC sequence count is consistent with ~2 billion sequences from 640 soil samples.
- MERC sequence count is consistent with ~292 million sequences from 775 TARA ocean metatranscriptomes.
- Release metadata confirms both catalogs were redundancy reduced at 90% sequence identity and 90% coverage.
- Downloaded FASTA files parse cleanly and header identifiers contain valid SRA accession identifiers.

## Limitations

- Catalogs are redundancy filtered at 90% identity / 90% coverage, so near-identical sequence variants are collapsed and absolute counts differ from raw assembly output.
- Catalog files are large single FASTA files (billions of sequences for SRC); streaming parsing and sufficient disk are required.
- MERC derives from TARA ocean expedition metatranscriptomes, so it represents marine eukaryotic expression space, not genomes.

## Evidence

- [abstract] We assembled two redundancy-filtered reference protein catalogs, 2 billion sequences from 640 soil samples (soil reference protein catalog) and 292 million: "We assembled two redundancy-filtered reference protein catalogs, 2 billion sequences from 640 soil samples (soil reference protein catalog) and 292 million"
- [readme] SRC was created by assembling 640 soil metagenome samples. MERC was assembled from the the metatranscriptomics datasets created by the TARA ocean expedition.: "SRC was created by assembling 640 soil metagenome samples. MERC was assembled from the the metatranscriptomics datasets created by the TARA ocean expedition."
- [readme] Both catalogues were redundancy reduced to 90% sequence identity at 90% coverage.: "Both catalogues were redundancy reduced to 90% sequence identity at 90% coverage."
- [methods] plass assemble examples/reads_1.fastq.gz examples/reads_2.fastq.gz assembly.fas tmp: "plass assemble examples/reads_1.fastq.gz examples/reads_2.fastq.gz assembly.fas tmp"
- [readme] Each catalog is a single FASTA file containing the sequences, the header identifiers contain the Sequence Read Archive (SRA) identifiers.: "Each catalog is a single FASTA file containing the sequences, the header identifiers contain the Sequence Read Archive (SRA) identifiers."
