---
name: contig-tip-truncation-detection
description: Use when calling alleles on draft (multi-contig) assemblies with chewBBACA
  AlleleCall and you must distinguish genuine size-divergent alleles (ASM/ALM) from
  apparently truncated matches caused by contig boundaries (PLOT) before building
  the allelic profile matrix.
when_to_use_negative:
- Input genomes are complete or chromosome-level assemblies where contig-edge truncation
  cannot occur and PLOT flagging is unnecessary
- Allele calling directly from raw reads — chewBBACA's allele-calling engine requires
  FASTA draft assemblies or complete genomes, not FASTQ
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3293
tools:
- name: chewBBACA
  role: Provides the AlleleCall engine that assigns PLOT, ASM, ALM, NIPH/NIPHEM, LNF
    and EXC classifications, and RemoveGenes to filter flagged loci from results
  repo: https://github.com/B-UMMI/chewBBACA
- name: BLAST
  role: Underlying BLASTp BLAST Score Ratio comparisons used to classify CDS hits
    before contig-tip (PLOT) and other tag assignment
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1099/mgen.0.000166
    title: 'chewBBACA: A complete suite for gene-by-gene schema creation and strain
      identification'
  source_content_hash: sha256:a71ba1acbc83daa6ce078db37d33c9fa9f34c12dd276e8d3eb6c7f685cc9def7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/chewBBACA
  tools:
  - BLAST
  - chewBBACA
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Contig-tip truncation detection (PLOT classification in chewBBACA AlleleCall)

## Summary

Detects CDS matches that fall at the tips of draft-assembly contigs during chewBBACA AlleleCall and flags them with the PLOT classification, because truncation at a contig edge can mimic a size-divergent allele and lead to erroneous allele assignment.

## When to use

Use when calling alleles on draft (multi-contig) assemblies with chewBBACA AlleleCall and you must distinguish genuine size-divergent alleles (ASM/ALM) from apparently truncated matches caused by contig boundaries (PLOT) before building the allelic profile matrix.

## When NOT to use

- Input genomes are complete or chromosome-level assemblies where contig-edge truncation cannot occur and PLOT flagging is unnecessary
- Allele calling directly from raw reads — chewBBACA's allele-calling engine requires FASTA draft assemblies or complete genomes, not FASTQ

## Inputs

- Draft genome assemblies in FASTA format (de novo assembled, multi-contig)
- Pre-defined cg/wgMLST schema seed (FASTA loci from Chewie-NS, CreateSchema, or adapted via PrepExternalSchema)
- Optional Prodigal training file for the species (e.g., Streptococcus_agalactiae.trn)

## Outputs

- Allelic profile matrix (allelic_profile_matrix.tsv / results_alleles.tsv) with PLOT tags at genome-locus cells where hits fall at contig tips
- Per-locus FASTA files of allelic diversity
- RepeatedLoci.txt listing paralogous loci (NIPH/NIPHEM) for downstream filtering

## How to apply

Run chewBBACA's AlleleCall module on FASTA draft assemblies against a schema seed; the engine first classifies CDSs with 100% nucleotide identity to schema alleles as exact matches (EXC), then uses a BLASTp-based BLAST Score Ratio (default BSR threshold 0.6) approach to classify remaining CDSs. For each non-exact hit, evaluate the match position within the contig: hits abutting a contig tip may be artificially truncated, so they receive the PLOT tag instead of being called as inferred alleles or ASM/ALM (size deviations greater than 20% of the locus mode). Loci matched in multiple distinct genomic regions are flagged as NIPH/NIPHEM and reported in RepeatedLoci.txt for potential removal with RemoveGenes. CDSs with no valid match receive LNF. Results are compiled into the allelic profile matrix where each genome-locus cell must be an integer allele identifier or one of the tags EXC/INF, ASM, ALM, NIPH/NIPHEM, PLOT, LNF.

## Related tools

- **chewBBACA** (Provides the AlleleCall engine that assigns PLOT, ASM, ALM, NIPH/NIPHEM, LNF and EXC classifications, and RemoveGenes to filter flagged loci from results) — https://github.com/B-UMMI/chewBBACA
- **BLAST** (Underlying BLASTp BLAST Score Ratio comparisons used to classify CDS hits before contig-tip (PLOT) and other tag assignment)

## Examples

```
chewBBACA.py AlleleCall -i complete_genomes/ -g tutorial_schema/schema_seed -o results32_wgMLST --cpu 6
```

## Evaluation signals

- Every genome-locus cell in allelic_profile_matrix.tsv contains either a valid integer allele identifier or one of the tags EXC/INF, ASM, ALM, NIPH/NIPHEM, PLOT, LNF
- PLOT tags occur only at loci whose matching CDS lies at a contig terminus of the draft assembly, and are absent when the same genomes are re-called as complete genomes
- Loci repeatedly tagged NIPH/NIPHEM appear in RepeatedLoci.txt and removal with RemoveGenes yields the expected reduced locus count (e.g., 3128 minus 20 paralogous loci = 3108 in the tutorial)
- Default BSR threshold of 0.6 was applied during the allele call
- Novel alleles called during the run are assigned identifiers and added to the schema (e.g., schema grew from 3,128 to 17,848 alleles in the tutorial)

## Limitations

- PLOT classification depends on assembly quality: fragmented assemblies produce more contig-edge hits, and a 95% core-genome threshold is recommended to account for loci missed due to sequencing coverage and assembly problems
- Proprietary GbG platforms' allele-calling algorithms are incompletely described, so PLOT-like handling of contig-edge truncation may not be comparable across tools
- select_highest_scores bug (fixed in 3.5.4) previously sorted BLASTp matches by sequence length instead of alignment raw score, which could misclassify some sequences in older versions

## Evidence

- [readme] The allele call used the default BSR threshold of 0.6: [evidence span withheld — anchor status: external_doc]
- [discussion] allele calling engine input type: "The allele-calling engine of chewBBACA uses FASTA files with draft assemblies or complete genomes as input"
- [readme] paralogous loci output: [evidence span withheld — anchor status: external_doc]
- [discussion] size threshold for ASM/ALM: "Size threshold is important for the definition of ASM and ALM (alleles smaller/larger than mode)."
- [discussion] allelic profile matrix output: "returns as output an allelic profile matrix and a set of FASTA files containing the full allelic diversity of each locus"
