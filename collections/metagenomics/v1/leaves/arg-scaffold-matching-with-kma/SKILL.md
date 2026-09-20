---
name: arg-scaffold-matching-with-kma
description: Use when you have assembled scaffolds (targeted ARGextender or full SPAdes)
  from metagenomic reads and need to identify which ARG database sequences are present
  and count ARG-scaffolds per sample.
when_to_use_negative:
- Input is raw reads rather than assemblies — ARGextender's KMA-based read recruitment
  stage, not scaffold matching, is the right entry point
- Data are long reads — ARGprofiler/ARGextender is designed to only work with short-read
  sequencing data
- The target gene family is not ARGs and the database has different scope — note PanRes
  combines sources of different scope, so subset appropriateness must be judged per
  research question
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3305
tools:
- name: KMA
  role: Aligner used (v1.4.12a, with '-ont' parameter) to match ARG database sequences
    to ARGextender scaffolds and SPAdes assemblies
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: ARGextender
  role: Produces the targeted de novo assemblies around ARGs that are then matched
    with KMA; combines KMA and SPAdes to recruit reads
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: SPAdes
  role: Produces the full de novo assemblies used as the comparison baseline in the
    benchmark (v3.15.5)
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: ARGprofiler
  role: Snakemake pipeline that orchestrates ARGextender assembly and provides the
    PanRes database of 14,078 unique ARGs
  repo: https://github.com/genomicepidemiology/ARGprofiler
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:712817ff180bf6a17395258b914f2bca0228ba79fd75a2807caff555858059fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/genomicepidemiology/ARGprofiler
  tools:
  - ARGextender
  - ARGprofiler
  - KMA
  - SPAdes
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3305
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# arg-scaffold-matching-with-kma

## Summary

Match ARG reference sequences (ResFinder/PanRes) against targeted ARGextender scaffolds or full SPAdes assemblies using KMA 1.4.12a, enabling per-sample counting of ARG-containing scaffolds and distinct ARGs detected. This is the comparison step that lets ARGextender's targeted de novo assembly be benchmarked against full metagenomic assembly.

## When to use

Use when you have assembled scaffolds (targeted ARGextender or full SPAdes) from metagenomic reads and need to identify which ARG database sequences are present and count ARG-scaffolds per sample.

## When NOT to use

- Input is raw reads rather than assemblies — ARGextender's KMA-based read recruitment stage, not scaffold matching, is the right entry point
- Data are long reads — ARGprofiler/ARGextender is designed to only work with short-read sequencing data
- The target gene family is not ARGs and the database has different scope — note PanRes combines sources of different scope, so subset appropriateness must be judged per research question

## Inputs

- Scaffolds from ARGextender targeted de novo assembly (per sample, FASTA)
- Scaffolds/contigs from full SPAdes metagenomic assembly (per sample, FASTA)
- ARG reference database sequences (ResFinder, or PanRes combined collection)
- Per-sample short-read FASTQ files (e.g. from ENA projects PRJEB40798, PRJEB40816, PRJEB40815, PRJEB27621, ERP015409)

## Outputs

- Per-sample KMA match results of ARG database sequences against scaffolds
- Per-sample count of ARG-containing scaffolds (with flanking regions)
- Per-sample count of distinct ARGs detected
- Summary table comparing ARGextender vs SPAdes means and ranges

## How to apply

Run KMA 1.4.12a with the '-ont' parameter to match the ARG database sequences (e.g. ResFinder, or PanRes's 14,078 unique ARGs) against each sample's scaffold set — either ARGextender targeted assemblies or full SPAdes de novo assemblies. For each sample, extract the flanking regions surrounding matched ARG hits and count (a) the number of ARG-containing scaffolds and (b) the number of distinct ARGs detected. Tabulate the two assembly strategies side by side; in the 951 urban sewage samples benchmark, expect ARGextender to yield comparable output (mean 101 ARG-scaffolds, range 1-350 vs 95, range 1-348 for SPAdes; mean 57 vs 58 distinct ARGs) while ARGextender avoids running the expensive full assembly when no target gene is identified. Rationale: KMA-based matching is faster than reassembly and is the same aligner ARGextender already uses internally to recruit reads, keeping the comparison internally consistent.

## Related tools

- **KMA** (Aligner used (v1.4.12a, with '-ont' parameter) to match ARG database sequences to ARGextender scaffolds and SPAdes assemblies) — https://github.com/genomicepidemiology/ARGprofiler
- **ARGextender** (Produces the targeted de novo assemblies around ARGs that are then matched with KMA; combines KMA and SPAdes to recruit reads) — https://github.com/genomicepidemiology/ARGprofiler
- **SPAdes** (Produces the full de novo assemblies used as the comparison baseline in the benchmark (v3.15.5)) — https://github.com/genomicepidemiology/ARGprofiler
- **ARGprofiler** (Snakemake pipeline that orchestrates ARGextender assembly and provides the PanRes database of 14,078 unique ARGs) — https://github.com/genomicepidemiology/ARGprofiler

## Examples

```
kma -ont -t_db resfinder -i sample_argextender_scaffolds.fa -o sample_arg_matches
```

## Evaluation signals

- Per-sample ARG-scaffold counts for ARGextender fall within the reported range 1-350 with mean ~101 on the 951 urban sewage samples
- SPAdes baseline yields mean ~95 ARG-scaffolds (range 1-348) and mean ~58 distinct ARGs (range 1-153)
- Distinct-ARG counts agree between tools: mean 57 for ARGextender (range 1-141) vs 58 for SPAdes
- Every matched scaffold can be traced to a ResFinder/PanRes database sequence in the KMA output; no unmatched scaffolds are counted
- Samples where no target ARG is identified skip the expensive assembly entirely (ARGextender behavior)

## Limitations

- ARGprofiler/ARGextender only supports short-read sequencing data; long-read support is planned but not available
- Results depend on the chosen database; PanRes combines sources with different scopes and it is up to the individual research question which subsets are appropriate
- Computational savings come from targeted assembly, so scaffolds cover only flanking regions around identified ARGs, not the full metagenome

## Evidence

- [results] KMA matching of ResFinder to both scaffold types: [evidence span withheld — anchor status: not_found]
- [abstract] ARGextender combines KMA and SPAdes for targeted assembly: "ARGextender is a bioinformatic approach combining KMA and SPAdes to recruit reads for a targeted de novo assembly"
- [discussion] Comparable flanking regions in less time: "ARGextender could extract comparable flanking regions to SPAdes but in a much shorter time frame"
- [discussion] Skipping expensive assembly when no target found: "This approach also avoids running the more expensive algorithm if none of the target genes is identified"
- [discussion] Short-read only limitation: "ARGprofiler is designed to only work with short-read sequencing data, thus not utilizing the advantages of long-read sequencing technologies"
