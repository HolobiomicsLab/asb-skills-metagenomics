---
name: csv-structured-output-formatting
description: Use when a paper reports a multi-condition benchmark table (e.g. OTU
  counts and total counts for CDSnake, DADA2, and deblur on Balanced, HMP, and Extreme
  mock datasets) that must be transcribed into a structured CSV for downstream comparison.
when_to_use_negative:
- The source values are already available in a machine-readable feature/OTU table
  (e.g. the pipeline's own OTU.txt output) rather than needing transcription from
  prose
- The task requires re-running the pipelines rather than reproducing published numbers
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: CDSnake
  role: Pipeline whose benchmark OTU counts (e.g. 55 OTUs / 524,604 counts on Balanced)
    are transcribed into the CSV
  repo: github.com/ydkondratenko/cdsnake
- name: DADA2
  role: Comparison pipeline whose reported OTU counts and total counts populate CSV
    rows
- name: deblur
  role: Comparison pipeline whose reported OTU counts and total counts populate CSV
    rows
- name: CD-HIT-OTU-Miseq
  role: Underlying utility whose results CDSnake wraps; its benchmark values are transcribed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-03591-6
    title: 'CDSnake: Snakemake pipeline for retrieval of annotated OTUs from paired-end
      reads using CD-HIT utilities'
  source_content_hash: sha256:8ad4daf67945cb78b68485c8b9f015f413bbe3758d37b1d4754ba08ece61fd21
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT-OTU-Miseq
  - CDSnake
  - DADA2
  - deblur
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# csv-structured-output-formatting

## Summary

Transcribes a benchmark results table (e.g. Table 1 of the CDSnake paper) into a validated CSV file with explicit columns (pipeline, dataset, otu_count, total_count, true_strains), enabling machine-readable comparison of pipeline outputs.

## When to use

Use when a paper reports a multi-condition benchmark table (e.g. OTU counts and total counts for CDSnake, DADA2, and deblur on Balanced, HMP, and Extreme mock datasets) that must be transcribed into a structured CSV for downstream comparison.

## When NOT to use

- The source values are already available in a machine-readable feature/OTU table (e.g. the pipeline's own OTU.txt output) rather than needing transcription from prose
- The task requires re-running the pipelines rather than reproducing published numbers

## Inputs

- Published benchmark table (Table 1 of the CDSnake article) reporting OTU counts, annotation statistics, and total counts
- Known true strain counts per mock community (Balanced 57, HMP 21, Extreme 27) from the results section

## Outputs

- CSV file (e.g. otu_counts_table1.csv) with columns pipeline, dataset, otu_count, total_count, true_strains and 9 data rows

## How to apply

Locate the results table (Table 1) and transcribe every pipeline-dataset combination: CDSnake, DADA2, and deblur each evaluated on the Balanced, HMP, and Extreme mock communities. Note that the paper's table strings run counts together (e.g. 'Balanced59dada29167573929,344' encodes dataset, true strain count, pipeline, OTU count, annotated, correctly annotated, total counts), so parse digits carefully: DADA2 on Balanced is 91 OTUs with 29,344 counts, deblur 34 OTUs with 200,317 counts, CDSnake 55 OTUs with 524,604 counts. Append a reference column with the true number of strains per mock community: 57 for Balanced, 21 for HMP, 27 for Extreme. Emit a 9-row CSV with columns pipeline, dataset, otu_count, total_count, true_strains, then cross-check every cell against the printed table for exact numerical agreement before saving (e.g. as otu_counts_table1.csv).

## Related tools

- **CDSnake** (Pipeline whose benchmark OTU counts (e.g. 55 OTUs / 524,604 counts on Balanced) are transcribed into the CSV) — github.com/ydkondratenko/cdsnake
- **DADA2** (Comparison pipeline whose reported OTU counts and total counts populate CSV rows)
- **deblur** (Comparison pipeline whose reported OTU counts and total counts populate CSV rows)
- **CD-HIT-OTU-Miseq** (Underlying utility whose results CDSnake wraps; its benchmark values are transcribed)

## Examples

```
python -c "import csv; rows=[('cdsnake','balanced',55,524604,57),('dada2','balanced',91,29344,57),('deblur','balanced',34,200317,57)]; w=csv.writer(open('otu_counts_table1.csv','w')); w.writerow(['pipeline','dataset','otu_count','total_count','true_strains']); w.writerows(rows)"
```

## Evaluation signals

- CSV contains exactly 9 data rows (3 pipelines × 3 datasets) plus a header
- Spot-check cells against Table 1: DADA2/Balanced = 91 OTUs and 29,344 counts; deblur/Balanced = 34 OTUs and 200,317 counts; CDSnake/Balanced = 55 OTUs and 524,604 counts
- true_strains column matches the results text: 57 (Balanced), 21 (HMP), 27 (Extreme)
- Header matches the agreed schema exactly: pipeline,dataset,otu_count,total_count,true_strains
- Number parsing of the concatenated table string is correct (e.g. 'Balanced59dada29167573929,344' is not read as OTU count 916)

## Limitations

- The published table concatenates multiple statistics into a single digit string (e.g. 'deblur34328024441200,317'), so mis-segmenting digits is the primary transcription failure mode
- The paper reports divergent strain counts for Balanced across sections (59 in one table string vs 57 in the results text); resolve using the results-section statement (57 bacteria and archaea)
- Software versions and dataset accessions are not stated in the article, so provenance of transcribed numbers is limited to the citation

## Evidence

- [results] DADA2 outputted 91 OTUs on the Balanced dataset (actual strains: 59), with 57 annotated and 39 correctly annotated: [evidence span withheld — anchor status: not_found]
- [results] Balanced table string with concatenated counts: "Balanced59dada29167573929,344"
- [results] True strain counts for the mock communities: "The Balanced community contained 57 bacteria and archaea at nominally equal frequencies"
- [readme] README description of the OTU table output that the CSV mirrors: "OTU table for all samples will be saved in file “OTU.txt” in “pooled” folder."
