---
name: naive-pileup-mapping-assembly-of-viral-genomes
description: Use when metagenomic paired-end reads must be reconstructed into virus-like
  super-scaffolds by mapping to Genbank viral/phage references (gbvrl, gbphage) with
  low-coverage or divergent viruses where de-novo assembly alone is insufficient.
when_to_use_negative:
- When sufficient coverage exists for a high-quality de-novo assembly with MEGAHIT
  or Tadpole and reference-based reconstruction is unnecessary
- When the goal is taxonomic classification only and no genome-scale reconstruction
  is required (VirMAP's bits-per-base classification path)
- When reads do not plausibly derive from viruses or phages represented in the Genbank
  gbvrl/gbphage reference divisions
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3174
tools:
- name: VirMAP
  role: Perl implementation orchestrating the tiered mapping assembly, pileup reconstruction,
    and super-scaffold merging
  repo: https://github.com/cmmr/VirMAP
- name: BBMap
  role: Aligns processed reads to the Genbank viral/phage nucleotide database (90%
    identity / 5% aligning-bases threshold)
- name: DIAMOND
  role: Performs translated alignment of reads to the viral protein database (20%
    identity / 8% aligning-bases threshold)
  repo: https://github.com/bbuchfink/diamond
- name: bbnorm (BBTools)
  role: Dereplicates and digitally normalizes paired-end input reads using default
    parameters before alignment
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:c4d3de6ba981e78b64fc4f38089f53f22192fe209795d5c96e71c2e048808436
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cmmr/VirMAP
  tools:
  - BBMap
  - DIAMOND
  - Perl
  - VirMAP
  - bbnorm (BBTools)
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# naive-pileup-mapping-assembly-of-viral-genomes

## Summary

VirMAP's mapping assembly stage builds virus-like super-scaffolds by combining nucleotide (BBMap) and translated/protein (DIAMOND) alignments in a tiered manner, then reconstructing each genome entry via naive pileup assembly and merging protein and nucleotide information into one super-scaffold per genome with nucleotide precedence.

## When to use

Use when metagenomic paired-end reads must be reconstructed into virus-like super-scaffolds by mapping to Genbank viral/phage references (gbvrl, gbphage) with low-coverage or divergent viruses where de-novo assembly alone is insufficient.

## When NOT to use

- When sufficient coverage exists for a high-quality de-novo assembly with MEGAHIT or Tadpole and reference-based reconstruction is unnecessary
- When the goal is taxonomic classification only and no genome-scale reconstruction is required (VirMAP's bits-per-base classification path)
- When reads do not plausibly derive from viruses or phages represented in the Genbank gbvrl/gbphage reference divisions

## Inputs

- Paired-end FASTQ metagenomic reads (e.g. SRR9875293_1.fastq / SRR9875293_2.fastq)
- Genbank viral and phage divisions (gbvrl and gbphage) nucleotide database
- Comprehensive viral protein database for DIAMOND translated alignment

## Outputs

- virmap_superscaffolds.fasta containing one merged virus-like nucleotide super-scaffold per non-filtered genome with tiled CDS protein calls

## How to apply

Dereplicate and digitally normalize paired-end reads with BBTools bbnorm (default parameters), then align the processed reads to the Genbank viral/phage nucleotide database with BBMap (90% identity / 5% aligning-bases threshold) and to the viral protein database with DIAMOND (20% identity / 8% aligning-bases threshold). Assign reads to viral genomes from both nucleotide and translated alignments to form a read set per genome, and filter genomic read sets by Jaccard distance when high numbers of non-unique read membership are observed relative to larger sets. For every non-filtered genome, individually reconstruct each nucleotide and amino-acid entry using a naive pileup assembly, then tile the reconstructed protein coding sequences (CDS-derived protein calls) on top of the nucleotide scaffold of their originating genome. Merge protein and nucleotide information into a single nucleotide super-scaffold per genome, resolving disagreements with nucleotide precedence, and write the result to virmap_superscaffolds.fasta.

## Related tools

- **VirMAP** (Perl implementation orchestrating the tiered mapping assembly, pileup reconstruction, and super-scaffold merging) — https://github.com/cmmr/VirMAP
- **BBMap** (Aligns processed reads to the Genbank viral/phage nucleotide database (90% identity / 5% aligning-bases threshold))
- **DIAMOND** (Performs translated alignment of reads to the viral protein database (20% identity / 8% aligning-bases threshold)) — https://github.com/bbuchfink/diamond
- **bbnorm (BBTools)** (Dereplicates and digitally normalizes paired-end input reads using default parameters before alignment)

## Examples

```
perl Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq --useMegahit --useBbnorm --sampleName SRR9875293
```

## Evaluation signals

- virmap_superscaffolds.fasta exists and parses as valid FASTA
- Output contains one merged virus-like nucleotide super-scaffold per non-filtered genome
- Reconstructed CDS-derived protein calls are tiled on top of the nucleotide scaffold of their originating genome
- Disagreements between protein and nucleotide reconstructions are resolved with nucleotide precedence
- Genomic read sets removed by Jaccard-distance filtering (high non-unique read membership) do not appear in the output

## Limitations

- VirMAP requires substantial SSD and RAM that expand as Genbank grows, since alignments target the comprehensive Genbank viral and phage divisions
- Genomic read sets with high numbers of non-unique read membership relative to larger sets are filtered by Jaccard distance, so ambiguous genomes may be dropped
- Software versions for VirMAP, BBMap, and DIAMOND are not stated, and the Genbank database snapshot/date is unspecified; Supplementary Data 14 (tunable sensitivity/specificity parameters) is not included in the main text

## Evidence

- [intro] Tiered nucleotide/protein mapping assembly: "a mapping assembly algorithm combining nucleotide and amino-acid alignment information in a tiered manner to build virus-like super-scaffolds"
- [intro] Putative genome/pseudo-scaffold construction: "A putative list of viral genomes and protein pseudo-scaffolds are constructed from clustered nucleotide and translated alignments to the Genbank viral and phage divisions (gbvrl and gbphage)."
- [discussion] Jaccard-distance filtering of genomic read sets: "Genomic read sets are then filtered by Jaccard distance if high numbers of non-unique read membership are observed relative to larger sets."
- [discussion] Naive pileup reconstruction and CDS tiling: "Each nucleotide and amino acid entry associated with a non-filtered genome is individually reconstructed using a naive pileup assembly."
- [discussion] Implementation language: "The VirMAP software is open-source and available online at https://github.com/cmmr/VirMAP. VirMAP is implemented in Perl."
- [discussion] Reference databases: "sourced from the Genbank viral (gbvrl) and phage (gbphg) divisions"
