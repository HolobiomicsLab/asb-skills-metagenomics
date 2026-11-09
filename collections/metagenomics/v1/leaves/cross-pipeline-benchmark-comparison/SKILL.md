---
name: cross-pipeline-benchmark-comparison
description: Use when two or more 16S OTU-calling pipelines must be compared on the
  same mock community FASTQ data, and you need to report OTU counts, total counts,
  and runtime rankings per dataset.
when_to_use_negative:
- The input is already an OTU/feature table — this skill benchmarks raw paired-end
  FASTQ through full pipelines, not table post-processing
- Only a single pipeline is available — a cross-pipeline comparison requires at least
  two pipelines run on identical input data
- Data are single-end or non-overlapping paired reads — the article's benchmark relies
  on highly overlapping paired-end 2 x 250 bp reads
tools:
- name: CDSnake
  role: Benchmarked Snakemake pipeline wrapping CD-HIT-OTU-MiSeq utilities; produces
    annotated OTU tables and is compared on OTU counts, total counts, and runtime
  repo: github.com/ydkondratenko/cdsnake
- name: DADA2
  role: QIIME2-based comparison pipeline; baseline for OTU counts (e.g. 91 OTUs /
    29,344 counts on Balanced) and runtime benchmarks
- name: deblur
  role: QIIME2-based comparison pipeline; fastest in every tested case and reference
    for runtime ranking (1371.72 s on HMP)
- name: CD-HIT-OTU-Miseq
  role: Source utilities that CDSnake wraps; timed separately to check CDSnake's Snakemake
    overhead (ran slightly longer than the source utilities)
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-03591-6
    title: 'CDSnake: Snakemake pipeline for retrieval of annotated OTUs from paired-end
      reads using CD-HIT utilities'
  source_content_hash: sha256:e8d6e3582ea3c989bbb749f7e94a50c5394e1cd4a3afe359814d0db2c257bdd7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT-OTU-Miseq
  - CDSnake
  - DADA2
  - deblur
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# cross-pipeline-benchmark-comparison

## Summary

Benchmark OTU-retrieval pipelines (CDSnake/CD-HIT-OTU-MiSeq, QIIME2-DADA2, QIIME2-deblur) head-to-head on mock community datasets by comparing output OTU counts, total read counts, annotation accuracy, and wall-clock runtimes against the true number of strains.

## When to use

Use when two or more 16S OTU-calling pipelines must be compared on the same mock community FASTQ data, and you need to report OTU counts, total counts, and runtime rankings per dataset.

## When NOT to use

- The input is already an OTU/feature table — this skill benchmarks raw paired-end FASTQ through full pipelines, not table post-processing
- Only a single pipeline is available — a cross-pipeline comparison requires at least two pipelines run on identical input data
- Data are single-end or non-overlapping paired reads — the article's benchmark relies on highly overlapping paired-end 2 x 250 bp reads

## Inputs

- Paired-end 16S FASTQ reads from mock community datasets (Balanced, HMP, Extreme)
- Formatted annotated 16S database (Greengenes or SILVA) for CDSnake/CD-HIT-OTU-MiSeq
- True strain counts per mock community (57 Balanced, 21 HMP, 27 Extreme) as reference
- Runtime measurements in seconds for DADA2, deblur, source CD-HIT-OTU-MiSeq, and CDSnake per dataset

## Outputs

- CSV/table of OTU counts and total read counts per pipeline per dataset with true_strains column (9 pipeline-dataset rows, Table 1 style)
- Annotated OTU table (OTU.txt in 'pooled' folder) per pipeline
- Per-dataset runtime ranking table with speed ranks and validation flags (Table 2 style)
- Qualitative comparison flags: OTU counts vs true strain number, deblur-minimal-runtime check, CDSnake total-count advantage

## How to apply

