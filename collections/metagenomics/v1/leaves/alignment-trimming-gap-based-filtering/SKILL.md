---
name: alignment-trimming-gap-based-filtering
description: Use when per-locus MAFFT alignments of Patchwork-recovered USCO (near-universal
  single-copy ortholog) sequences contain gap-rich columns that must be removed before
  concatenating a supermatrix and inferring a maximum-likelihood phylogeny.
when_to_use_negative:
- Sequences are not yet aligned — trimming requires a multiple sequence alignment,
  so run MAFFT (or equivalent) first
- 'Reference sequences are not near-USCOs: the article recommends limiting references
  to near-USCOs to avoid chimeric sequences, which trimming thresholds alone will
  not fix'
- The goal is marker retrieval itself from WGS data rather than post-alignment cleaning
  — use Patchwork's retrieval/stitching workflow instead of (or before) trimming
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: trimAl
  role: Performs the gap-based alignment trimming with -gt 0.9 (remove positions with
    >90% gaps) and -cons 60 (retain at least 60% of each alignment)
- name: MAFFT
  role: Builds the per-locus multiple sequence alignments (--globalpair --ep 0.123)
    that serve as trimming input
- name: FASconCAT-G
  role: Concatenates the trimmed alignments into a single supermatrix with a gene
    partition file
- name: IQ-TREE 2
  role: Infers the maximum-likelihood phylogeny from the trimmed/concatenated supermatrix
    (-m MFP, -B 1000), the downstream check that trimming did not destroy phylogenetic
    signal
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:bbec678796befbebaf3d1e5e895b88240325b41b96c08922fbccae6592280036
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FASconCAT-G
  - FASconcat-G
  - IQ-TREE 2
  - MAFFT
  - Patchwork
  - trimAl
  edam_operation: http://edamontology.org/operation_0492
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# alignment-trimming-gap-based-filtering

## Summary

Gap-based alignment trimming cleans per-locus MAFFT alignments of Patchwork-recovered USCO markers with trimAl (-gt 0.9, -cons 60), removing columns with more than 90% gaps while retaining at least 60% of each alignment, before concatenation and ML tree inference. In the louse phylogenomic benchmark this retained 3,454,320 positions (~64% of the untrimmed supermatrix) and still yielded a well-supported, congruent tree.

## When to use

Use when per-locus MAFFT alignments of Patchwork-recovered USCO (near-universal single-copy ortholog) sequences contain gap-rich columns that must be removed before concatenating a supermatrix and inferring a maximum-likelihood phylogeny.

## When NOT to use

- Sequences are not yet aligned — trimming requires a multiple sequence alignment, so run MAFFT (or equivalent) first
- Reference sequences are not near-USCOs: the article recommends limiting references to near-USCOs to avoid chimeric sequences, which trimming thresholds alone will not fix
- The goal is marker retrieval itself from WGS data rather than post-alignment cleaning — use Patchwork's retrieval/stitching workflow instead of (or before) trimming

## Inputs

- Per-locus MAFFT multiple sequence alignments (e.g. --globalpair --ep 0.123) of Patchwork-recovered marker/USCO sequences in FASTA format, one alignment per locus (e.g. 957 loci)

## Outputs

- Trimmed per-locus alignments (gap columns >90% removed, ≥60% of columns retained per alignment)
- Downstream-consumable trimmed alignments for concatenation (e.g. into a FASconCAT-G supermatrix with partition file; reference result: 3,454,320 concatenated positions, ~64% of untrimmed)

## How to apply

For each of the ~957 loci recovered against near-USCO references (e.g. the 957 near-USCOs from the H. robusta annotated genome), align the Patchwork-recovered marker sequences with MAFFT (--globalpair --ep 0.123). Trim each alignment with trimAl using -gt 0.9 so that positions consisting of more than 90% gaps are removed, and -cons 60 so that at least 60% of each alignment is retained; these two thresholds together guard against both gap-dominated columns and over-trimming of short or divergent loci. Feed the trimmed alignments to FASconCAT-G to build the concatenated supermatrix plus its gene partition file, which is then used by IQ-TREE 2 (-m MFP, -B 1000) for tree inference. Rationale: trimming gap-heavy columns removes poorly covered or nonhomologous regions from genome-skimming-derived markers, and the article reports that most markers were well above the 90% aligned position trimming threshold, so heavy trimming signals problematic loci.

## Related tools

- **trimAl** (Performs the gap-based alignment trimming with -gt 0.9 (remove positions with >90% gaps) and -cons 60 (retain at least 60% of each alignment))
- **MAFFT** (Builds the per-locus multiple sequence alignments (--globalpair --ep 0.123) that serve as trimming input)
- **FASconCAT-G** (Concatenates the trimmed alignments into a single supermatrix with a gene partition file)
- **IQ-TREE 2** (Infers the maximum-likelihood phylogeny from the trimmed/concatenated supermatrix (-m MFP, -B 1000), the downstream check that trimming did not destroy phylogenetic signal)

## Examples

```
trimal -in locus_001.aln.fasta -out locus_001.trim.fasta -gt 0.9 -cons 60
```

## Evaluation signals

- Each trimmed alignment retains at least 60% of its original columns (trimAl -cons 60) and no column with >90% gaps survives (-gt 0.9)
- The concatenated supermatrix and gene partition file both exist after FASconCAT-G; the reference run retained 3,454,320 positions (~64% of the untrimmed alignment)
- Most individual markers remain well above the 90% aligned position trimming threshold; markers falling near it should be inspected as suspect/low-quality loci
- The downstream IQ-TREE 2 run completes 1,000 ultrafast bootstrap replicates and produces a well-supported tree (reference result: congruent with the original louse phylogeny, Fig. 6)

## Limitations

- The article does not provide specific cutoff threshold values for gap-excluded identity trimming in all contexts, so thresholds may need tuning per data set
- Trimming thresholds do not detect artifactual noncoding stretches by themselves; the authors suggest PREQUAL could be used to detect and remove such regions
- Over-trimming can occur for low-coverage or divergent loci — at very low sequencing depth (e.g. 1x) marker quality degrades, so retained columns may be few

## Evidence

- [results] trimal thresholds: "most markers were well above the 90% aligned position trimming threshold"
- [methods] sliding window trimming: "we have implemented a sliding window–based alignment trimming approach to rid the alignments from these unwanted regions"
- [results] tree result: "Our phylogenetic reconstruction resulted in a well-supported tree (Fig. 6)"
- [discussion] PREQUAL suggestion: "the tool PREQUAL could be used to detect such and remove such regions"
