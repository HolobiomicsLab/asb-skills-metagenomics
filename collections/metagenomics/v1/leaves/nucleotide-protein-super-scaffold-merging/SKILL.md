---
name: nucleotide-protein-super-scaffold-merging
description: Use when metagenomic paired-end reads must be reconstructed into virus-like
  super-scaffolds by combining nucleotide (BBMap) and translated-protein (DIAMOND)
  alignments against Genbank viral/phage references, especially for low-coverage or
  divergent viruses.
when_to_use_negative:
- The goal is purely de-novo assembly of novel genomes absent from gbvrl/gbphage references
  rather than mapping-assembly reconstruction against Genbank viral/phage divisions.
- Only taxonomic classification (bits-per-base scoring / LCA) of already-assembled
  contigs is needed, not per-genome read-set reconstruction and super-scaffold building.
- Input is a single non-viral genome sample where alignment to viral/phage reference
  databases would mostly produce filtered or empty genomic read sets.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0797
tools:
- name: VirMAP
  role: 'Perl pipeline implementing the mapping assembly: read assignment to genomes,
    Jaccard filtering, naive pileup reconstruction, and nucleotide/protein super-scaffold
    merging'
  repo: https://github.com/cmmr/VirMAP
- name: BBMap
  role: Aligns processed reads to the Genbank viral/phage nucleotide database at 90%
    identity / 5% aligning-bases threshold
- name: DIAMOND
  role: Performs translated alignments of reads to the viral protein database at 20%
    identity / 8% aligning-bases threshold
  repo: https://github.com/bbuchfink/diamond
- name: BBTools bbnorm
  role: Dereplication and digital normalization of paired-end input reads prior to
    alignment (default parameters)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:aeee235a9c30ae226f5972d8858c7fce07708da45e76723b80e5caf63a4e267e
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
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# nucleotide-protein-super-scaffold-merging

## Summary

VirMAP's mapping-assembly stage merges nucleotide and amino-acid alignment information in a tiered manner: reads are assigned to viral genomes from clustered BBMap/DIAMOND alignments to Genbank gbvrl and gbphage, each genome entry is naive-pileup reconstructed, and CDS-derived protein calls are tiled onto the nucleotide scaffold to produce one virus-like super-scaffold per genome.

## When to use

Use when metagenomic paired-end reads must be reconstructed into virus-like super-scaffolds by combining nucleotide (BBMap) and translated-protein (DIAMOND) alignments against Genbank viral/phage references, especially for low-coverage or divergent viruses.

## When NOT to use

- The goal is purely de-novo assembly of novel genomes absent from gbvrl/gbphage references rather than mapping-assembly reconstruction against Genbank viral/phage divisions.
- Only taxonomic classification (bits-per-base scoring / LCA) of already-assembled contigs is needed, not per-genome read-set reconstruction and super-scaffold building.
- Input is a single non-viral genome sample where alignment to viral/phage reference databases would mostly produce filtered or empty genomic read sets.

## Inputs

- Dereplicated, digitally normalized paired-end metagenomic FASTQ reads (e.g. SRA sample SRR9875293)
- Comprehensive Genbank viral and phage nucleotide reference databases (gbvrl and gbphage divisions)
- Genbank viral protein reference database

## Outputs

- virmap_superscaffolds.fasta containing one merged virus-like nucleotide super-scaffold per non-filtered genome, with tiled CDS protein calls

## How to apply

Dereplicate and digitally normalize paired-end reads with BBTools bbnorm (default parameters), then align them to the Genbank viral/phage nucleotide database with BBMap (90% identity / 5% aligning-bases threshold) and to the viral protein database with DIAMOND (20% identity / 8% aligning-bases threshold). Assign reads to viral genomes from both nucleotide and translated alignments to form a read set per genome, filtering genomic read sets by Jaccard distance when high numbers of non-unique read membership are observed relative to larger sets. Reconstruct each nucleotide and amino-acid entry of every non-filtered genome with a naive pileup assembly, then tile the reconstructed protein coding sequences (CDS-derived protein calls) on top of the nucleotide scaffold of their originating genome. Merge protein and nucleotide information into a single nucleotide super-scaffold per genome, resolving disagreements with nucleotide precedence, and write the merged super-scaffolds to virmap_superscaffolds.fasta. The rationale is that merging nucleotide and protein information permits taxonomic classification and reconstruction independent of genome coverage or read overlap.

## Related tools

- **VirMAP** (Perl pipeline implementing the mapping assembly: read assignment to genomes, Jaccard filtering, naive pileup reconstruction, and nucleotide/protein super-scaffold merging) — https://github.com/cmmr/VirMAP
- **BBMap** (Aligns processed reads to the Genbank viral/phage nucleotide database at 90% identity / 5% aligning-bases threshold)
- **DIAMOND** (Performs translated alignments of reads to the viral protein database at 20% identity / 8% aligning-bases threshold) — https://github.com/bbuchfink/diamond
- **BBTools bbnorm** (Dereplication and digital normalization of paired-end input reads prior to alignment (default parameters))

## Examples

```
Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq --useMegahit --useBbnorm --sampleName SRR9875293
```

## Evaluation signals

- virmap_superscaffolds.fasta exists and parses as valid FASTA.
- The file contains exactly one merged virus-like nucleotide super-scaffold per non-filtered genome (genomes removed by Jaccard-distance filtering are absent).
- CDS-derived protein calls from the reconstructed amino-acid entries appear tiled on top of their originating genome's nucleotide scaffold.
- Disagreement positions between protein and nucleotide reconstructions are resolved with nucleotide precedence in the final super-scaffold sequences.
- Downstream taxonomic calls on the super-scaffolds resolve to the superkingdom virus, consistent with reads having been assigned from clustered gbvrl/gbphage alignments.

## Limitations

- Genomic read sets are filtered by Jaccard distance when high numbers of non-unique read membership are observed relative to larger sets, so heavily shared phage/viral regions can lose their read sets and produce no super-scaffold.
- Reconstruction depends on coverage of the Genbank viral and phage reference divisions; database coverage gaps of the viral tree of life restrict recovery of divergent viruses, and the Genbank snapshot/version is not specified.
- Tunable sensitivity/specificity parameters are documented only externally (Supplementary Data 14 and the GitHub options table), and no software versions are stated for VirMAP, BBMap, or DIAMOND.

## Evidence

- [intro] A putative list of viral genomes and protein pseudo-scaffolds are constructed from clustered nucleotide and translated alignments to the Genbank viral and phage divisions (gbvrl and gbphage).: "A putative list of viral genomes and protein pseudo-scaffolds are constructed from clustered nucleotide and translated alignments to the Genbank viral and phage divisions (gbvrl and gbphage)."
- [discussion] Naive pileup assembly per genome and CDS tiling: "Each nucleotide and amino acid entry associated with a non-filtered genome is individually reconstructed using a naive pileup assembly."
- [discussion] Jaccard-distance filtering of genomic read sets: "Genomic read sets are then filtered by Jaccard distance if high numbers of non-unique read membership are observed relative to larger sets."
- [abstract] Merging rationale: coverage-independent classification: "merge nucleotide and protein information to taxonomically classify viral reconstructions independent of genome coverage or read overlap"
- [methods] Virmap.pl command line invocation: "`Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq"
