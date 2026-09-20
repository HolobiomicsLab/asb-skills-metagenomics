---
name: iterative-contig-merging-reassembly
description: 'Decision support for refining assembled metagenomic contigs down to
  a purely viral set: use when contigs from a mapping-assembly stage must be merged,
  filtered against Genbank, and rebuilt until convergence.'
when_to_use_negative:
- Input is raw reads with no upstream mapping assembly — VirMAP requires contigs/super-scaffolds
  from its first-stage mapping assembly before improvement
- Goal is de-novo assembly alone without taxonomic filtering — this skill's filter
  step discards anything not resolving to the viral superkingdom
- Analysis targets non-viral sequences (e.g. bacterial or eukaryotic genomes) — the
  >50% viral-base filter will remove them by design
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0781
tools:
- name: VirMAP
  role: Implements the iterative merging and improvement loop that merges overlapping
    contigs, rebuilds them via pileup assembly, and filters non-viral contigs until
    convergence
  repo: https://github.com/cmmr/VirMAP
- name: BBMap
  role: Nucleotide mapping of contigs to the viral nucleotide database during the
    improvement-loop filter and merge steps
  repo: https://github.com/bbuchfink/diamond
- name: DIAMOND
  role: Translated (protein) alignment of contigs to the viral protein database for
    taxonomic validation
  repo: https://github.com/bbuchfink/diamond
- name: MEGAHIT
  role: Optional multi-kmer de-novo assembly supporting the rebuild stage of the improvement
    cycle
  repo: https://github.com/voutcn/megahit
- name: Tadpole
  role: Alternative multi-kmer de-novo assembly supporting the rebuild stage of the
    improvement cycle
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:41a85d6acbec9d6b7dc2cbc8f3f5fc89a74356b239e5102a95300ca0d0e4c95c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cmmr/VirMAP
  tools:
  - BBMap
  - DIAMOND
  - MEGAHIT
  - Tadpole
  - VirMAP
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0781
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# iterative-contig-merging-reassembly

## Summary

VirMAP's improvement algorithm that iteratively merges overlapping contigs and rebuilds them via pileup (and optional de-novo) assembly, filtering out non-viral contigs at each cycle so the final dual assembly consists solely of viral sequences.

## When to use

Decision support for refining assembled metagenomic contigs down to a purely viral set: use when contigs from a mapping-assembly stage must be merged, filtered against Genbank, and rebuilt until convergence.

## When NOT to use

- Input is raw reads with no upstream mapping assembly — VirMAP requires contigs/super-scaffolds from its first-stage mapping assembly before improvement
- Goal is de-novo assembly alone without taxonomic filtering — this skill's filter step discards anything not resolving to the viral superkingdom
- Analysis targets non-viral sequences (e.g. bacterial or eukaryotic genomes) — the >50% viral-base filter will remove them by design

## Inputs

- Super-scaffolds/contigs from the VirMAP mapping-assembly stage
- Dereplicated, digitally normalized FASTQ reads (khmer normalize-by-median.py or bbnorm)
- VirMAP custom viral nucleotide database (from Genbank gbvrl/gbphage divisions)
- VirMAP viral protein pseudo-scaffold database
- Comprehensive Genbank database spanning 16 divisions (for improvement-loop validation)

## Outputs

- Converged set of merged, rebuilt viral contigs/genomes
- Taxonomic classifications for final viral genomes
- Per-genome read abundance estimates
- Run diagnostics written to VirmapRun.err

## How to apply

Start from the super-scaffolds/contigs produced by the upstream mapping-assembly stage. Align each contig to the custom VirMAP viral nucleotide database with BBMap and to the protein database with DIAMOND, then filter out any contig whose top scoring pass-one taxon is not in the superkingdom virus, or where >50% of its aligning bases do not resolve to virus (evaluated against a comprehensive Genbank database spanning 16 divisions). Run the merging step by self-mapping contigs against each other and joining those sharing significant overlap. Rebuild contigs by naive pileup assembly of the dereplicated, digitally normalized reads (normalize-by-median.py or bbnorm), optionally seeded with a multi-kmer de-novo assembly using MEGAHIT and/or Tadpole, and iterate the merge-and-rebuild cycle until convergence. Finally compute per-genome read abundance from the converged read assignments.

## Related tools

- **VirMAP** (Implements the iterative merging and improvement loop that merges overlapping contigs, rebuilds them via pileup assembly, and filters non-viral contigs until convergence) — https://github.com/cmmr/VirMAP
- **BBMap** (Nucleotide mapping of contigs to the viral nucleotide database during the improvement-loop filter and merge steps) — https://github.com/bbuchfink/diamond
- **DIAMOND** (Translated (protein) alignment of contigs to the viral protein database for taxonomic validation) — https://github.com/bbuchfink/diamond
- **MEGAHIT** (Optional multi-kmer de-novo assembly supporting the rebuild stage of the improvement cycle) — https://github.com/voutcn/megahit
- **Tadpole** (Alternative multi-kmer de-novo assembly supporting the rebuild stage of the improvement cycle)

## Examples

```
Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq --useMegahit --useBbnorm --sampleName SRR9875293
```

## Evaluation signals

- Every retained contig has a top scoring pass-one taxon in the superkingdom virus and ≤50% of its aligning bases resolving outside virus
- The merge-and-rebuild cycle terminates at convergence (no further contig joins from significant overlap)
- VirmapRun output directory is populated with taxonomic classifications and abundance results; diagnostics appear in VirmapRun.err without fatal errors
- Rebuilt contigs are derived from the dereplicated, digitally normalized read pileup, not from the original raw reads
- Final assembly contains no contigs aligned to non-viral Genbank divisions

## Limitations

- Filter thresholds (top pass-one taxon not virus, >50% non-viral aligning bases) are fixed, so divergent but genuinely viral contigs with poor database hits may be discarded
- The dynamically calculated LCA radius and majority threshold formulas are not fully specified in the main text; tuning parameters reside in Supplementary Data 14, which may not be available
- Sensitivity/specificity of the improvement loop depends on Genbank database coverage, which the paper notes is insufficient for parts of the viral tree of life
- No software versions stated for VirMAP, BBMap, DIAMOND, MEGAHIT, or Tadpole; Genbank database snapshot/date also unspecified

## Evidence

- [discussion] A contig is filtered if its top scoring pass one taxon is not in the superkingdom virus, or if > 50% of its aligning bases do not resolve to the superkingdom virus.: "A contig is filtered if its top scoring pass one taxon is not in the superkingdom virus, or if > 50% of its aligning bases do not resolve to the superkingdom virus."
- [intro] VirMAP uses an improvement algorithm that iteratively merges and rebuilds contig information.: "an improvement algorithm that iteratively merges and rebuilds contig information"
- [discussion] alignments of the improved contigs to a comprehensive Genbank database spanning 16 separate divisions (Supplementary Note 4): "alignments of the improved contigs to a comprehensive Genbank database spanning 16 separate divisions"
- [discussion] a multi-kmer de-novo assembly using either MEGAHIT or Tadpole, or both: "a multi-kmer de-novo assembly using either MEGAHIT21,22 or Tadpole19, or both"
- [discussion] using BBMap19 and DIAMOND20, respectively: "using BBMap19 and DIAMOND20, respectively"
