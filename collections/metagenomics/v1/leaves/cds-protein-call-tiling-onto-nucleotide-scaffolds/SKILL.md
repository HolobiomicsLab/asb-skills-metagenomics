---
name: cds-protein-call-tiling-onto-nucleotide-scaffolds
description: Use when reconstructing virus-like genomes from metagenomic reads where
  translated (DIAMOND) and nucleotide (BBMap) alignments must be merged per genome
  into one super-scaffold with nucleotide precedence over protein calls.
when_to_use_negative:
- The goal is de-novo assembly of a high-coverage single virus — VirMAP's mapping
  assembly and de-novo strategies (MEGAHIT/Tadpole) are separate; tiling CDS calls
  is specific to the reference-guided super-scaffold stage.
- Input reads have not been assigned to Genbank viral/phage genomes or the genomic
  read sets were filtered out by Jaccard distance — there is no originating genome
  scaffold to tile onto.
- Classification-only analysis is needed (bits-per-base taxonomic scoring) rather
  than genome reconstruction/super-scaffold building.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0622
tools:
- name: VirMAP
  role: Perl pipeline that performs the naive pileup reconstruction, CDS tiling onto
    the originating genome's nucleotide scaffold, and nucleotide-precedence merge
    into the super-scaffold
  repo: https://github.com/cmmr/VirMAP
- name: BBMap
  role: Produces the nucleotide alignments used to build the per-genome nucleotide
    scaffold that the CDS calls are tiled onto
- name: DIAMOND
  role: Produces the translated protein alignments from which the CDS-derived protein
    calls are reconstructed before tiling
  repo: https://github.com/bbuchfink/diamond
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:80a6d4b6c608fc437f97191bea5f2fc62f8d636e84e060b6ce35d2065bc2952a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cmmr/VirMAP
  tools:
  - BBMap
  - DIAMOND
  - Perl
  - VirMAP
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# CDS protein call tiling onto nucleotide scaffolds

## Summary

A VirMAP mapping-assembly step that tiles reconstructed protein coding sequences (CDS-derived protein calls) onto the nucleotide scaffold of their originating viral genome and merges the two layers into a single nucleotide super-scaffold per genome, written to virmap_superscaffolds.fasta.

## When to use

Use when reconstructing virus-like genomes from metagenomic reads where translated (DIAMOND) and nucleotide (BBMap) alignments must be merged per genome into one super-scaffold with nucleotide precedence over protein calls.

## When NOT to use

- The goal is de-novo assembly of a high-coverage single virus — VirMAP's mapping assembly and de-novo strategies (MEGAHIT/Tadpole) are separate; tiling CDS calls is specific to the reference-guided super-scaffold stage.
- Input reads have not been assigned to Genbank viral/phage genomes or the genomic read sets were filtered out by Jaccard distance — there is no originating genome scaffold to tile onto.
- Classification-only analysis is needed (bits-per-base taxonomic scoring) rather than genome reconstruction/super-scaffold building.

## Inputs

- Normalized paired-end metagenomic reads (dereplicated/digitally normalized with BBTools bbnorm)
- Per-genome read sets assigned from BBMap nucleotide and DIAMOND translated alignments to Genbank gbvrl and gbphage divisions
- Naive pileup reconstructions of nucleotide entries per non-filtered genome
- Reconstructed protein coding sequences (CDS-derived amino-acid calls) per genome

## Outputs

- virmap_superscaffolds.fasta containing one merged virus-like nucleotide super-scaffold per non-filtered genome with tiled CDS protein calls

## How to apply

After VirMAP assigns reads to viral genomes from BBMap nucleotide alignments (90% identity / 5% aligning bases) and DIAMOND translated alignments (20% identity / 8% aligning bases) and filters genomic read sets by Jaccard distance, each nucleotide and amino-acid entry associated with a non-filtered genome is reconstructed with a naive pileup assembly. The reconstructed protein coding sequences (CDS-derived protein calls) are then tiled on top of the nucleotide scaffold of their originating genome. Disagreements between the protein and nucleotide layers are resolved with nucleotide precedence, and the merged result is written as a single nucleotide super-scaffold per genome to virmap_superscaffolds.fasta. This tiered merge is what lets VirMAP recover highly divergent or low-coverage viruses whose protein signal exists even when nucleotide signal is weak.

## Related tools

- **VirMAP** (Perl pipeline that performs the naive pileup reconstruction, CDS tiling onto the originating genome's nucleotide scaffold, and nucleotide-precedence merge into the super-scaffold) — https://github.com/cmmr/VirMAP
- **BBMap** (Produces the nucleotide alignments used to build the per-genome nucleotide scaffold that the CDS calls are tiled onto)
- **DIAMOND** (Produces the translated protein alignments from which the CDS-derived protein calls are reconstructed before tiling) — https://github.com/bbuchfink/diamond

## Examples

```
Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq --useMegahit --useBbnorm --sampleName SRR9875293
```

## Evaluation signals

- virmap_superscaffolds.fasta exists and parses as valid FASTA
- One merged super-scaffold is present per non-filtered viral genome
- Each super-scaffold contains the tiled CDS-derived protein calls integrated into the nucleotide backbone
- Disagreement positions between protein and nucleotide layers resolve to the nucleotide sequence (nucleotide precedence)
- Genomes whose read sets were filtered by Jaccard distance are absent from the output

## Limitations

- Sensitivity/specificity tradeoff depends on tunable parameters (Supplementary Data 14), which are not provided in the main text
- Merging quality depends on the underlying Genbank gbvrl/gbphage database snapshot; the article does not specify database version or download date
- The tiered merge yields a putative super-scaffold rather than a finished genome; accuracy depends on coverage and on reads being present in the non-filtered per-genome read sets

## Evidence

- [discussion] Reconstructed protein coding sequences are tiled on top of the originating genome scaffold: "Each nucleotide and amino acid entry associated with a non-filtered genome is individually reconstructed using a naive pileup assembly. Reconstructed protein coding sequences are tiled on top of"
- [intro] Putative genomes and pseudo-scaffolds from clustered alignments: "A putative list of viral genomes and protein pseudo-scaffolds are constructed from clustered nucleotide and translated alignments to the Genbank viral and phage divisions (gbvrl and gbphage)."
- [abstract] Merging protein and nucleotide information: "merge nucleotide and protein information to taxonomically classify viral reconstructions independent of genome coverage or read overlap"
- [discussion] Genomic read set filtering by Jaccard distance precedes reconstruction: "Genomic read sets are then filtered by Jaccard distance if high numbers of non-unique read membership are observed relative to larger sets."
- [discussion] VirMAP implemented in Perl, open-source: "The VirMAP software is open-source and available online at https://github.com/cmmr/VirMAP. VirMAP is implemented in Perl."
