---
name: naive-pileup-viral-contig-reconstruction
description: Use when refining VirMAP super-scaffolds/contigs so the final dual assembly
  consists solely of viral sequences, after viral filtering and contig merging, by
  rebuilding each genome's contigs from its assigned read set.
when_to_use_negative:
- Input contigs are already classified as non-viral (top scoring pass one taxon not
  in superkingdom Virus) — these are filtered out rather than rebuilt.
- Goal is de-novo assembly of a whole metagenome without viral targeting — use MEGAHIT/standalone
  assembly instead of VirMAP's iterative rebuild loop.
- Genomic read sets show high non-unique read membership requiring Jaccard-distance
  filtering — resolve assignment ambiguity before pileup reconstruction.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3174
tools:
- name: VirMAP
  role: Orchestrates the merge-and-rebuild improvement loop, per-genome read assignment,
    and naive pileup reconstruction of nucleotide and amino-acid entries
  repo: https://github.com/cmmr/VirMAP
- name: MEGAHIT
  role: Optional multi-kmer de-novo assembly supporting the rebuild step (--useMegahit)
  repo: https://github.com/voutcn/megahit
- name: khmer
  role: Digital normalization of assigned reads before pileup assembly via normalize-by-median.py
  repo: https://github.com/dib-lab/khmer
- name: DIAMOND
  role: Translated alignment supporting per-genome read assignment and taxonomic scoring
    in the improvement loop
  repo: https://github.com/bbuchfink/diamond
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:26f3e278089d6782eccc21a2cfc33d340645b0783e2cec5e3aef639266dabf27
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
  - khmer
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# naive-pileup-viral-contig-reconstruction

## Summary

Rebuild viral contigs within VirMAP's improvement loop by naive pileup assembly of dereplicated, digitally normalized reads assigned to each viral genome, optionally supported by MEGAHIT or Tadpole multi-kmer de-novo assembly, and iterate the merge-and-rebuild cycle until convergence.

## When to use

Use when refining VirMAP super-scaffolds/contigs so the final dual assembly consists solely of viral sequences, after viral filtering and contig merging, by rebuilding each genome's contigs from its assigned read set.

## When NOT to use

- Input contigs are already classified as non-viral (top scoring pass one taxon not in superkingdom Virus) — these are filtered out rather than rebuilt.
- Goal is de-novo assembly of a whole metagenome without viral targeting — use MEGAHIT/standalone assembly instead of VirMAP's iterative rebuild loop.
- Genomic read sets show high non-unique read membership requiring Jaccard-distance filtering — resolve assignment ambiguity before pileup reconstruction.

## Inputs

- Dereplicated, digitally normalized reads assigned per viral genome (post Jaccard-distance filtering of genomic read sets)
- Viral nucleotide and protein databases (Genbank gbvrl and gbphage divisions)
- FASTQ reads (e.g. SRR9875293_1.fastq from SRA)

## Outputs

- Rebuilt viral contigs per genome (naive pileup assemblies with tiled reconstructed CDS)
- Converged final dual viral assembly with read abundance per viral genome
- VirmapRun output directory with taxonomic classifications and abundance results; diagnostics in VirmapRun.err

## How to apply

After VirMAP filters contigs (top scoring pass one taxon not in superkingdom Virus, or >50% of aligning bases not resolving to virus) and merges overlapping contigs via self-mapping, rebuild contigs for each non-filtered genome: dereplicate and digitally normalize the assigned reads (normalize-by-median.py from khmer or bbnorm, --useBbnorm), then perform a naive pileup assembly of each nucleotide and amino-acid entry associated with the genome. Optionally augment this with a multi-kmer de-novo assembly using MEGAHIT and/or Tadpole (--useMegahit). Reconstructed protein coding sequences are tiled on top of their originating genome. Iterate the merge-and-rebuild cycle until convergence, then compute read abundance for the final converged viral genomes.

## Related tools

- **VirMAP** (Orchestrates the merge-and-rebuild improvement loop, per-genome read assignment, and naive pileup reconstruction of nucleotide and amino-acid entries) — https://github.com/cmmr/VirMAP
- **MEGAHIT** (Optional multi-kmer de-novo assembly supporting the rebuild step (--useMegahit)) — https://github.com/voutcn/megahit
- **khmer** (Digital normalization of assigned reads before pileup assembly via normalize-by-median.py) — https://github.com/dib-lab/khmer
- **DIAMOND** (Translated alignment supporting per-genome read assignment and taxonomic scoring in the improvement loop) — https://github.com/bbuchfink/diamond

## Examples

```
Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq --useMegahit --useBbnorm --sampleName SRR9875293
```

## Evaluation signals

- Run completes and populates the VirmapRun output directory with taxonomic classifications and abundance results; run diagnostics captured in VirmapRun.err.
- All retained contigs pass the viral filter: top scoring pass one taxon in superkingdom Virus and ≤50% of aligning bases resolving outside virus.
- Reconstructed protein coding sequences are correctly tiled on top of their originating genome.
- Merge-and-rebuild iterations reach convergence (no further contig changes) before computing final read abundance.

## Limitations

- Achieving the necessary coverage for a quality assembly remains challenging; the pileup rebuild depends on per-genome read coverage.
- VirMAP's sensitivity/specificity tradeoff requires tuning via Supplementary Data 14 parameters, which are not fully specified in the article text.
- No software versions are stated for VirMAP, BBMap, DIAMOND, MEGAHIT, or Tadpole; Genbank database snapshot/version is unspecified, so results depend on database content at run time.
- As Genbank expands, the minimum SSD and RAM requirements will expand as well.

## Evidence

- [intro] Iterative merge-and-rebuild improvement loop: "an improvement algorithm that iteratively merges and rebuilds contig information"
- [discussion] Per-genome naive pileup assembly: "Each nucleotide and amino acid entry associated with a non-filtered genome is individually reconstructed using a naive pileup assembly."
- [discussion] Viral filtering threshold: [evidence span withheld — anchor status: not_found]
- [discussion] Read dereplication and normalization: "input reads are dereplicated and digitally normalized18 using default parameters"
- [methods] MEGAHIT/Tadpole support and CLI flags: "--useMegahit --useBbnorm --sampleName SRR9875293"
