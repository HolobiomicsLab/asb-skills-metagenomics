---
name: viral-read-alignment-to-genbank-nucleotide-protein-databases
description: Use when metagenomic FASTQ reads must be assigned to known viruses by
  both nucleotide and translated protein alignments to build per-genome read sets
  for mapping assembly.
when_to_use_negative:
- The goal is de-novo assembly of novel viruses with no database representatives —
  VirMAP's mapping assembly tier requires reference genomes; use MEGAHIT or Tadpole
  de-novo assembly instead.
- Input is already assembled contigs rather than raw reads — the read-assignment and
  per-genome pileup logic assumes read-level alignments.
- The target organisms are non-viral (e.g. bacterial or eukaryotic classification)
  — the databases and thresholds here are tuned to the viral superkingdom and VirMAP
  filters contigs mapping outside it.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0781
tools:
- name: VirMAP
  role: Perl-implemented pipeline that orchestrates the tiered nucleotide/protein
    alignment, read assignment, and Jaccard-distance filtering
  repo: https://github.com/cmmr/VirMAP
- name: BBMap
  role: Nucleotide alignment of processed reads to the Genbank viral/phage database
    (90% identity / 5% aligning-bases threshold)
- name: DIAMOND
  role: Translated alignment of reads to the viral protein database (20% identity
    / 8% aligning-bases threshold)
  repo: https://github.com/bbuchfink/diamond
- name: BBTools bbnorm
  role: Dereplication and digital normalization of paired-end input reads with default
    parameters prior to alignment
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:8f8b20cb6c561e774606bd64118599938a90e8c0189b1c44d8f3637f59cd3642
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cmmr/VirMAP
  tools:
  - BBMap
  - BBTools bbnorm
  - DIAMOND
  - Perl
  - VirMAP
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0781
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# viral-read-alignment-to-genbank-nucleotide-protein-databases

## Summary

Align digitally normalized metagenomic reads against comprehensive viral nucleotide (Genbank gbvrl/gbphage) and viral protein databases using BBMap and DIAMOND, forming the tiered alignment layer VirMAP uses to assign reads to putative viral genomes and protein pseudo-scaffolds.

## When to use

Use when metagenomic FASTQ reads must be assigned to known viruses by both nucleotide and translated protein alignments to build per-genome read sets for mapping assembly.

## When NOT to use

- The goal is de-novo assembly of novel viruses with no database representatives — VirMAP's mapping assembly tier requires reference genomes; use MEGAHIT or Tadpole de-novo assembly instead.
- Input is already assembled contigs rather than raw reads — the read-assignment and per-genome pileup logic assumes read-level alignments.
- The target organisms are non-viral (e.g. bacterial or eukaryotic classification) — the databases and thresholds here are tuned to the viral superkingdom and VirMAP filters contigs mapping outside it.

## Inputs

- Paired-end metagenomic FASTQ reads (e.g. SRA runs such as SRR9875293 fetched via fasterq-dump)
- Digitally normalized read pairs (bbnorm-processed)
- Genbank viral (gbvrl) and phage (gbphage) division nucleotide database
- Viral protein database

## Outputs

- Nucleotide alignments of reads to Genbank viral/phage genomes (BBMap, 90% identity / 5% aligning-bases)
- Translated protein alignments of reads to viral proteins (DIAMOND, 20% identity / 8% aligning-bases)
- A read set assigned per putative viral genome (post Jaccard-distance filtering)

## How to apply

First dereplicate and digitally normalize paired-end reads with BBTools bbnorm using default parameters, mirroring VirMAP's pre-processing stage. Align the processed reads to the comprehensive Genbank viral/phage nucleotide database (gbvrl and gbphage divisions) with BBMap using a 90% identity / 5% aligning-bases threshold, and independently to the viral protein database with DIAMOND using a 20% identity / 8% aligning-bases threshold. The nucleotide and translated alignments are combined in a tiered manner: clustered alignments construct a putative list of viral genomes and protein pseudo-scaffolds, from which a read set per genome is formed. Genomic read sets with high numbers of non-unique read membership relative to larger sets are filtered by Jaccard distance to resolve ambiguous mapping across related genomes. Output alignment assignments feed the naive pileup assembly and super-scaffold merging stages, so alignments must be complete and correctly thresholded for downstream reconstruction to succeed.

## Related tools

- **VirMAP** (Perl-implemented pipeline that orchestrates the tiered nucleotide/protein alignment, read assignment, and Jaccard-distance filtering) — https://github.com/cmmr/VirMAP
- **BBMap** (Nucleotide alignment of processed reads to the Genbank viral/phage database (90% identity / 5% aligning-bases threshold))
- **DIAMOND** (Translated alignment of reads to the viral protein database (20% identity / 8% aligning-bases threshold)) — https://github.com/bbuchfink/diamond
- **BBTools bbnorm** (Dereplication and digital normalization of paired-end input reads with default parameters prior to alignment)

## Examples

```
Virmap.pl --threads $(nproc) --useBbnorm --sampleName SRR9875293 --readF /home/$USER/VirmapTest/SRR9875293_1.fastq
```

## Evaluation signals

- Output virmap_superscaffolds.fasta exists, parses as valid FASTA, and contains one merged virus-like nucleotide super-scaffold per non-filtered genome with tiled CDS protein calls.
- Each putative genome produced by the clustered alignments has an associated non-empty read set from both nucleotide and translated alignment channels.
- Read sets flagged for high non-unique read membership relative to larger sets are removed by the Jaccard-distance filter before assembly.
- Alignment thresholds match the published parameters: BBMap at 90% identity / 5% aligning bases and DIAMOND at 20% identity / 8% aligning bases.
- Classified reconstructions are consistent with VirMAP's bits-per-base scoring: sequences mapping outside the viral superkingdom are absent from the output.

## Limitations

- No software versions are stated for VirMAP, BBMap, or DIAMOND, and the Genbank database snapshot/version or download date is unspecified, complicating reproducibility.
- The minimum SSD and RAM requirements expand as Genbank grows, so large reference alignments may be resource-intensive.
- Alignment sensitivity/specificity is tunable (Supplementary Data 14 parameters), but the referenced supplementary data is not provided in the article text.
- High non-unique read membership among related viral genomes is only partially resolved by Jaccard-distance filtering; ambiguous assignments can propagate into super-scaffolds.

## Evidence

- [discussion] Nucleotide/protein alignment thresholds: "Processed reads are aligned to a comprehensive viral nucleotide and viral protein database"
- [intro] Putative genome construction: "A putative list of viral genomes and protein pseudo-scaffolds are constructed from clustered nucleotide and translated alignments to the Genbank viral and phage divisions (gbvrl and gbphage)."
- [discussion] Read assignment and Jaccard filtering: "Reads are assigned to viral genomes by nucleotide and translated alignments resulting in a set of reads per genome. Genomic read sets are then filtered by Jaccard distance"
- [discussion] Pre-processing with bbnorm: "input reads are dereplicated and digitally normalized18 using default parameters"
- [discussion] Database divisions: "sourced from the Genbank viral (gbvrl) and phage (gbphg) divisions"
- [methods] Example invocation from README: "`Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq"