Run each pipeline on the same three mock datasets (Balanced with 57 bacteria and archaea, HMP with 21 bacteria, Extreme with 27 bacterial strains), all with highly overlapping paired-end 2 x 250 bp reads. For CDSnake, edit the Snakefile (INPUT_FOLDER, DATABASE, TRIMMOMATIC, CDHIT, p=200 and q=180 bp 'good' quality lengths, MINLEN=100) and run Snakemake; DADA2 and deblur run inside the QIIME2 workflow. Collect per-pipeline OTU counts and total counts into a table (Table 1 pattern: e.g. Balanced — DADA2 91 OTUs/29,344 counts, deblur 34 OTUs/200,317 counts, CDSnake 55 OTUs/524,604 counts) and append the true strain count as a reference column. Separately time each pipeline (including the source CD-HIT-OTU-MiSeq utilities) per dataset and pooled 'All' set, as in Table 2 (e.g. HMP: DADA2 4614.81 s, deblur 1371.72 s, source CD-HIT 2149.59 s, CDSnake 2263.83 s). Validate by checking that CDSnake's OTU counts are closer to the true strain numbers on Balanced and HMP, that deblur is fastest in every case, and that CDSnake yields higher total counts (preserving more information from raw data).

## Related tools

- **CDSnake** (Benchmarked Snakemake pipeline wrapping CD-HIT-OTU-MiSeq utilities; produces annotated OTU tables and is compared on OTU counts, total counts, and runtime) — github.com/ydkondratenko/cdsnake
- **DADA2** (QIIME2-based comparison pipeline; baseline for OTU counts (e.g. 91 OTUs / 29,344 counts on Balanced) and runtime benchmarks)
- **deblur** (QIIME2-based comparison pipeline; fastest in every tested case and reference for runtime ranking (1371.72 s on HMP))
- **CD-HIT-OTU-Miseq** (Source utilities that CDSnake wraps; timed separately to check CDSnake's Snakemake overhead (ran slightly longer than the source utilities))

## Examples

```
cd input_folder && snakemake
```

## Evaluation signals

- Each pipeline-dataset combination yields an OTU count and total count that matches the published Table 1 values exactly (9 rows for CDSnake, DADA2, deblur x Balanced, HMP, Extreme)
- OTU counts are compared against true strain numbers (57/21/27); expected pattern is CDSnake closer than DADA2 and deblur on Balanced and HMP
- Runtime ranking per dataset has exactly four pipelines and deblur attains the minimum runtime in every tested case (e.g. 1371.72 s on HMP)
- CD-HIT-based pipelines (source CD-HIT-OTU-MiSeq and CDSnake) take substantially longer on the Extreme dataset than the QIIME2-based DADA2 and deblur
- CDSnake produces higher total counts than DADA2/deblur (e.g. 524,604 vs 29,344 counts on Balanced), consistent with preserving more information from raw data

## Limitations

- CDSnake is generally slower than the other pipelines, so runtime cost must be weighed against higher total counts
- Software versions of DADA2, deblur, CD-HIT-OTU-MiSeq, and Snakemake are not stated in the article
- No accessions or availability details are given for the three mock community datasets, and the hardware/execution environment for time benchmarking is unspecified
- CDSnake requires unzipped .fastq files with _R1_/_R2_ (or _1/_2) filename conventions and a database formatted per CD-HIT-OTU-Miseq installation step 4

## Evidence

- [abstract] CDSnake outputted less OTUs than DADA2 and deblur.: "CDSnake outputted less OTUs than DADA2 and deblur."
- [abstract] Slower but preserves more reads: "Though generally slower than other pipelines, CDSnake outputted higher total counts, preserving more information from raw data."
- [results] Mock community composition: "The Balanced community contained 57 bacteria and archaea at nominally equal frequencies"
- [discussion] Runtime benchmark table: "Time, secDADA2deblursource CD-HITCDSnakeHMP4614.81371.722149.592263.83"
- [methods] 97% OTU clustering threshold: "16S reads clustered with 97% similarity result in read groups corresponding to species or close taxonomic levels."
- [readme] How to launch CDSnake: "Then place your edited Snakefile in input folder containing reads and cd into this folder. To run pipeline with default Snakemake settings, use command snakemake."
