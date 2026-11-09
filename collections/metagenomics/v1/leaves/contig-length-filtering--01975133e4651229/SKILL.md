---
name: contig-length-filtering
description: Use when assembler output contigs must be length-filtered before computing
  reference-based assembly quality metrics, or when comparing assemblers fairly after
  removing very short contigs.
when_to_use_negative:
- Evaluating raw assembly statistics (e.g. total contig count, N50) where short contigs
  should be retained
- Input is not assembler-generated contig FASTA (e.g. raw reads or reference genomes)
- Assemblies that have already been filtered to the same --minLength threshold (re-filtering
  is redundant)
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: BBTools
  role: Applies the sequence-length filter (v38.44) to retain only contigs of the
    filtered set (FS)
- name: minimap2
  role: Aligns the FS contigs to the reference replicons (v2.22) to produce reference-dependent
    mappings
- name: LMAS
  role: Nextflow workflow that orchestrates filtering and reference mapping per assembler
    and sample
  repo: https://github.com/B-UMMI/LMAS
- name: Nextflow
  role: Workflow engine running the modular, containerized filtering/mapping processes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giac122
    title: 'LMAS: evaluating metagenomic short <i>de novo</i> assembly methods through
      defined communities'
  source_content_hash: sha256:52ead656e75bb19c5c943914a6057a713fe630ae7d03eeb63f587f99f2f18355
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/LMAS
  tools:
  - BBTools
  - LMAS
  - Nextflow
  - minimap2
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# contig-length-filtering

## Summary

Filters assembler-generated contig FASTA files down to the filtered set (FS) by discarding contigs shorter than a user-defined minimum length (BBTools v38.44, --minLength, default 1000 bp) before reference-based quality assessment in LMAS.

## When to use

Use when assembler output contigs must be length-filtered before computing reference-based assembly quality metrics, or when comparing assemblers fairly after removing very short contigs.

## When NOT to use

- Evaluating raw assembly statistics (e.g. total contig count, N50) where short contigs should be retained
- Input is not assembler-generated contig FASTA (e.g. raw reads or reference genomes)
- Assemblies that have already been filtered to the same --minLength threshold (re-filtering is redundant)

## Inputs

- Per-assembler contig FASTA files from results/assembly/<assembler>/
- Single multi-FASTA file of reference replicons (--reference)
- --minLength parameter value (default 1000 bp)

## Outputs

- Filtered-set (FS) contig FASTA files containing only contigs >= --minLength
- One minimap2 alignment file per sample mapping FS contigs to reference replicons

## How to apply

Load the contig FASTA files produced by each assembler (results/assembly/<assembler>/) together with the single multi-FASTA of reference replicons supplied via --reference. Apply the BBTools (v38.44) sequence-length filter using the minimum contig length from the --minLength parameter (default 1000 bp) to retain only the filtered set (FS). Index the reference replicons and align the FS contigs against them with minimap2 (v2.22), writing one alignment file per sample. The rationale is that differences between assemblers tend to be attenuated after filtering for contigs <1 kbp, so downstream global and reference-specific metrics are computed on comparable FS sets.

## Related tools

- **BBTools** (Applies the sequence-length filter (v38.44) to retain only contigs of the filtered set (FS))
- **minimap2** (Aligns the FS contigs to the reference replicons (v2.22) to produce reference-dependent mappings)
- **LMAS** (Nextflow workflow that orchestrates filtering and reference mapping per assembler and sample) — https://github.com/B-UMMI/LMAS
- **Nextflow** (Workflow engine running the modular, containerized filtering/mapping processes)

## Examples

```
nextflow run B-UMMI/LMAS --reads '*_R{1,2}.fastq.gz' --reference replicons.fna --minLength 1000
```

## Evaluation signals

- A valid alignment file exists for every sample after the mapping step
- All retained (FS) contigs meet the --minLength length threshold (default 1000 bp)
- No contig shorter than --minLength appears in any filtered output FASTA
- Downstream metric comparisons behave as expected, e.g. assembler differences attenuate after filtering for contigs <1 kbp

## Limitations

- The default 1000 bp cutoff is user-adjustable via --minLength, so results depend on the chosen threshold and are not comparable across runs with different values
- Filtering discards very short contigs entirely, so metrics on the FS do not reflect the full assembler output

## Evidence

- [methods] BBTools FS filter rationale: [evidence span withheld — anchor status: not_found]
- [methods] minLength parameter definition: "The **X** in the **Contigs > X** section corresponds to the minimum contig size as defined in the ``--minLength`` parameter"
- [results] Effect of 1 kbp filtering on assembler comparison: "no significant difference was observed between the genomic and metagenomic assemblers, particularly after filtering for contigs <1 kbp"
